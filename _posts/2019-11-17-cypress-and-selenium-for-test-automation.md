---
layout: page
title:  "Cypress and Selenium for Test Automation"
date:   2019-11-17 10:07:19 +0200
categories: testing
---

It's been a long time since my last post here. Well, I've been busy with some new testing projects, so naturally that was my priority. However, let's have a look at a fairly new topic for me, which is [Cypress](https://www.cypress.io/).

Since I'm now more responsible for test automation and I have the freedom to choose tools I want, I've decided to broaden my horizons and try out something new. Not that I wanted to use Cypress from the first moment, but I'd heard it's a useful tool, so I wanted to give it a try and see if it's worth it. The long story short, after two weeks of writting tests in it every day, I've decided to use this framework on one of the projects. Since the only other technology for frontend test automation I know is Selenium (is there really something else beside this?), I'll try to highlight some differences and why I think Cypress is beneficial, plus some bits on what I wish was different in Cypress.

### Major Differences

Let's start with **installation**. Cypress is fairly straightforward at the beginning of the project, you can set it up in like 1 minute. All you need to do is use some javascript package manager and install it, e.g. `$ npm install cypress --save-dev`. That's it, no need to install Selenium server, download browser drivers, choose a wrapper around Selenium you want to use, create a project structure, download a suitable IDE for something like Robot framework. I do think this is a huge advantage especially for someone new or for a situation when you quickly need some automated tests.

So when we've installed Cypress, we need to create some tests. A good thing is, **Cypress will automatically scaffold out a project structure**, so you are ready to go from the start, no need to learn where to put things and how to organize it when your testing project grows in size. Just very quickly: your tests go into `integration/` folder, test data into `fixtures/` folder, some support functions, custom commands etc. into `support/` folder. Easy and already set up for you, no need to waster any mental power on figuring this out. Of course if you need to add to this structure (like some elements and page objects), you can just create it the way you need.

However, creating the tests is not always that easy, especially if you're coming from a "traditional" programming/scripting language where you're used to doing something like: `var my_var = my_object.my_function()`. Cypress with its javascript and jQuery syntax just doesn't work this way. Most of what you will ever write in Cypress won't give you a return value, you need to start thinking in chainables, so a typical Cypress syntax would be:

```javascript
cartOne.getTotalPrice().invoke('text').then((totalPriceCartOne) => {
	// do something with the text, hold areference, use more commands here and then compare etc.
})
```

I might be just me, but this doesn't seem as straightforward as the previous example. You just don't get a return value and you need to take it into accout and work differently. It will take additional time and effort for you to learn it.

Another huge and practical advantage of Cypress over Selenium is, it's **fast**. From whatever reason (I don't know enough of the internals of either one of these technologies to be more concrete here), Cypress just runs faster, therefore it takes less time to run my tests.

Cypress is really a **product on its own**. As I already mentioned, with Selenium, you just get the API and you need to figure out the rest, just plain Selenium is useless, you need more stuff to take off. Cypress is already a product on its own, you can run it from a command line, or you can open it in a browser and you get plenty of information about test execution right there.

One more disadvantage might be that Cypress can, at the time of writing this article, run **only with Chrome environment**, so you can use Chrome, Chromium, Electron, Canary. Selenium could be used with more browsers (e.g. Firefox, IE).

Then, if you run your tests with Cypress, you can **go back in time** and see a state of the application at a particular time in history of the executed tests. Cypress won't delete anything when a test case finishes. So you can just review what's been done. This comes really handy.

Cypress also automatically **supports waiting for pages to load and elements to be visible** etc. This is actually more complex, what Cypress actually does is it queries the DOM repeatedly for a given period (default timeout), if conditions are met within this period, it moves on, if not, it eventually runs out of this timeout period and fails the test. So no need to add "Wait For Element To Be Visible" keywords and functions every now and then, Cypress has this solved internally. In reality, it allows for faster creation of your tests and you get to focus more on actual tests, not on making the tests stable. A huge benefit

### Conclusion

All in all, Cypress proved useful in my context on my project, so I'm using it right now. There might be more to it, but these are some main ideas I've got in last two weeks.