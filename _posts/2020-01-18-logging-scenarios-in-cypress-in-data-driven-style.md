---
layout: page
title:  "Logging Scenarios in Cypress in Data Driven Style"
date:   2020-01-18 16:07:19 +0200
categories: testing
---

Data driven style of writing tests is popular in many different testing tools and/or frameworks. You can use `@pytest.mark.parametrize` in [pytest](https://docs.pytest.org/en/latest/), you can use [templates in Robot Framework ](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#data-driven-style) and I guess there're many other examples I don't have experience with. However, I want to try to use such a style in Cypress. Not that this particular style would be any better than something else, but some people might like the readability of it.

### Example of Eshop Logging

For the following example, I'll use something easy, straighforward. I've chosen an example of logging into a web page.

I won't explain some basics like how to set up node, npm, Cypress, how to find what requests go to the server, so we can use them in tests, etc. I'll just show a working example (well, I won't show any links and credentials as this is an example from my place of work :)), and explain a thing or two.

However, it's reasonable to say that some of the ideas in my example come from [one of the official examples provided by Cypress](https://github.com/cypress-io/cypress-example-recipes/tree/master/examples/logging-in__single-sign-on), especially the custom Cypress command:

```javascript
const _ = Cypress._

Cypress.Commands.add('logInReq', (overrides = {}) => {   

	Cypress.log({
    	name: 'logInReq',
    })

    const options = {
    	method: 'POST',
      	url: 'http://abc_example_site/Account/Login',      
      	form: true,
      	headers: {
      		referer: 'http://abc_example_site/'
      	},
      	body: {
        	Username: 'pavel.saman@test.cz',
        	Password: '1234',
      	}
    }

    _.extend(options, overrides)

    cy.request(options)
})
```

The difference in my setup is I've put this Cypress command into `cypress/support/commands.js` file, which seems as a bit more appropriate place for any custom commands you might wish to create for your tests.

### Tests

Let's dive into the problem. I'll first create a new integration file in `cypress/integration`, which is a place for any integration test files (it's really good to keep some structure, the number of tests might grow).

The initial structure/outline will be as follows:

```javascript
describe('Log In Eshop', function() { 
	
	
})
```

Good start :) Let's add one test:

```javascript
describe('Log In Eshop', function() { 
	
	it('Log In With No Error', function() {
		
	})	
})
```

Ok, you can name it whatever fits your context. However, I've decided that this test case will check I don't end up with any 500 response code for various combinations of input data.

Now comes the part with **data driven approach**. In Cypress/Javascript, I can use `forEach`:

```javascript
describe('Log In Eshop', function() { 
	
	[my test data].forEach((data) => {
		it('Log In With No Error', function() {
		
		})	
	})	
})
```

It turns out I can put the whole test case (`it()`) inside the `forEach`. It will loop over all my data:

```javascript
[{ url: Cypress.config().baseUrl + '/Account/Login', body: { Username: 'pavel.saman@test.cz', Password: '1234' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: 'pavel.saman@test.cz', Password: '12345' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: '', Password: '' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: 'abc', Password: '' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: '', Password: '123' } }
].forEach((login_data) => {

	it('Log In With No Error ' + JSON.stringify(login_data), function() {
		
	})
})
```

It doesn't matter I have added just 4 examples, the style is what I want to show here. So, a few things happened:

- I defined key value pairs of what I need to override in each request so the requests are actually different from what's hardcoded in `options` object.
- I'm using `Cypress.config().baseUrl` which will get `baseUrl` from `cypress.json` located in the root dir of my project (I can have multiple config files, so I don't want any hard coded values in my tests)
- I changed the name of the test case to `it('Log In With No Error ' + JSON.stringify(login_data)`, so it will actually change with every test data used. The benefit is obvious, I can see at the first glance what request is being made.

Now, I just need to create the last part:

```javascript
[{ url: Cypress.config().baseUrl + '/Account/Login', body: { Username: 'pavel.saman@test.cz', Password: '1234' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: 'pavel.saman@test.cz', Password: '12345' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: '', Password: '' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: 'abc', Password: '' } },
 { url: Cypress.config().baseUrl + '/Account/Login', body: { Username: '', Password: '123' } }
].forEach((login_data) => {

	it('Log In With No Error ' + JSON.stringify(login_data), function() {
		cy.logInReq(login_data).then((resp) => {
			expect(resp.status).not.to.eq(500)        		
   		})
	})
})
```

I'm passing each `login_data` to my custom command `logInReq`. It will override those hard-coded values and make a request. Then I just test I don't get any 500 response code.

### Conclusion

This example shows just one possible option of how to add test data into test cases. I wouldn't go as far as saying this is the best option, it's probably not (it depends on a situation). And it's definitely not the only option. Cypress teaches that test data could go into `cypress/fixtures` folder, which makes it more separate from test cases. I find that useful and I use it that way in many cases. I'd probably go for this data-driven approach in situations where I don't need many test data.