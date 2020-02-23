---
layout: page
title:  "304 Not Modified - Retrieving Resources From Local Cache"
date:   2020-02-23 17:07:19 +0200
categories: other
---

Knowing how HTTP protocol works is essential for anyone who is involved in developing web applications. Let's see just a small example how fetching resources works, Ï'll show that on an example of this website, plus I'll shortly show what could be done in Firefox DevTools, which is another important thing that can bring tons of useful information to any tester.

(Note at the beginning: I'm using Firefox 73.0.1-0.1, if you open this article in 2021, half of what you read might be irrelevant.)

When you press `F12`, you'll see Developer Tools, or DevTools as people usually refer to it. Since we're talking about HTTP protocol, you can switch to Network tab and ignore the rest. Once you're on the tab, go to some website, I'll go to this one as an example. After a while (if everything goes smoothly), I can see 5 requests in the window. I've highlighted some parts of the window that I consider important:

![image](/images/dev_tools_firefox.png)


1. all the requests that have happened during loading my github website
2. option **Persist logs** is quite useful if you don't want to lose request from the request section with every reload of the site, I have this permanently checked, option **Disable Cache** could be useful when you always want to fetch a resource from the remote server without relying on cache
3. if you want to clear your window, you press this bin icon

You can also adjust columns in the request section, e.g. I've added columns Protocol, Cookies, ETag, and Last-Modified:

![image](/images/dev_tools_firefox_2.png)

The last two mentioned columns are response columns, I've added them because of what I want to show in a bit in this article.

Let's interpret what actually happened. From the request section I can see that my browser asked for 5 resources and got all of them from the remote server (response code is 200 and I can actually see concrete values in Transferred column). The result of that is what a user normally sees in the browser window. Let's take the first request-response pair, click on it and see some details:

![image](/images/dev_tools_firefox_3.png)

The remote server sent me two header fields I'm interested in with regard to today's article: `Last-Modified` and `ETag`. The former one means simply a date and time when this particular resource was last modified. ETag is an identifier that a server assigned to this particular version of the resource. So, in this case, I can see that this resource `/` was last modified on 23rd of February at 16:50:27.

A question: if nothing in the browser changes (cache deletion, header fields sent in the next request), nothing on the server changes (the same resource version) and a user presses `F5`, what happens?

![image](/images/dev_tools_firefox_4.png)

Quite a different view, isn't it? First of all, the status code has changes, it's no longer 200, it's 304. Second of all, the transferred column is showing me not a number of kB transferred, but `cached`. Let's see some details:

![image](/images/dev_tools_firefox_5.png)

When I pressed `F5`, my browser generated a bunch of header fields that were sent in the request to the server. Among these header fields, `If-Modified-Since` and `If-None-Match` were also sent. The former one had a value that was taken from the first response to the same request, it's the value of `Last-Modified` header field from the third screen above. What my browser is saying with sending this header field is something along these lines: _"if the resource I'm asking for has changed after this date and time I'm giving you, send me a new version of the resource"_. `Last-Modified` is one of the [conditional header fields described in HTTP](https://tools.ietf.org/html/rfc7232). Similar logic follows the header field `If-None-Match`, but not with a time, but with an etag token, so again, my browser is saying something along these lines: _"if you don't have a version of the resource under this token(s) (it could be a list), give me a new version"_.

Having said that, when these two header fields were sent and nothing else has changed, the server considered unnecessary to send the same version of the resource again, so only 304 status code along with some other header fields was returned and no payload was transferred, therefore Firefox understood that and just had a look into its local cache from where the resource was eventually retrieved to the user. A nice way to safe some bandwidth.

Let's experiment further. Firefox DevTools allow for changing the request and sending it again, let's delete `If-None-Match` header field and update the value of `If-Modified-Since` header field to some value before where the resource was last changed:

![image](/images/dev_tools_firefox_6.png)

What happens now?

I'm asking the server to give me a new version of the resource if there's one newer than `Sun, 23 Feb 2020 13:50:27 GMT`. Since nothing else has changed from the previous situation, we know that the resource was last modified on the 23rd of Feb at about 16:50. So, the condition must evaluate to true and the server should send the resource. Let's hit the send button and see the result:

![image](/images/dev_tools_firefox_7.png)

And there it is, no retrieval from the local cache, the resource was transferred from the server.

Similarly, I can mess around with the `If-None-Match` header field, changing its value to some non-existent one and sending the request:

![image](/images/dev_tools_firefox_8.png)

And the response is not cached either:

![image](/images/dev_tools_firefox_9.png)

Remember when I highlighted the option **Disable cache** in the first screen? It turns out that when checked, these two header fields will not be send in subsequent request, so the server has no data to check whether the client already has the resource (and what version of it), so payload is always transferred:

![image](/images/dev_tools_firefox_10.png)

and details of the request headers:

![image](/images/dev_tools_firefox_11.png)

Alright, that'll do for now. I hope this short peek into HTTP and DevTools was useful. Similar understandings can really help you at your job if you're a tester involved in developing web applications. You should definitely know your way around DevTools and what information you can get from there. And knowing a bit more about HTTP, its status codes, header fields and basic workings/logic is just as useful as DevTools.