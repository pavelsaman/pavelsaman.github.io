---
layout: page
title:  "MSSQL and SMSS for Testers"
date:   2019-11-27 17:07:19 +0200
categories: testing
---

Testers usually don't need to know every small detail of SQL and/or DBMS, however, they still need to know enough not to get lost in the middle of their work. I'll show a few examplex of what makes MS SQL unique and how to use this DBMS within SMSS.

### Install SMSS and snippets

First of all, you want to install [SMSS](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15). I have experience with version 18.4 and 17.x, they are very similar, but I'll point out some differences later.

One of the first things you want to do is to make your work more efficient. You'll likely write a lot of selects, so you want to use some snippets in order not to always write `select * from []`. There are more ways how you can get snippets inside SMSS, I actually use an external tool. The result should be something like this:

![image](/images/smss.png)

### Be On The Safe Side

Another thing you actually can see in the picture above is this `BEGIN ROLLBACK` block. It turns out there's no native support for transactions (like there's in e.g. Oracle tools), so if you manage to execute:

`delete from [Order]`

on a production database, you're gonna spend a nice evening restoring data from backups.

Level two of the same would be to create snippets for DML statements that always look like this:

```sql
update top(1) [] set where
delete top(1) from []
```

So even if you manage to leave you the transaction block and execute something that would potentially alter many rows, you always end up updating only one row with such statements. Much safer.

### Some Specifics

I'm sure there are many specifics in MSSQL, especially for developers. Testers will mostly need to query some data so they can check it. Let's suppose you want to get orders ordered by a user id (just an example, you can find a more real one), you'd write something like:

```sql
select * from [Order] o order by o.UserId
```

However, it turns out that orders could be made by unregistered users as well. In that case, column `UserId` is `NULL`. MSSQL will give you these rows first:

![image](/images/mssql.png)

But from whatever reason, you want to keep the order, but you just want to put all the NULL rows at the end. In Oracle, you'd write this

```sql
select * from Order o order by o.UserId nulls last
```

But MSSQL doesn't know this `NULLS LAST`. How to go aroud this? MSSQL allows you to put a case statement into the order part of it:

```sql
select * from [Order] o order by case when o.UserId is null then 1 else 0 end, o.UserId
```

The result is just the way we wanted:

![image](/images/mssql_2.png)

Another trouble might be comparing dates. It seems to me every DBMS has its own functions in this area, so might get a bit confusing at times. In MSSQL, you'd mostly need something like:

```sql
select top(10) * from [Order] o where o.Created>=Convert(datetime, '2019-11-27')
```

So remember there's this `convert()` function that could be used with two params.

### Why []?

That'd be all I wanted to share, but one thing that I can't wrap my head around is why MSSQL uses these `[]` around table names? Two useless characters I have to type. I mean it sometimes works without them, but it's almost in all examples and by default, MSSQL uses them in their intellisense. Does anybody has an idea, why? I'd very much like to know that.

