---
layout: page
title:  "JavaScript Default Package Setting with npm config"
date:   2021-01-14 13:30 +0200
categories: other
---

Npm package manager has much more to it than just `npm init` or `npm install`. There's also a bunch of config options that could save some time later or save a lot of trouble with managing package versions.

### current config

First off, it's useful to know what config values are currently set:

```
$ npm config ls -l
```

That will print config values. There'll be `userconfig` section that starts like so:

```
; userconfig C:\Users\PavelSaman\.npmrc
```

All user option are saved (by default) in `.npmrc` file.

### init

`$ npm init -y` is very often how we start a new project. It has some reasonable default options, but many times we still have to overwrite them sooner or later. You can simplify this task by setting up what default values should go into `package.json` upon issuing npm init command.

If I want to set `author` to my name by default, I can do it with:

```
$ npm config set init-author-name="Pavel Saman"
```

Similarly with email, I can use `init-author-email` option.

Default version is 1.0.0 (it's also recommended to start with this version), if I want to start with 0.0.1, I can set such a default value with npm config as well:

```
$ npm config set init-version="0.0.1"
```

### version control

Your package will likely be dependent on other packages and some other packages might over time become dependent on your package. The problem we might encounter is that versions of all of these packages change, which might break something over time. Therefore, it would be a good idea to control the versions of packages we use.

If you have a look into `package.json`, by default, you'll see such values for dependencies:

```json
{
    "devDependencies": {
        "chai": "^4.2.0",
        "eslint": "^7.17.0",
        "eslint-plugin-import": "^2.22.1",
        "mocha": "^8.2.1"
    }
}
```

The carets (`^`) basically mean that it's possible to perform minor upgrades. If I start with `~`, it's possible to do only patch upgrades. If I include only a version (e.g. `"chai": "4.2.0"`), it will install only this very version.

The caret is a default option, but if I want to be a bit more certain about what versions I use, I can use `~`, or refuse to upgrade completely with en ampty prefix. To override the default option, I can use these commands:

```
$ npm config set save-prefix='~'
$ npm config set save-prefix=''
```

You can read more about the logic behind versioning [here](https://docs.npmjs.com/about-semantic-versioning).

There's much more to it, these are the values I've recently overriden, so I don't need to change everything manually later. All the option could be found [here](https://docs.npmjs.com/cli/v6/using-npm/config), so have a look if you need to do more tricks with it.