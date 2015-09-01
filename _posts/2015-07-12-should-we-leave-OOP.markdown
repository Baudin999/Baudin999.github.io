---
layout: post
title:  "Should we leave OOP?"
date:   2015-07-12 08:42:46
categories: programming
---
## Why can't I sleep?

I've been doing a lot of 'soul searching' these last weeks. I've been facing a difficult problem which I cannot
reconcile within myself. The symptoms started when I really dove into JavaScript, creating a breeding ground of
ideas and code styles. 

For example; the following code describes a decorator in JavaScript:
    
{% highlight javascript linenos=table %}
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
JavaScript, but it is a wonderful language which we should all learn. It opens your mind to what you really 
need to program instead of building a scaffold which an Object Oriented application is. 

I will try and get these comments into perspective, imagine an application where we're doing some type of 
order with orderlines. In C# we would have to scaffold this structure along the lines of:

{% highlight C# linenos=table %}
public class Order {
  public string Number { get; set; }
  public ICollection<OrderLine> OrderLines { get; set; }
  public Order() {
    this.OrderLines = new HashSet<OrderLine>();
  }
}
public class OrderLine {
  public string Number { get; set; }
}
{% endhighlight %}

This would be a typical 'bare' implementation of a structure. And when we want to instantiate these objects we 
will get something like:

{% highlight C# linenos=table %}
var order = new Order();
order.Number = "1234567890";
order.OrderLines.Add(new OrderLine() {
  Number = "0987654321"
});
{% endhighlight linenos=table %}

If we were to adopt a more "javasscripty" style of programming we would have to accept that there "is no 
spoon". We do not write data classes. We just return objects. So instead of writing the class definition
we would just create the object:

{% highlight javascript %}
var order = {
  number: '1234567890',
  orderLines: [
    { number: '0987654321' }
  ]
};
{% endhighlight %}

In F# this would probably look something like:

{% highlight fsharp linenos=table %}
type OrderLine = {
    Number: string
}

type Order = {
    Number: string;
    OrderLines: OrderLine list
}

let order = { 
    Number = "1234567890"; 
    OrderLines = [ { Number = "0987654321" } ] 
}
{% endhighlight %}

In F# the object `order` is already immutable. In C# and JavaScript we would still need to do a bit of plumbing
to get this done. The JavaScript version is the easiest to achieve. `Object.freeze(order)` would be all we needed
to do to make our order object immutable. One thing to note here is that while the object is immutable it is
still possible to add and orderLine to the orderLines property of the order object.

In C# we would have to rewrite our entire class to achieve this immutability:

{% highlight C# linenos=table %}
public class OrderLine {
    private readonly string _Number;
    public string Number {
        get {
            return _Number;
        }
    }
    public OrderLine(string number) {
        _Number = number;
    }
}
public class Order {
    private readonly string _Number;
    private readonly ICollection<OrderLine> _OrderLines;
    
    public string Number {
        get {
            return _Number;
        }
    }
    
    public ReadOnlyCollection<OrderLine> OrderLines {
        get {
            // we should return a copy of the data because we
            // do not want it passed as a reference and have 
            // other pieces of code change it.
            return _OrderLines.ToList().AsReadOnly();
        }
    }
    
    public Order(string number, ICollection<OrderLine> orderLines) {
        _Number = number;
        _OrderLines = orderLines;
    }
}
{% endhighlight %}

This C# code while verbose is better than the JavaScript version because the Collection of orderlines returned
from the property will never mutate the original collection. The part which gives me nightmares is the amount 
of code we need to write to give this class the illusion of immutability.

In JavaScript the code to formally create an immutable object becomes another type of nightmare:

{% highlight javascript linenos=table %}
// create a self executing function
var order = (function createOrder() {

  // create the order lines
  var _orderLines = [
    { number: '0987654321' }
  ];
  
  // create the order
  var _order = {
    number: '1234567890'
  };
  
  // add a property to the _order object
  Object.defineProperty(_order, 'orderLines', {
    get: function() {
      return _orderLines.map(function(_o) { return _o; });
    }
  });
  
  // return the _order
  return Object.freeze(_order);
  
}());


order.orderLines.push({ number: '123123123' });
console.log(order.orderLines.length);
{% endhighlight %}

But seeing how we already know how to write decorators, we might be able to abstract all of this away. Let's
look at an implementation of this solution:

{% highlight javascript linenos=table %}
(function() {

  var _old = Object.freeze;
  
  Object.freeze = function(obj) {
    // filter out all the properties which are arrays
    var arrays = Object.keys(obj).filter(function(key) {
      return Array.isArray(obj[key]);
    }).map(function(key) {
      return { key: key, value: obj[key] };
    });
       
    // for each array property delete the old property and
    // create a new one with just a getter which returns a 
    // copy of the array
    arrays.forEach(function(a) {
      
      // delete the old property
      delete obj[a.key];
      
      // define a new property and let it return a 'clone' of the collection
      Object.defineProperty(obj, a.key, {
        get: function() {
          return a.value.map(function(_o) { return _o; });
        }
      });
    });
    
    return _old(obj);
  };
}());

var order = {
  number: '1234567890',
  orderLines: [
    { number: '0987654321' }
  ]
};

Object.freeze(order);


order.orderLines.push({ number: '123123123' });
console.log(order.orderLines.length);
{% endhighlight %}

This decorator is still readable enough to serve it's purpose and it has changed the default implementation
of the `Object.freeze()` implementation. Now, to be honest, I do not like changing behavior of existing 
functionality. Appart from that, we will still need to do a lot of work to get this implementation to a 
level which is acceptable for production. We cannot, for example, add items to our collection or change the
`order.number` property. We will need to create some sort of `list.add(...)` function and maybe a `change(..)`
function as well.

What we do get, for free, is a simple extension point for things like memoisation and hash lookups. Because 
of the fact that we parse our object before we return it we can add a lot of behaviour to our code; like caching
in a localStorage of memoisation where needed.  

There is no clear and easy way of doing this in C#. We might create an `Object.Freeze(...)` function and have
that return an object, but this object cannot have the same type as the original one and it most certainly 
doesn't help with readability or maintainability. 

For now F# seems to wins a the point on immutability whilst remaining easy to read.

## Write more for less

The second feeling I have when I look at JavaScript is that every keystroke somehow "means more".