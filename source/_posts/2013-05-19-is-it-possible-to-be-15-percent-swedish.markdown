---
layout: post
title: "Is it possible to be 15% Swedish?"
date: 2013-05-19 13:32
comments: true
categories: 
---
This question came up as a joke during a team standup a few months ago.  Although the obvious answer is "no," if you're willing to play fast and loose with your metaphysics for a bit, the answer can be "yes" and there's a cute solution that ties together binary numbers and binary trees.  This post itself is a bit of a joke in that it's just for fun, but it might be nice to see the familiar concepts of binary numbers and binary trees in a new light.

## The obvious answer is "no"

Let's quickly see why the real life answer is "no."  But first we should lay out the assumptions implicit in the problem.  We're going to assume that at some point in time, everyone was either entirely Swedish or entirely non-Swedish.  There's a chicken-and-egg problem that we're sweeping under the rug here, but that's what rugs are for.  Next we're assuming that every person after that point in time has their Swedishness wholly and equally determined by their parents Swedishness.  So if mom is 17% Swedish and dad is 66% Swedish, then baby is ½ x 17% + ½ x 66% = 41.5% Swedish.
<!--more-->

So why is 15% impossible?  Or, for that matter, all the numbers in the previous example: 17%, 66%, 41.5%?  The reason is that any person's Swedishness must be a fraction which, in lowest terms, must have a denominator that is a power of 2.  There's an easy proof by induction.  Initially everyone is either entirely Swedish or entirely non-Swedish.  In lowest terms these fractions can be expressed as 1/1 and 0/1, respectively.  The denominator, 1, is a power of 2 (fyi $2^0 = 1$).  Now, assuming a mom and dad are Swedish in proportions $m/2^M$ and $d/2^D$ respectively, their offspring will be this Swedish:

$$\frac{2^Dm + 2^Md}{2^{D+M+2}}$$

The denominator is a power of 2, and reducing this fraction to lower terms will not change that fact.  Numbers like 15%, a.k.a. 15/100, or 3/20 in lowest terms, have denominators which aren't powers of 2, and that's why no one can ever really be 15% Swedish.

## But what if...?

What if we keep the assumption that Swedishness is determined equally by the parents' Swedishnesses, but without assuming there was some point in time where everyone was either entirely Swedish or entirely non-Swedish?  Let's weaken that assumption to simply state that every person has an ancestor that's either entirely Swedish or entirely non-Swedish.  And let's do one more crazy thing: let's allow human history to go back infinitely through the generations, with no beginning.  So there could be an infinitely long lineage of Swedes without there being a first Swede.  If it helps, imagine a family tree that's infinitely tall, with no original top level.  In this universe of bastardized metaphysics, can you be 15% Swedish?  Why yes!

## Binary numbers and trees

We know how decimal numbers work.  12.34 as a decimal number means

$$1 \times 10^1 + 2 \times 10^0 + 3 \times 10^{-1} + 4 \times 10^{-2}$$

Binary numbers work the same way, with 2's instead of 10's.  So the number four is:

$$1 \times 2^2 + 0 \times 2^1 + 0 \times 2^0$$

So it's represented as $100$ in binary.  Similarly one-half is:

$$0 \times 2^0 + 1 \times 2^{-1}$$

So it's represented as $0.1$ in binary.  What about a number like one-third?  It's equal to the value of the following infinite sum:

$$0 \times 10^0 + 3\times 10^{-1} + 3\times 10^{-2} + \dots$$

So it's represented as $0.33...$ in decimal.  It's also equal to the following infinite sum:

$$0 \times 2^0 + 0 \times 2^{-1} + 1 \times 2{-2} + 0 \times 2^{-3} + 1 \times 2^{-4} + 0 \times 2^{-5} + 1 \times 2^{-6} + \dots$$

So it's represented as $0.010101...$ in binary. 

Now why do we care?  Well, if you've read this far, then you care because you can use the binary representation of a number to figure out what a person's family tree could look like if their Swedishness was equal to that number.  For example, how can you be $1/2$ Swedish?  Well $0.1$ is the binary representation of $1/2$, and this tells us that if we have 1 parent who is entirely Swedish (and hence all the ancestors on that side are entirely Swedish), and one parent who is entirely non-Swedish (along with all their ancestors), then you can be 1/2 Swedish.  

How about a more involved example.  To be $3/16$ Swedish, which is $0.0011$ in binary, you can accomplish this by first having 1 great-grandparent who is entirely Swedish, let's call her Agnetha.  Agnetha's parents will of course have to be entirley Swedish too.  In addition to them you'll need one more great-great-grandparent who's entirley Swedish, let's call him Bjorn.  If you have great-grandma Agnetha and great-great-grandpa Bjorn who are entirely Swedish (as must be all their ancestors), and if all of your ancestors who aren't descendents of Agnetha and Bjorn are entirely non-Swedish, then you'll be exactly $3/16$ Swedish.  How does this look in terms of your family tree?  If we say you are at level 0, your parents at level 1, etc. then what we get is the following:

The first full Swede is Agnetha, on level 3.  
The next full Swede who isn't logically "forced" to be Swedish on account of being Agnetha's ancestor is Bjorn, on level 4.  
There are no other full Swedes except Agnetha's and Bjorn's ancestors.  
Everyone else is entirely non-Swedish unless they're "forced" to be a bit Swedish on account of being descendents of Agnetha and/or Bjorn.

Notice how the "level 3" and "level 4" correspond to the locations of the 1's in the binary expansion of 3/16?  If you go back to the simpler example of 1/2, which was $0.1$ in binary, you'll see that we have one full Swede on level 1 and everyone except that person's ancestors is fully non-Swedish.

## Putting it all together

So here's how you can be 15% Swedish:

1. Express 15% in binary, it'll be infinitely long.
1. Use the binary number as a recipe for marking the certain nodes at certain levels of an infinite binary (family) tree as "entirely Swedish".
1. Any ancestor of one of the marked nodes also gets marked as "entirely Swedish", and any node that's not an ancestor or descendent of a marked node is entirely non-Swedish.  Every remaining node is Swedish to the degree determined by its parents.
1. The person at the root of this family tree will be the desired 15% Swedish.

$$15\% = 2^{-3} + 2^{-6} + 2^{-7} + 2^{-10} + \dots$$

So if you have great-grandma Agnetha, great-great-great-great-grandpa Bjorn, great-great-great-great-great-granpa Benny, and great-great-great-great-great-great-great-great-grandma Anni-Frid, ....  And all of them are entirely Swedish, and none of them are ancestors/descendents of one another.  And if anyone else on your family tree that isn't a blood relative of theirs is entirely non-Swedish.  Then you will be 15% Swedish!

## Back to real life, sort of

Okay, so all that was incredibly silly.  Can we say anything that's merely very silly?  Say you want to know if you can be 15% Swedish in real life, but within some error bounds.  Maybe you want to know if you can be 15% Swedish, give or take 1%.  Easy: find a *finitely-long* binary number that's between 0.14 and 0.16, and repeat the above steps with that number.  One simple way to do that is to start finding the binary expansion of 0.15, and stopping once you're within the desired range:

$2^{-3}$?  
Nope, that's 0.125, too small.  
$2^{-3} + 2^{-6}$?  
Yup, that's 0.140625.

Could we have known ahead of time how much of the binary expansion of 0.15 we'd have to calculate before reaching the desired range?  Yup, we can do that too.  Once you've started writing a binary number out to $n$ digits, no matter what digits you add on next, the most you can add to your current number is $2^{-n}$.  For example, all the binary numbers that start with $0.11010...$ must be within $2^{-5} = 0.03125$ of one another.  So if I know I want to be within $0.01$ (in decimal, i.e. 1%) of 15%, I just have to apply the above reasoning backwards.  $\log _2(0.01) ~ -4.605...$ so if I figure things out back more than 4.605 generations, that's enough.  So I really only need to figure things out 5 generations back.  5 generations back I have 32 ancestors.  The closest fraction of the form x/32 to 0.15 is 5/32.  So I know that if I have exactly 5 totally Swedish level-5 ancestors and the remaining 27 level-5 ancestors are totally non-Swedish, I will be within 1% of being 15% Swedish (I'll be 15.625% Swedish to be exact).

![](http://images3.wikia.nocookie.net/__cb20080726160313/muppet/images/thumb/3/34/Swedishchef-myspace.jpg/300px-Swedishchef-myspace.jpg)
