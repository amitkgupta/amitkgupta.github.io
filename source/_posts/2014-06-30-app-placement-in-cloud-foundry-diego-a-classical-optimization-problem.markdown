---
layout: post
title: "App Placement in Cloud Foundry Diego: A Classical Optimization Problem"
date: 2014-06-30 22:48
comments: true
categories: 
---
I work on the Cloud Foundry [Diego](https://github.com/cloudfoundry-incubator/diego-release) team.  Diego started as a rewrite of the DEA in Go, but by the time I joined it had evolved into a ground-up rearchitecture of the core service. Our team's fearless leader [Onsi Fakhouri](https://twitter.com/onsijoe) recently [gave a talk](https://www.youtube.com/watch?v=1OkmVTFhfLY) explaining the Diego rewrite.  Seriously, watch that talk, it might literally be *the* best engineering talk you'll ever see.

Onsi talks about the problems associated with having the Cloud Controller responsible for knowing about all the apps on all the DEAs and orchestrating the placement of newly started apps.  I thought it'd be fun to introduce some of the key players in the Diego architecture and explain how Diego solves the app placement problem by modelling it as a sort of classical **constrained optimization problem**.

<!--more-->

Caveat: In what follows, I actually describe a slightly idealized version of Diego.  For instance, I take in account the desire to balance apps across multiple Availability Zones (if it were deployed on AWS) or Clusters (if it were vSphere), which Diego doesn't currently do.  These are just my own musings, but I hope this provides an enjoyable little glimpse at how parts of Diego and Cloud Foundry work, along with links to some of our code repositories, documentation, and simulation tools so you can learn more if you're so inclined.


## Cast of Characters

An interesting dimension to the complexity of this decision problem is that it's a distributed problem.  Beyond just finding the input which optimizes some objective function subject to some constraints, we have to take into account that the inputs, objective function, and constraints may be coming from different sources, and that these data need to be communicated between sources in a machine-readable way over network and messaging protocols such as HTTP and NATS.  Instead of the old model where CC had the state of the world in its head, we want to gather state from the appropriate sources of truth at decision-time.  Before diving into modeling the optimization problem, let's look at the players involved and their responsibilities.

* [**The App Manager**](https://github.com/cloudfoundry-incubator/app-manager): sits behind the user-facing API, and handles requests to start apps.  It knows things about desired apps, such as how many instances of the app are desired to start, where the source code for the app lives (in some blobstore), what the compute requirements are for an instance of the app in terms of memory, disk, and so on.  It (eventually) causes app instances to be run by requesting an "auction" to be held where machines bid to run the app.
* [**The Auctioneer**](https://github.com/cloudfoundry-incubator/auctioneer): watches for "start-auction" requests for desired app instances, and then holds the auctions.  It gathers "bids" and then chooses a "winner", and the winner is then responsible for running the app.
* [**The Executors**](https://github.com/cloudfoundry-incubator/executor): form a pool of process, each running on a separate VM, with each one capable of running multiple apps in isolated containers within its VM.
* [**The Reps**](https://github.com/cloudfoundry-incubator/rep): represent the Executors, one Rep for one Executor.  They participate in the auctions, providing bids requested by the Auctioneer on behalf of their Executors.  A bid encapsulates relevant information about an Executor, such as what apps its currently running, and how much more free memory it can allocate to new containers.

(For more, check out the [Diego Design Notes](https://github.com/cloudfoundry-incubator/diego-design-notes))

## Setting

Say a user pushes an app, and wants to have 3 instances of it running, so that it's highly available.  The Executors are running on different VMs spread out across different AZs (substitue Clusters or whatever the correct term is on your IaaS of choice).  These instances should ideally be balanced across Executors so that they land in different AZs; if they all end up in the same AZ, or worse, on the same Executor, then the high availability you'd expect to achieve by running 3 instances is a lot more brittle.  Moreover, the balancing of app instances across AZs can't be as simple as saying, "put instance 0 in the first AZ, instance 1 in the next AZ, etc." since that'll clearly skew app placement towards the first AZs and away from the last AZs (in whatever ordering you have for your AZs).

In addition to having their app instances balanced across AZs, the user wants to ensure a certain amount of memory and disk are allocated to each running instance.  So when placing an app instance, in addition to choosing an Executor in an AZ that's ideally not already running too many instances of the app, we want to choose one that has enough free compute resources.  Furthermore, if there are several candidate Executors, it's better to go with the one with the most free resources so as to better balance the load across Executors.  Whether it's AZs or Executors, balancing the load hedges against the risk associated with any one of the AZs or Executors going down.

The App Manager is going to take data about an `AppInstance` and trigger a "start-auction", which will include an **objective function** and **constraints** specific to that `AppInstance`.  The Auctioneer will start the auction and gather `RepBid`s from the Reps.  These `RepBid`s will constitute the **inputs** for the objective function, and the Auctioneer will find the input which minimizes the objective function amongst those that satisfy the constraints.

## Plot (...eh, sorry to stretch the metaphor)

Now let's actually formally define the problem, the data structures involved, and see what pieces are already in place in Diego that implement solutions and measure the quality of solutions.

### Data Structures

```go
type AppInstance struct {
	RequiredDiskMB		uint
	RequiredMemoryMB	uint
	AppID				uint
	InstanceNumber		uint
	TotalInstances		uint
	AppSourceBlobID		string
	Stack				string
}

type RepBid struct {
	AvailableMemoryMB	uint
	AvailableDiskMB		uint
	TotalMemoryMB		uint
	TotalDiskMB			uint
	RunningAppIDs		[]uint
	AvailZoneNumber		uint
	CachedBlobIDs		[]string
	Stack				string
}			
```
	
### Problem

Choose the "highest-scoring" Rep to run a particular `AppInstance`:

$$ \argmax _r f_{\mathrm{AM},ai}(r) $$

where $f_{AM,ai}$ is the objective function for the `AppInstance` provided by the $\mathrm{AM}$ (App Manager), and $r$ ranges over all the `RepBid`s collected by the Auctioneer from the Reps.

### Constraints

A Rep's bid is only feasible if its Executor has enough memory and disk, and is running the same stack (e.g. Lucid 64-bit) as required for the `AppInstance`:

$$r.AvailableMemoryMB \geq ai.RequiredMemoryMB $$
$$r.AvailableDiskMB \geq ai.RequiredDiskMB $$
$$r.Stack = ai.Stack $$


### Actual Problem

The **actual** problem is to express an objective function that captures the business requirements, and then find a near-optimal solution efficiently.  Furthermore, any solution must take into account that the data in a Rep's bid can change in the course of an auction it's participating in, since it maybe participating in multiple simultaneous auctions.  For instance, a Rep might report having 4096MB free memory simultaneously in many different auctions.  If it wins one of the auctions and agrees to start a 512MB app instance, then its original bid in any of the auctions that haven't been decided yet is somewhat invalid, since it now only has 3584MB free.

### Meta-Constraints

The objective function $f_{\mathrm{AM},ai}$ itself must be expressible in terms of $ai$ and $r$ in a fairly simple way.  Here are some meta-constraints on the expression used to define the function by allowing only the following operations:

* **Attributes:** Access public attributes of $r$ and $ai$.
* **Arithmetic:** $+, \times, \div, \mathrm{mod}$ where the modulus for $\mathrm{mod}$ must be a constant, not a computed value.
* **Counting:** `count(x, xs)` which counts how many times `x` occurs in the slice `xs`.

It's not terribly important what these constraints are, just that they ensure that a very simple, strict DSL will suffice to express the objective function.  Nobody wants to be sending arbitrary expressions involving $ai$ and $r$ from the App Manager to be `eval`ed on the Auctioneer.

### A Proposal for the Objective Function

$$ f_{\mathrm{AM},ai}(r) = (ai.InstanceNumber + ai.AppID + r.AvailZoneNumber)(\mathrm{mod}\  \# AZs) + 1$$
$$ +\ \alpha \cdot \mathrm{count}(ai.AppSourceBlobID, r.CachedBlobIDs)$$
$$ +\ \beta \cdot (r.AvailableMemoryMB / r.TotalMemoryMB)$$
$$ +\ \gamma \cdot (r.AvailableDiskMB / r.TotalDiskMB) $$
$$ +\ \delta \cdot (1 - \mathrm{count}(ai.AppID, r.RunningAppIDs) / ai.TotalInstances) $$

where $\# AZs$ is the total number of AZs, and $\alpha, \beta, \gamma, \delta$ are non-negative weighting constants that add up to $1$.

1. The first line is a way of making sure that **each instance of an app gives preference to a different AZ**.  Furthermore, assuming that AppIDs are somewhat random, the addition of the $ai.AppID$ term ensures that we don't always have instance number 0 of every app prefering AZ number 0 every time.  Since the result of the modulo is an integer, and then we add 1, and then the rest adds up to $\alpha + \beta + \gamma + \delta = 1$ at best, we're ensured that balancing instances across AZs is the "most-significant bit", and is given highest consideration compared to free memory, for instance.
2. The second line gives some weight to an **Executor that has a cached copy of the app's source** locally.  The $\mathrm{count}$ expression on that line will only ever evaluate to $0$ or $1$.
3. The third line considers the **percentage of free memory on an Executor**.  
4. And the fourth considers the **percentage of free disk**.  
5. The fifth line takes into account **how many instances of the app the Executor is already running**.  So, for instance, if two Reps are "tied" before considering this criteria, both have the same percentage of Memory and Disk free, but one of them is already running many instances of the app, it would be preferable to put the current instance in question on the other Executor.

(Note: to implement that function in Golang most things will have to be typecast to `float64`.  Not only for the division, turns out Go wants floats for [$\mathrm{mod}$](http://golang.org/src/pkg/math/mod.go) as well.) 


### Evaluating Solutions

Solutions to this optimization problem have to be evaluated based on time taken, CPU and memory cost, number of network calls, how well-balanced it places apps, and all in the context of being run multiple times concurrently.

## What Diego Already Does

### Solutions

Diego already runs auctions.  Not only that, the [auctionrunner](https://github.com/cloudfoundry-incubator/auction/tree/master/auctionrunner) package already defines multiple ways to run an auction.  There are auctions such as [randomAuction](https://github.com/cloudfoundry-incubator/auction/blob/master/auctionrunner/random.go) which optimizes for performance but obviously does miserably at choosing a near-optimal, feasible `RepBid`.  Other auctions make tradeoffs between quality of the `RepBid` it chooses, total number of communications, etc.  Any one of these auctions can be run in a multi-round format, where a winner is tentatively selected but multiple rounds are played, to adjust for the fact that a Rep's bid can quickly become invalid if it's participating in multiple auctions.

What's currently missing is the elaboratness of the objective function.  Diego does not take AZ placement or cached blobs into account, as though the first two lines on the right-hand side of the equation weren't there.  It also weighs free memory, disk, and currently running apps equally, as though $\beta = \gamma = \delta = 1/3$).

Moreover, the objective function is not currently something dynamic.  In fact, the Auctioneer has no notion of an objective function, and Rep bids aren't structured objects.  The Reps compute the scores and just give those scores directly to the Auctioneer, and the formula for computing scores is [hard-coded](https://github.com/cloudfoundry-incubator/auction/blob/master/auctionrep/auction_rep.go#L238-L247).

### Evaluating Solutions

Diego has a [Simulator](https://github.com/cloudfoundry-incubator/auction/tree/master/simulation)!  You can run the simulator on its own, and it also runs as part of the test suite.  You can play with all sorts of different variables in the system.  For instance, you can obviously run different types of auctions (random vs. pick-the-best).  But you can also play around with different communication modes, e.g. in-process vs. NATS, to see how performance is affected when the communication between processes is "realistically" handled by a central message bus.  Simulation runs are accompanied by a report that will show up in your browser, giving you statistics such as total number of communications and standard deviation in the number of apps placed on each Executor, and various visualizations so you can see how well the apps were balanced, or how frequently the auction required more than 1 round to choose a winner.  Here's what the output looks like in the browser and the terminal:

<div style="margin:auto"><a href="/images/simulation_browser.png" rel="shadowbox"><img src="/images/simulation_browser.png" width="40%" height="40%" style="vertical-align: middle"></a>
<a href="/images/simulation_terminal.png" rel="shadowbox"><img src="/images/simulation_terminal.png" width="40%" height="40%" style="vertical-align: middle"></a></div>
