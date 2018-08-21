---
layout: post
title:  "My Takeaways from Working with an OSS Project"
date:   2018-08-21 01:43:08 +0800
author: "Guangyuan Yang"
header-img: "img/about-bg-walle.jpg"
tags:
    - tech
---

For anyone who hasn't been familiar, coala is a popular open-source project that provides a unified command-line interface for linting and fixing all your code, regardless of the programming languages you use. The coala community is known to be welcoming and inclusive to newcomers at the open-source wonderland.

I have been promoted as a developer in this awesome coala community for months, and here I would like to summarize some of my experiences and thoughts here.

# What I Have Learned

There is already plenty of general advice around, but I would like to share 3 most important things that, based on my current experience, could have made my performance better.

## Communicate, Navigate, Aviate

Pilots are taught an important set of priorities that they should follow through their entire flying career: Aviate, Navigate, Communicate. As a student pilot myself, such order of priorities also seems intuitively reasonable when I am coding solo - since no one depends on what you are working, you just have to code whatever you like, and write documentation when you have *plenty* of time.

However, working inside an open-source organization is a different story. Mentors are your co-pilot instead of ATC, you are meant to fly the plane together, thus it is important to keep them aware of your status and your intentions even before you act. Note that almost always they will have more domain knowledge about the project and the problem you are trying to solve than yourself (that's why they are mentors!), so communicating often will give you better understanding of the problem and help you find the optimal solution. Don't spam your mentors though.

I used to prefer not asking for help. Sometimes it feels good, sometimes you will find yourself stuck at a problem for really long, or solve a problem that is unexist/unimportant. Thus, I reprioritize the idiom to be "Communicate, Navigate, Aviate":

1. Communicate: Create an issue regarding the exact problem you are trying to solve. Clearly state the problem itself and how you design to solve it.
2. Navigate: Ask mentors, maintainers or related developers for their input, as they may have different perspectives / better underdstanding. Identify design choices and find the optimal one.
3. Aviate: Shut up and code. Go back to the previous steps whenever you feel unclear, don't get stuck on an issue for more than 12 hours.

## Mind the Technical Debt

Another set of domain knowledge that I learned from John, our Org Admin, is about one best practice of maintaining an OSS project - pay huge attention to technical debt and avoid creating more as much as possible.

coala has a very strict review process, in which some of my Merge Requests take weeks to go through. Needless to say the Git knowledge and community guidelines that submitters must know. Before it got marked as `pending_review` and handed to the maintainers for review, one Merge Request needs to pass CI tests on multiple platforms, achieve 100% code coverage, meet style requirements enforced by coala bears, have logically separated commits with clear messages, and get correctly rebased. And then, maintainers will work with the author to ensure the implementation uses the best practices, which will almost always consume more time than your actual coding.

I once had an intuitive design choice of making explicit changes for a platform instead of using prefixes, wrappers and environmental variables. John, as the Org Admin, commented his reason of why he preferred the other way:

> This reduces the effort required to review your patch, and ensures that all future enhancements also need to support such platforms ...

> If you don't do this, you have created 'technical debt', and that will mean that if you disappear from coala, the responsibility for keeping it working will fall onto other maintainers that are unfamiliar with it putting extra effort in.

> If you do it as requested, ... minor breakages will be a learning opportunity for coders and reviewers alike, and occasionally someone will rise to that challenge and learn a bit of the new platform to get the breakage fixed.

> (Also, from the prospective of scalability,) what about any other platform which may also need similar infrastructure?

I agreed with him and took notes.

Does the process sounds paranoid? I would be honest and say it somewhat is, but thinking as an OSS project maintainer when you code is worth it.

*The discussion referred to is at [https://gitlab.com/coala/package_manager/merge_requests/127#note_86416461](https://gitlab.com/coala/package_manager/merge_requests/127#note_86416461). The very MR took months to get merged, but I learned a lot from it.*

## Always Plan Things Ahead and Keep Motivated

I believe everyone mentions this, but it just can't be emphasized enough. Plan things ahead, start early, keep motivated, and expect the unexpected, so you won't easily be affected by unplanned circumstances. There will be no "deadline" to push you forward in an open-source project. Without proper planning and motivation, you will find yourself achieve very little in the open-source world.

# See You Guys Soon at coala!

I would like to recommand anyone interested in getting involved in a friendly, inclusive community to ping us on [coala Gitter channel](https://gitter.im/coala/coala).
