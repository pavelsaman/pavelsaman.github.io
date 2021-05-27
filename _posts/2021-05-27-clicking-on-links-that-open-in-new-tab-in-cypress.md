---
layout: page
title:  "Cypress: Clicing on Links That Open in New Tab"
date:   2021-05-27 09:50 +0200
categories: testing
---

Cypress does not have multi-tab support, which comes as a surprise to many, usually after they have dived headlong into automating some UI checks. However, not having multi-tab support is not really that much of a problem because many times the multi-tab support is not really needed and it is a suboptimal solution anyway. Let's have a look at the most common use case, clicking on links that open in a new tab.

First off, how does such a link look like? Let's see its html:

```html
<a href="https://abc.com/" target="_blank">Click me</a>
```

That `target` attribute is the key, it means that a browser interprets it in a way that the link will be opened in a new tab.

In fact, there're nothing to be tested anyway in regards to new tabs, this is a browser behaviour, I don't need to test what browser developers already tested. I can easily just assert that the link does have the target attribute:

```javascript
it('Link opens in a new tab', () => {
  cy
    .get('a')
    .should('have.attr', 'target', '_blank');
});
```

Another thing a Tester might want to check here is that the link really leads somewhere. That basically means I need to request the resource somehow now, but it doesn't mean I have to use multi-tab support for that. Two solutions comes to mind:

### Changing DOM

Thanks to jQuery methods, I can change the DOM easily and open this link in the same tab:

```javascript
it('Link resource exists', () => {
  cy
    .get('a')
    .invoke('attr', 'target', '')
    .click();

  // some assertions or other testing code
});
```

What `.invoke('attr', 'target', '')` does here is it changes the link to this:

```html
<a href="https://abc.com/" target="">Click me</a>
```

And when such a link is clicked, a browser opens it in the same tab. Then I can go and work with the page in Cypress because it has been opened in the same tab.

However, there's still one limitation. The link's href attribute has to be of the same origin, because Cypress can't (easily) work with cross origin frames.

### `cy.request()`

Another solution to the same problem is to simply request the resource:

```javascript
it('Link resource exists', () => {
  cy
    .get('a')
    .invoke('attr', 'href')
    .then(link => {
      cy
        .request(link)
        .then(res => {
          expect(res.status).to.equal(200);

          // possibly some other checks
        });
    });
});
```

This way I can also work around the same origin policy.

Any of these two solutions is much better than trying to work with multiple tabs just because all I need to do is open a link and check the resource exists.
