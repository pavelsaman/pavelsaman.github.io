---
layout: page
title:  "TestCafe: Setting Up Base Url Based On Environment"
date:   2020-06-02 16:45:19 +0200
categories: testing
---

Following up from the previous article, I'll leave one quick setup here that I currently use to set up base url for different environments.

First of all, I use the custom config file TestCafe recommends. I don't have many settings in there, but I currently use it for urls of different environments:

```json
{
    "baseUrlDev": "https://something-dev.azurewebsites.net/",
    "baseUrlStaging": "http://something-staging.azurewebsites.net/",
    "baseUrlProd": "https://www.something.cz/"
}
```

This allows me to decide later on where I want to run my tests. The way I go about it is I set up an environment variable, in my case `TESTCAFE_ENV`.

Later in `Helpers/`, I import the config and decide what environment I'm on:

```javascript
import config from '../config';

const baseUrlOf = {
    "dev": `${config.baseUrlDev}`,
    "staging": `${config.baseUrlStaging}`,
    "prod": `${config.baseUrlProd}`
};

export function getBaseUrl () {
    return baseUrlOf[`${process.env.TESTCAFE_ENV}`];
}
```

I return the correct url based on the environment variable.

Using all this in tests is just a matter of calling the exported function `getBaseUrl()`:

```javascript
const baseUrl = getBaseUrl();   
```

And I can start tests on the desired environment:

```javascript
fixture `Log In`    
    .page(baseUrl);
```

I find it appropriate to keep settings in `config.json` and not somewhere else (in `Helpers/` or worst of all hard-coded in tests). this way, it could be changed fast with no need to update any other part.