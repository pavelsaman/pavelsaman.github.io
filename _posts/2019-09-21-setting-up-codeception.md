---
layout: page
title:  "Setting Up Codeception"
date:   2019-09-21 13:07:19 +0200
categories: testing
---

Some time ago, I started using and learning Codeception which I find a pretty good testing tool. In here, I'll once again go over the initial steps of setting it up since there're a few paths to take.

### What is Codeception

First off, there's an [official site](https://codeception.com/) of the project. It says:

```
Codeception collects and shares best practices and solutions for 
testing PHP web applications.
With a flexible set of included modules tests are easy to write,
easy to use and easy to maintain.
```

I'm a bit skeptical about the best practices part, since I guess it depends on a particular project and context. They also don't really say what Codeception is, only that it collects something etc. So in my opinion, better to scroll all the way down where they say a bit more in the footer:

```
Codeception is a BDD-styled PHP testing framework,
brought to you by Codeception Team.
OpenSource MIT Licensed.
```

Finally some useful information. So it's a testing framework, based on PHP. And it's open source so no extra expenses for the project. Now we can decide if it's suitable for us or not.

On the main page, there're some other hints about what Codeception can help you do. I'll go briefly over some of them:

1. `user-centric tests`

Fine, it probably means we can write tests the way non-programmers understand them. Could be a benefit, but my experience is, no one other than testers and programmers want to have anything to do with such technical thing anyway.

2. `browser testing`

Testing the more or less final product, the UI, so we're talking about acceptance testing here.

3. `framework testing`

This seems interesting. Testing in browser (UI testing) is also slow, if Codeception has the ability to test on lower level, it's might come in handy. Now we're talking about integration and system testing.

4. `API testing`

There're likely be some modules that allow for API testing. That might be interesting so we don't need to use different tools for different testing tasks, but stick to Codeception only.

5. `data-drive tests`

Seems it will be easy to use test data from various databases.

6. `unit and integration testing`

Aha, so we can do even unit testing with Codeception. Along with integration and acceptance tests, we come to a situation where Codeception can cover a wide range of tests, no need to use various tools and framework. Could be an advantage, but as I said, I think it depends on a context.

### Installation

Let's try it out. There ar a few requirements, so let's make sure, you have:
1. php installed
There's no php on [Manjaro](https://distrowatch.com/table.php?distribution=manjaro) for example, so first I need to get this over with:
```bash
$ sudo pacman -Syu
$ sudo pacman -S php
```

Then I find out there are two (at least) options to get Codeception working.
#### Global Installation
In order to install Codeception globally, all I need to do is download `codecept.phar` file:
```bash
$ wget https://codeception.com/codecept.phar
```

To make it easier for me, I put the file in into `PATH`, make it executable and either create an alias `codecept` or a symlink:

```bash
lrwxrwxrwx  1 pavel pavel   13 Sep  7 23:31 codecept -> codecept.phar
-rwxr-----  1 pavel pavel 7.0M Aug 25 00:01 codecept.phar
```

This is the situation on my machine.

#### Local Installation
Sooner or later, you're gonna discover that Codeception has some modules and extensions that are not in the default installation. They need to be added, installed pretty much the same way Python installes additional packages via pip or Ruby uses gems. Then it becomes a bit too overwhelming to do the task globally. And just like there're virtual environments in Python in which you can have different packages that are not accessible globally, the similar situation could be achived for Codeception via [composer](https://getcomposer.org/).

Again, composer needs to be installed first:

```bash
$ sudo pacman -Syu
$ sudo pacman -S composer
```

Then go to your project root directory and install codeception:

```bash
$ composer require codeception/codeception --dev
```

If you already know you won't need acceptance tests, you can get codeception without stuff like WebDriver:

```bash
$ composer require codeception/codeception --dev
```

Either approach you take here, the final outcome of this effort should be you can execute codeception, for instance:

```bash
$ codecept -V
Codeception 3.0.3
```

If you used the local installation, there're gonna be 3 more files (2 files, one directory) in your project root folder:
```bash
-rw-r--r--  1 pavel pavel   73 Sep 21 14:40 composer.json
-rw-r--r--  1 pavel pavel 117K Sep 21 14:41 composer.lock
drwxr-xr-x 21 pavel pavel 4.0K Sep 21 14:41 vendor
```

Any time you want to execute the local installation of codeception, you'll be executing the one in `vendor/bin/codecept`.

### Generate Tests

After the successful installation, it's time to generate some test structure and write some tests.

First you need to initialize the config files and create some basic structure for the tests:

```bash
$ php vendor/bin/codecept bootstrap
```

Upon successful completion, 2 things happen:

1. it adds `codeception.yml` into the project root dir
2. it created directory `tests`
This dir has the following structure ![image](/images/codeception_structure.png)

I find this useful because it already guides me how to structure files within my tests and where to put what. So there's less change to end up with one big mess where test data are among tests or everything is in one file. The disadvantage is obvious, this can take longer to learn, as a newcomer to the framework, I might be lost in this structure.

Another interesting thing to notice is, codeception already created 3 types of files in `_support/_generated`, files for unit, functional, and acceptance testing. Only the work functional might get confusing here. It doesn't mean that acceptance or unit tests within codeception are not functional, th word simply related to integration tests, the ones run without the UI.

From now on in this article, I'll go for some acceptance tests.

I let codeception generate a basic structure:

```bash
$ php vendor/bin/codecept generate:cest acceptance FirstSmokeTest
```

This creates a new file named `FirstSmokeTest.php` inside `tests/acceptance/`. The content of the file is pretty simple:

```bash
$ cat FirstSmokeTest.php
<?php 

class FirstSmokeTestCest
{
    public function _before(AcceptanceTester $I)
    {
    }

    // tests
    public function tryToTest(AcceptanceTester $I)
    {
    }
}
```

This is where the tests go into.

I should probably mention right here, that it's a good idea to use some sort of structure, or design pattern if you want, for the tests. It makes it more readable and easy to maintain in the long run. There's nothing worse than to put all test steps inside this FirstSmokeTest.php (or any similar file for that matter), only to wonder later how to reuse some of those test steps. Therefore, I will follow this path right from the beginning, because I believe it's worth it to learn the right way right from the beginning, not to unlearn bad habits later. On the othe hand, it will become a bit more difficult now.

However, I ned to cover one more thing right now. That is how to execute the tests. There are pretty much 3 option:

#### Selenium Server

This is a recomended path by Codeception team and I guess the most used one today. However, it requires some other prerequisites:
1. Java for running a selenium server
2. browsers
3. browser drivers
4. GUI to actually display something

The idea behind Selenium Server is that you connect to the server and tell tge server what tests you want to run and the server manages other things like firing up browser instances. This could, and likely will be, done on a different machine to which you connect to executethe tests.

#### Chromedriver

Chromedriver could be used even as a separate tool (not with Selenium Server), to which you can connect directly to execute some tests. This is definitely easier to set up.

#### PhpBrowser

PhpBrowser is so-called headless browser. You don't actually need to have any browsers installed and it doesn't require a GUI. What it does instead is it uses a PHP scraper, which basically means it sends requests and then parses the responses. But it has some limitations (see [the official documentation](https://codeception.com/docs/03-AcceptanceTests)), so I won't go for this option. I also think that when we decide to run acceptance tests, we should do acceptance tests, so why leave out GUI.



