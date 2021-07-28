---
layout: page
title:  "Generating Test Email Addresses"
date:   2021-07-28 15:15 +0200
categories: testing linux
---

Reading many online resources makes you believe that test automation is only about automating pushing the keys - fill in this form, click the send button, and check something. However, test automation should be thought of in a broader sense when tools are used to make the Tester more powerful. One such example where test automation is useful is for generating test data. One particular example I'll show in this blog post is generating email addresses.

Many times a Tester needs quite a lot of email addresses, for example when testing a sign-up functinality.

First problem arises with the email addresses themselves. How to set up so many of them? Fortunately there're the known ["trick"](https://gmail.googleblog.com/2008/03/2-hidden-ways-to-get-more-from-your.html) to use a Gmail email address and attach `+<any letters or numbers>`.

Suppose you email address is `abc@gmail.com`. Then you can use `abc+01@gmail.com` or `abc+a001@gmail.com`, and if you send an email to one of these, the emails will arrive in your email inbox.

Another problem is that now you want to generate many of such email addresses, and you don't want to type it manually. This would be a waste of time.

How else should you do it then?

If you know any programming, or scripting languge, it should be easy enough to type a few instructions and run the program to generate these email addresses for you. I like to use what's already available to me without the need to install any other software, interpreter, library, and the like. I'll use bash.

```
$ echo abc+{01..99}@gmail.com
abc+01@gmail.com abc+02@gmail.com abc+03@gmail.com abc+04@gmail.com abc+05@gmail.com abc+06@gmail.com abc+07@gmail.com abc+08@gmail.com abc+09@gmail.com abc+10@gmail.com abc+11@gmail.com abc+12@gmail.com abc+13@gmail.com abc+14@gmail.com abc+15@gmail.com abc+16@gmail.com abc+17@gmail.com abc+18@gmail.com abc+19@gmail.com abc+20@gmail.com abc+21@gmail.com abc+22@gmail.com abc+23@gmail.com abc+24@gmail.com abc+25@gmail.com abc+26@gmail.com abc+27@gmail.com abc+28@gmail.com abc+29@gmail.com abc+30@gmail.com abc+31@gmail.com abc+32@gmail.com abc+33@gmail.com abc+34@gmail.com abc+35@gmail.com abc+36@gmail.com abc+37@gmail.com abc+38@gmail.com abc+39@gmail.com abc+40@gmail.com abc+41@gmail.com abc+42@gmail.com abc+43@gmail.com abc+44@gmail.com abc+45@gmail.com abc+46@gmail.com abc+47@gmail.com abc+48@gmail.com abc+49@gmail.com abc+50@gmail.com abc+51@gmail.com abc+52@gmail.com abc+53@gmail.com abc+54@gmail.com abc+55@gmail.com abc+56@gmail.com abc+57@gmail.com abc+58@gmail.com abc+59@gmail.com abc+60@gmail.com abc+61@gmail.com abc+62@gmail.com abc+63@gmail.com abc+64@gmail.com abc+65@gmail.com abc+66@gmail.com abc+67@gmail.com abc+68@gmail.com abc+69@gmail.com abc+70@gmail.com abc+71@gmail.com abc+72@gmail.com abc+73@gmail.com abc+74@gmail.com abc+75@gmail.com abc+76@gmail.com abc+77@gmail.com abc+78@gmail.com abc+79@gmail.com abc+80@gmail.com abc+81@gmail.com abc+82@gmail.com abc+83@gmail.com abc+84@gmail.com abc+85@gmail.com abc+86@gmail.com abc+87@gmail.com abc+88@gmail.com abc+89@gmail.com abc+90@gmail.com abc+91@gmail.com abc+92@gmail.com abc+93@gmail.com abc+94@gmail.com abc+95@gmail.com abc+96@gmail.com abc+97@gmail.com abc+98@gmail.com abc+99@gmail.com
```

Pretty straightforward. The only problem is this is one line, and I likely want to have multiple lines, each email should be on a separate line:

```
$ echo abc+{01..99}@gmail.com | tr ' ' '\n'
abc+01@gmail.com
abc+02@gmail.com
abc+03@gmail.com
abc+04@gmail.com
abc+05@gmail.com
abc+06@gmail.com
abc+07@gmail.com
abc+08@gmail.com
abc+09@gmail.com
abc+10@gmail.com
...
```

Much better.

How about if I want to attach letters, not numbers:

```
$ echo abc+{a..z}@gmail.com | tr ' ' '\n'
abc+a@gmail.com
abc+b@gmail.com
abc+c@gmail.com
abc+d@gmail.com
abc+e@gmail.com
abc+f@gmail.com
abc+g@gmail.com
abc+h@gmail.com
abc+i@gmail.com
abc+j@gmail.com
abc+k@gmail.com
abc+l@gmail.com
abc+m@gmail.com
abc+n@gmail.com
abc+o@gmail.com
abc+p@gmail.com
abc+q@gmail.com
abc+r@gmail.com
abc+s@gmail.com
abc+t@gmail.com
abc+u@gmail.com
abc+v@gmail.com
abc+w@gmail.com
abc+x@gmail.com
abc+y@gmail.com
abc+z@gmail.com
```

Or a combination of letters and numbers:

```
$ echo abc+{a..z}{0..9}@gmail.com | tr ' ' '\n'
abc+a0@gmail.com
abc+a1@gmail.com
abc+a2@gmail.com
abc+a3@gmail.com
abc+a4@gmail.com
abc+a5@gmail.com
abc+a6@gmail.com
abc+a7@gmail.com
abc+a8@gmail.com
abc+a9@gmail.com
abc+b0@gmail.com
abc+b1@gmail.com
abc+b2@gmail.com
abc+b3@gmail.com
abc+b4@gmail.com
abc+b5@gmail.com
abc+b6@gmail.com
abc+b7@gmail.com
abc+b8@gmail.com
abc+b9@gmail.com
...
```

Bash sequence expressions are a great way for (test) data generation when you need to generate a big amount of similar data.

This is a nice example of two things - test automation could be about many things, (test) data generation is one example, and you don't really need to know much if you use the right tools in the first place.
