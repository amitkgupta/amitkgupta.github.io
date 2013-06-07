---
layout: post
title: "So You Still Don't Understand Hindley-Milner?  Part 3"
date: 2013-06-07 00:22
comments: true
categories: 
---
In Part 2, we finished defining all the formal terms and symbols you see in the StackOverflow question on the Hindley-Milner algorithm, so now we're ready to translate what that question was asking about, namely the rules for deducing statements about type inference.  Let's get down to it!

##The rules for deducing statements about type inference

[Var]  

$$\underline{x:\sigma \in \Gamma}$$
$$\Gamma \vdash x:\sigma$$

This translates to: If "$x$ has type $\sigma$" is a statement in our collection of statements $\Gamma$, then from $\Gamma$ you can infer that $x$ has type $\sigma$.  Here $x$ is a variable (hence the name of this rule of inference).  Yes, it should sound that painfully obvious.  The terse, cryptic way that [Var] is expressed isn't that way because it contains some deep, difficult fact.  It's terse and succinct so that a machine can understand it and type inference can be automated.  
<!--more-->

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

