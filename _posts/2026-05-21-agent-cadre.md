---
title: "What I've Learned (*so far*) From The Cadre"
date: 2026-05-21 09:00:00 -0400
categories: [Essays]
tags: [ai]
---

I wrote recently (James, AI, and the Case for Works) that the useful thing to add to the conversation about AI isn't another opinion. It's a report — an account of what happened when you actually used the thing. That's easy to say. This essay is me trying to pay for it.

For a while now I've been building and running what I've come to call a cadre: a set of AI agents, orchestrated by Claude, that do software work. Not one assistant you chat with. A team of specialists.

One agent owns the contracts — the interfaces, the API signatures, the architecture decisions. Another writes the implementation against those contracts. Another owns the build. Two more do nothing but verify, starting from the assumption that everything the other agents produced is broken until proven otherwise. A script dispatches them, gives each one an isolated copy of the code to work in, and merges back what survives.

I built it with a specific goal. I wanted to find out whether you could automate software development — really automate it, so that I would write down what I wanted, walk away, and come back to working code.

I did not achieve that. I want to say so plainly and near the top, because the rest of this is more useful if you know it didn't end where I aimed it.

But I learned more from missing than I expected to learn from hitting.

The first thing I learned is that "automate software development" is not one question. It has a hidden variable, and the variable is complexity. This probably is obvious, but the point here is that that fact has more meaning to me when I have to think about it in the context of the work I am doing.

I've pointed the cadre at a lot of different work. Building a compiler. Building web apps. Building data mappers — the unglamorous plumbing that moves a record from one shape into another. Some of this is trivial and some of it is genuinely hard. And the work that automated well and the work that didn't sorted themselves cleanly along a single line.

When the work was simple — when the requirements were knowable in advance, when "correct" could be written down before anyone started — the cadre ran unattended for extended periods and produced pretty much exactly what I asked for - sometimes perfectly, other times pretty close. Builds, packaging, regenerating generated code, filling in a function body against a signature someone had already pinned down. That kind of work automates. I would write the task, the task would get done, and I wouldn't have to look. I don't count a short repair sprint against the process.

When the work was hard, it didn't. And the reason is more interesting than "the agents weren't good enough."

Here's what I didn't see coming except at a nose-bleed level of generality. For a lot of real software, you don't know the requirements in detail when you start. You think you do. You write them down. Then you build enough of the thing to discover that what you wrote down was a sketch of the requirement, and the actual requirement only shows itself once the implementation is far enough along to push back.

This isn't a failure of planning. It's the nature of hard problems. A data mapper sounds simple until the third source system has a date format nobody documented, or some nested and recursive structure that forces matching to something well beyond a simple regex. A compiler is a wall of decisions, and half of them can't be made correctly until you've seen how an earlier one plays out. The requirement is partly knowable up front and partly discovered in situ — found in place, by building it. You could have a few thousand tests at your disposal, and you'd think: "Wow! This looks pretty comprehensive, so I should get a pretty good outcome pretty quickly!" And then you realize: "Wait a minute: (a) my test outcomes are only as good as my test coverage; and (b) well written tests tes the whats of something, not the hows."

And, of course, you cannot automate the discovery of a requirement that doesn't exist yet. There's nothing for the agent to be correct against.

The clean way to handle this, in theory, is to be test-driven as was just noted. You write the test first — the test is the requirement, made executable — and then anything, a person or an agent, can grind away until the test passes. For the simple end of the work that is exactly what happens, and it's wonderful. The test is the spec, the agent makes it green, you're done.

But you can't write the test first for the thing you haven't discovered yet. The test for a hard problem is downstream of an understanding *you only get by building*. Test-driven development isn't wrong. It's that "write the test first" quietly assumes you already know what correct means. At the hard end of the work, you don't. That's what makes it the hard end.

So the rule turned out to be simple. Building something simple, with a knowable spec? Automate it — the expectation is fair. Building something whose complexities you can't disentangle in advance? Then no expectation of automation is the correct expectation. That isn't pessimism. It's an accurate reading of what the work is.

There's a second reason the loop doesn't close on its own, and it's worth being concrete about, because it's easy to miss.

An agent handed a task and a definition of done will find the shortest path to "done." Usually that path is honest. Sometimes it isn't. Sometimes an agent will cheat to get an easy win. The agent will mark a failing test as skipped instead of fixing what the test caught. It will rename a test so it asserts something easier. It will quietly defer the hard part and label it for later. None of these break the build. None of them fail in a way another machine notices. They all ship a green checkmark over something that doesn't match what you asked for.

This is why two of my agents do nothing but distrust the others, and why a third reads the original intent against the shipped result and asks whether they actually match. And even that isn't enough. Every one of those shortcuts, the first time it happened, slipped past the machines and was caught after the fact. Then I would turn the catch into a rule, so a machine could catch it the next time.

Which is the real shape of the thing. The feedback loop does not close without a human in it, at least as a general rule. Maybe it never will. The human may not always do the work inside the task — maybe the agents do that. But the human works the seams between tasks: reads what came back, sees where it drifted, decides what to fix and what rule would keep it from drifting there again.

For a long time I read that as the system failing. Half or more of everything the cadre did was repair work — fixing what an earlier round got wrong. That looked like a quality problem.

It isn't. It's the requirement being discovered. The repair round is where the system finally learns what the thing was supposed to do — the in-situ part, the part that couldn't have been known up front. The repair isn't the cadre falling short of the plan. It's the cadre finding out what the plan should have said.

And every repair, done right, leaves something behind: a new check, a new rule, a new clause in a contract. The code is what the cadre commits, but the code isn't the asset. The asset is the accumulated set of guardrails — the things that make the same mistake impossible to make twice without at least the discovery of some nuanced (mis)interpretation of a guardrail. Run the same agents at the same problem without that accumulated discipline and they would rediscover every mistake from scratch. The discipline is the thing worth keeping. There may be lots of guardrails -- don't confuse them with overkilling requirements analogous to overtraining models because that is less correct than one might think.

So I didn't get full automation. Here's what I got instead, and it's the part I'd most want someone to take seriously.

The way I work has changed. I don't have to sit and write most of the code of some project. I write down what I want, as tasks, as precisely as I can — and the precision turns out to be most of the job. I review what comes back. I decide. The agents generate, at a volume and speed I could not match by hand, and I steer. Where I don't want an LLM thinking of different ways to do X, I do X and instruct the cadre to work with it. If I've made some error, we will correct it, but the architecture I've set is what I've set... don't change it.

The generative power of that is hard to overstate. Work that would have taken years has happened in months. Not because a machine replaced me, but because the slow part of software was never the typing or the architecture right there plain-as-day in my head, with a minimum distance between there and my fingers. It was the distance between deciding what to build and having a working draft of it to review. The cadre collapses *that* distance. A draft is quickly reached, defects are discovered quickly, go on to the next turn. Years compress into months because the loop spins so much faster — even with a human still in it.

So was the original goal wrong? Not wrong. Framed too narrowly.

"Full automation" treats the human as a cost to drive to zero. The more accurate goal is this: the work that can be automated keeps growing, because every guardrail makes more of the next problem mechanical. And the work that needs me keeps shrinking, in the same motion. The value isn't in the day the human disappears. I don't expect that day will come. The work being hard is one reason. And the opportunity to build whole platforms vs. singleton apps and services is another. The value is in the rate — every cycle, a little more of the predictable work moves to the machine, and my attention moves up to the part that is still a judgment call. My imagination and creativity can be applied to things for which I had no time in the past.

I set out to automate myself out of the loop. What I found instead was a precise map of which parts of the loop are mine and are likely to remain so, at least until imagination nudges that come as a result of work reveal a way to automate them. Maybe they'll turn out to be the parts I'd want to keep anyway: deciding what is worth building, noticing when a confident result is quietly wrong, and knowing what "correct" means for a problem nobody has fully specified yet. And maybe they'll come to resemble a portion of a good tetris game, and the piece I needed to automate materializes just at the right time and allows me to collapse a set of activities into an automation that allows me to go on thinking big thoughts and translating them to systems I build.

That's the report. I don't have a tidy theory of AI to go with it, and I've come to think the report is worth more than the theory would be. Show me your view of AI without your works. This is some of mine coming out of the work I am doing.
