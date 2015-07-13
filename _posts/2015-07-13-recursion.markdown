---
layout: post
title:  "Recursion"
date:   2015-07-12 08:42:46
categories: programming
---
## Getting rid of variables?

In this post I would like to begin a multipart series on how functional programming can help us write better and more 
consistent code. One of the first items I would like to tackle is that of variables, also called state. Within a more 
procedural piece of code we often encounter variables which contain state. This state can be in the form of a temporary 
value or object. This blog post is about explaining how the number of these variables might be reduced. I give one 
(the first in many) examples on how to do this. 

The following code might seem trivial but you can extrapolate a multitude of scenarios each and every one of us have encountered.

*Explaining the domain*: Now these code examples are all about tests students make. I think about education a lot and 
one of the things I've discovered over the years is that everybody has an opinion about education; furthermore; 
everyone has at least a basic understanding about this "domain". This is why I have chosen these examples for these 
blog posts.

A *Test* consists of multiple *Questions*. A student can score *Points* on a *Question*. The following code example will sum 
the points a *Student* has scored on a test: 


{% highlight javascript linenos=table %}
int total = 0; 
for (int i = 0; i < test.Questions.length; ++i) { 
    total += test.Questions[i].Points; 
} 
{% endhighlight %}

This is an extremely common scenario. Loop through a set and do something with a value on an object. A more functional 
approach would be to use something called recursion; here we call the same function from within the function:

{% highlight fsharp linenos=table %}
let rec sumPoints questions = 
    match questions with 
    | h::t -> h.Points + sumPoints t 
    | [] -> 0 

let total = sumPoints test.Questions 
{% endhighlight %}

There are a few things that will need some clarification at this point. These blog posts are about F#. And as such I 
will write examples in both F# and C#, the two languages I am most familiar with. In this case the example in C# is 
much shorter then the one written in F#, but, what is the main difference between them?

The C# example perfectly states how to sum the points of the questions in the test, but, it does so in a very explicite 
way.  If I now want to Parallelise this; or do something else which will make people very edgy, I might run into problems. 
The recursive definition does not really care how it runs. You just give it a `List` of questions and it will "do the right 
thing". Now I know that this case is fairly trivial. But look at it again. 

In the first C# example you have to really read what is happening. It might even be possible to increment the i inside 
the body of the for loop, potentially breaking the execution and killing your application. With the recursive function 
I match, I do not explicitly state. This is a lot safer. If I have a head on my array; well, use it. If I have an empty 
array return 0. There are no other options for an array.  And with there being no more options comes a feeling of safety.

## Tail recursion

There is this big'ol messed up confusion about recursion in general. The first is that it's fast. 
[It isn't](http://stackoverflow.com/questions/2651112/is-recursion-ever-faster-than-looping). We can optimise compilers 
and we can do all kinds of muck, but the fact of the matter is, shifting bits on a stack will trump recursively calling a 
function any day of the week. But does this matter? I don't think it does. I think maintainability and extreme Type Safety 
result in an increase of code quality which will eventually result in more income. 

The second part about recursion is that compilers optimise for it. Well, they do and they don't at the same time. Imagine 
having one question, the result would look something like:

{% highlight fsharp linenos=table %}
total = ( 3 + ( 0 ) ) 
{% endhighlight %}

Now imagine having four questions:

{% highlight fsharp linenos=table %}
total = ( 3 + ( 4 + ( 2 + ( 1 + ( 0 ) ) ) ) )
{% endhighlight %}

All of the brackets are results of function calls. With the last function call being evaluated first. If I have a few 
million questions this will result in an infamous StackOverflow exception. How can I fix this? I could fix this be doing 
something called Tail Recursion. Look at the following code for an example:

{% highlight fsharp linenos=table %}
let sumPoints questions = 
    let rec sp questions accumulator = 
        match questions with 
        | h::t -> sp t accumulator + h.Points 
        | [] -> accumulator 
    sp questions 0 

let total = sumPoints test.Questions 
{% endhighlight %}

Here we've complicated things even more by creating an inner function called sp which takes as parameters the questions 
and an accumulator. We call this function immediately thus returning the actual sum of the points of all of the questions. 
This way is completely Type Safe and it would be hard to generate a StackOverflow exception with this implementation.

Back to the title of this post. Getting rid of variables. Just a simple count of the variables in the C# code gives me 2. 
The total and the i. The variable count of the F# code gives me 1, the total. This might not seem like much but imagine 
a big application, with thousands of lines of code. Reducing the number of variables reduces the number of accidental 
mistakes you can make, producing better quality code. If you do not believe variables cause bugs, read up on the 
[NullReferenceException](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)! 

## For the unbelievers
While there are many ways to write software, C# compared to other languages has many...many ways of writing the same 
piece of functionality. Some of you will say: "I never use the for loop anymore. I always use foreach!". To those of 
you I say: "Count the variables, a foreach also has two!".

Then there are those of you that say: "I only use LINQ for this kind of thing!". Here we get into murky waters. LINQ 
is a functional approach to working with collections. If you are using LINQ you are already eating from the functional 
apple, why not take it one step further and dip that apple into the yummy saus that is F#?

## For the JavaScripters
Writing in a recursive style might be fun for F# developers. C# developers always see it as an unnecessary complexity,
but how does recursion look in JavaScript? And what can we do in JavaScript that we cannot, or at least, can't easilly
do in C#?

{% highlight javascript linenos=table %}
function sumPoints(questions) {
    // internal function to get some 'closure'
    function internal(_questions, _accumulator) {
        var head = _questions.shift();
    
        if (!head) return _accumulator;
        else return internal(_questions, _accumulator + head.Points);
    }
    
    var tempQuestions = questions.map(function(q) { return q; });
    return internal(tempQuestions, 0);
}
var total = sumPoints(test.Questions); 
{% endhighlight %}

This code looks clean and easy to understand. We simply either return the accumulator or we reiterate our collection.
The fun part is how we treat our collection. The map function creates a shallow copy or clone of our array, making sure
we do now break our data. Because JavaScript does not have a good concept of immutabble types we will have to revert to
this type of *hackery*.

We can remove a few variables from this code by writing it like:

{% highlight javascript linenos=table %}
function sumPoints(questions) {
    // internal function to get some 'closure'
    function internal(_questions, _accumulator) {
        var head;
        return (head = _questions.shift()) ?   
          internal(_questions, _accumulator + head.Points) :
          _accumulator;
    }
    
    return internal(questions.map(function(q) { return q; }), 0);
}
{% endhighlight %}

