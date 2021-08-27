---
layout: page
title:  "Mock Server in JS"
date:   2021-08-27 14:15 +0200
categories: testing
---

What I did in the [previous article](https://pavelsaman.github.io/mock-server-for-testing-vue-app.html) has two obvious disadvantages. Postman allows only 1000 requests for mock servers a month, plus your mock server data and everything is stored in a cloud. To overcome that, it's better to set up a mock server locally with a different tool. I'll use [MockServer](https://www.mock-server.com/) with JS to do the same I did with Postman in the previous article.

I'll skip all that talk about why mocking in the first place and just get down to writing the mock server in JS.

It turns out I need to define expectations. It works like this: when the mock server receives a request, it looks at active expectations and tries to match the requests agains one of them, if found, it proxies the request, otherwise it will return 404.

After playing with it for a bit, I defined an expectation in a JSON files like so:

```json
{
    "priority": 2,
    "httpRequest": {
        "path": "/ads",
        "method": "GET",
        "queryStringParameters": [
            {
                "name": "name",
                "values": ["abc"]
            }
        ]
    },
    "httpResponse": {
        "statusCode": 200,
        "headers": [
            {
                "name": "Content-Type",
                "values": ["application/json; charset=utf-8"]
            },
            {
                "name": "access-control-allow-origin",
                "values": ["*"]
            }
        ],
        "body": {
            "data": [
                {
                    "identifier": "94942095558582279289",
                    "name": "abc",
                    "categoryName": "Světla, xenony, žárovky",
                    "userName": "Bruno",
                    "userId": 81928,
                    "adUrl": "https://petklub.cz/Ads/Detail/94942095558582279289",
                    "created": "2016-12-01T16:15:11.0000000Z",
                    "status": "Archived"
                }
            ],
            "paging": {
                "limit": 25,
                "offset": 0,
                "total": 1
            }
        }
    }
}
```

So the mock server tries to match requests agains what's inside `httpRequest`:

```json
{
    "path": "/ads",
    "method": "GET",
    "queryStringParameters": [
        {
            "name": "name",
            "values": ["abc"]
        }
    ]
}
```

That means that if I send a GET request to my mock server instance to `/ads` with a query param `name=abc`, the mock server will return what's inside `httpResponse`. That looks pretty straigforward, so I'll defined one more expectation:

```json
{
    "priority": 1,
    "httpRequest": {
        "path": "/ads",
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "headers": [
            {
                "name": "Content-Type",
                "values": ["application/json; charset=utf-8"]
            },
            {
                "name": "access-control-allow-origin",
                "values": ["*"]
            }
        ],
        "body": {
            "data": [
                {
                    "identifier": "94942095558582279281",
                    "name": "Ford-boční blikač",
                    "categoryName": "Světla, xenony, žárovky",
                    "userName": "Bruno",
                    "userId": 81928,
                    "adUrl": "https://petklub.cz/Ads/Detail/94942095558582279281",
                    "created": "2015-11-01T16:15:11.0000000Z",
                    "status": "Blocked"
                }
            ],
            "paging": {
                "limit": 25,
                "offset": 0,
                "total": 1
            }
        }
    }
}
```

One peculiarity is the `priority` property. That's because I need to make sure more concrete requests get matched first. If I send a request to `/ads?name=abc`, even the expectation for `/ads` is matched, so I first need to try to match it against `/ads?name=abc`.

Now I need to create the rest of the mock server, one part of that is what's called `MockServerClient`, that's where I define all the expectations:

```javascript
const config = require('./config.json');
const mockServerClient = require('mockserver-client').mockServerClient;

const simpleAdReqRes = require('./resources/simple-ad-response.json');
const adSearchQuery = require('./resources/ad-response-search-query.json');

function mockMatchers () {
    mockServerClient(config.host, config.port).mockAnyResponse(adSearchQuery).then(
        function () {
            console.log('created "/ads?name=abc" expectation');
        },
        function (error) {
            console.log(error.body);
        }
    );
    mockServerClient(config.host, config.port).mockAnyResponse(simpleAdReqRes).then(
        function () {
            console.log('created "/ads" expectation');
        },
        function (error) {
            console.log(error.body);
        }
    );
}

module.exports = mockMatchers;
```

Then I need to run the mock server:

```javascript
const config = require('./config.json');
const mockServer = require('mockserver-node');
const mockClient = require('./server-client');

mockServer
    .start_mockserver({serverPort: config.port, verbose: true})
    .then(
        mockClient,
        function (error) {
            console.log(JSON.stringify(error, null, " "));
        }
    );
```

And just to see the config file as well:

```json
{
    "host": "localhost",
    "port": 1080
}
```

The whole project could be seen [here](https://github.com/pavelsaman/mock-server-example).

Then I run the mock server:

```
$ node server.js
```

If everything is ok, I should be able to get the defined responses for the defined requests:

```
$ curl 'localhost:1080/ads'
{
  "data" : [ {
    "identifier" : "94942095558582279281",
    "name" : "Ford-boční blikač",
    "categoryName" : "Světla, xenony, žárovky",
    "userName" : "Bruno",
    "userId" : 81928,
    "adUrl" : "https://petklub.cz/Ads/Detail/94942095558582279281",
    "created" : "2015-11-01T16:15:11.0000000Z",
    "status" : "Blocked"
  } ],
  "paging" : {
    "limit" : 25,
    "offset" : 0,
    "total" : 1
  }
}
```

And the other one:

```
$ curl 'localhost:1080/ads?name=abc'
{
  "data" : [ {
    "identifier" : "94942095558582279289",
    "name" : "abc",
    "categoryName" : "Světla, xenony, žárovky",
    "userName" : "Bruno",
    "userId" : 81928,
    "adUrl" : "https://petklub.cz/Ads/Detail/94942095558582279289",
    "created" : "2016-12-01T16:15:11.0000000Z",
    "status" : "Archived"
  } ],
  "paging" : {
    "limit" : 25,
    "offset" : 0,
    "total" : 1
  }
}
```

So now I can go back to the Vue app and change the `vars.js` file to:

```javascript
const env = process.env.VUE_APP_ENV

const vars = {
  local: {
    apiUrl: "http://localhost:1080"
  },
  dev: {
    apiUrl: ""
  },
  staging: {
    apiUrl: ""
  },
  prod: {
    apiUrl: ""
  },
}

export default {
    install: app => {
        app.config.globalProperties.$vars = vars[env]
    }
}
```

Then after building and running the app on localhost, I can see in the browser the app is already making requests to my mock server and it's getting the responses as defined above.

I achieved the same result as in the [previous article](https://pavelsaman.github.io/mock-server-for-testing-vue-app.html) but with a different tool. It has the advantage that I can run an unlimited number of requests against my local mock server, plus no data is stored outside my local machine.
