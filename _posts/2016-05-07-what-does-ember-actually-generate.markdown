---
layout: post
title: "What code does ember actually silently auto-generate at runtime?"
---
You may have noticed that ember does a bunch of things magically for you. This is, in general, great; convention over configuration and all that. It can get confusing when you need to start adjusting the code though; where do I put it? Do I need to include any boilerplate? Are [xml sit-ups](http://www.recursion.org/2006/1/20/xml-situps) involved?

(NB this is orthogonal to ember-cli's blueprint generation)

## Routing

#### Route Handlers

You don't need to actually define route handlers for 75% of your routes.

#### Subroutes

[Index routes](https://guides.emberjs.com/v2.5.0/routing/defining-your-routes/#toc_index-routes) are well documented in the guides; the short version is that

```javascript
Router.map(function() {
  this.route('posts', function() {
    this.route('favorites');
  });
});
```

is equivalent to

```javascript
Router.map(function(){
  this.route('index', { path: '/' });
  this.route('posts', { path: '/posts' }, function() {
    this.route('index', { path: '/' });
    this.route('favorites', { path: '/favorites' });
  });
});
```

All routes also automatically have `loading` and `error` subroutes, as per <https://guides.emberjs.com/v2.5.0/routing/loading-and-error-substates/>

#### Data
 Data is loaded even when no `model()` has been specified on a route handler; the default implementation of model for a route like
```
this.route('photo', { path: '/photos/:photo_id' });
```

does something like
```
return this.store.findRecord('photo', params.photo_id);
```

The actual low-level implementation is [more magic](https://github.com/emberjs/ember.js/blob/master/packages/ember-routing/lib/system/route.js#L1500-L1527), but this is the intended equivalent.

## Controllers

Unless you need to specify properties or actions, you can ignore controllers entirely. They're going to disappear once routable components finally land.

## Templates

Ember will happily generate blank templates for you, and if the route has children it'll act as an `outlet` for nested content.

## Components

All components are rendered as a `div` around their template by default; this can be adjusted using the `tagName` property.

The default component template is `{{yield}}` so that it can be used block-style as well as inline.

## Etc

Let me know what other in-memory generated magic you find!

## Acknowledgements

Thanks are due to [Alisdair](https://twitter.com/alisdair) for proof-reading and reminding me about components.
