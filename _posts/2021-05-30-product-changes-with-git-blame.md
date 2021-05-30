---
layout: page
title:  "Product Changes with git blame"
date:   2021-05-30 10:50 +0200
categories: testing
---

Testers do have a lot of questions about the product they test, but sometimes they need to discuss things with the author. What I usually see Testers do in such a situation is they either go into a ticketing system like Jira to try to find who did some changes to the area of their interest, or they ask a random person on the team. But how about using git blame to find exactly who to ask?

Software products nowadays are versioned in some sort of versioning system, most likely git. And it's exactly git that comes with a ton of functionality that could be leveraged even by Testers.

A common use case is that a Tester needs to ask about some changes to the codebase. What usually happens is that the changes are tracked in some sort of ticketing system like Jira or Redmine. It could be easy to find them there, except that sometimes it's not (for various reasons I don't want to discuss in this article). Then the Tester is left wondering who to ask, especially if we're talking about bigger teams or about remote teams where it's not only about asking a person sitting at a desk next to you.

So, another way of going about this is using what git has to offer.

A basic command everyone should know is `$ git log` that already produces a lot of information:

```
commit 76b9b828d2982f492910ca4a188642ca91337e13 (HEAD -> master, origin/master, origin/HEAD)
Author: Pavel Saman <random@email.cz>
Date:   Thu May 27 10:27:30 2021 +0200

    new cypress article about links

commit 5aeacdb65026076724bae775915af393dd1022af
Author: Pavel Saman <random@email.cz>
Date:   Fri May 21 11:23:35 2021 +0200

    cypress api checking article
```

But that doesn't really answer my question about who did some particular changes to this concrete file.

That's when `$ git blame` comes in. Let's say I want to see who updated `.gitignore` file:

```
$ git blame .gitignore
```

This command produces output like so:

```
84475ab8 (pavelsam 2019-09-22 14:48:20 +0200 1) concepts/
```

From that, I can see what commit changed the file, who and when changed each line of the file, and to what each line was changed. This file has only one line, so perhaps I will show a better example.

To see what the changes were to my previous article on this blog, I'd type:

```
$ git blame 2021-05-27-clicking-on-links-that-open-in-new-tab-in-cypress.md
```

That will produce a longer output, I'm showing only first 10 lines:

```
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  1) ---
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  2) layout: page
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  3) title:  "Cypress: Clicing on Links That Open in New Tab"
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  4) date:   2021-05-27 09:50 +0200
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  5) categories: testing
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  6) ---
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  7) 
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  8) Cypress does not have multi-tab support, which comes as a surprise to many, usually after they have dived headlong into automating some UI checks. However, not having multi-tab support is not really that much of a problem because many times the multi-tab support is not really needed and it is a suboptimal solution anyway. Let's have a look at the most common use case, clicking on links that open in a new tab.
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  9) 
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200 10) First off, how does such a link look like? Let's see its html:
```

As you can imagine, this is really handy, but can leave you with a lot of output you need to navigate. It'd be nice to be able to narrow down my blaming:

```
$ git blame -L 5,10 2021-05-27-clicking-on-links-that-open-in-new-tab-in-cypress.md
```

This will give me only changes to lines 5 through 10:

```
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  5) categories: testing
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  6) ---
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  7) 
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  8) Cypress does not have multi-tab support, which comes as a surprise to many, usually after they have dived headlong into automating some UI checks. However, not having multi-tab support is not really that much of a problem because many times the multi-tab support is not really needed and it is a suboptimal solution anyway. Let's have a look at the most common use case, clicking on links that open in a new tab.
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200  9) 
76b9b828 (Pavel Saman 2021-05-27 10:27:30 +0200 10) First off, how does such a link look like? Let's see its html:
```

Much better, this will be more helpful since I can narrow down my search to only that one part I'm interested in.

`git blame` has many other options like `-e` that will show user email instead of user name. That can help you find that person more easily. Other options could of cource be found in man pages (`$ man git-blame`) provided you use a decent operating system that comes with man pages.

Another useful option is `-w` that ignores white-space changes.

I suppose this method of using `git blame` might be much faster in some situations, so it's useful to know it and have another option of how to work effectively with the rest of your team.
