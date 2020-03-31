---
layout: page
title:  "Difference Between Having No Tester and Having One"
date:   2020-03-31 20:20:19 +0200
categories: testing
---

In case someone wonders why a team should have a Tester, I'm bringing this article to the table. Not that I think a Tester should be the only one who tests (that should be done by more team members), or they should be quality guardians (testing <> quality), but I think that a Tester brings a significant value to the team and finally I have some data I can support my theory with.

It's been 6 months since I joined a company I currently work for. The situation before me joining was in short the following:

- one team with multiple (3 - 4) projects at the same time
- about 4 developers working on all the projects (who has time does some work on project A, then goes on to project B etc.)
- no tester - not really any tester ever before on any of these projects, or to be more precise, one overwhelmed tester also assigned to a different and more urgent project with no real capacity to do anything on any of these 4 projects, and one Tester in the past who left the company during the first 3 months, all in all, it sounds like no Tester to me

So, this was the situation when I came on board. Now after 6 months, I'm the only Tester on these projects and basically the only Tester ever on these projects (see the point above).

Moving on, I wouldn't say I'm a big proponent of metrics in testing. There's not really any metric that would make sense in various scenarious and situations I find myself as a Tester. However, one metric I sort of like is a ratio of opened/solved bugs. I'd say there are a few points I like this metric for:

- it helps me figure out if, as a team, we're improving the system and not falling behind with fixing stuff
- it's sort of feedback to me whether or not I'm focusing on something that matters, say there're significantly more opened than solved bugs, then I'm probably not filling in bugs the rest of the team considers important
- in situation like mine I can see what the difference between having no Tester on the team and having one

The following plot shows the situation in the company I work for on all those 4 projects:

![image](/images/tester_no_tester.png)

That seems to be a pretty significant change, isn't it? Some takeways for me are:

- it does matter if you have a Tester who does testing fulltime or not
- as a Project Manager, you should hire a Tester (even though it will give you headaches at times as well)
- there can be many bugs lying around in the product no one's noticed for weeks/months/years

Obviously there could be many more assumptions I can make out of this chart, but let's not exaggerate. I think other people can test and many times know how to, but there're other problems such as how interested they are to do it (people don't really like testing that much I think), how much time they have for testing (if someone with a different role is asked to test). All in all, I don't think that a fulltime Tester can be replaced by some other role, or by just developers testing their stuff after implementation. As these projects make it clear, it's not what happens in reality.

