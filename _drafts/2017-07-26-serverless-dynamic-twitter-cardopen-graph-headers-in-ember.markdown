---
layout: post
title: "Serverless Dynamic Twitter Card/Open Graph Headers in Ember"
---

(For a brief summary of the serverless paradigm, see <https://martinfowler.com/bliki/Serverless.html>.)

Using static Firebase hosting for our ember app greatly simplifies many things, but means we can't provide any dynamic headers for services like twitter or facebook. Until now! Using Firebase's [Cloud Functions](https://firebase.google.com/docs/functions/) lets us write javascript that's run server-side and triggered by a url that we specify.

So, we can write a function that loads an html page using e.g. <https://github.com/request/request>, edits it and returns the changed page to the browser:

```javascript
exports.getIndex = functions.https.onRequest((request, response) => {
  var rqst=require('request');

  rqst.get('https://flowerpot-function-test.firebaseapp.com/',{},function(err,res,body){
    if(err){response.send("something went wrong" + err)} //TODO: handle err
    if(res.statusCode === 200 ){
      response.send(res.body.replace(/{{placeholder}}/g, 'Hello from the server!'));
    }
  });
});
```

There's a limitation on firebase's free tier that stops http requests from going to non-google services; sadly as of late July 2017 this makes it impossible to load files from firebase hosting without being on a paid plan, which may be overkill for your project. There are several potential solutions to this problem; the one I've worked on first is to store the index.html file in our Firebase database and fetch it in the function.

Use cheerio to edit headers
