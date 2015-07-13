---
layout: post
title:  "Should we leave OOP"
date:   2015-07-12 08:42:46
categories: programming
---
## Why can't I sleep?

I've been doing a lot of 'soul searching' these last weeks. I've been facing a difficult problem which I cannot
reconcile within myself. The symptoms started when I really dove into JavaScript, creating a breeding ground of
ideas and code styles. 

For example; the following code describes a decorator in JavaScript:
    
{% highlight javascript %}
function decorateConsoleLog () {
    var old = console.log;
    
    console.log = function () {
        var args = Array.prototype.slice.call(arguments);
        args.unshift(new Date().toString());
        old.apply(this, args);
    };
    
    // return a function which can clear the decorator
    return function() {
        console.log = old;
    };
};

// apply the decorator to the console.log function
var clearDecoration = decorateConsoleLog();

// log something to the console
console.log('FP, really?');

// with the result of the decorator function we can now clear the 
// decorator and resume our normal function calls...
clearDecoration();

// log something to the console.
console.log('Cleared');
{% endhighlight %}

This code is wonderful in it's simplicity. It does what it is supposed to do and that is create a decorator for
`console.log`. And it also shows a valid pattern in JavaScript which are closures. We tend to think poorly of
JavaScript, but it is a wonderful language which we should all learn. It opens up your mind to what you really 
need to program instead of building a scaffold which an OO application is. 