---
layout: page
title:  "Testing Input Fields"
date:   2019-09-28 10:07:19 +0200
categories: testing
---

Like it or not, a lot of time as a tester will be spent checking inputs and whether the system fails or not. One way we can do this is basically using the final product, that is via a GUI interface as a user. We can call a user testing or differently, it's not that important to remember the eve-growing list of names in software testing, it's important to use your own brain and think about the testing you do.

### Input Field Under Test

Not surprisingly, there are many different fields a web page or any other app (mobile, desktop) can have. There're input fields for text, for numbers, checkboxs, radio buttons etc. For each of them, you need to consider a bit different set of checks/tests.

For this article, I'll choose as an example a simple search field as shown here:

![image](/images/search_field.png)

It's a search field from a web application.

### What to Test For?

Long time ago, I was asked this very question (a bit broader in fact, but this was part of it) during an interview. Well, as a newcomer to SW testing, I stuttered out 1 or 2 thinks in the list I utilize nowadays. I don't feel particularly good about it now, but everyone has to start somehow.

Below you can see my simple table I ca use as a checklist when testing input fields. There're more than just 2 inputs now.

**Test**|**Check if the result is expected/valid**|**Problem**
:-----:|:-----:|:-----:
Allowed value| | 
Empty value| | 
Far below LB| | 
LB-1| | 
LB (low boundary)| | 
LB+1| | 
UB-1| | 
UB (upper boundary)| | 
UB+1| | 
Far above UB| | 
Different data type (e.g. float into int field, different datetime format)| | 
String value| | 
National characters (e.g. Czech, Russian, Chinese)| | 
White character(s)| | 
Leading spaces + valid value| | 
Trailing spaces + valid value| | 
Leading zeros| | 
Special characters (e.g. &, @, /)| | 
Type in a value| | 
Copy in a value (Ctrl + c and Ctrl + v)| | 
Copy in a value using a mouse| | 
max value (e.g. max int, max # of chars in a string field)| | 
Negative value (for number fields)| | 
0| | 
Null| | 
NaN| | 
nill| | 
Upper case| | 
Lower case| | 
Combination of upper and lower case| | 
File name absolute path| | 
File name relative path| | 
File name forward slashes /| | 
File name backward slashes \\ | | 
File name combination of slashes / and \\ | | 
File name more forward slashes //| | 
File name more backward slashes \\\\ | | 
ASCII 255| | 
Escape characters (‘, “, \, \t, \n, \0, \\)| | 
Is the value in the field remembered when going away from the page and back?| | 
Different browsers| | 

As you've figured out, not all these tests are applicable for every input field. When testing a search field, I don't need to check for different ways to enter a file path. Again, take this as a general guidance and use your own judgement.

There might be more to it, no I might have missed something. One interesting input field is the one for telephone numbers... when you consider all these formats one might input their phone number (722222222, 722 222 222, 0042722222222, 0042 722 222 222, +420722222222, +420 722 222 222, foreign phone numbers, how about leading and trailing white characters (someone might press space by mistake) etc.), on the other hand, I consider all these nuances under the "different data type" row. That's where your creativity and judgement come in.

Hope it will make you consider more tests in the future. If you want to download a spreadsheet with the table, [get it here](/files/input_fields_checkbox.ods).
