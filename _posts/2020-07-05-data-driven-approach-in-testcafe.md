---
layout: page
title:  "Data-driven approach in TestCafe"
date:   2020-07-05 18:30:19 +0200
categories: testing
---

Data-driven approach is a well-known approach in test automation that allows for running the same test case multiple times, each time with a different set of test data. It's handy because there's a high level of code reuse and the rest is only about some sort of parametrization (different test data). TestCafe on the other hand is a e2e test tool that uses Javascript to write test cases. Writing test cases in the data-driven approach might not be the most straighforward in JS-based tools, but let's have a look on how to do it.

Let's assume my file with test data looks like this:

```
{
	"credentials": [
        {
            "username": "pavel.saman@inveo",
            "password": "123456"
        },
        {
            "username": "pavel.samaninveo.com",
            "password": "123456"
        },
        {
            "username": "pavel",
            "password": "123456"
        }
    ]
}
```

A simple JSON format with various usernames and passwords.

Now I want to run a simple test script that takes one set of credentials at a time and performs some steps (like logging into an application).

```javascript
test
    .meta({ author: 'Pavel Saman', creationDate: '27/05/2020',
        env: process.env.TESTCAFE_ENV, url: baseUrl
    })
    ('Log Into User Account', async t => {

    // some test steps      
});
```

But this is hardly a data-driven approach, so I need to do something about the test case.

TestCafe (and some other JS-based tools like Cypress) doesn't support the data-driven approach out of the box, but it's possible to use syntax that's already available in Javascript. I'm talking about simple iteration over an array with `forEach`.

First, I need to import my file with the test data:

```javascript
const testData = require(`../Resources/${process.env.TESTCAFE_ENV}/logIn.json`);
```

Then I use `forEach`:

```javascript
testData.credentials.forEach(credentials => {
    test
        .meta({ author: 'Pavel Saman', creationDate: '27/05/2020',
            env: process.env.TESTCAFE_ENV, url: baseUrl
        })
        ('Log Into User Account', async t => {
            
        await LogIn.logIn(credentials.username, credentials.password); 

        // some other test steps   
    });
});
```

The only disadvantage in my opinion is that the beginning of a line doesn't start with `test.` because the `forEach` block is wrapped around the test case. So it's not that obvious at the first sight this is actually a test case.

I hope this will be helpful, it's still a valid syntax and higly recommended if you need to run same test steps multiple times with different test data. JS is probably not the most straighforward language, but JS-based tools have some other advantages over other e.g. Selenium-based tools.
