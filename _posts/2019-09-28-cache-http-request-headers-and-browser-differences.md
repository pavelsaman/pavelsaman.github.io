---
layout: page
title:  "Cache HTTP Request Headers and Difference in Browsers"
date:   2019-09-28 12:07:19 +0200
categories: browser
---

It's not a big surpise that different browsers implement different things and behave differently. I've, however, focused on one particular area - HTTP cache request headers and how some of the main browsers of today respond to three different user actions.

There are a few HTTP headers that control caching. For a full description, see [this documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#Caching) and [this](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#Conditionals) as well. Out of these headers that are related to caching, only the following list of headers fall into either general or request header category:
1. Cache-Control
2. Pragma
3. Warning
4. If-Match
5. If-None-Match
6. If-Modified-Since
7. If-Unmodified-Since

That's still a lot of headers, leaving us with many options since they can be combined when a browser sends a request for a resource.

However, I've taken the following browsers and tried to observe what they do (what request headers they send) after they have already visited (loaded) a particular resource (a web page).

The browsers are:
1. Firefox 69.0.1
2. Chromium 76.0.3809.132
3. Opera 63.0.3368.66
4. Brave 0.67.119 (Chromium: 76.0.3809.72)
5. Pale Moon 28.7.0
6. Otter Browser 1.0.01

As I said, I've first visited a site (main site of this blog actually) and all testing was done after this point. It included the following 3 actions:
1. Enter the same URL into address bar and press Enter
2. Use `F5` on the site
3. Use reload button on the site

Three simple action, yet the result surprised me.

### Entering URL into Address Bar

Let's start with the first action on the list and see what the browsers did:

**Browser**|**If-None-Match**|**If-Modified-Since**|**Cache-Control**|**Status code**
:-----:|:-----:|:-----:|:-----:|:-----:
Mozilla|N|N|N|200 (cached)
Chromium|Y|Y|max-age=0|304
Opera|Y|Y|max-age=0|304
Brave|Y|Y|max-age=0|304
Pale Moon|N|N|N|200 (cached)
Otter Browser|N|N|N|200 (data transferred)

Values `Y` and `N` were actually concrete values, but I was more interested whether these headers got sent or not, so I don't show these concrete values here, only `Y` if such a header was sent or `N` if it wasn't.

As it's obvious even here, there are differences among these browsers. Those that sent headers `If-None-Match` and `If-Modified-Since` ended up with code `304`, so the final page was retrieved from local cache after the resource version had been validated with the server. In these cases, no actual resource data were sent over the network. Firefox returned status code 200 but didn't actually ask the server, the answer was retrieved from local cache. This is the fastest way to get a resource as there's no traffic over the network.

### F5

How about when pressing `F5` on the site?

**Browser**|**If-None-Match**|**If-Modified-Since**|**Cache-Control**|**Status code**
:-----:|:-----:|:-----:|:-----:|:-----:
Mozilla|Y|Y|max-age=0|304
Chromium|Y|Y|max-age=0|304
Opera|Y|Y|max-age=0|304
Brave|Y|Y|max-age=0|304
Pale Moon|Y|Y|max-age=0|304
Otter Browser|N|N|max-age=0|200 (data transferred)

The browsers tend to behave in the same way, except Otter Browser.

### Reload Button

Finally, the last action I took in all these browsers was pressing a reload button in menu.

**Browser**|**If-None-Match**|**If-Modified-Since**|**Cache-Control**|**Status code**
:-----:|:-----:|:-----:|:-----:|:-----:
Mozilla|Y|Y|max-age=0|304
Chromium|Y|Y|max-age=0|304
Opera|Y|Y|max-age=0|304
Brave|Y|Y|max-age=0|304
Pale Moon|Y|Y|max-age=0|304
Otter Browser|N|N|max-age=0|200 (data transferred)

These results are actually the same as in the previous situation.

### Conclusion

Different browsers behave differently in terms of caching and sent cache HTTP headers. It needs to be taken into account when e.g. testing since it matters a lot where I get a resource from.

Chromium, Opera, and Brave browsers seem to behave the same when it comes down to caching and cache HTTP request headers. Otter Browser doesn't seem to use resource version at all and seem to use less caching than the other browsers. Firefox and Pale Moon seem to utilize 200 cache response in one particular situation, so the display a resource with no network traffic at all.

It'd be interesting to see other browsers as well, namely IE and Edge, but my current OS platform doesn't allow me to do so. I might go into this topic later as well, so come and see if I have an update.
