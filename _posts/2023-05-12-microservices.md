---
title: "Microservices, Babies and Bathwater"
date: 2023-05-12 09:00:00 -0500
categories: [Essays]
tags: [technology]
---

A few months ago Amazon published a case study that read (gone now from the original Amazon publishing), at first glance, like a recantation. They'd moved a service off a microservice architecture and back onto a monolith, and the numbers were good enough that the whole thing started to sound like something larger than a case study. Around the same time David Heinemeier Hansson was making a similar argument in [essays](https://signalvnoise.com/svn3/the-majestic-monolith/) [of](https://world.hey.com/dhh/how-to-recover-from-microservices-ce3803cc) his own. The internet being what it is, these were soon read as obituaries. Microservices are dead. Long live the monolith.

I don't think that's quite right.

The trouble with declaring an architecture dead is that architectures aren't really the sort of thing that can be dead. An architecture is just a set of decisions. Some of those decisions turn out to be good for your situation and some turn out to be bad, and the hard part isn't picking a side in advance but knowing which is which.

When microservices first appeared, they came with a sales pitch. Each team could pick its own language. Each team could deploy independently. A bug in one service wouldn't take down the whole platform. Tests would be easier to write because each piece was small. It sounded great. It still sounds great. But if you read the pitch closely, every one of those promises is about behavior in the small, not the large.

Take polyglot. "Use the right tool for the right job" sounds reasonable. But like most things that sound reasonable, it depends on what you mean by "right job" and how many of them you want. If you have one service in Go and another in Python because one is doing numeric work and the other isn't, fine. If every team picks a language because they felt like it that morning, you have a problem. Not a technical problem, exactly. A staffing problem. Because in five years half those engineers will have left, and whoever replaces them will have to get fluent in whatever was chosen in the first week.

Then there's granularity. The word "micro" did a lot of damage here. People took it literally. A microservice should be small, they reasoned, and the smaller the more micro. But there's a floor below which a service stops being a unit of business and becomes a function with a network between it and its caller. Functions are a fine thing. Networks are a fine thing. But putting a network in the middle of every function call is a bad trade, because now every call can fail in ways functions don't usually fail, and you have to reason about timeouts and retries in places you used to think of as arithmetic.

People will tell you this is what tooling is for. That's mostly an article of faith. The tools for debugging a distributed system across ten languages and three continents are, in my experience, either nonexistent or bad. Say what you want about monoliths. You can at least set a breakpoint in one.

I've done technology diligence on companies where microservices went all the way. Different stacks, different teams, most of them in time zones far from the people paying the bills. The orchestration layer, which was supposed to hold the whole thing together, turned out to be a source of mystery and expense on its own. Everything that could go wrong did, and then some things no one had thought of. Total cost of ownership, that phrase consultants say and nobody reads, turned out to be real.

So does this mean the Amazon team is right, and we should all be writing monoliths again?

No. Or at least, not for that reason.

The trouble with monoliths isn't that they're bad; it's that they drift. If you don't watch them, they start to grow parts that shouldn't touch each other. The UI knows something about the database it shouldn't. Two modules that were separate become one tangled thing because it was faster, once, to make them touch. Testing becomes a full regression every time, because nobody can tell which parts depend on which. Rails apps in particular seem prone to this, but it isn't Rails' fault. You can write a clean architecture in Rails. You just have to know you want one.

Is Rails the problem? No. The problem is architecture.

This is the thing everyone seems to miss in the microservices debate. Both sides have their cautionary tales, and both cautionary tales have the same moral. When you ignore the architecture — when you just pick a style and run — you get in trouble. That's true of microservices. That's true of monoliths. It'll be true of whatever comes next.

So what should you actually do?

A few things. If you're going to use serverless — and serverless is worth using, because the pricing is remarkable — measure what it costs you before you build your company on top of it. Cloud orchestration is cheap when you're playing and expensive when you're in production, and you often find this out only after you've been billed. When you factor in the cost, you'll sometimes discover that some of your fine-grained services would be happier as larger ones.

Think about services in terms of business functions, not technical ones. This is the single most useful distinction I know in this area. A service should do something a person could describe in one sentence to a non-programmer. If you can't do that, the service is either too small or too vague.

Remember that code outlives you. Programmers don't absorb new languages or new codebases overnight, no matter what they say in interviews. A polyglot stack is a tax your company pays forever. You can pay it. But you should know you're paying it.

And think about scale from the start. Not because you need it from the start, but because the moment you do need it, you won't have time. Success tends to show up as a pile of new functional demands that feel urgent, and performance work is always the first thing to get deferred. If your architecture can't scale and you become successful, congratulations: you now have two problems instead of one.

So is serverless bad? Are microservices bad? Are monoliths bad?

None of them are bad. None of them are good. They're tools, and like any tool they reward people who understand what they're for. The real mistake, the one under every "X is dead" essay ever written, is reading one case study as a verdict on a whole category of ideas. Amazon moved one service and saved money. That tells you something about that service. Hansson built [a successful product](https://signalvnoise.com/svn3/the-majestic-monolith/) as a monolith. That tells you something about that product. Neither of them tells you what to do.

If you read those stories as arguments for monoliths over microservices, you're throwing the baby out with the bathwater. And the baby, in this case, is the part everyone keeps forgetting: what matters isn't which architecture you pick, but whether you thought about it hard enough to have reasons.
