---
layout: page
title:  "Decorators in Robot Framework"
date:   2020-07-05 11:30:19 +0200
categories: testing
---

Robot Framework (RF) in its version 3.2 released in April of 2020 allows for a new syntax regarding custom test libraries. I've already used it once on one project, so let's have a look.

RF is basically a wrapper around Python, so if something could be done in Python, it could also be done in RF. It brings along one obvious advantage, RF could be easily [extended](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#creating-test-library-class-or-module), which makes it easy to use even in more difficult situations that require the use of a proper language.

Writting custom test libraries is virtually possible in two versions - as modules, or as classes.

Example of a test library defined as a Python module could look like this:

```python
import random

def random_number(min: int, max: int):
    return random.randint(min, max)
```

Or I can define test libraries as Python classes:

```python
import random

class Random:

    ROBOT_LIBRARY_SCOPE = 'GLOBAL'
    ROBOT_LIBRARY_VERSION = '0.1'

    def random_number(min: int, max: int):
        return random.randint(min, max)
```

Which version you choose is perhaps based on how big the library will be, easy keywords do not necessarily be wrapped in classes.

The new syntax I talked about is related to `ROBOT_LIBRARY_SCOPE`, `ROBOT_LIBRARY_VERSION` that I used in the last example. And also to `ROBOT_LIBRARY_DOC_FORMAT`, `ROBOT_LIBRARY_LISTENER`, and `ROBOT_AUTO_KEYWORDS` that I didn't use. All these options could also be defined as part of the decorator `@library`:

```python
import random
from robot.api.deco import library

@library(scope='GLOBAL', version='0.1')
class Random:

    def random_number(min: int, max: int):
        return random.randint(min, max)
```

This decorator comes from the `robot.api.deco` API, so it has to be imported as shown in the example.

In my view, it can make the whole class a little bit more clean, not cluttering it with meta information. So I must say, I like this option.

However, there comes one different behaviour with the decorator. When used, `ROBOT_AUTO_KEYWORDS` is automatically set to `False`, so RF won't automatically discover keywords. Therefore, you have to tell RF what methods are meant to be used as keywords in RF with another deocrator `@keyword`.

So my previous example would actually not import any keywords into RF, to correct it, I need to type:

```python
import random
from robot.api.deco import library, keyword

@library(scope='GLOBAL', version='0.1')
class Random:

    @keyword
    def random_number(min: int, max: int):
        return random.randint(min, max)
```

From now on, I can use `Random Number` as a keywords in RF.

This makes the whole syntax a little bit more explicit, which I personally like, plus you don't need to prefix methods with an underscore to tell RF not to use such a method as a keyword. It's very much a different approach than before when all methods except those starting with an underscore were automatically added as keywords, now with `@library` decorator, none are automatically added.

The `@keyword` decorator (introduced before version 3.2) could be used in various ways as well. For example, I specified what types the arguments are using function annotation, the same could be achieved like this:

```python
import random
from robot.api.deco import library, keyword

@library(scope='GLOBAL', version='0.1')
class Random:

    @keyword(types={'min': int, 'max': int})
    def random_number(min, max):
        return random.randint(min, max)
```

Note that `types` doesn't have to be a dict, it could be a list as well, but than you need to know that it matches the types to arguments from left to right, which seems a little bit more prone to error, especially when you want to skip conversion for one argument somewhere in the middle of your argument list (I know, you can use `None` for that).

I can also turn off the automatic type conversion, change the keyword's name, specify tags, or allow for a custom name with embedded arguments like I do in this example:

```python
import random
from robot.api.deco import library, keyword

@library(scope='GLOBAL', version='0.1')
class Random:

    @keyword('Random Number Between ${min:\d+} And ${max:\d+}')
    def random_number(min: int, max: int):
        return random.randint(min, max)
```

In terms of types, I just used a function annotations, which wasn't previously possible (before version 3.0.2) in RF either, but was fixed in [here](https://github.com/robotframework/robotframework/issues/2627).

All in all, these changes are more about different syntax (except the [type conversions](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#supported-conversions)), which seems to be a little more clean and lets you focus on implementing actual keywords.