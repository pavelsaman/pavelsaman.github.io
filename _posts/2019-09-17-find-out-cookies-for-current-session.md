---
layout: post
title:  "Find Out Cookies For Current Session"
date:   2019-09-17 21:07:19 +0200
categories: browser
---

When working with web applications, I sometimes need, or I'm just curious, to find what cookies the app is using. There are a few ways how to get hold of them. Let's have a look at how to use dev tools in a browser.

When you go into your browser, you can most likely press `F12` and get a new section like this:

![image](/images/dev_tools.png)

Many different things could be done from here. For example, there's a console:

![image](/images/dev_tools_2.png)

This console could be used to execute some Javascript. Then it's only one Google search away from finding out how to show all cookies for the current session:

```javascript
>> document.cookie;
```

And here we go:

![image](/images/dev_tools_3.png)

No need to install any addon like Cookie editor if you just need to check what cookies you got.