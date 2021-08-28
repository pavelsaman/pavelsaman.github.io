---
layout: page
title:  "Variables In Testing"
date:   2020-02-02 17:07:19 +0200
categories: testing
---

Variables are a well-known concept in basically any programming and/or scripting language. You assign a name to a space in memory, so you can write and/or read from that space later on in the code. Knowing all this is perhaps useful in some situation, but a Tester does not usually operate on such a deep level, yet even Testers can speak about variables, which is usually a little different concept in their understanding. Let's have a look.

More often than not, if you want to uncover some bugs, you need to find ways how to exercise different scenarios. That could be achieved by finding things you can change, or vary, in other words, finding variables that could be changed.

However, I'm not speaking about those variables you might know from programming/scripting languages. Sure, you can change their values, but as a Tester, you usually don't have a direct access to the code, and even if you have, unit testing is still just a part of your job, not the only task you have, and definitelly not the only approach that should be used (unless you want to deliberately leave some bugs in the product :))

Having said that, let's have a look at some high level variables, or things you can vary in order to exercise different scenarios, or different flows in the software.

### Concrete values in input fields

This one is obvious, an app has some inputs (well, most likely anyway), so you can fill in various values, send them to the server, and see what happens. I won't go into much detail here, but how you choose these values is also important. If you want to research more, google terms such as _equivalence partitioning_ and _boundary value analysis_. These approaches are a good start.

### Count

In almost any app you come across there'll be things you can count. That's another nice variable. Try to figure out ways how you can change the count - perhaps you might delete all orders assigned to a user and see if a page that lists all user orders handles such a case in a proper manner. Or you assign a lot of orders/subordinates/departments to one user. The bottom line is, you find ways how you can change count of things in the system.

### Size

Size is another nice variable. One example that comes to mind is uploading/downloading files. If the system allows file upload, what if you try to upload a file of 50 MB, 100 MB, etc.? Will it still be processed in a proper and timely manner? If not, will the error be handled gracefully without any stack trace on the screen?

How about screen size? If you open a web app on a mobile device instead of on a desktop/laptop screen, what will you see?

### Frequency

Can you do something repeatedly? Resend a web form twice, 3 times, ... Click on that nice "Finish Order" button 5 times in a quick succession?

### Speed

One of the systems I've tested was built for corporate users. These people usually worked at the same place for at least months, and they knew the system in and out. In practice, it meant they could navigate through the system really fast, not waiting for everything to load, because they already knew what was going to come and where to click next. But that was exactly the problem at times. The system couldn't handle the speed with which they used it. If I only used the system in a slow(er) manner, I'd've never found some of those problems.

### Time

Could you stay in a certain state/on a certain page for minutes/hours and then resume your work? Can you leave the system open/working over night and come in the morning to a fully functional system?

### Platform

Some systems are designed to work only on a particular platform, other might not even have this specified. Either way, you should find ways to test the system on different platforms. In terms of a web app, it might mean testing in different browsers, on different operating systems (same or similar versions of web browsers e.g. for Windows and Linux might not be necessarily the same when it comes to displaying web content), with different protocols allowed (what if your app works with WebSocket protocol and you turn it off in your browser?).

### Resources

What I mean by resources is things like connection, CPU, memory, printer, other systems that are in integration with your system. What if your app is designed to get some data from a different app and you cut off the connection? Will you be notified? Will you get an ugly stack trace on the screen? Will you be able to save unsaved work? What if your app prints something during some flow and the printer is not available? Such cases are not even that far-fetched, it happens all the time that something goes down or is unavailable.

### URL

Tweaking a URL is a nice way to test how the app handles invalid URLs, unescaped parameters and the like. As a normal user, can you go past the last page (e.g. a query parameter like `&page=2000`)? What will you see?

### Formats

Time formats are one example where you might find some interesting bugs. But even other formats like float numbers. Is a valid way to enter a floating number `10,5` or `10.5`? Will the system behave the same? How about phone formats? Even within one country, there might be different variations of phone numbers, e.g. `732548963`, `732 548 963`, `+420732548963`, `+420 732 548 963`, `0042732548963`, `0042 732 548 963` are all valid in the CZE, so all of these should be processed as valid phone numbers.

How about invalid data formats? You send/save/open a JSON like this `"a": 5 }`, what will happen?

### Location

Some app might be dependent on geographic location. Or they might restrict users from a different geographic location, or they might show different content to different countries (e.g. Netflix), so you'd better change your geographic location when testing such an app as well.

### Language

Some apps might have a language mutation (or more). Change it and see if it's correct. Some words might be shorter/longer, so there even might be cases whan some text overflows some box designed to contain such a text. You can't be sure unless you see it.


That's a good start :) You might find much more that these, some might be context specific. The bottom line, however, is you need to work with such variables when testing.
