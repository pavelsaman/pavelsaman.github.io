---
layout: page
title:  "Rounding Is Not Always Straightforward"
date:   2020-08-24 19:30 +0200
categories: testing
---

Many people have a well-formed expectation when it comes to rounding. We've all learnt this during the early years in school, right? But such expectations can become a source of bugs that are hard to find. Let's have a look.

When we think of rounding, many will automatically come up with the following rules which I show on a few examples:

- 5.5 => 6
- 5.55 => rounding to one decimal place => 5.6
- 5.65 => rounding to one decimal place => 5.7
- 4.946 => rounding to 2 decimal places => 4.95
- 4.945 => rounding to 2 decimal places => 4.95

and so on and so forth. In other words, many assume that 0 - 4 are rounded down and 5 - 9 are rounded up. That's probably how most of us have learnt it in school and we go on thinking this is the only way.

Until there's a production bug.

You start exploring and you find out:

```csharp
using System;
                    
public class Program
{
    public static void Main()
    {
        Console.WriteLine(Math.Round(5.5, 0));   // 6, expected
        Console.WriteLine(Math.Round(5.55, 1));  // 5.6, expected
        Console.WriteLine(Math.Round(5.65, 1));  // 5.6, what?
        Console.WriteLine(Math.Round(4.946, 2)); // 4.95, expected
        Console.WriteLine(Math.Round(4.945, 2)); // 4.94, whaaaat???
    }
}
```

At this point, you probably start writing on stackoverflow thinking you've just found a bug in .NET Core. There have been such people as you can see [here](https://stackoverflow.com/questions/977796/why-does-math-round2-5-return-2-instead-of-3) for example.

If you are lucky and read the stackoverflow discussion before submitting a duplicate and getting downvoted into oblivion, you find out that all your expectations about there being only one type of rounding are wrong. There's no bug in .NET Core or similar, but the point is that `Math.Round()` uses so called [Banker's Rounding](https://en.wikipedia.org/wiki/Rounding) by default. It's even explained in the [ducumentation](https://docs.microsoft.com/en-us/dotnet/api/system.math.round?view=netcore-3.1), but no one usually reads that, which is the reason why there's a production bug in the first place.

Then you go and fix it with an additional argument:

```csharp
using System;
					
public class Program
{
	public static void Main()
	{
		Console.WriteLine(Math.Round(5.5, 0, MidpointRounding.AwayFromZero));   // 6
		Console.WriteLine(Math.Round(5.55, 1, MidpointRounding.AwayFromZero));  // 5.6
		Console.WriteLine(Math.Round(5.65, 1, MidpointRounding.AwayFromZero));  // 5.7
		Console.WriteLine(Math.Round(4.946, 2, MidpointRounding.AwayFromZero)); // 4.95
		Console.WriteLine(Math.Round(4.945, 2, MidpointRounding.AwayFromZero)); // 4.95
	}
}
```

This actually happened in my case, takeways for myself as a Tester are mostly that I shouldn't assume anything even when it comes to standard library methods in a well-known framework like .NET Core. I still think this default behaviour is not very well-chosen, because it seems (from my example, from the example of my colleagues, and from the example of many on the Internet like the stackoverflow discussion) that people simply don't expect this, so it might eventually cause trouble.
