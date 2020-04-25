---
layout: page
title:  "Premature Optimization, Benchmarking, and Why IT Seems Difficult"
date:   2020-04-25 13:00:19 +0200
categories: perl
---

It's a widespead notion that [premature optimization](https://softwareengineering.stackexchange.com/questions/80084/is-premature-optimization-really-the-root-of-all-evil) is bad, or as many say, the root of all evil. Some add other bits into the discussion such as optimization without prior measurement is always premature. I don't have specific developer advice in this area (well, I'm not a developer after all), but I'll share my personal opinion on why I think IT world seems difficult to many in regard to this very topic.

If you've gone to an IT high school just like me, you were probably tought some programming language. Many times we did just some basics such as learning the syntax, some basic stuctures of the language, so we were able to piece together some useless stuff such as various ways to sort a list etc.

To be a bit more specific, a typical way we were tought to use any programming language (we mostly learnt some C, C#) was something like this (I'll show all these examples in Perl):

```perl
sub unique {
    my @arr = @_;    

    my @unique = ();
    foreach my $i (0..scalar @arr - 1) {
        my $not_found = 1;        
        foreach my $j (0..scalar @arr - 1) {
            if ($i != $j && $arr[$i] == $arr[$j]) {
                $not_found = 0;                               
            }
        }

        if ($not_found) {
            push @unique, $arr[$i];
        }
    }

    return @unique;
}
```

If it seems messy, useless, and too long, that's about right. It also won't work for string lists, so it's not even scalable. Yet, this is the way we were tought.

However, I still have no data to be completely certain about why this is a horrible piece of code (well, it's too long, buggy and not scalable, that's probably enough), so a logical next step would be to benchmark it against another solution.

Knowing some modules in Perl, I'll simply use [`List::MoreUtils`](https://metacpan.org/pod/List::MoreUtils). I should perhaps stop in my article for a second and highlight one thing: **it's important to know standard libraries and other libraries and modules**. This is also a thing we were **not** tought in school, so nearly everyone ended up writing those horrible pieces of code with no real benefit at all.

Having said that, I can read the documentation of `List::MoreUtils` and find out there's a nice subroutine `singleton` which does exactly what I tried to do with the first example. So instead of writting buggy and long code, I can simply come up with:

```perl
sub unique {
    return singleton @_;
}
```

What a change - clean, readable, scalable, only 3 lines of code.

When I finally have two options, I can benchmark them, the whole piece of code would now look like this:

```perl
#!/usr/bin/env perl

use List::MoreUtils qw(singleton);

sub unique_a {
    my @arr = @_;    

    my @unique = ();
    foreach my $i (0..scalar @arr - 1) {
        my $not_found = 1;        
        foreach my $j (0..scalar @arr - 1) {
            if ($i != $j && $arr[$i] == $arr[$j]) {
                $not_found = 0;                               
            }
        }

        if ($not_found) {
            push @unique, $arr[$i];
        }
    }

    return @unique;
}

sub unique_b {
    return singleton @_;
}

################################################################################

our @data = (1, 2, 3, 4, 5, 4, 5);

use Benchmark qw(cmpthese);
cmpthese -10, {
    uniq_a => 'my @r = unique_a(@data)',
    uniq_b => 'my @r1 = unique_b(@data)'
};

exit 0;
```

I'm using [`Benchmark`](https://metacpan.org/pod/Benchmark) module, you can read up on the details in the link.

If I run this script for only those 7 examples, the results are as follows:

```
$ ./benchmark.pl 
           Rate uniq_a uniq_b
uniq_a 147508/s     --   -77%
uniq_b 654177/s   343%     --
```

In other words, that horrible solution with loops is not even close to using an existing solution. In this case (on this data and machine), the existing solution is 3.43 times faster! And we're only talking about such a small data set. If I increase the data set 100 times:

```perl
@data = (@data) x 100;
```

the results are even more remarkable:

```
$ ./benchmark.pl 
          Rate uniq_a uniq_b
uniq_a  2454/s     --   -91%
uniq_b 26946/s   998%     --
```

Furthermore, you can improve the singleton variant as well. You can use a compiled version [`List::MoreUtils::XS`](https://metacpan.org/pod/List::MoreUtils::XS) which will likely give you another speed improvement.

Now I even have data needed for deciding why such optimization was premature - because of its speed.

My point is that in most cases, optimization not only creates a buggy, not scalable, messy, and slow code, but it also takes away one's mental energy, so instead of focusing on real problems, you focus on fixing some bugs that you have unnecessary created in the first place. And that is, in my opinion, why so many people feel IT is so difficult, because many times, starting in school like in my case, it's about writting those little messy pieces of code, not focusing on code reuse and real problems.

To take this a step further, it's not only about "low-level" code as in these examples, but about whole frameworks and solutions as well, let's see some examples:

- your company decides to write their own testing framework instead of using one of those hundreds/thousands [that already exist](https://techbeacon.com/app-dev-testing/top-11-open-source-testing-automation-frameworks-how-choose)
- your company decides to create their own CRM system instead of using some other product already on the market
- you hire programmers to create a new eshop for you instead of [googling for 10 seconds](https://www.ecommerceceo.com/open-source-ecommerce/) and reusing an existing solution

The results in such cases are similar to the results in my little examples - the final outcome is messy, buggy, expensive, not scalable, slow etc.

The programmers I know do not really use much of "their" code either. Most of their work is somewhere in the following areas:

- analysis of what the client/project manager wants
- finding appropriate solutions they can reuse
- focusing on business logic == solving the client's problem, **not trying to solve how to sort a list**
- finding a good (project) structure for those bits and pieces they're gonna reuse
- setting up their environment, server's environment, working with scaling (e.g. in Azure) etc.
- writing documentation

I don't really see them focusing on low-level code. I'm sure such programmers/projects do exist, but the vast majority of today's jobs in this area are not on this side.

Don't take me wrong, knowing internals of something is beneficial in many ways, but it should not be the goal to always build our own solution to a problem. And if someone is entering the world of IT, it might be much less intimidating to them if they get to build something quickly, rather than spending much more time messing around with some own solution that will not really be used later anyway. This was definitely my experience in school, I didn't like that and only after years I returned to IT and found out many problems are already solved and I can just reuse them.
