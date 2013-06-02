---
layout: post
title: "So you still don't understand Hindley-Milner?"
date: 2013-05-14 01:12
comments: true
categories: 
---

I was out for drinks with some friends from work, and one of them found out I "speak math."  He had come across [this StackOverflow question](http://stackoverflow.com/questions/12532552/what-part-of-milner-hindley-do-you-not-understand) and asked me what it meant:

![](http://i.stack.imgur.com/hZhjl.png)  
<br>

Before we figure out what it means, let's get an idea for why we care in the first place.  [Daniel Spiewak's blog post](http://www.codecommit.com/blog/scala/what-is-hindley-milner-and-why-is-it-cool) gives a really nice explanation of the purpose of the HM algorithm, in addition to an in-depth example of its application:

> Functionally speaking, Hindley-Milner (or “Damas-Milner”) is an algorithm for inferring value types based on use.  It literally formalizes the intuition that a type can be deduced by the functionality it supports.

Okay, so we want to formalize an algorithm for inferring types of any given expression.  I'm going to quickly touch on what it means to formalize something, then describe the building blocks of the HM formalization, and finally translate that StackOverflow question.  

##What it means to formalize

Okay, so we want to talk about expressions.  Arbitrary expressions.  In an arbitrary language.  And we want to talk about inferring types of these expressions.  And we want to figure out rules for how we can infer types.  And then we're going to want to make an algorithm that uses these rules to infer types.  So we're going to need a meta-language.  A language to talk about expressions in an arbitrary programming language.  This meta-language should:

* Be abstract and generic, so that it allows us to reason about statements of type inference purely based on their *form* (hence, *formalization*), without having to worry about their content.
* Give a precise, unambiguous, yet intuitive definition for what an expression is.
* Give those definitions in terms of a small number of uncontroversial primitive concepts.
* Similarly give definitions for types, the idea that an expression has a type, and the idea that we can infer that a given expression has a given type.
* Lend itself to a simple, terse symbolic representation, e.g. rather than saying "the expression formed by applying the first expression to the second expression has the type of a function from strings to some type we don't care to specify in the current context" we could simply say "$e_1(e_2):\mathrm{String} \rightarrow t$".
* Be easily translated to something a computer can understand and implement, so we can ultimately automate type inference.

To make all that a little more concrete, let's look at a really quick example of a formalization.  If, instead of formalizing a language for talking about inferring types of expressions in an arbitrary programming language, what if we wanted to formalize a language for talking about truths of sentences in arbitrary natural languages?  Without formalization, we might say something like

> Suppose I know that if it's raining, Bob will carry an umbrella.  
> And suppose I also know that it's raining.  
> Then, I can conclude that Bob will carry an umbrella.  
> 
> And any argument that takes this form is a valid way to reason.

Propositional Calculus formalizes that whole things as a rule known as Modus Ponens:

$$\underline{A,\ \ A \rightarrow B}$$
$$B$$

where $A$ and $B$ are variables representing propositions (a.k.a. sentences or clauses) in an arbitrary natural language.  

Okay, so let's enumerate the building blocks of the HM formalization:  

##Building blocks of the formalization

We will need:

1. A formal way to talk about expressions.  This formalization should meet the criteria enumerated above; for this purpose we use the **Lambda Calculus**.  I'll be explaining that in a minute, but there's nothing crazy going on here.
1. A formal way to talk about types, and a formal way to talk about expressions and types together.  After all, the purpose of the HM algorithm is to be able to deduce statements of the form "expression $e$ has type $t$".
1. A formal set of rules for deriving statements about expression types from other such statements. Rules along the lines of: "if I can already demonstrate that some expression has this type, and another expression has that type, then this third expression has this other type".  *Such a set of rules is exactly what you're seeing in that SO question*.  I'll be translating this in full detail.
1. An algorithm that intelligently uses the deduction rules to get from a starting point to deducing/inferring a desired conclusion statement: "the expression $e$ that I'm interested in has type $t$".  This is the "algorithm" part for the "HM algorithm", and that's not something I'll be going into in this post.

##Formalizing the concept of an expression

We'll give a [recursive definition](http://en.wikipedia.org/wiki/Recursive_definition) of what an expression is; in other words, we'll state what the most basic kind of expression is, we'll say how to create new, more complex expressions out of existing expressions, and we'll say that only things made in this way are valid expressions.  

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

##The rules for deducing statements about type inference

So let's get down to it.  

[Var]  

$$\underline{x:\sigma \in \Gamma}$$
$$\Gamma \vdash x:\sigma$$

This translates to: If "$x$ has type $\sigma$" is a statement in our collection of statements $\Gamma$, then from $\Gamma$ you can infer that $x$ has type $\sigma$.  Here $x$ is a variable (hence the name of this rule of inference).  Yes, it should sound that painfully obvious.  The terse, cryptic way that [Var] is expressed isn't that way because it contains some deep, difficult fact.  It's terse and succinct so that a machine can understand it and type inference can be automated.  

[App]  

$$\underline{\Gamma\vdash e_0:\tau\rightarrow\tau '\ \ \ \Gamma\vdash e_1:\tau}$$
$$\Gamma\vdash e_0(e_1):\tau \rightarrow \tau '$$
  
This translates to: If we can infer that $e_0$ is an expression whose type is $\tau \rightarrow \tau '$ (e.g. $e_0$ might be an anonymous function which, according to $\Gamma$, takes input of type $\tau$ and returns output of type $\tau '$), and we can infer that $e_1$ has type $\tau$, then we may deduce that we can infer that $e_0(e_1)$, the expression obtained by applying $e_0$ to $e_1$, has type $\tau '$.  The intuitive gist is if we can infer the types of the input and output of a function, and we can infer some expression has the same type as the input of the function, then when we apply the function to that expression, we can infer the result expression has the type of the output of the function.  Nothing bewildering here.  By the way, this is the formalization of our `speak(duck)` example.  
  
[Abs]  

$$\underline{\ \ \Gamma, x:\tau \vdash e:\tau '\ \ }$$
$$\Gamma \vdash \lambda x.e:\tau \rightarrow \tau '$$
  
This translates to: If allowing us to assume that $x$ has type $\tau$ we were able to infer that $e$ has type $\tau '$, then we may deduce that we can infer that the abstraction/anonymization of $e$ with respect to the variable $x$, $\lambda x.e$, has type $\tau \rightarrow \tau '$.  So, for example, we know that if $x$ has type $\mathrm{String}$, then the expression $x[0]$ has type $\mathrm{Char}$.  Now [Abs] allows us to deduce that  

```javascript
function(x) { return x[0]; }
```

has type $\mathrm{String} \rightarrow \mathrm{Char}$.  

Aside. I mentioned polytypes earlier.  Let's revisit them it in this example, just to help hammer it home.  So note that this function above also has type $\mathrm{Array}[\mathrm{Int}] \rightarrow \mathrm{Int}$.  In fact, for any type $t$, the function has type $\mathrm{Array}[t] \rightarrow t$.  So it has many different types, $\mathrm{String} \rightarrow \mathrm{Char}$ being just one of them.  *Each* of its types of the form $\mathrm{Array}[t] \rightarrow t$ is monotype.  We can express that this function has *all* of these monotypes by saying that it has the polytype $\forall t(\mathrm{Array}[t] \rightarrow t)$.  We read that as "for all $t$, the type $\mathrm{Array}[t] \rightarrow t$" and we treat that whole thing as a single, yet more abstract, type.  So note that when we infer the type of some expression, that doesn't mean that said type is the *only* type of that expression.  An expression can have many types, and some of these types can be specializations of more abstract types.  The simplest kinds of types are monotypes: $\mathrm{Int}$, $\mathrm{String}$, $\mathrm{String} \rightarrow \mathrm{Char}$, etc. but we can have more abstract/general types called polytypes.  
  
[Let]  

$$\underline{\Gamma \vdash e_0:\sigma\ \ \ \ \Gamma , x:\sigma \vdash e_1 : \tau}$$
$$\Gamma \vdash \mbox{let } x = e_0 \mbox{ in } e_1:\tau$$ 

Easy:  
    
If we can infer that $e_0$ has type $\sigma$, and    
If we were to assume $x$ had type $\sigma$ we could infer that $e_1$ has type $t$,  
Then we may deduce that we can infer that the result of letting $x = e_0$, and substituting it into $e_1$, has type $t$.  

These last four rules do nothing more than formally capture our intuition about what type inferences we can make when we have variables and we do things like create anonymous functions, apply functions, and substitute expressions into other expressions.  It's something we as programmers can do intuitively, and here we're just saying that this is something we can formally describe, what's happening in our brains isn't necessarily magical.  It's also worth noting that these last four rules correspond precisely with the four rules for defining what a valid expression is in the Lambda Calculus.  This is not a coincidence.

[Inst]

$$\underline{\Gamma \vdash e:\sigma '\ \ \ \ \sigma '\sqsubseteq \sigma}$$
$$\Gamma \vdash e:\sigma$$

This is about instantiation.  You can think of the monotype $\mathrm{Array}[\mathrm{Int}] \rightarrow \mathrm{Int}$ as an instantiation of the polytype $\forall t. \mathrm{Array}[t] \rightarrow t$.  Another word for this is "specialization": $\mathrm{Array}[\mathrm{Int}] \rightarrow \mathrm{Int}$ is more special than for $\forall t.\mathrm{Array}[t] \rightarrow t$. We denote the "more special than" relation between types with $\sqsubseteq$.  So  

$$\forall t. \mathrm{Array}[t] \rightarrow t \ \ \sqsubseteq\ \  \mathrm{Array}[t] \rightarrow t$$  

So the direct translation of [Inst] is: If we can infer $e$ has type $\sigma '$, and $\sigma$ is a specialization/instantiation of $\sigma '$, then we can deduce that we can infer that $e$ has type $\sigma$.  And you can think of $\sigma$ and $\sigma '$ as being types like $\mathrm{Array}[t] \rightarrow t$ and $\forall t. \mathrm{Array}[t] \rightarrow t$ respectively.

[Gen]

$$\underline{\Gamma \vdash e:\sigma\ \ \ \ \alpha \notin \mathrm{free}(\Gamma)}$$
$$\Gamma \vdash e:\forall \alpha.\sigma$$

This is the hardest one to understand.  It really only makes sense in the context of doing a type inference using these restricted set of rules we're outlining.  It doesn't have a very concrete analogue since it heavily depends on the concept of a variable type, something that never occurs in any real programming language, but is an indispensable concept when we're trying to work in a meta-language that talks about types in any arbitrary real programming language.  The idea can sort of be captured in this "example":

Suppose you have some variables $x$ and $y$, and for the time being you're assuming they have type $\alpha$, where $\alpha$ is a variable standing for a type.  You later come across an expression that you somehow manage to infer has type $\alpha \rightarrow \alpha$ in *this* context (the context where you're assuming $x$ and $y$ have type $\alpha$).  The question is, will this function have the polytype $\forall \alpha. \alpha \rightarrow \alpha$?  I.e. does this function generally map objects to things of the same type, or does that only appear to be the case because you assumed $x$ and $y$ had the same type $\alpha$?  

Since $\alpha$ is a variable type, i.e. it could stand for any type, we might like to think that, since we've inferred that $e$ has type $\alpha \rightarrow \alpha$ that it has the polytype $\forall \alpha. \alpha \rightarrow \alpha$.  But we can't necessarily make this generalization without more insight into how $e$ is related to $x$ and $y$;  In particular, if our inference that it has type $\alpha \rightarrow \alpha$ is tightly coupled to our prior assumptions involving $\alpha$, then we shouldn't conclude that it generally has the polytype $\forall \alpha .\alpha \rightarrow \alpha$.

Here's the translation:

If some variable type $\alpha$ hasn't "freely" been mentioned in our current context/set of knowledge/assumptions, and we can infer that some expression $e$ has some type $\sigma$, then we can infer that $e$ has type $\sigma$ independent of what $\alpha$ turns out to be.  Slightly more technically, $e$ has the polytype $\forall \alpha . \sigma$.  

Okay, but what does "freely mentioned" mean?  In a polytype like $\forall \alpha . \alpha \rightarrow \alpha$, $\alpha$ isn't "really" being mentioned.  That type is the exact same as this one: $\forall \beta . \beta \rightarrow \beta$.  An expression with either type is just that of a function that sends any type to itself.  On the other hand, $x:\alpha$ "really" does mention $\alpha$.  

$$x:\alpha$$
$$y:\beta$$

and

$$x:\alpha$$
$$y:\alpha$$

mean different things.  The latter means $x$ and $y$ definitely have the same type (even though what that type is may not have been pinned down).  The former tells you nothing about how the types of $x$ and $y$ are related.  The difference is, when $\alpha$ is mentioned inside the scope of a $\forall$, as is the case in $\forall \alpha . \alpha \rightarrow \alpha$, that $\alpha$ is just a dummy, and can be swapped out for any other type variable regardless of the rest of the context.  So we can interpret the statement "$\alpha$ isn't freely mentioned in the context $\Gamma$" to say, "$\alpha$ is either never mentioned at all, or, if it is, it's only ever mentioned as a dummy and could in principle be swapped out for something entirely different without changing the semantics of the assumptions/knowledge in our context."  

And that's it.  Questions?  Comments?  Let me know.
