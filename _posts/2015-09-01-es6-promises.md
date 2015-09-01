---
layout: post
title:  "ES2015 Promises"
date:   2015-09-01
categories: programming
---

## Promises?

Normally I would start a blog post with a bit of a back-story about the subject, but promises are a topic which
should, but often do, need some explanation. This post will be different I will start from a simple implementation
of a promise and we will end with how ES2015 will implement them.

{% highlight javascript linenos=table %}

// The promise class implementation
function Promise() {
  var self = this;

  this.resolve = function() {
    if (self._resolver) {
      self._resolver.apply(self, arguments);
    }
  }

  this.promise = {
    then: function(_resolver) {
      self._resolver = _resolver;
      return this;
    }
  };
}
Promise.defer = function() {
  return new Promise();
}

{% endhighlight %}

Now we can use this piece of code like so:

{% highlight javascript linenos=table %}

// how to use the new Promise class...
function doSomething() {
  var deferred = Promise.defer();
  setTimeout(function() {
    deferred.resolve('I\'ve been resolved!!');
  }, 2000);
  return deferred.promise;
}

doSomething().then(function(message) {
  console.log(message);
});

{% endhighlight %}

Let's step though this code. Because this blog is for medium to advanced programmers I'm not going to explain
the basics of JavaScript like lambda's and closures. If you do not have a solid understanding of these concepts 
I suggest you read other tutorials on these topics, for example: 
[Closures](http://material.diophantic.nl/#/content/javascript/05_closures.md) & 
[Functions](http://material.diophantic.nl/#/content/javascript/04_functions.md)

In the first code block on `line 2` I create a class, the `Promise` class; which is a function; which has a static 
method: `defer()`; which returns an instance of the `Promise` class. While I understand that this is somewhat of
a strange way to describe the functionality of this class I think it is formally correct. Another way to look 
at this class is in the way of instances and functions. A static function is not an instance member. This is 
something we can use. Because I can pass in parameters to this static function I can use closures to maintain 
instance specific 'data'.

The implementation is somewhat the same as the [Q](https://github.com/kriskowal/q) library. The difference is that 
I did not implement any error handling or chaining of promises. This is to keep the example code as light as possible.

Now let's rewrite this using ES2015's class keyword and other goodies:

{% highlight javascript linenos=table %}

class Promise {
  
  constructor() {
    this.promise = {
      then: (_resolver) => {
        this._resolver = _resolver;
      }
    }
  }
  
  resolve(...args) {
    if (this._resolver) {
      this._resolver.apply(undefined, args); 
    }
  }
  
  static defer() { 
    return new Promise(); 
  }
}

{% endhighlight %}

And the usage would look something like:

{% highlight javascript linenos=table %}

// how to use the new Promise class...
function doSomething() {
  var deferred = Promise.defer();
  setTimeout(function() {
    deferred.resolve('I\'ve been resolved!!');
  }, 2000);
  return deferred.promise;
}

doSomething().then(function(message) {
  console.log(message);
});

{% endhighlight %}

Now let's look at the Promise library in ES2015:

{% highlight javascript lineos=table %}
function doSomething() {
  return new Promise((_resolve, _reject) => {
    setTimeout(() => _resolve('I\'ve been resolved!!'), 0);
  });
}

doSomething().then((_m) => console.log(_m));
{% endhighlight %}

It feels a bit wrong to pass in the `resolve` and `reject` handlers to the `Promise` constructor. But considering
how JavaScript's closures work it is actually a very nice way to work around having to create objects inside of
your implementation. While looking at the spec and working with the new promises I've come to appreciate the 
choices made by the ECMA committee.  