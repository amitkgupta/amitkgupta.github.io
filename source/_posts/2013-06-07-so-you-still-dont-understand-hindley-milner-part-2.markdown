---
layout: post
title: "So You Still Don't Understand Hindley-Milner?  Part 2"
date: 2013-06-07 00:19
comments: true
categories: 
---
In Part 1, we said what the building blocks of the Hindley-Milner formalization would be, and in this post we'll thoroughly define them, and actually formulate the formalization:

##Formalizing the concept of an expression

We'll give a [recursive definition](http://en.wikipedia.org/wiki/Recursive_definition) of what an expression is; in other words, we'll state what the most basic kind of expression is, we'll say how to create new, more complex expressions out of existing expressions, and we'll say that only things made in this way are valid expressions.  
<!--more-->

1. Variables are valid expressions.  
1. If $e$ is any expression, and $x$ is any variable, then $\lambda x.e$ is an expression.  Here it helps to think of e as typically (thought not necessarily) a more complex expression involving $x$, e.g. $x^2+2$, and then $\lambda x.e$ as the anonymous function that takes an input $x$ and returns the result of evaluating the expression $e$ with the given value of $x$.  In other words, think of it like this:  
```javascript
function(x) { return x^2 + 2; }
```
1. If $f$ and $e$ are valid expressions, and $f$ can take arguments (e.g. if it's an abstraction like explained in point 2), then $f(e)$ is a valid expression.  This is called Application, for obvious reasons.
1. If $x$ is a variable, and $e_1$ and $e_0$ are valid expressions, then substituting every occurrence of $x$ in $e_0$ for $e_1$ yields a valid expression.  So, for example, if $e_0$ is the expression $x^2 + 2$, and $e_0$ is the expression $y/3$, then if we let $x = e_0$ in $e_1$, we get the expression $(y/3)^2 + 2$.

And nothing else is a valid expression.

Aside: anyone paying close attention will wonder, wait, how can I make any useful expressions out of this?  How do I even get $x^2 + 2$, or in fact $2$ for that matter, out of the above?  Heck, what about $0$?  There is nothing in the rules above which *obviously* yield the expression $0$.  The solution is to create expressions in the Lambda Calculus which behave like $0, 1, \dots, +, \times, -, /$ when interpreted correctly.  In other words, we have to encode numbers, numerical operations, strings, etc. into patterns we can create with the Lambda syntax. [This blog post](http://palmstroem.blogspot.com/2012/05/lambda-calculus-for-absolute-dummies.html) has a nice little section on numbers and numerical operations.  This is a great feature of the Lambda Calculus: we have a simple syntax which we can define recursively in 4 simple clauses, and this therefore allows us to prove many things about it inductively in 4 main steps, yet the language itself has the expressive power to capture numbers, strings, and all the types and operations we could ever care about.

##Formalizing statements about types

Let $e$ be any expression, that is, "$e$" is a variable in our meta-language which stands for any expression in our base language, like any of the following:

```javascript
x
Math.pow(x,2)
[1,2].forEach ( function(x) { print(x); } )
```

Then if $t$ is any type, we can express "$e$ is of type $t$" by  

$$e:t$$  

Just like $e$, $t$ is a variable in our meta-language, and it can stand for any type in the base language, like $\mathrm{Int}$, $\mathrm{String}$, etc.  And just like $e$, $t$ doesn't necessarily need to stand for any one type in particular.  

One can give a formal definition for what counts as a type, just as we did for expressions above.  However the abstraction gets fairly twisted, so we'll leave it at that.  I should just point out a few two key things to keep in mind:   

1. If $s$ and $t$ are types, then so is $t \rightarrow s$; it's the type of a function with $t$ inputs and $s$ outputs.  
1. If $r$ is a type, possibly made up of other types (just as $t \rightarrow s$ is made up of $t$ and $s$, which could each potentially have been made up of other types), and $\alpha$ is a variable for a type, then $\forall \alpha.r$ is a type.  That makes no sense without an example:

```javascript
function (x) { return x; }
```

This function is type $\mathrm{String} \rightarrow \mathrm{String}$.  But it's also $\mathrm{Int} \rightarrow \mathrm{Int}$.  In fact, for any type $t$, it's type $t \rightarrow t$.  We're gonna say that it has type $\forall t. t \rightarrow t$.  Each of the types $\mathrm{String} \rightarrow \mathrm{String}$, $t \rightarrow t$, are "monotypes". $\forall t. t \rightarrow t$ is a "polytype".  The identity function above has the abstract polytype $\forall t. t \rightarrow t$ which, in practice, means that for every real type $t$, it has type $t \rightarrow t$.  If all of this has been sinking in, then we can compactly express this as:

$$\lambda x.x:\forall\alpha.\alpha \rightarrow \alpha$$

##Formalizing statements about statements about types

Now we're going to want to formalize a bunch of rules for how we can go from some knowledge of expressions and their types to inferring types of more expressions.  Remember how propositional calculus formalized Modus Ponens?  We're going to do something similar.  For instance, say we want to formalize the following piece of reasoning:

> Suppose I've already been able to infer that a variable `duck` has type `Animal`.  
> Suppose furthermore that I've inferred that `speak` is a method of type `Animal -> String`.  
> Then I can infer that `speak(duck)` has type String.  
>   
> And any reasoning of this form is valid type inference.

We'll formalize that as follows:

$$\underline{\Gamma\vdash e_0:\tau\rightarrow\tau '\ \ \ \Gamma\vdash e_1:\tau}$$
$$\Gamma\vdash e_0(e_1):\tau \rightarrow \tau '$$

Let's first get a handle on all the symbols you see above:  

* $\Gamma$, this will stand for the collection of statements we already know, or perhaps, the statements we're assuming.  More generally, $\Gamma$ should just be thought of as some collection of statements (about expressions and their types).  And of course, there's nothing special about the letter $\Gamma$; capital greek letters are commonly used for sets of statements however.  
* $\vdash$, the "turnstile", denotes that something can be inferred.  For instance, $\Gamma \vdash x:t$ says that if we take the statements in $\Gamma$ as our assumptions/axioms/current knowledge, then we can infer that $x$ has type $t$.  
* $\in$, "epsilon", denotes membership.  $x:t \in \Gamma$ says that the statement $x:t$ is a member of $\Gamma$.
* That long horizontal bar; this line tells us that we can make the conclusion below the line if the things above the line are taken as premises in the argument.   It lets us express things like, "if we can infer such and such, then we can infer such and such", e.g.  

$$\underline{\Gamma \vdash y:\sigma}$$
$$\Gamma \vdash x:\tau$$  

If we can infer that $y$ has type $\sigma$ from $\Gamma$, then we can infer $x$ has type $\tau$ from $\Gamma$.  
