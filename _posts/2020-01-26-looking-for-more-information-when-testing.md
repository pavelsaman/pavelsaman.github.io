---
layout: page
title:  "Looking For More Information When Testing"
date:   2020-01-26 16:07:19 +0200
categories: testing
---

Testing can be difficult because of the fact that a Tester might not have enought information to see whether or not there's a bug. In addition to that, some Testers don't even look for anything beyond what the see on the screen when interacting with the software. I don't think this is the best way to gather enough information and uncover enough bugs. Let's see some other ways how Testers can find information.

#### Frontend

This is an obvious source of information. A Tester goes through the app and notices what they see. This is what the majority of Testers do. There's nothing wrong about it and it's a valid way to find bugs and problems with the app. But it should never be the only way.

#### Consoles

`F12` in a browser. But every such console has more to it. There's the network tab where you can see requests, there's the console itself, there's quite a lot about performance, there's the whole DOM object. You can even use the debugger.

#### Logs

Every app generates some logs (or at least it should, at least on test environments). It could be a text file in a certain format on a FTP server, it could be a table in a database, etc. Find where the logs are and focus on them as well.

#### Database

Many apps work with some data which is stored in some sort of repository. It could be a relational DB, some other non-relational DB like Mongo or Redis. Whatever the storage, there'll likely be one (unless you test something specific, or some embedded device, etc.). As a Tester, you should be able to follow the data as you interact with the app. Then you might uncover some interesting things that happen to the data.

#### Code

Sometimes it could be easier to read a bit of code to see the logic and get a new perspective on the app and its behaviour. It could be priceless to see some code when testing an API. Get familiar with at least some bits or the code that's used on the project you're on and it will help you a lot (plus save you some time).

#### Monitoring tools

You can use some additional monitoring tools. If you're testing an app that has something to do with network communication, you might want to sniff some packets and see what's going on on the wire.

#### APIs

An API can sometimes give you an answer to your questions much faster than other ways. If your system should have sent an email, but the email gets sent by a different system, maybe that different system has an API you can call and ask what's going on with your email. It might give you the email status, when it was sent, etc. This might be better than waiting at your email client pressing a refresh button every 5 seconds and hoping it will be received soon.

#### People

There're other people who work with the app. It's even better if you can directly ask users who actually use the app. You might see how they really work with the app and based on that input, you might update your test cases. If you're testing some corporate app, you might find out the (power) users might use the app much faster than you've ever imagined. What a good idea for further testing.


These are some ways I've come up with. Not all of them might be available in every situation, but the key is to look for more than just the frontend.
