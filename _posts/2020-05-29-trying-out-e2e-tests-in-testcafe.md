---
layout: page
title:  "Trying Out e2e Tests in TestCafe"
date:   2020-05-29 01:30:19 +0200
categories: testing
---

There has been more JS based testing frameworks recently, Cypress is perhaps the most popular out of them, but today, it's time for TestCafe. I've decided to write a complete set of e2e tests for a small system in TestCafe. I made the decision on Tuesday when I only knew there's such a thing as TestCafe, but never worked with it before. I finished on Thursday (technically on Friday at about 1 a.m. at night :D), that's about 2 days from zero to 48 e2e tests across multiple pages in the app. 25 commits in total. Let's sum up what I experienced and perhaps compare TestCafe with Cypress and others a bit.

First of all, getting TestCafe is as easy as installing one additional module from `npm`, it could be done globally, or inside your project directory:

```
$ npm install --save-dev testcafe
```

And you're ready to roll.

On the other hand, there's no automatic scaffolding like we know from Cypress, that might make things a bit tough if you don't choose a strong directory structure from the off. I made the following one before starting to write tests:

```
.
├── config.json
├── .testcaferc.json
├── Helpers
├── node_modules
├── Objects
├── package.json
├── package-lock.json
├── Resources
├── Results
└── Tests
```

It probably doesn't need much explanation, there's hardly anything special about this. Page objects go into `Objects/`, helper functions into `Helpers/`, reports, screenshots and videos TestCafe generates during test execution all go to `Results/`, test data are in `Resources/`. TestCafe keeps configuration in `.testcaferc.json`, so when executing tests, configuration from this file will be used, or it could be overwriten with command line options. `config.json` is a file for custom options passed to TestCafe, custom options can't go into `.testcaferc.json`, which is a bit weird, but there's already a raised issue on Github for this.

Let's see my `.testcaferc.json` file to have a look at a subset of possible settings:

```json
{
    "src": "./Tests/*.js",
    "browsers": ["chrome"],
     "reporter": [
        {
            "name": "spec"
        },
        {
            "name": "json",
            "output": "Results/report.json"
        },
        {
            "name": "xunit",
            "output": "Results/report.xml"
        }
    ],
    "screenshots": {
        "path": "./Results/Screenshots/",
        "takeOnFails": true,
        "fullPage": true,
        "pathPattern": "${DATE}_${TIME}/${BROWSER}_${BROWSER_VERSION}_${OS}_${OS_VERSION}/${FIXTURE}/${TEST_INDEX}_${TEST}.png"
    },
    "videoPath": "./Results/Videos/",
    "videoOptions": {
        "pathPattern": "${DATE}_${TIME}/${BROWSER}_${BROWSER_VERSION}_${OS}_${OS_VERSION}/${FIXTURE}/${TEST_INDEX}_${TEST}.mp4"
    },
    "skipJsErrors": true,
    "selectorTimeout": 4000,
    "assertionTimeout": 2000,
    "pageLoadTimeout": 2000
}
```

TestCafe seem to be pretty strong with video, it just works out of the box and it supports `ffmpeg`, so all its options could be used.

TestCafe also supports a varienty of reporters, or you can write your own one. One (and only one) reporter could be used to print stuff to stdout, any number of reporters could be used to print stuff to files on a disk, all those reporters I'm using are natively in TestCafe, so no additional module has to be installed.

Moving on, let's have a look at some page object:

```javascript
import { Selector, t } from 'testcafe';

const item = Selector('li');

class MenuItem {
    constructor (text) {
        this.element = item.withExactText(text);
    }
}

class Profile {
    constructor () {
        this.leaveButton = Selector('a').withExactText('Odhlásit');
        this.menuItemObj = {
            "profile": new MenuItem('Můj profile'),
            "children": new MenuItem('Moje děti'),
            "cards": new MenuItem('Moje karty'),
            "eshops": new MenuItem('E-shopy'),
            "membership": new MenuItem('Členství')
        };   
    }    

    async logOut () {
        await t
            .click(this.leaveButton);
    }

    async goToMyProfile () {
        await t
            .click(this.menuItemObj.profile.element);
    }

    async goToMyKids () {
        await t
            .click(this.menuItemObj.children.element);
    }    

    async goToMyCards () {
        await t
            .click(this.menuItemObj.cards.element);
    }

    async goToEshops () {
        await t
            .click(this.menuItemObj.eshops.element);
    }

    async goToMembership () {
        await t
            .click(this.menuItemObj.membership.element);
    }
}

export default new Profile();
```

That could be used in tests like so:

```javascript
import { Selector } from 'testcafe';
import { reload } from '../Helpers/reload';
import { yearStringFor } from '../Helpers/czechYearString';
import config from '../config';
import Popup from '../Objects/popup';
import LogIn from '../Objects/logIn';
import Profile from '../Objects/profile';
import UserChild from '../Objects/userChild';

const testData = require(`../Resources/${process.env.TESTCAFE_ENV}/updateUserChild.json`);

fixture `Update User Child`    
    .page `${process.env.TESTCAFE_BASE_URL}`;    

testData.updateChildInfo.forEach(updateChildInfo => {
    test
        .meta({ author: 'Pavel Saman', creationDate: '28/05/2020',
            env: process.env.TESTCAFE_ENV, url: process.env.TESTCAFE_BASE_URL
        })
        .before(async t => {
            await Popup.close();

            await LogIn.logIn(testData.username, testData.password);
            await Profile.goToMyKids();            
            await UserChild.addChild(testData.childInfo);
        })        
        ('Update Child', async t => {     

            await UserChild.updateChild(testData.childInfo.name, updateChildInfo);    

            if (updateChildInfo.name) {
                await t
                    .expect(UserChild.child.withExactText(updateChildInfo.name).visible).ok();
            }   

            if (updateChildInfo.age) {
                await t
                    .expect(UserChild.child.withExactText(updateChildInfo.age + " " + yearStringFor(updateChildInfo.age)).visible).ok();
            }
    })
    .after(async t => {
        if (updateChildInfo.name) {
            await UserChild.deleteChild(updateChildInfo.name);
        }
        else {
            await UserChild.deleteChild(testData.childInfo.name);
        }        
    });  
});
```

I've chosen this example on purpose. Yes, TestCafe supports conditional testing, which is an unknown concept in Cypress. I can even write conditional actions in my page objects like so:

```javascript
import { Selector, t } from 'testcafe';

class Popup {
    constructor () {
        this.popupCloseButton = Selector('.popup__close');
    }

    async close () {
        if (await this.popupCloseButton.exists) {
            await t
                .click(this.popupCloseButton);
        }
    }
}

export default new Popup();
```

Other than that, a pretty typical structure and syntax - imports, data driven testing, and hooks are probably what could be seen from this example.

One thing I got stuck on for a bit is how to handle multiple environments. I need to be able to test on a dev environment as well as our staging environment. Normally in other testing tools and frameworks, I would do something like:

```
$ testcafe --env=staging
```

which would pass the option further and I'd choose a corrent url and other parameters for the environment. This is not supported in TestCafe. There's a module called minimist that should be able to handle this, but I didn't make it work (that might have been just me), so I solved it with environment variables (`process.env.TESTCAFE_ENV` and `process.env.TESTCAFE_BASE_URL`). It's still a nice solution and one that TestCafe official documentation suggests as well.

One thing I like in TestCafe is that is alows you to annotate tests and fixtures. I use it only with tests now:

```javascript
test
    .meta({ author: 'Pavel Saman', creationDate: '28/05/2020',
        env: process.env.TESTCAFE_ENV, url: process.env.TESTCAFE_BASE_URL
    })
```

but the same syntax could be used with fixtures. You can come up with your own attributes in this meta object, so you can even assign something like severity to your tests and later choose what tests to execute based on these custom meta attributes. It's handy.

TestCafe doesn't support any `reload()` function to reload the current page, so you perhaps want to write your own helper function:

```javascript
import { t } from 'testcafe';

export async function reload () {
    await t
        .eval(() => location.reload(true));
}
```

One thing that blew my mind was that TestCafe supports so many [browsers](https://devexpress.github.io/testcafe/documentation/guides/concepts/browsers.html) out of the box. Chrome, Firefox are sort of a standard (Cypress only supports Firefox in beta as of 05/2020), but supporting even IE, Edge, Opera, Safari, Chromium, Canary is not that common. And what's more, it just really works. I tried my tests in Chrome, Chromium, Firefox, IE, Edge, and Opera and there is no real trouble with any of these browsers. Definitely not the case in many other tools, including Selenium and Cypress. TestCafe also supports a headless mode with some (?) of these browsers.

My `package.json` so far looks like this:

```json
{
    "scripts": {
        "test": "testcafe",
        "test:headless": "testcafe \"chrome:headless\"",
        "test:headless:chrome": "testcafe \"chrome:headless\"",
        "test:headless:firefox": "testcafe \"firefox:headless\"",
        "test:live": "testcafe -L",
        "browsers": "testcafe --list-browsers",
        "linux:clean": "rm -rf Results/ && mkdir -p Results/Videos && mkdir -p Results/Screenshots"
    },
    "devDependencies": {
        "testcafe": "^1.8.5",
        "testcafe-reporter-json": "^2.2.0",        
        "testcafe-reporter-xunit": "^2.1.0"
    }
}
```

`linux:clean` is funny... yeah, try to make this happen in Windows awful command line environment, that's sort of unreal, so I don't bother.

That could be about everything I wanted to summarise. All in all, TestCafe seems like a decent tool that could be learnt fast. It could be further extended by writing your own reporters or whole modules, so if there's something missing, feel free to reinvent the wheel :)