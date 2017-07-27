---
layout: post
title: "Serverless Dynamic Twitter Card/Open Graph Headers in Ember"
---

(For a brief summary of the serverless paradigm, see <https://martinfowler.com/bliki/Serverless.html>.)

This post will use an example blogging app using Firebase and Ember, but the concepts apply equally to any static hosting, FaaS and front-end framework.

Using static Firebase hosting for our ember app greatly simplifies many things, but means we can't provide any dynamic headers for services like twitter or facebook. Until now! Using Firebase's [Cloud Functions](https://firebase.google.com/docs/functions/) lets us write javascript that's run server-side and triggered by a url that we specify.

So, we can write a function that loads an html page using e.g. <https://github.com/request/request>, edits it and returns the changed page to the browser:

```javascript
exports.getIndex = functions.https.onRequest((request, response) => {
  var rqst=require('request');

  rqst.get('https://flowerpot-function-test.firebaseapp.com/',{},function(err,res,body){
    if(err){
      response.send("something went wrong" + err);
    }
    if(res.statusCode === 200 ){
      response.send(res.body.replace(/{{placeholder}}/g, 'Hello from the server!'));
    }
  });
});
```

There's a limitation on firebase's free tier that stops http requests from going to non-google services; sadly as of late July 2017 this makes it impossible to load files from firebase hosting without being on a paid plan, which may be overkill for your project. There are several potential solutions to this problem; the one I've worked on first is to store the index.html file in our Firebase database and fetch it in the function.

So we can adapt our previous example to look something like this:

```javascript
exports.getIndex = functions.https.onRequest((request, response) => {
  const admin = require('firebase-admin');
  admin.initializeApp(functions.config().firebase);

  admin.database().ref('/index_html').once("value", function(snapshot) {
    response.send(snapshot.val().replace(/{{placeholder}}/g, "Hello from the server!"));
  });
})
```

I've built [an ember deploy plugin](https://github.com/ibroadfo/ember-cli-deploy-firebase-database) to automate storing the html in your database, but you can put the content there however you like.

Because we want to edit existing headers rather than just replace a placeholder, it would be useful to use something like <https://cheerio.js.org>, which is a jquery substitute for use in node environments. The function then becomes:

```javascript
exports.rewriteSocialHeaders = functions.https.onRequest((request, response) => {
  const admin = require('firebase-admin');
  admin.initializeApp(functions.config().firebase);

  admin.database().ref('/index_html').once("value", function(snapshot) {
    var $ = require('cheerio').load(snapshot.val());

    $('meta[property="og:title"]').attr('content', `A blog post`);
    $('meta[property="og:description"]').attr('content', `A blog post on PLATFORM`);

    response.send($.html());
  });
})
```

Finally, we want to load the individual post and capture some details about it to place them into the headers. Our `firebase.json` file should contain something like the following so that requests to /posts/$post_id get sent to the function.

```json
"rewrites": [
  {
    "source": "/posts/**",
    "function": "rewriteSocialHeaders"
  },
  {
    "source": "**",
    "destination": "/index.html"
  }
],
```

And here is the complete example function:

```javascript
/* eslint-env node */

const functions = require('firebase-functions');
const admin = require('firebase-admin');
const cheerio = require('cheerio')

var config = functions.config().firebase;
config.databaseURL = 'https://flowerpot-staging.firebaseio.com'
admin.initializeApp(config);

exports.rewriteSocialHeaders = functions.https.onRequest((request, response) => {
  var indexPromise = admin.database().ref('/index_html').once("value");
  var postId = request.path.split('/').pop();
  var postPromise = admin.database().ref('/posts/' + postId).once("value");

  var fullUrl = require('url').format({
    protocol: request.protocol,
    host: request.headers["x-forwarded-host"] || request.get('host'),
    pathname: request.originalUrl
  });

  return Promise.all([indexPromise, postPromise]).then((results) => {
    var index = results[0].val();
    var post = results[1].val();

    const $ = cheerio.load(index);

    $('meta[property="og:url"]').attr('content', fullUrl);
    $('meta[property="og:title"]').attr('content', `Flowerpot: ${post.title} [${post.note}]`);
    $('meta[property="og:description"]').attr('content', `A post on Flowerpot about ${post.title}, with warning note '${post.note}'`);

    return response.send($.html());
  });
});
```
