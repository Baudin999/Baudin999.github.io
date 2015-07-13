---
layout: post
title:  "Why void is an antipattern"
date:   2015-07-12 08:42:46
categories: programming
---
## Why void is an antipattern

WARNING: You might learn some math!!

Today I want to talk about void. Specifically about why I think functions should never return void. Before 
you think I'm stark raving mad; let me assure you that by the end of this post I will have at least made you 
think about the concept if not outright accepted it. First things first, no good blog post about programming 
should be devoid of some maths. So here we go! 

Consider the following image. This image shows  what a *function* does in a graphical way:

![functions]({{ site.url }}/static/img/function_01.png){: .centered }

A function in mathematical terms might be the projection of an element from one collection (let's call this 
first collection X) to an element of another collection (let's call this collection X'). I specifically 
say 'might be' in the previous sentence because, as with every piece of mathematics, nothing is as clear as 
it might seem at first glance.

In mathematical terms a function must have a result. There is simply no way for something to be a projection 
if there is no start element and no end element. Computer Science (CS) tends to have roots in theoretical 
mathematics; meaning that a lot of the CS concepts come straight from mathematics. This is the reason why 
[monads](https://en.wikipedia.org/wiki/Monad_(category_theory)) are founded in Category Theory, and why 
[p vs np](https://en.wikipedia.org/wiki/P_versus_NP_problem) is a mathematical problem.

Now, we're not going to dive into mathematics this deeply. We are just going to skim the surface and dip our 
toes. Hopefully at the end I can convince you that returning void might not, in most cases, be the best way to go. 

As I've previously stated, a function always returns something. In computer programming this is also true. But 
sometimes we return void. Now void (in C#, and maybe the CLR) can have three meanings:

1. return 'nothing'
2. a pointer to an unknown type
3. alias for System.Void

I am fully aware that I am skirting some corners here, but just as I do not want to dive too deeply into the 
math behind these theories I do not want to dive too deeply into the CS theories and compiler specific 
instructions. As you can see only the first one jumps out. The second and the third meaning of the keyword void 
are perfectly valid in mathematical terms. 

Maybe we can look at the problem from another perspective. Maybe this void element is the only element in a 
collection V (from Void). Now we can have a solid projection from some collection X to a collection V!

The following image illustrates this principal:

![multiple functions]({{ site.url }}/static/img/function_02.png){: .centered }

Here I have four functions which project one element from a collection to the collection V(oid). My main problem 
with this model is that we can never go [back](https://en.wikipedia.org/wiki/Bijection). There is no way to reverse 
our operation, effectively cancelling out our chance of ever implementing a rollback.


## Quit it with the math

Now, let's bring it back to reality. We're sitting behind our desks and we're writing software. We have our 
[SOLID](https://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29)
principles and we're running with some architecture which some architect has envisioned out of boredom (there will 
someday be an extensive article about this some day). We're writing Unit Tests. We're even, either toying with, or 
actually doing TDD! Now how do we write our tests when we have functions returning void?

It might be something like:

{% highlight fsharp %}
// given some question, set some answer...
question.giveAnswer(answer);
Assert.AreEqual(question.answer, answer);
{% endhighlight %}

This of course is a very terse test. We set an answer on a question object and expect it to equal the answer we 
supplied. For now we will ignore the fact that most equality comparers compare objects by reference and we'll 
just assume that the AreEqual function of the Assert class just "does the right thing".

Now imagine the giveAnswer function doing something like this:

    public void giveAnswer(Answer answer) {
       // let this be the Question object
       this.answer = answer;
       answer.text = answer.text + "oh la la!!";
    }

Our test, while being reasonable, will have broken. There are many reasons why something like this implementation 
is wrong. You might argue that we're violating Single Responsibility when altering the answer object. The other 
thing we can argue is that we've introduced a side effect, and we have mutated an object which did not deserve 
mutating. 

Our first point was about the Single Responsibility Principle; we can nullify this argument by looking at the 
definition of the SRP. The definition of the Single Responsibility Principle is (as defined by Wikipedia and 
Robert C. Martin):

> In object-oriented programming, the single responsibility principle states that every class should have 
  responsibility over a single part of the functionality provided by the software, and that responsibility 
  should be entirely encapsulated by the class. All its services should be narrowly aligned with that responsibility.

So basically, when we look at methods we can't really say anything about single responsibility. This principle 
is defined on objects. 

When thinking in terms of [functional programming](https://en.wikipedia.org/wiki/Functional_programming) (FP) I 
like to think of functions as 
["static functions on static classes"](http://stackoverflow.com/questions/155609/difference-between-a-method-and-a-function) 
when I have to explain things to my CSharpy-friends. This means that functions in general have no context; 
just parameters and return values. When thinking in terms of FP I also like to think about my objects (mostly data) 
as being [immutable](https://en.wikipedia.org/wiki/Immutable_object).  And last but not least, in FP you do not 
want to get caught introducing [side effect](https://en.wikipedia.org/wiki/Side_effect_%28computer_science%29), people will 
literally tie you to a chair and paint a mustachio on you! 

Now I do not think that 100% FP is the right way to go. You still need to get the job done. Sometimes, throwing an 
exception is really the right thing to do. Sometimes mutating state is the only possible way to get the performance 
you want. However, I also think that in most cases holding yourself to the simple truth that side effects result in 
bugs and mutating state is dangerous will save you a lot of time in big projects!

Now back to my original subject. Why is returning nothing a bad thing? 

## Turn it around

Imagine not returning void! What would you do if your functions could never ever return void? Would you create a 
fluent APIs? Would you return state? Would you return the next step in your process? Would you return the id of 
the event you created so that your system can poll some queue? There are so many things we could do if we just 
didn't return void. However, this still does not answer the question why we shouldn't! 

It all boils down to what you want to do with your application. The idea of programming is that you chain a lot 
of little blocks together to create bigger blocks. The idea behind not returning anything is that you create a 
dangling chain:

![dangeling chain](/static/img/dangeling_chain.jpg){: .centered }

Each dangle is a method not returning something. You will have to tie all of these dangling pieces of code 
together some how. Usually you'll have to refer to some sort of Orchestrator to manage this, all the while having 
no real security as each method (and I'm using method now!!) can introduce side effects. 

The idea of returning something and continuing with this, either through continuations or by doing some sort of 
piping or fluent api, will give you something akin to:

![pipe line](/static/img/pipe_line.jpeg){: .centered }

It might not look pretty or be straight, but it'll get where it's going. Together with the idea of always returning 
something comes the idea of having your "state" returned from the functions. This state is what you are continuing 
with. This state is what you are validating in Unit Tests and this state, because it is immutable, will be valid now 
and forever!

## Conclusion

So to conclude this epic tale of chaos: returning values from functions goes hand in hand with immutable state and 
some sort of piping architecture. If you were to introduce immutability into a system accepting void you would never 
get anything done. The simple truth of the matter is, not returning anything results into not being able to continue 
with anything. The only way you could create a system based on returning void is if you maintain state and mutate 
this state through all of your void methods. As I've shown in this post, mutating state can be really, really 
dangerous.

So, to conclude my conclusion. Even though there was no F# code in this post, I did manage to throw in some useless 
maths, make references to compilers and monads and added images of bracelets and pipelines. All in all an evening 
well spent! I hope you enjoyed reading it and I hope this article will trigger something, a thought, maybe a 
reflective beer on a warm starry night, you never know. Maybe you'll end up writing F# code!

