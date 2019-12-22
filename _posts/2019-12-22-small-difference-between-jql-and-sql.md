---
layout: page
title:  "Small Difference Between JQL and SQL"
date:   2019-12-22 16:07:19 +0200
categories: other
---

I've recently written a small [Python script](https://github.com/pavelsaman/jira-tasks) for getting information out of Jira. My motivation was mainly this: going into a browser, refreshing the page or even going to my dashboard can easily take up to 20 seconds (Jira is extremely slow). So I created a small console Python app that gets me all the information I need on a daily basis when testing. However, in the process of doing this, I've found out a small but interesting difference between **J**ira **Q**uery **L**anguage and SQL.

Both JQL and SQL exist for the same purpose, you need to query data from a database, so there's a language for exactly this very purpose. JQL has been pupularized (and created) for one of the most popular issue & project tracking system Jira. SQL has, on the other hand, a different and longer history, and I won't go into this because I assume just about everybody knows some of it.

### Operator `in`

In both languages, there's an operator `in`, one example in SQL could be:

```sql
select * from Order where Status in (1, 2, 3)
```

This would return all rows in `Order` table that have one of the following statuses: 1, 2, or 3. If one of the statuses doesn't exist, but some other do, no error is returned, it simply returns those rows that meet the condition. Perfectly understandable, for me as a SW Tester, this `in` operator is something I use often for getting data from databases.

### JQL

Atlassian's documentation about the operator `in` could be found [here](https://confluence.atlassian.com/jirasoftwarecloud/advanced-searching-operators-reference-764478341.html). But how does it work in reality?

My search in the Python script is this:

```python
query = {
    'jql': 'project in ({0}) AND status in ("{1}")'.format(','.join(PROJECTS), env)
}
```

This can translate to a query like this:

```
project in ("Bambule CZ E-commerce", "Alpine Pro CZ&SK E-commerce", "Asko CZ&SK E-commerce", "Sparkys CZ E-commerce", "Toys club CZ E-commerce", "E-commerce DEV", "Teta drogerie CZ DEV", "Pongo CZ E-commerce") AND status in ("Needs testing on DEV")
```

And I use `/search` resource with GET. More about this [here](https://developer.atlassian.com/cloud/jira/platform/rest/v3/?utm_source=%2Fcloud%2Fjira%2Fplatform%2Frest%2F&utm_medium=302#api-rest-api-3-search-get).

I simply need to find tickets I need to test. And I limit the result to just some projects where I'm a Tester.

In Postman, after sending such a request, I got status code 200 and some json data:

![image](/images/jira_req.png)

However, when testing my Python script, I managed to add a non-existing project into the list of projects after the `in` operator, the query looked something like this:

```
project in ("Bambule CZ E-commerce", "Alpine Pro CZ&SK E-commerce", "Asko CZ&SK E-commerce", "Sparkys CZ E-commerce", "Toys club CZ E-commerce", "E-commerce DEV", "Teta drogerie CZ DEV", "Pongo CZ E-commerce", "blabla") AND status in ("Needs testing on DEV")
```

And Jira returned status code 400:

![image](/images/jira_req.png)

So it turns out that if one of the values in the list of values after `in` operator is e.g. mispelled, the result will always be "bad query". But why? What if I had, that's exactly my situation actually, tickets that meet the condition, but just one project is mispelled or non-existing? Why do I need to check the whole query, correct it, and try it again?

This doesn't seem very convenient, because that's not how SQL works and I guess many people are so used to it they don't even expect something so fundamental to work so differently. You also sort of imagine `or` between the values (`project = "a" or project = "b" ...`, which is even what Atlassian says in the [documentation](https://confluence.atlassian.com/jirasoftwarecloud/advanced-searching-operators-reference-764478341.html)), so I don't understand why I have to make sure all the values exist when I use `or`.

### Conclusion

I just need to be a bit more aware that Atlassian made their own rules that are not necessarily the same as in SQL. In this example, it wouldn't help reading their documentation either since they don't mention this important condition, nor the difference from SQL, which further complicates things, because one only finds out by trial and error.

