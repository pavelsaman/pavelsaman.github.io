---
layout: page
title:  "Learning the Product with Testing Tours"
date:   2019-12-05 08:07:19 +0200
categories: testing
---

I've recently written [a post on sqa stackexchange](https://sqa.stackexchange.com/questions/41720/how-to-test-a-system-alone-with-very-little-experience/41724#41724) where I shared a few thoughts about testing software with no or little experience. I believe my post has been received well because I wrote the post remembering my first days in testing. In this blog article, I'd like to pick up on that and share a few ways a tester can do to learn about a product. These ways could be used by anybody, even inexperienced testers who might find them especially useful because they can lead them towards some interesting bugs very quickly.

First of all, I'm not the first one who shares this. These testing tours are something described in this [blog post](https://www.developsense.com/blog/2009/04/of-testing-tours-and-dashboards/), I simply use them and add a bit here and there to make it suitable for my context.

For the sake of this article, let's imagine you are give an application you need to test. This is your first time you see the app and there's no other documentation and everybody's busy, so no one will explain much to you. Pretty standard scenario in many companies. However, you still want to come up with useful feedback on the app. What do you do?

Well, I'd set off on a few tours with the sole purpose of exploring the app and learning about it.

### Functionality Tour

Going through the app and enlisting functionalities. I'd keep this high-level so not to get bogged down on details. Example of an ecommerce site could be:

- customer can buy a product (this can be later broken down to several functinalities such as adding an item into cart, increasing quantity etc.)
- customer can create a personal account using their email and/or social media account
- customer can log in and see their personal information
- customer can change their personal information in their profile
- customer can flag products as favourite, which makes them appear in user profile
- customer can reserve products in stores
- user gets notified on email (when chosen in their profile) about orders, ...


By doing this, I've already learnt quite a lot, and it has also given me some good testing ideas that I can pursue later.

### Interruption Tour

This simply means interrupting actions when not finished. For example, have you ever paid for some products online? The scenario usually goes like this: you finish your order on an eccommerce site, you click a button or you're automatically transferred to a 3rd party payment provider where you add your card details and pay. Then you are either transferred automatically back, or you stay there and you can click a button to go back to the site. But what if you don't go back to the site? You've deviated a bit from what the creators of the app might've imagined. I've recently faced an issue where the order status hasn't been updated when a user didn't click on that "go back after payment button, making the orders incomplete.

Or this tour can mean something simple, e.g. clicking cancel buttons.

### Empty Data Tour

This one is a classic, you go through the app and try to finish actions with empty values. There might be many fields for you to fill in, but what happens if you leave them empty? The bugs might not immediatelly present themselves, you should also look for some background processes that work with the inserted data.

### Language Tour

Typos, grammar mistakes, not clear instructions. All these might confuse customers, making them not want to go back to the site. Read texts carefully, read error messages as if it was the first time you see them. Are they clear? Do you know what to do next to finish what you, as a customer, want to finish?

### Configuration Tour

Explore ways in which the app could be configured. Does it still do what it's designed to do? How does the behaviour change when you change configuration? If you test a website, is it presented in the same way when you use different browsers?

### Data Tour

Follow the data you insert into the system, where do they go next? Where do they end up? How are they transformed?

### Variability Tour

Look for things that vary in the app. If possible, vary them in different ways and directions and see what happens.

### Interaction Tour

How does the app interact with other apps? You might find some APIs or other ways how apps can talk to each other. Find why this interaction is important and why it exists from the business perspective. You can learn the technical details later.

### Documentation Tour

Try to find documentation for the project, ask for whatever documentation has been created, find old bugs and read some. What has been documented? Usually those parts are a bit more prone to errors, that's why busy developers and other people on the project created documentation for these parts in the first place.

### Automation Tour

Is there some test automation done on the project? If so, explore it and find what it does, what it doesn't do and how you can use it to your benefit.

### Process Tour

Testing is not only about the product, but about processes as well, find how they work.

### Security Tour

How about security? It's something that matters a lot and you should have a look at least on some basic and/or most common security threads.

### Performance Tour

Just like the previous tour, this can get a little overwhelming, but you should be able to at least give some subjective feedback on performance. What takes a long time? Where do you get stuck waiting until something loads/appears? This is valuable feedback you should gather.

Alright, these are just a few tours you can take to get familiar with a product/project that you are supposed to work on as a tester. Thinking along these lines can help you speed up the initial phase of learning and exploring the product. However, you can use these tours even later on the project, they prove useful anytime.