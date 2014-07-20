---
layout: post
title: "Is math programming? You're asking the wrong question. Part 2"
date: 2014-07-19 23:21
comments: true
categories: 
---

In [Part 1](/blog/2014/07/19/is-math-programming-youre-asking-the-wrong-question-part-1/) I started to address [Sarah Mei](https://twitter.com/sarahmei)'s recent blog post about how [programming is not math](http://www.sarahmei.com/blog/2014/07/15/programming-is-not-math/).  In this post, I'll continue my critique of her article and highlight the ways in which math can be valuable to a programmer.

## Understanding math to understand programming problems

In her article, Sarah attacks the claim that "without a mathematical foundation, you’ll have only a surface understanding of programming" by first reducing it to an alleged "common variation" of the same sentiment: "without a CS degree, you can’t build anything substantial."  But wait, those aren't the same thing at all.
<!--more-->

#### Moving the Goalposts

I can argue that a mathematical foundation can help give you a deep, valuable understanding of certain programming problems.  No one can argue that you need math to build anything of substance.  Especially if "substance" is correlated with VC funding.  ~~Yes, you can build "Yo!" without a deep understanding of anything.~~  Worthwhile endeavours and opportunities for gainful employment abound, which don't require understanding much math, or any deep understanding of the programming problems at hand.  

She has a point when she says "computer science is not programming".  Yes.  But understanding some math, and mathy computer science concepts, can certainly be valuable when it comes to reasoning about a programming problem, and communicating that reasoning with your collaborators.

Here's an example from something I was working on recently.  We were building an appliance for our enterprise clients, to streamline the process of deploying [Cloud Foundry](https://www.gopivotal.com/platform-as-a-service/cloud-foundry) to their on-premise datacenters.  Users would enter configuration, including an IP subnet in CIDR notation and a set of blacklisted IP ranges in a format like `10.10.0.0-10.10.0.15, 10.10.0.129-10.10.0.255`.  They would then select and configure any number of distributed services to be deployed, such as the Cloud Foundry PaaS, an add-on MySQL DBaaS cluster, and let's say a [Pivotal HD](http://www.gopivotal.com/big-data/pivotal-hd) Hadoop cluster.  Our appliance would spin up VMs on which to run these services, assigning IPs to these VMs within the given subnet, but outside the blacklist.

Everything was fine and dandy, until a field rep told us that one of his customers was experiencing some really slow performance.  We noticed that they were using a `/16` subnet.  First off, tell me that a solid confidence and facility with numbers doesn't make understanding [CIDR blocks](http://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_blocks) a walk in the park.  Anyways, after some debugging we traced the problem to some code that looked like this:

```ruby  
class IpPool
	def initialize(subnet, blacklist)
		@subnet = subnet           # eg. NetAddr::CIDR.create('192.168.1.1/24')
		@ips = subnet.enumerate    # an Array of strings representing IPs in the subnet
		@blacklist = blacklist     # an Array of strings representing IPs to avoid using
		@taken_ips = {}				# a Hash of ip:String => purpose:String pairs
	end
	
	### snip ###
	
	def find_or_take_ips_for_job(job, num_ips)
		already_taken_ips = @taken_ips.select { |_ip,purpose| purpose == job }.keys
		result = already_taken_ips
		
		(num_ips - already_taken_ips.size).times do
			if next_ip = next_available_ip
				@taken_ips[next_ip] = job
				result << next_ip
			else
				raise AvailableIpNotFoundError
			end
		end
		
		result.first(num_ips)
	end
    
	def next_available_ip
		@ips.find { |ip| !@blacklist.include?(ip) && !@taken_ips.has_key?(ip) }
	end
end
```

Actually, it was a fair bit more complicated than that.  Because this code was being called by something that was first looping over all already-installed products and hydrating the IP pool, then then looping over all the yet-to-be-installed products to determine what IPs to assign for each of its jobs. For each product, it had to loop over each job that comprises that product and find-or-take IPs from the pool depending on how many VMs it would be running.

My pair and I were able to reason through this problem and communicate with each other by breaking down the runtime complexity of the problem in terms of big O.  Having the training of thinking about problems in that way helped.  Having a common language within which to frame the problem, a language that's precise and unambiguous, really helped.  I'm not saying the problem in any way *requires* familiarity with runtime analysis and big O notation.  But just imagine what two people with no exposure to these things would go through when reasoning about this problem and communicating with each other.  Then appreciate **how much friction just disappears** if those two collaborators knew this stuff.

The fix?

```ruby
	def next_available_ip
		(@ips - @blacklist).find { |ip| !@taken_ips.has_key?(ip) }
	end
```

When $n = 2^{16}$, an $O(n^2) \to O(n)$ improvement is significant.  Calculations that were taking many hours (or rather, hitting the nginx timeout and never finishing) in the worst case were now being done in a couple of seconds at most!

As evidence for the claim that understanding math doesn't help much with understanding programming, Sarah cites her experience: "I have found little connection between a person’s formal qualifications and the depth of their understanding."

#### Stawman Argument

Formal qualifications and understanding math are *not* the same thing!  She talks specifically about big O in this context.  So here's a challenge: amongst all your colleagues, think about the ones who have limited formal education (e.g. early college dropout) but whom you admire for their skills and passion for solving hard and interesting problems.  Now amongst those folks, find one who doesn't get big O notation or who doesn't care to think about runtime analysis.  Can you even find one?

I think the example above and the two examples in Part 1 all show how a mathematical approach can certainly help you better understand and tackle certain problems that come up, even in your day-to-day, non-mathy work.  Here's yet another example: [Using the framework of classical optimization theory to understand the problem of designing a PaaS to schedule resources for running user applications in a highly available manner](http://blog.gopivotal.com/cloud-foundry-pivotal/products/app-placement-in-cloud-foundry-diego-a-classical-optimization-problem).

## Learning math to improve how you approach programming problems

You'll often hear that even if you don't use math as a programmer, math teaches you abstract thinking and problem solving, and you need that for programming.  Sarah retorts: "learning to program is more like learning a new language than it is like doing math problems."

#### Another Strawman Argument

Doing math problems is not the same as learning math.  And there's *so much more* to learning math than learning calculus, matrix arithmetic, a bit of graph theory and some combinatorics.  Especially in America, the state of modern math education in college is quite poor for the vast majority of students.  What they're exposed to is excessively computational (doing problems vs. learning), centuries old, and severely limited in both breadth and depth.

So, what about *learning* math?  Is there something special about it that helps someone learn programming later on?  Absolutely.  First off:

> Doing math is about taking abstract and unfamiliar concepts and bodies of knowledge, and internalizing them by building mental models so that they feel concrete and familiar, often by building upon existing mental models, so that you can pose conjectures and prove assertions about these concepts.

You can try to say that this description could apply to other fields, but there's no field for which this is as fitting a description as it is for math.  This habit of building mental models to understand the unfamiliar and make it concrete is something I use every time I'm introduced to a new problem domain.  And I use the word "habit" deliberately, because it's not that you need math to be able to do this, it's that learning math necessitates making this a habitual part of your thinking process.

Furthermore, because so much of math is so abstract:

> Doing math develops tenacity -- the confidence that things which seem entirely opaque to you know can eventually be learned and understood in a way that seems clear through patience and effort.

I came to programming knowing nothing.  I didn't know the difference between Java and JavaScript.  I didn't know about GET, POST, PUT, DELETE.  I had hardly used Unix.  I'd never heard of "client-server".  Didn't know a thing about networks, IaaS, virtualization, databases, etc.  My head would be throbbing after work every day for my first few weeks as I was being flooded with knowledge.  And I would come in to work the next day and notice that a lot of the stuff from the previous day had leaked out overnight.

But that was okay.  I could tell myself, there's no way this stuff is more abstract than combinatorial characterizations of compactness and incompactness of the second uncountable cardinal, the stuff I never wrote my thesis about.  It took several months, but I was eventually able to penetrate those concepts in set-theory enough to start (but not finish, obviously) being productive.  These programming concepts?  Just give me a few weeks and I can be productive there too.

The ability to feel comfortable while having absolutely no clue what you're doing is something I take out of my time devoted to learning math.  Again, math is not the only path to this goal.  I feel that any artistic practice where the creations take a long time to take shape, be it drawing, sculpture, literature, etc. all develop the same sort of confidence.  But math is at least one way to achieve it.  And when it comes to programming, it certainly helps that math, much more so than art, requires a fairly similar "type" of thinking to programming.

## Summary

No matter where you are in your career as a programmer or what problem domain or business vertical you work on, there is value to knowing math, thinking mathematically, and experiencing the process of learning math.  Knowing math adds a powerful set of tools to your toolbelt.  It won't be the right tool for *every* task, but that's true of any tool.  You know the old chestnut about hammers and nails.  That said, there will undoubtedly be times that an application of math will be the best tool for the job, where that "job" could mean optimizing, designing, debugging, experimenting, or anything else.

Thinking in terms of mathematics, and in terms of those mathy concepts from computer science, can sometimes provide an incredibly productive way to reason about a particular programming problem.  Furthmore, the precise nature of mathematics often makes it an ideal way to communicate about a particular programming problem, especially when all your collaborators are familiar with the language.

Finally, the process of learning math (real math, not multiplying matrices) is a great way to build habits and attitudes towards problem solving that transfer readily to programming.