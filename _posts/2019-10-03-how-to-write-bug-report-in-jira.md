---
layout: page
title:  "How to Write Bug Report in Jira"
date:   2019-10-03 15:07:19 +0200
categories: testing
---

Jira is a popular tool for software development. One of the ways in which Jira is used for is bug reporting. I've recently faced a situation where there was no structure to bug reports in Jira, leaving it completely up to a Tester. I'll try to argue for why this is not the best practice and how it could be improved.

The following printscreen is how bugs are added into Jira.

![image](/images/jira_empty.png)

A new bug is simply described in field: That't exactly what I want to write about on the following lines.

### How to Describe Bug

A bug needs to be described somehow. Depending on a context, it could be a long report or one sentence. However, in any situation, what needs to be described is at least:

1. environment, app version
2. what data was used - like testing users, values in input fields etc.
3. how to reproduce the bug, so some steps
4. what the current state is
5. what the desired state is

Again, each of these steps could be more or less formal, depending on a context! But a Tester always needs to think along these lines, and only then decide what format (formal, informal) to choose.

### How to Improve Jira

Fairly easy, what I have a good experience with is adding a simple teplate that would automatically add these 4 guidlines. A Tester doesn't need to keep everything in their head and if there're some less experienced testers on the team, they might even use this as a reminder what not to forget when filling in new bug reports.

The final Jira could look like this:

![image](/images/jira_full.png)
