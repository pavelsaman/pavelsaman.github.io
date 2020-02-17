---
layout: page
title:  "Copying Objects in Javascript"
date:   2020-02-17 20:07:19 +0200
categories: testing
---

Many modern testing frameworks and tools use Javascript, so it's worth knowing some key concepts and pitfalls. I'll share just one particular thing regarding objects and how they could be copied.

Let's take the following object as an example:

```javascript
let obj = { name: "ABC", locations: [ { name: "a" }, { name: "b" } ] }
```

You can imagine there's a name of a product and then its locations, or a name of a shop and its locations. Whathever the case, it will do for today's example.

Now, let's assume you want to have such an object as a template and once in a while change a name of this object, e.g. before sending its new version in a request to a server. So you might want to do something like this:

```javascript
// experiement 1
let objNew = obj
objNew.name = "DEF"
``` 

And then after a while, you hope object `obj` is still in its initial version... Well, wrong! When I print out both of these objects, this is a result:

```javascript
console.log(objNew) // { name: 'DEF', locations: [ { name: 'a' }, { name: 'b' } ] }
console.log(obj) // { name: 'DEF', locations: [ { name: 'a' }, { name: 'b' } ] }
```

What happened is that no new objects has actually been created, only a reference to `obj` object has been assigned to variable `objNew`. So when changing `objNew`, you're actually changing `obj` object.

Fine, so you might come up with the following code:

```javascript
// experiment 2
objNew2 = Object.create(obj)

objNew2.name = "DEF"
objNew2.locations[0].name = "c"
```

And the result will be:

```javascript
console.log(objNew2) // { name: 'DEF' }
console.log(obj) // { name: 'ABC', locations: [ { name: 'c' }, { name: 'b' } ] }
```

Well, it's an improvement, at least we didn't change the first level in the object. But `obj.locations[0].name` was still changed!

Then you find the following syntax on StackOverflow:

```javascript
// experiment 3
objNew3 = { ...obj}

objNew3.name = "DEF"
objNew3.locations[0].name = "c"
```

And the result:

```javascript
console.log(objNew3) // { name: 'DEF', locations: [ { name: 'c' }, { name: 'b' } ] }
console.log(obj) // { name: 'ABC', locations: [ { name: 'c' }, { name: 'b' } ] }
```

Not quite! This is only a **shallow copy**, only the first level has been copied, the rest references the initial object `obj`, so if `objNew3.locations[0].name` changes, `obj.locations[0].name` is in fact changed.

At this point, you probably want to give up, but then you finally find the following syntax:

```javascript
// experiment 4 - deep copy
objNew4 = JSON.parse(JSON.stringify(obj))

objNew4.name = "DEF"
objNew4.locations[0].name = "c"
```

And when I print out the objects:

```javascript
console.log(objNew4) // { name: 'DEF', locations: [ { name: 'c' }, { name: 'b' } ] }
console.log(obj) // { name: 'ABC', locations: [ { name: 'a' }, { name: 'b' } ] }
```

Finally! This is what's called a **deep copy**, meaning, a completely new object is created in memory, no reference to the initial object, so if you change `objNew4`, nothing else will be changed.

Such a simple thing to grasp in fact, but it can bring you a huge headache and quite a lot of debugging if you're not aware of this concept of shallow and deep copies.