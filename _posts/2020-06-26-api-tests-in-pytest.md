---
layout: page
title:  "API Tests In Pytest"
date:   2020-06-26 12:00:19 +0200
categories: testing
---

I've recently writen 200+ API tests in pytest across two different APIs, so I feel like summarising (mostly for my future self) some key points. Perhaps it might be useful to others as well, so I'll write it as a blog post.

Before getting straight to code, it's better to think a bit about various things such as:

- where is a documentation for the web service, do I understand it?
- do I understand what the API is for? What do I need to test for?
- can I access the API's controllers and other code so I can fill blank spots where the documentation left me wondering
- project structure - messy project dir and structure lead to messy, flaky, and hard to maintain tests
- do I need to get some test data from e.g. a database? what will I use for that?
- what documentation will I write for my tests - it's reasonable to assume that one day someone else will take over my tests, I should make their job easy
- how to store test data?
- how to write concrete test cases
- what will be my environment for the project?

In my case, I can answer many of these:

- yes, there's a documentation on Swagger UI, I can read it and see examples that I can explore using e.g. Postman
- I can also access the code so I can get another insight into the API; plus I can always ask devs
- yes, I do need to get test data from a database, in my case, it's a MSSQL database for which I can use `pyodbc` module
- I try to name tests properly, keep them as short as possible and strictly independent on other test cases, that way, it creates little need for heavy documentation, yet I choose to write a short README file that explains how to run tests or e.g. what custom marks I use and what tests will be run if some marks are specified on the command line or in `pytest.ini`
- in terms of test data, I've chosen an easy tabular format (csv), which could be created by anybody, even less technical people, which might come in handy later when I can ask someone else to come up with more test cases
- test cases should be short, easy to understand, ideally written in BDD style, little to none conditionals in them, and one test case tests one thing only, it leads nowhere if I create one huge test method spanning tens of lines where I test for 5 different things, it's not readable, it's easy to make a mistake, so I try to avoid it
- of course, test cases has to be independent on each other; this is such a common problem, just see how many times something like _"how can I run my tests in a particular order"_ has been asked on stackoverflow and elsewhere; if you need to run your tests in a particular order, your design is wrong in the first place and you should change it in the first place
- regarding the environment, I try to separate it as well, I use `pipenv`, so it creates a requirement file that enumerates all dependencies needed to run the tests

Project structure requires a little bit more thoughts in my opinion, some key points I have identified:

- I want to separate test data, tests (test cases), config values, helpe functions, custom libraries; ideally, I want to only use variables and parametrization, plus perhaps some constants, but no hard-coded values in my tests
- I want to keep a config file somewhere separately, where I can e.g. store values needed for DB connection, etc.
- I tend to choose the following project dir structure:

```
Config/
Libraries/
Helpers/
Resources/
Results/
Tests/
```

It should be clear enough for anyone who comes to my project later on.

Having said that, I can get into coding now. I'll show just some bits.

In order to get test data from csv files, I need to create a simple piece of code for that:

**Libraries/CSVReader.py**
```python
import csv
from Config.config import Config


class CSVReader:

    @staticmethod
    def read_from_csv(filename):
        with open(filename, newline=Config.UNIVERSAL_NEWLINE) as csv_file:
            data = csv.reader(csv_file, delimiter=Config.DELIMETER)
            next(data)
            test_data = [row for row in data]
        return test_data

```

I can later use it in my tests:

```python
import pytest
import requests
from Config.config import Config
from Libraries.CSVReader import CSVReader


API_ENDPOINT = "/Token"


@pytest.mark.parametrize("authorization, header_value, expected_status_code, expected_reason",
                         CSVReader.read_from_csv("Resources/get_token_authorization.csv"))
def test_authorization(authorization, header_value, expected_status_code, expected_reason):
    pass
```

If I need to connect to a database that I need to use in my tests (e.g. retrieve something), I should do it via fictures:

**Tests/conftest.py**
```python
import pytest
from Libraries.Tenant import Tenant


@pytest.fixture(scope="session")
def tenant_db():
    return Tenant()
```

I can use it later in my tests:

```python
@pytest.mark.db
def test_new_token(tenant_db):
	pass
```

From now on in the test case, I can use `tenant_db` object and all its methods. This way, I can talk to the DB, to a particular table in this case. I also use a custom mark `db` that I use for all tests that connect to the DB, I define the mark in `pytest.ini`:

**pytest.ini**
```python
markers =
    db: tests that require access to the DB
```

And I can run e.g. all test that do not require the access to the DB like so: `$ pytest -m "not db"`

I also talked about BDD style in relation to writing tests. All it means is that I should keep the following structure:

```python
def test_bdd_example():
	# pre-conditions, given something

	# action, the actual API call

	# then, assert(s)
```

Just thinking in this way helps keep the test case simpler and clean.

One example of a test case at the end:

```python
import pytest
import requests
import dateutil.parser
from Helpers.datetime_utils import get_current_day
from Config.config import Config
from Libraries.CSVReader import CSVReader

@pytest.mark.db
@pytest.mark.parametrize("tenant_name", ["MyTenantName"])
def test_new_token(access_token_db, tenant_db, tenant_name):
    print(tenant_name)
    tenant_id = tenant_db.tenant_id(tenant_name)
    access_token_db.delete_tokens_for_tenant(tenant_id)

    api_key = Config.Tenant[Config.env()][tenant_name]["ApiKey"]
    response = requests.get(f"{Config.base_url()}/{API_ENDPOINT}",
                            headers={"Authorization": f"Basic {api_key}"})

    assert response.status_code == 200
    assert response.reason == "OK"
    print(response.text)
    response_body = response.json()
    valid_until = dateutil.parser.parse(response_body["validUntil"])
    assert valid_until.day == get_current_day() + Config.TOKEN_VALIDITY_IN_DAYS
```

In pytest, it's also possible (and sometimes beneficial) to structure test cases in test classes. I've, however, taken these examples from a simple API test suite where I didn't go for this approach since it'd unreasonably create more complicated structure.

I hope this will be usuful not only to myself later, but to others as well.