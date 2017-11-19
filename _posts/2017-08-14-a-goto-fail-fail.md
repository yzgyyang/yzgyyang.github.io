---
layout: post
title:  "A 'goto fail' Fail"
date:   2017-08-14 21:43:48 -0400
author: "Guangyuan Yang"
header-img: "img/post-bg-2015.jpg"
tags:
    - tech
    - story

---

# It was a warm, peaceful Tuesday afternoon...

At office, I was developing a patch to the FreeBSD userland [acpidump(8)](https://www.freebsd.org/cgi/man.cgi?query=acpidump&sektion=8) tool to report ACPI NVDIMM information in a user-friendly manner. The implementation seemed to be pretty straightforward. Then I noticed the following error-handling condition in code:

```c
if (subtable->Length < sizeof(ACPI_NFIT_HEADER))
        return;
```

"Hmm, that could be better." I improved it (or at least I thought I did) in a flash:

```c
if (subtable->Length < sizeof(ACPI_NFIT_HEADER))
        warnx("invalid subtable length %u", subtable->Length);
        return;
```

An hour later, I was desperately staring at the screen and scratching my head, wondering what went wrong that caused the most of my code skipped. Then my supervisor Ed came. It took him two seconds to point out the problem, and while I was blaming myself for asking the stupiest question (it's not true, I had worse), he smiled and said, "You had a `goto fail` problem. Apple did that as well."

So what am I doing wrong, and what exactly did Apple do?

# `goto fail`

## The Famous Apple Inc.'s `goto fail` story

> On 2014-02-21, Apple released a security update for its implementation of SSL/TLS in many versions of its operating system. It is formally named CVE-2014-1266, but informally it’s often referenced as the Apple “goto fail” vulnerability (or “goto fail goto fail” vulnerability).

The essence of the problem is more than straightforward. [The code](https://opensource.apple.com/source/Security/Security-55471/libsecurity_ssl/lib/sslKeyExchange.c) included these lines in the SSLVerifySignedServerKeyExchange() function:

```c
hashOut.data = hashes + SSL_MD5_DIGEST_LEN;
hashOut.length = SSL_SHA1_DIGEST_LEN;
if ((err = SSLFreeBuffer(&hashCtx)) != 0)
    goto fail;
if ((err = ReadyHash(&SSLHashSHA1, &hashCtx)) != 0)
    goto fail;
if ((err = SSLHashSHA1.update(&hashCtx, &clientRandom)) != 0)
    goto fail;
if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
    goto fail;
if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
    goto fail;
    goto fail;  /* WHOOPS!! */
if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
    goto fail;

err = sslRawVerify(...);
```

You will immediately notice that there is one duplicate line of the `goto fail` statement. However, if the `if` statement is true, the first `goto fail` got executed and the code will still be correct.

Or is that the case?

The real problem here is that the indentation is very misleading. There are no curly braces after the `if` statement. That is, if the `if` statement failed, the second `goto fail` will be executed, with no errors to report. Notice that the second `goto fail` also skipped the rest of the tests. As a result, the function will return 0 (all OK) in every case.

## How bad could it be?

The bug itself seems simple and silly, however, the impact of this issue is quite serious and disturbing.

> In context, that meant that vital signature checking code after the second `goto fail` was skipped, so both bad and good signatures would be accepted. Invalid certificates were quietly accepted as valid. An attacker now has a way to trick users of OS X 10.9 into accepting SSL/TLS certificates that ought to be rejected.

This made it easy for attackers to do [Man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) against any secured connection using SSL/TLS (including “https://” URLs). Later, Apple published [iOS 7.0.6](https://support.apple.com/en-ca/HT202934), a security update for iOS devices, which fixed this problem.

![About the security content of iOS 7.0.6](/img/in-post/post-goto-fail/apple-ios706.jpg)

## What caused this?

I didn't find any official reports anywhere on the Internet. However, I do have some good guesses of possible causes:

- ~~This is intentional (very unlikely).~~
- An incorrect merge operation (for example, the line from both version got included regarding whitespace issues)
- Code review with naked eye
- There must be something wrong with the coding style.

# Thoughts

## About coding style

The debate of curly braces (or Vim / Emacs, or Tab / Space) has never stopped in the community. However, it would now be obvious that which one is more correct:

```c
if (condition)
{
    statement;
}
```
```c
if (condition)
    statement;
```
There are several examples from [a post on the StackExchange](https://softwareengineering.stackexchange.com/questions/16528/single-statement-if-block-braces-or-no) that could happen to anyone if you are using the second style.

Let's say you are temporarily commenting out code to debug something:

```c
if (condition)
//    statement;
otherStatement;
```

Or adding code in a hurry:

```c
if (condition)
    statement;
    otherStatement;
```

Unlike Python, C is whitespace insensitive, which means the indentation means nothing to the computer, and the above code will never work as intended. On the human side though, it is sometimes natually assumed that code within the same indentation belongs to the same code blocks. This is obviously bad.

Code is written to let both **human** and computer understand (~~That's why Python rules!~~). Personally, I would also prefer verbose to fancy in the following situations:

```c
//if (!p)
if (p == NULL)

//if(check())
if (check() == true)
```

*If you think it is too verbose to always add curly braces, I suggest that the only situation to omit it would be a one-liner:*

```c
if (condition) statement;
```

## About testing

Using static analysis methods, unreachable code could be properly detected and checked. Sadly, warning flags are not available in this case.

> The gcc compiler once reported unreachable code. However, years ago the gcc developers removed this capability because the code was unstable. Even worse, it still accepts a parameter intended for the purpose, namely -Wunreachable-code, without any notification that it no longer does anything.

As David A. Wheeler suggested in [this article](https://www.dwheeler.com/essays/apple-goto-fail.html), dynamic analysis methods like thorough negative testing in test cases should also be addressed.

## About code merge

The problem could also be caused by a merge operation. That being said, maybe both versions of code were correct before code merge. Always do a review and **never** fully trust the auto-merge tools.

# Conclusion

I couldn't imagine this being an issue that I would write a blog post about. I couldn't even imagine myself writing such code before I literally wrote it myself. But as the Apple developers have told us in the code, if you do not care enough about the code quality and not try to follow good practices in your career, you will likely just `goto fail`!

# References

[Apple's code which caused the `goto fail` vulnerability](https://opensource.apple.com/source/Security/Security-55471/libsecurity_ssl/lib/sslKeyExchange.c)

[About the security content of iOS 7.0.6](https://support.apple.com/en-ca/Hhttps://softwareengineering.stackexchange.com/questions/16528/single-statement-if-block-braces-or-noT202934)

[Single statement if block - braces or no?](https://softwareengineering.stackexchange.com/questions/16528/single-statement-if-block-braces-or-no)

[The Apple goto fail vulnerability: lessons learned](https://www.dwheeler.com/essays/apple-goto-fail.html)

[Anatomy of a “goto fail”](https://nakedsecurity.sophos.com/2014/02/24/anatomy-of-a-goto-fail-apples-ssl-bug-explained-plus-an-unofficial-patch/)
