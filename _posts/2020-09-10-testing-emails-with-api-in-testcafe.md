---
layout: page
title:  "Testing Emails With API In TestCafe"
date:   2020-09-10 22:30 +0200
categories: testing
---

A common trouble with e2e automation testing is how to test sending/receving emails. I've seen various Gmail accounts Testers create, I've seen how Testers want to check a received email on Gmail via its GUI. Neither approach is effective because testing via a GUI is slow and flaky, the other approach is undoable completely since Google blocks automation scripts like Selenium. How should we do it then?

First of all, forget Gmail and forget that GUI is the only way. This is a good start because it will set you up for thinking about better and usually faster ways. Such a better way could include the following steps:

1. you send an email via your app - you want to do it once via a GUI because you want to simulate a real user, nothing wrong with it here
2. you issue a request to your mail server that will return the sent email (or not if the app doesn't work here) - no GUI here!

Without further ado, I'll recommend this [mailosaur](https://mailosaur.com/) service, which is basically your new mail server along with an API. You can find it yourself after some time of googling. It's not the only service, I believe, but I've used it successfully on a project, so I'm used to it.

After you register, you'll find out a server ID, along with an example email in a similar format `birds-visitor.<your-server-id>@mailosaur.io`. The point here is you can change the first part before the dot, so that could be some id like [`nanoid()`](https://www.npmjs.com/package/nanoid).

Going back to my previously mentioned two points, the first one in TextCafe could look like this:

```javascript
await t
    .typeText(Newsletter.newsEmailInput, mailosaurFullEmail(id))
    .click(Newsletter.consent)
    .click(Newsletter.sendButton)
```

Email sent!

Now the second point.

A little trouble with TestCafe is it doesn't have (not yet) any `t.request()` method for network requests (unlike Cypress for instance). But it's all just Node environment, so it's a matter of knowing little bit of Javascript. I can for example create my own request method:

**Helpers/networkRequest.js**
```javascript
import axios from 'axios';

export async function request (reqObject) {  
    try {
        return await axios(reqObject);            
    } catch (error) {
        console.error(error);
    }
}
```

I've used [`axios`](https://www.npmjs.com/package/axios) that I've seen recommended somewhere (probably on Stackoverflow).

When this is ready, I can continue with my test:

```javascript
let res = await request({
    method: 'POST',
    url: config.mailosaurUrlEmail + serverId(),
    headers: {
        'Authorization': 'Basic ' 
            + Buffer.from(process.env.MAILOSAUR_API_KEY)
                .toString('base64'),
        'Content-Type': 'application/json'
    },
    data: {
        sentTo: mailosaurFullEmail(id)
    }
});       
```

I don't show much here, so the request url should be in the following format `https://mailosaur.com/api/messages/await?server=<server-id>`. The `await` method will timeout after 30 seconds (maybe less?) after no email has been received. If an email has been received, it'll return 200 OK along with a json body with all data from the email. You also need an Authorization header where the API key is base64 encoded. And the body contains a json with an address you expect to receive an email from.

The rest of the test is easy, I just assert what I want to assert:

```javascript
await t
    .expect(res.status).eql(200);
```

This is virtually all. A much more straighforward way of checking sent/received emails. It's still not perfect because not every system/app sends emails immediately (or within those 30 seconds) because emails could be handled by some other system that handles a big amount of emails and the email queues might be too full to send your email fast. Or the app might deliberately postpone sending emails till later, etc. This approach would be much more fit for cases like sending forgotten password emails, order confirmation emails, etc. Such emails are usually sent immediately because we don't want to leave the user/customer waiting.
