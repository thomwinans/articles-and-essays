---
title: "The Idle Grid"
date: 2025-08-15 09:00:00 -0400
categories: [Essays]
tags: [ai, energy, infrastructure]
---

There's a paradox in the AI energy story that almost nobody mentions. We're told we don't have enough electricity to train AI. We're told we need to build a parallel grid of hyperscale data centers, and that those data centers will strain the existing grid, and that the existing grid will have to be expanded to keep up. All of that is true.

And yet, every night, a very large fraction of American electrical capacity sits idle.

That's not a rhetorical flourish. The grid is built for peak demand, which happens during the day. At night, demand drops off a cliff. The wires are there. The substations are there. The generators in many cases are still spinning, because it's expensive to throttle them up and down. The capacity exists. We just don't put it to use, because there's nothing to do with it after everyone goes to bed.

So why is the AI industry building a second power grid for itself?

The honest answer is that it doesn't know how to use the one we already have.

The numbers are striking once you look at them straight on. U.S. residential electricity consumption is projected to be somewhere around 1,517 billion kilowatt-hours in 2025. If even ten percent of that were redirected to AI training during off-peak hours — the small hours of the night, when most of the country is asleep and most of the appliances are off — it would cover roughly a thousand GPT-3-scale training runs a year. Not ten. Not a hundred. A thousand. And not as a future build-out requiring tens of billions in capex and a decade of permitting. As an existing, paid-for resource, sitting unused on a grid we've already built.

GPT-3 reportedly took about 1,287 megawatt-hours to train. GPT-4, by some accounts, took fifty gigawatt-hours — enough to power San Francisco for three days. These are big numbers if you compare them to a household. They are not big numbers if you compare them to the country.

So the constraint isn't electricity. The constraint is *where* the electricity is.

The reason a data center is centralized is historical, not physical. We started training models on a single machine because that's what we had. We scaled to clusters because the models got too big for one machine. We scaled to data centers because the clusters got too big for one room. At each step the architecture was the same shape: bigger box, faster interconnects, more electricity in one place. Nobody ever questioned the underlying assumption — that the training had to happen in one place.

But it doesn't. There's a technique called federated learning in which each device trains a small version of the model on its own data, and then sends a summary of what it learned — not the data, not the full weights, just a distilled signal — back to a central server. The server aggregates the summaries into a larger model. The data never moves. Only the lessons learned from it do. And the network bandwidth involved is small enough that a home internet connection can handle it without anyone noticing.

Combine that with model distillation — where a small "student" model is trained against the outputs of a bigger "teacher" model — and you have a path to training pieces of an AI on hundreds of millions of devices that are already sitting plugged in, idle, on free Wi-Fi, in the middle of the night.

This isn't speculative. Published work on communication-efficient federated learning via knowledge distillation has shown bandwidth reductions of around ninety-five percent compared to naive synchronization, with model quality competitive with centralized training. The pieces exist. They have for years.

The architecture that falls out of this is not the one we have. It's a tiered one. At the edge, your phone or your laptop trains a small model on something it's local to — your typing, your photos, your habits — and pushes only the distilled signal upward. In the middle, a desktop with a consumer GPU or a small institutional cluster aggregates signals from a neighborhood's worth of edge devices. At the top, a relatively modest set of high-end servers consolidates everything into the model people actually use. The hyperscale data center isn't gone. It's just the small tip of a very large pyramid, instead of the whole thing.

What's striking about this picture is how little of the technology is the hard part. Federated learning is a real field. Distillation is in production. Smart meters are widespread. Home internet is fast enough. The hardware in a five-year-old laptop, ten million of them, is more than enough to substitute for a small data center.

So why hasn't it happened?

The reason isn't technical. It's structural.

The companies that train the frontier models also own the data centers. The utilities don't think of AI as their business. The homeowners don't know they have something valuable. And the venture money is much better at funding a ten-billion-dollar moonshot than a coordinated contribution of a few kilowatt-hours each from a hundred million participants. The incentives all point inward, toward the centralized build-out. Nobody is structurally inclined to make the distributed version happen, so nobody does, and people start mistaking the absence for impossibility.

This is the part that should make us uncomfortable. The reason AI is becoming the province of a small handful of hyperscalers isn't that the underlying physics requires it. It's that we organized the industry this way, and the organization is starting to mistake itself for a law of nature.

If a million people contributed a kilowatt-hour each per night, and the resulting training pool were open, what would AI development look like? It would look more like Linux than Windows. More like the Web than AOL. It would look like the kind of thing that, in retrospect, would make people wonder why it took so long.

It would also do something interesting to the privacy conversation. If the data never leaves the edge — if only a distilled, statistical signal moves — then most of the things we currently worry about with centralized training stop being worries. There's nothing in the central server to breach. There's nothing aggregated to subpoena. The model knows something *like* what your phone knew, but the something is a shadow, not a copy. That's a different posture toward user data than the current one, and it's a better one.

There are obstacles, and it's worth being honest about them. Consumer devices are less reliable than enterprise hardware. Coordinating tens of millions of nodes is harder than coordinating a thousand in one building. The grid isn't quite smart enough yet to route load preferentially to AI training during off-peak hours, because we haven't asked it to. Incentives for participation have to be designed — energy credits on your bill, priority access to the model you helped train, something. None of these are show-stoppers. They are engineering problems and policy problems, of the kind we solve all the time when we decide we want to.

We have to decide we want to.

Maybe the lesson is bigger than AI. We're surrounded by idle capacity we don't see. Empty seats in cars. Empty rooms in houses. Empty hours in the evening. Empty CPUs while the screen savers run. Every so often, a piece of software notices one of these and turns it into something — Uber for the cars, Airbnb for the rooms — and when it does, the result feels like value conjured out of nothing. It isn't, of course. The capacity was always there. We just hadn't bothered to look.

The electric grid is the biggest piece of idle capacity in the country, and it's sitting on us right now while we worry about brownouts a decade from now and quietly resign ourselves to handing AI over to whichever five companies can finance their own substations.

The grid doesn't need to be expanded before we can train AI. It needs to be noticed.

And once we notice it, the question stops being "how much new power can we build for AI?" and becomes "who gets to participate in AI when the power is already there?" Those are very different questions, and they have very different answers. The first one ends with a handful of hyperscalers and a strained grid. The second one ends with a few hundred million people, the grid we already have, and a model trained by something that looks much more like a country than a company.

We get to choose which question we're answering. We just have to remember that we're choosing.
