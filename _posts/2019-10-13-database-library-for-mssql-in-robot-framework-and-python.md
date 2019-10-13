---
layout: page
title:  "Database Library for MS SQL in Robot Framework and Python"
date:   2019-10-13 09:07:19 +0200
categories: testing
---

Robot Framework is a useful tool for test automation. It's mostly used, and suitable, for UI testing because it can be easily integrated with Selenium API. But there could also be many other libraries, either written by other people or we can come up with our own libraries that extend the number of use cases for which Robot could be used. I've recently needed to use Robot in a way when it looks into a DB and checks a few things there. I'm going to share with you a part of my outcome.

### Structure

First of all, having a certain structure for the whole test project is important. As the number of test cases grows, it becomes more and more difficult to know where you keep things (test cases, variables, page objects, ...). You can easily get lost in your project, only having to spend a lot of time every time you need to change a few test cases (or add new ones).

Having said that, I keep the following structure for my test project:

```
.
├── Keywords
├── Libraries
├── Objects
├── README.md
├── Resources
├── Results
├── Settings
└── Tests
```

This is how an empty project in Robot looks when I use the tool. Of course I can further divide some of the folders, so a more real project would end up with a structure similar to this one:

```
.
├── arguments.txt
├── Keywords
│   ├── alpine_admin_keywords.txt
│   └── alpine_keywords.txt
├── Libraries
│   ├── AlpineDB.py
├── Objects
│   ├── alpine_admin_objects.py
│   ├── alpine_objects.py
├── README.md
├── red.xml
├── Resources
│   ├── alpine_admin_test_data.py
│   ├── alpine_sql.py
│   ├── alpine_test_data.py
│   ├── alpine_texts.py
│   ├── alpine_variables.py
├── Results
│   └── Screenshots
├── Settings
│   ├── alpine_admin_keywords.txt
│   ├── alpine_admin_tests.txt
│   ├── alpine_keywords.txt
│   └── alpine_tests.txt
└── Tests
    ├── Admin
    │   └── login_page.robot
    └── Eshop
        └── login_registration.robot
``` 

It's still pretty similar to the empty project structure, you can just see more subdirectories in `Tests/` where I try to keep things more structured, so I know where some particular tests are by just looking at it.

Anyway, this is the basis I'm gonna use in this article.

### DB Library

As I said at the beginning, I want to write a library that connects to a MS SQL database and e.g. gets a number of rows for a particular sql statement. First, I didn't know much about this, but after some time spent googling, I found `pyodbc` module that does exactly what I need. So the first step would be to install it `pip install pyodbc`.

Robot framework allows you to write custom libraries in Python (more natural since Robot is basically a wrapper around Python) and Java. You can read all the details in the [official documentation](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#creating-test-libraries).

Ok, I can't use Java as efficiently as Python, so I go for Python here. However, another decision I need to make fairly early now is what scope I want to use with my library. Robot allows for either global, suite, or test case scope. Again, I'm not going to rewrite the official documentation here, just [read it there](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#test-library-scope), if you want a more detailed explanation. Since I don't want to create a new instance for every test case or a test suite where I use my library, I'm going to go for global scope.

So, a good start would be something like:

```python
from robot.api import logger
import pyodbc


class AlpineDB(object):
    """
    Library for connecting into AP MS SQL DB.

    = Table of content =
    - Keywords
    """

    ROBOT_LIBRARY_VERSION = 1.0
    ROBOT_LIBRARY_SCOPE = 'GLOBAL'

    PARAM_SUBSTITUTE = '{}'
    AP_SHOP_ID_CZ = 121
    AP_SHOP_ID_SK = 1810   

    def __init__(self, url='https://inveo-alpineprocz.net'):
        if 'dev' in url.lower() or 'test' in url.lower():
            self.driver = '{SQL Server}'
            self.server = 'server'
            self.database = 'database'
            self.username = ''
            self.pwd = ''
            if 'alpineprosk' in url.lower():
                self.shop_id = self.AP_SHOP_ID_SK
            else:
                self.shop_id = self.AP_SHOP_ID_CZ
        elif 'staging' in url.lower():
            self.driver = '{SQL Server}'
            self.server = 'server1'
            self.username = ''
            self.pwd = ''
            if 'alpineprosk' in url.lower():
                self.database = 'database1'
                self.shop_id = self.AP_SHOP_ID_SK
            else:
                self.database = 'database2'
                self.shop_id = self.AP_SHOP_ID_CZ

        self.conn = pyodbc.connect(Driver=self.driver,
                                   Server=self.server,
                                   Database=self.database,
                                   uid=self.username,
                                   pwd=self.pwd)    

```

A few words about this. I've of course deleted all sensitive data like usernames, passwords, database names. I hope it's still understandable enough. The point here is I want to use this library with different environments (against different databases and servers), so I need to create the right connection first. Therefore, there's a bit of logic in the `__init__` method.

As soon as I have the right connection into a database, I can write some methods that later become available as keywords in Robot framework. Let's start easy with a method that counts a number of rows:

```python
def build_sql_statement(self, sql_statement, params):
        """
        Builds a SQL statemets given a SQL statements and parameters. String parameters are automatically enclosed in quotes, so no need to use quotes in params list.
        
        Example:
        | @{params}= | Create List | 1234 |
        | ${sql_statement}= | Build Sql Statement | Select * from [User] where UserId = {} | ${params} |
        """
        # enclose strings in quotes
        for index, value in enumerate(params):
            if isinstance(value, str):
                params[index] = '\'' + value + '\''     

        if sql_statement.count(self.PARAM_SUBSTITUTE) != len(params):
            logger.console('SQL statement requires different number of parameters.')
            logger.error('SQL statement requires different number of parameters.')
            raise AssertionError('SQL statement requires different number of parameters.')

        # add shopid condition
        sql = sql_statement + ' and ShopId=' + str(self.shop_id)
        return sql.format(*params)

    def row_count(self, sql_statement):
        """
        Returns a row count given a SQL statement.
        
        Example:
        | @{params}= | Create List | 1234 |
        | ${sql_statement}= | Build Sql Statement | Select * from [User] where UserId = {} | ${params} |
        | ${row_count}= | Row Count | ${sql_statement} |
        """        
        cursor = self.conn.cursor()
        cursor.execute(sql_statement)
        return len(cursor.fetchall())
```

Instead of just one method, I also wrote method `build_sql_statement`. That's because I want to keep just sql templates that could be used with different conditions. I want to hard code as few things as possible. Plus the part with `ShopId` is very much specific to my project. It turned out that one of the databases is shared across more than one test environment. The parameter that distinguishes whether a particular row belongs to this or that environment is `ShopId`, so that's why I have this little logic here when building sql statements.

Method `row_count` is just a simple one, I guess no explanation is needed here.

Also notice that because I'm gonna use this library with Robot framework, I'm using `robot.api` and `logger`. I'm also using a particular [documentation style](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#writing-documentation) that allows for auto-generation of documentation for this library. It comes in handy when someone else wants to use the library after me.

Just to summarize, the whole library at this point looks like this:

```python
from robot.api import logger
import pyodbc


class AlpineDB(object):
    """
    Library for connecting into AP MS SQL DB.

    = Table of content =
    - Keywords
    """

    ROBOT_LIBRARY_VERSION = 1.0
    ROBOT_LIBRARY_SCOPE = 'TEST SUITE'

    PARAM_SUBSTITUTE = '{}'
    AP_SHOP_ID_CZ = 121
    AP_SHOP_ID_SK = 1810   

    def __init__(self, url='https://inveo-alpineprocz.net'):
        if 'dev' in url.lower() or 'test' in url.lower():
            self.driver = '{SQL Server}'
            self.server = 'server'
            self.database = 'database'
            self.username = ''
            self.pwd = ''
            if 'alpineprosk' in url.lower():
                self.shop_id = self.AP_SHOP_ID_SK
            else:
                self.shop_id = self.AP_SHOP_ID_CZ
        elif 'staging' in url.lower():
            self.driver = '{SQL Server}'
            self.server = 'server1'
            self.username = ''
            self.pwd = ''
            if 'alpineprosk' in url.lower():
                self.database = 'database1'
                self.shop_id = self.AP_SHOP_ID_SK
            else:
                self.database = 'database2'
                self.shop_id = self.AP_SHOP_ID_CZ

        self.conn = pyodbc.connect(Driver=self.driver,
                                   Server=self.server,
                                   Database=self.database,
                                   uid=self.username,
                                   pwd=self.pwd)   

    def build_sql_statement(self, sql_statement, params):
        """
        Builds a SQL statemets given a SQL statements and parameters. String parameters are automatically enclosed in quotes, so no need to use quotes in params list.
        
        Example:
        | @{params}= | Create List | 1234 |
        | ${sql_statement}= | Build Sql Statement | Select * from [User] where UserId = {} | ${params} |
        """
        # enclose strings in quotes
        for index, value in enumerate(params):
            if isinstance(value, str):
                params[index] = '\'' + value + '\''     

        if sql_statement.count(self.PARAM_SUBSTITUTE) != len(params):
            logger.console('SQL statement requires different number of parameters.')
            logger.error('SQL statement requires different number of parameters.')
            raise AssertionError('SQL statement requires different number of parameters.')

        # add shopid condition
        sql = sql_statement + ' and ShopId=' + str(self.shop_id)
        return sql.format(*params)

    def row_count(self, sql_statement):
        """
        Returns a row count given a SQL statement.
        
        Example:
        | @{params}= | Create List | 1234 |
        | ${sql_statement}= | Build Sql Statement | Select * from [User] where UserId = {} | ${params} |
        | ${row_count}= | Row Count | ${sql_statement} |
        """        
        cursor = self.conn.cursor()
        cursor.execute(sql_statement)
        return len(cursor.fetchall())

```

It doesn't do much, but it's a good start, more methods could be easily added into this structure.

All I need to do is save this as a new python file inside `<my_project>/Libraries/`.

One last note on this. Notice the default parameter value of `url` in `__init__`. That's not so much because I really want to use it like this, but because I use Eclipse and RED as my IDEs for Robot framework and Eclipse/RED has a particular bug here that doesn't allow for object creation (AlpineDB in my example) when no default value is not present. The symptoms of this are that the whole library would be marked as invalid when I have my project open in Eclipse/RED, which complicates things like seeing documentation for my custom library or using intellisense. However, when run, there's no problem even without the default value. So it's basically just a bit more comfortable to use this workaround.

### Robot Framework

Let's now use this library inside Robot framework.

To keep the structure, I'm gonna first create a new test suite inside `<my_project>/Tests/`.

```bash
$ touch db.robot
```

Then let's create a test case that checks if only one user exists in a database using my library from above:

```
*** Settings ***
Documentation    Database checks and tests.  
Resource    ../Settings/alpine_tests.txt
Library    ../Libraries/AlpineDB.py    ${URL_DEV}

*** Variables ***
${no_user_in_db}=    User was not found in DB.
${too_many_users_in_db}=    User was not found in DB.

*** Test Cases ***
One User Exists In Database
    [Tags]    smoke    db
    ${params}=    Create List    ${valid_username}
    ${sql_statement}=    Build Sql Statement    ${select_user_by_email}    ${params}
    ${number_of_users}=    Row Count    ${sql_statement}
    Run Keyword If    ${number_of_users} == 0    Fail    ${no_user_in_db}
    Run Keyword If    ${number_of_users} > 1    Fail    ${too_many_users_in_db}
```

As you can see, I try to avoid hard-coded values. So I'm using variables that live inside files in `<my_project>/Resources` and are imported here via file `<my_project>/Settings/alpine_tests.txt`.

I've also mentioned something about sql templates. That's exactly what I'm using here. One such template is in variable `${select_user_by_email}`, the value of this variable is in fact `select_user_by_email = 'Select * from [User] u where u.Email={}'`, this lives inside `<my_project>/Resources/alpine_sql.py` and is again imported via `<my_project>/Settings/alpine_tests.txt`. It helps keep things organised, because I can keep all sql statements in one file, so there's no need to look at various places if I want to change something.

The rest is probably clear enough, notice how my python methods have become keywords here in Robot. So I can easily call my `row_count` using keyword `Row Count` and my `build_sql_statement` using keyword `Build Sql Statement`.

I'll leave running the test case up to you, but you'd do something like this:

```bash
$ robot --argumentfile ../arguments.txt --test "One User Exists In Database" db.robot
```

Based on how you set it up, you'd get reports and logs. In my case, I have them timestamped (`--timestampoutput` option inside `<my_project>/arguments.txt`) and saved (option `--outputdir ../Results` inside `<my_project>/arguments.txt`) into `<my_project>/Results`.

### Documentation

I mentioned something about auto-generating the documentation for my library. It can't be easier, all you need to do is follow the guidlines on how to write comments in the source file and then execute `python.libdoc`:

```bash
$ python -m python.libdoc -f html AlpineDB.py AlpineDB_Library.html
```

It will generate a file in the format that you know from [official Robot documentation](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html).

I strongly recommend using it, especially if you're not the only Tester that's gonna use your libraries.


### Not Reinventing The Wheel

It'd be strange if I was the first one to think about this topic. Of course other people have done exactly that before me. But I think it's always a good thing to take things apart and learn how they work, that's why I wrote my own simple database library.

However, if you don't want to do the same, just use what's already done. There is a [database library](https://franz-see.github.io/Robotframework-Database-Library/api/0.5/DatabaseLibrary.html) already written, you can get in on [pypi.org](https://pypi.org/project/robotframework-databaselibrary/) and see documentation [here](https://franz-see.github.io/Robotframework-Database-Library/api/0.5/DatabaseLibrary.html).

