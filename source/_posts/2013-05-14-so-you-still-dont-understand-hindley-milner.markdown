---
layout: post
title: "So you still don't understand Hindley-Milner?  Part 1"
date: 2013-05-14 01:12
comments: true
categories: 
---

I was out for drinks with [Josh Long](https://twitter.com/starbuxman) and some other friends from work, when he found out I "speak math."  He had come across [this StackOverflow question](http://stackoverflow.com/questions/12532552/what-part-of-milner-hindley-do-you-not-understand) and asked me what it meant:

![](http://i.stack.imgur.com/hZhjl.png)  
<br>

Before we figure out what it means, let's get an idea for why we care in the first place.  [Daniel Spiewak's blog post](http://www.codecommit.com/blog/scala/what-is-hindley-milner-and-why-is-it-cool) gives a really nice explanation of the purpose of the HM algorithm, in addition to an in-depth example of its application:

> Functionally speaking, Hindley-Milner (or “Damas-Milner”) is an algorithm for inferring value types based on use.  It literally formalizes the intuition that a type can be deduced by the functionality it supports.

Okay, so we want to formalize an algorithm for inferring types of any given expression.  In this post, I'm going to touch on what it means to formalize something, then describe the building blocks of the HM formalization.  In [Part 2](/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-2/), I'll flesh out the building blocks of the formalization.  Finally in [Part 3](/blog/2013/06/07/so-you-still-dont-understand-hindley-milner-part-3/), I'll translate that StackOverflow question.  
<!--more-->

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
1. An algorithm that intelligently uses the deduction rules to get from a starting point to deducing/inferring a desired conclusion statement: "the expression $e$ that I'm interested in has type $t$".  This is the "algorithm" part for the "HM algorithm", and that's not something I'll be going into in these posts.

