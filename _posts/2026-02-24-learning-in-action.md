---
title: "Lisp, and Learning in Action"
date: 2026-02-24 09:00:00 -0400
categories: [Essays]
tags: [technology, ai]
---

[This is a working paper! I've learned much from John Seely Brown, and his influence here is felt, especially vis-a-vis learning in action or in situ, and abduction.]

I've been learning Lisp. Not for the first time — I used it decades ago on a Symbolics, and used Smalltalk before that — but this time is different enough to be worth writing down.

Back then I was a user. I wrote Lisp the way a tourist speaks a language: enough to get around, not enough to think in. I knew the syntax. I'd heard the famous sentences — that code is data, that functions are just lists, that eval and apply are the whole engine. I could have repeated them at a party. But I hadn't lived inside them.

But I've now built an interpreter with a bit of compiler mixed in, with the help of an LLM. And the thing that surprised me wasn't that I finally understood Lisp (there is quite a way to go yet) — people have been learning Lisp by writing interpreters for sixty years. What surprised me was how much I learned while doing it, and how different that learning felt from anything I'd done before, including the years I'd spent at the Symbolics keyboard.

When I sit down to build something with an LLM, I'm always in one of two situations, and they're surprisingly different. In the first, I know exactly what I want. I can say "give me a function that returns the running average of a list," and I can write a test before I write the function. Pass or fail. In this mode, using an LLM is mostly dispatch. In the second, I sort of know what I want. I have a shape in my head. I couldn't tell you what success looks like because I don't yet know enough to know. I'll know it when I get there. And getting there is the point.

Most writing about LLMs assumes the first mode. But most of the interesting work — the kind where you learn something — happens in the second. And the second mode has its own rules, which nobody has written down as far as I can tell.

When I started the interpreter a bit more than one year ago, I was firmly in mode two, despite having used Lisp before. That's the first strange thing. You'd think prior exposure would put you in mode one: you've seen the language, you know what it does, the task is just reconstruction. But that wasn't my experience at all. Being a past user of something turns out to be almost indistinguishable from being a stranger to it, *once you try to build it from the inside*. I knew what Lisp looked like. I didn't know what it *was*. So I couldn't have written a test for the whole thing. I didn't know what success was. But I could write tests for the small things. "When I type `(+ 1 2)`, the interpreter should print 3." That's a test. "When I define a function and call it, it should work." Another. As I built, the tests accumulated, and the pile of tests became something strange and useful: a slowly forming definition of what I was trying to build.

Software people will recognize this as TDD — test-driven development, the practice Kent Beck formalized in 2002, though he was careful to note he'd rediscovered it rather than invented it.[^1] Write the test first. Watch it fail. Write just enough code to make it pass. Refactor. Repeat. It has two nice properties: you end up with code that does what you said it should, and the tests accumulate into a living specification.

But it seems that TDD was never really about code. It was about a way of thinking, and the way of thinking generalizes.

You can write an essay this way. Before drafting, write down what the essay has to do. Not a thesis — more like a set of checks. "The reader should come away believing X." "Section three should answer the question from section one." These are tests. They aren't automatic, but they're checkable in the sense that matters: you can look at the draft and say yes or no.

Research works the same way. You don't know what you'll find. But you can write: "By the end of this week, I should be able to explain in one paragraph why this protein folds this way." If you can't, you've failed the test. Maybe the question was wrong; maybe the answer doesn't exist. But the test gave you something to push against.

Almost any act of creation has, buried inside it, an implicit set of tests. Usually they live in the creator's head. What's interesting about working with an LLM is that you have to pull them out.

Why? Because LLMs love detail. If you say "write something about taxes," you get a page of mush. If you say "a 300-word explanation, aimed at a first-time freelancer, of why estimated quarterly taxes exist, in a voice that sounds like a friendly accountant," you get something much better. The detail isn't decoration. It's the specification. It's the test.

This sounds like a design constraint, but it's a gift. Working with LLMs forces you to articulate what you want, and articulating what you want is most of the work of any creative task. Most bad writing is by people who didn't know what they wanted to say. Most bad software is by people who didn't know what they wanted it to do. The LLM won't let you get away with that — or rather, it will, cheerfully, produce mush if you give it mush. The discipline is yours to impose.

So the first thing that happens, when you start building with an LLM in mode two, is that you have to digitally write down what you want in more detail than you normally would. The act of writing down a test is itself an act of discovery. You find out what you don't know as soon as you try to specify it. I should note that this is not implying writing an incredibly detailed requirements document replete with epic and story detail, though that doesn't necessarily hurt. But designing systems usually requires *bigger* thinking than that, too. Certainly this is true when implementing Lisp. You need to think about data types, lists and conses, functions and macros and backquote processing, I/O, and much more. You're almost forced to simultaneously think both *outside in* and *inside out*. What is nice about LLMs is that there is a bit of forgiveness regarding precision as long as you try. AND there are ways to interrogate the LLMs to see if understanding is clear...

Back to the interpreter. Early on I asked the LLM to do things in a general way, e.g.: "Write me an eval function that evaluates Lisp expressions." This is mush. And it's worth saying that I asked in this way precisely because I was a former user. An LLM, while hyperspatially and statistically marvelous, is software. My request, in which I seemingly assumed that the LLM could divinely interpret my thoughts, was ridiculously vague. A complete stranger to Lisp would have known to ask something smaller. I asked something big because I half-remembered the shape of the answer, which is a worse starting position than having no memory at all. What's a Lisp expression? What should happen for each kind? What about errors? Environments? I didn't really know, but I thought I did, and because I thought I did, the LLM couldn't know either. What it produced was plausible and mostly wrong. And worse, it was the kind of wrong I might have accepted a decade ago as "close enough," because my user-level familiarity let me read past the problems.

So I backed up. "When I eval `(+ 1 2)` in an empty environment, the result should be 3." Small, but because it was small, it forced me to decide things. How are numbers represented? What does `+` mean — is it a built-in, or a symbol to look up? Every test answered one question and raised two more. Pretty soon, I was having to think through special operators, functions, macros, and evaluating arguments before they're processed by some function vs. understanding the importance of *not* pre-evaluating in the case of macros.

After a few days I had numerous small tests. And something interesting happened: they started to triangulate in on a concrete implementation of Lisp. When I added new features like closures, tests broke — not because closures were wrong, but because my earlier model of variable lookup had been incomplete. Tests passed, but for wrong reasons.

This is what programmers call regression, and when it happens you have three possibilities. The new code is broken. The old test was wrong. Or the old test was right as far as it went, but incomplete — it only covered the easy cases, and you've now discovered one it didn't anticipate.

All three are common. All three are, in their own way, learning. The regression isn't a bug. It's evidence.

Here's a way to picture what the tests are actually doing. When I started, I had many possible framings of what Lisp was — the tourist's framing, half-remembered Symbolics habits, textbook slogans, various sediments of misconception. You can think of these as candidate territories, each one a possible answer to the question "what kind of thing am I building?" The first test, `(+ 1 2) → 3`, was like dropping a bit of color on one of those territories to mark it as warmer — more likely to be where I should be standing. Each subsequent test deposited color somewhere. When the closures test broke, it wasn't just a bug report; it was evidence that I'd been coloring the wrong valley. The correct territory was adjacent, and I had to redistribute. Over time, the coloring stabilizes. New tests stop moving the boundary much. Old tests don't contradict new ones. At that point the territory is the specification, and the specification is what you actually understand. The tests are how you map what you're trying to know.

This is more than a metaphor, I think. Learning something unfamiliar is mostly a matter of figuring out which of several plausible framings you're actually in. The tests are how you commit to one and discover where its edges are.

There's a word for this kind of reasoning that people don't use much outside philosophy or detective stories (a tip of the hat to Sherlock Holmes fans): abduction. Deduction goes from rules to conclusions. Induction goes from examples to rules. Abduction goes backward: you have a surprising observation, and you work back to the best explanation. The word is due to Charles Sanders Peirce, who put the schema roughly this way: the surprising fact C is observed; but if A were true, C would be a matter of course; hence there is reason to suspect A.[^2] Peirce called abduction "the only logical operation which introduces any new idea." Deduction and induction rearrange what you already have.

When a test regresses, you're doing abduction. The observation is surprising. The "why" isn't in the code — it's in your model of the code. The regression is evidence that your model was wrong, and you have to revise it. In the coloring picture, it's the moment when the existing map fails to explain a new observation, and you have to raise heat somewhere else or redraw a boundary.

This is also, incidentally, why being a former user was such a liability. A former user has a *lot* of coloring to revise. A beginner has almost none, so every surprise is a clean addition. I had twenty-year-old intuitions that were subtly wrong and that I had to break before I could replace. The regressions weren't teaching me Lisp; they were teaching me which parts of what I already believed about Lisp I needed to unlearn.

We usually talk about learning as induction: see enough examples, infer the pattern. But a lot of real learning, the kind that happens when you're building something you don't fully understand, is abduction. You have a model. The world does something you didn't expect. You revise. That's the loop.

This isn't how we usually think about working with LLMs. The usual story is that they're tools for producing output: text, code, summaries. Prompt in, result out. But in the mode I'm describing, the LLM isn't producing output. It's helping you run the loop. Your model is wrong in some way. The LLM generates the next candidate. The test tells you if it's right. The regression tells you which belief was wrong. You revise. Go again.

The LLM here isn't quite a tool, and not quite a collaborator. It's more like a fast way to manifest your guesses. Without one, you'd code each guess yourself, which is slow, and the slowness would tempt you to commit to wrong beliefs. With one, you can try things. And trying things, cheaply, is the whole game.

None of this is new. Donald Schön, who spent years watching how architects and doctors and town planners actually worked, called something very close to this "reflection-in-action."[^3] The real competence of a professional, he argued, wasn't in applying theories from school. It was in a running conversation with the situation in front of them. They try something, the situation pushes back in a way they didn't expect, and they revise. Schön's more striking observation was that skilled practitioners usually know more than they can put into words. Their knowledge is tacit, embedded in practice.

What's funny is that Smalltalk users in the 80s were doing reflection-in-action at the keyboard before Schön's book was a decade old, and so were Symbolics users. The live image, the inspector, the browser, the debugger that let you fix a method mid-stack and resume execution — that was the loop, in machine form. You had a guess, you tried it, the image pushed back, you revised. The environment made the cost of a revision close to zero, which is why people who used those systems tended to talk about them with a kind of grief after they left. They weren't just editors. They were a medium for thinking.

What I've come to realize, building my interpreter with an LLM this past year, is that I'm back in something like that medium. Not the same — the LLM doesn't give me a live image of my program the way Smalltalk did. But it gives me something Smalltalk never did, which is a live image of the space of *possible* programs. I can ask for three different implementations of something and see them side by side. I can ask why one approach would be better than another and get a real answer, not a guess. The loop isn't just fast now; it's wider. At each step I can see more of the terrain I'm choosing from.

I didn't know Lisp in words. But I came to know it through a running conversation with the thing I was building. Each test was a probe. Each regression was the system pushing back. My understanding lived in the accumulated tests before it lived in any sentence I could have said. And — this is the part that surprised me most — I came to know the Lisp I'd used thirty years ago better than I had when I was using it, because I was finally building the thing from underneath.

David Kolb formalized a broader version of reflection-in-action as the experiential learning cycle: concrete experience, reflective observation, abstract conceptualization, active experimentation.[^4] You do a thing; you think about what happened; you form a theory; you try the theory; you do it again. Real knowledge, Kolb argued, gets made by cycling through experience and reflection until a theory earns its keep — not by being handed the theory at the start. That's worth saying because it means my user-era exposure to Lisp wasn't really knowledge in Kolb's sense at all. I'd skipped the experimentation step. I'd done the experience and a bit of the conceptualization, and then stopped. The theory never had to earn its keep, so it didn't.

Working with an LLM in mode two is a recognizably Schön-ish, Kolb-ish experience. It's reflective practice, sped up. And the speed matters more than people realize. When the cost of running another loop drops from five days to five minutes, the kind of thing you can learn changes. Things that used to require an apprenticeship — because only in an apprenticeship could you run the loop often enough to learn from it — are now approachable at a desk.

Using this technique and LLMs beyond software feels mechanical at first, e.g., like it might kill the part of writing that's alive. What I've found is the opposite. Constraints on what is being developed free me to focus on the parts that actually matter, the same way a carpenter with a good jig can focus on the cut instead of on holding the wood. You outsource the stable parts to the structure and put your attention where it counts.

And tests help me notice when I'm quitting too early, which is maybe the most important point. If you have no tests, you stop when you're tired. If you have tests, you stop when you pass them. Very different stopping conditions.

Research works the same way. The old workflow was: go to Google, open ten tabs, get lost, forget what you were looking for, come back an hour later mildly depressed. The new way is: ask the model, in detail, what you want to know; have it search; ask follow-ups. Your attention stays on the question.

This sounds like a small improvement. I don't think it is. The old search-engine workflow drained you because every tab was a rabbit hole, and every rabbit hole was a competing test for your attention. With an LLM you can hold a question in your head for an hour, which is not something most people have been able to do for twenty years.

There's a piece of evidence from education research I want to bring forward. In 1984, Benjamin Bloom published what became known as the "2 sigma problem." Students who got one-on-one tutoring plus mastery learning performed about two standard deviations better than students in a normal classroom — roughly 50th percentile to 98th.[^5] It was extraordinary, and depressing, because one-on-one tutoring is, as Bloom put it, "too costly for most societies to bear on a large scale." His challenge to researchers was to find methods of group instruction that could match it.

Forty years later the challenge has shifted. Randomized trials and meta-analyses of LLM-based tutoring report meaningful gains in performance, engagement, and retention — one RCT found students learning significantly more in less time than with in-class active learning.[^6][^7] The gains aren't uniform, and the studies flag real problems: over-reliance, uneven reliability, the way LLMs can confidently produce wrong answers that a tired student will take on faith.

What's interesting isn't that LLM tutors work better than lectures. It's that the mechanism seems similar to what I am discovering in my own work. They give the learner a feedback loop. They let the learner specify what they don't understand and try again, as many times as it takes. They supply the thing a human tutor supplies when there's only one student in the room: constant, particular, responsive feedback. Bloom guessed that if society couldn't afford a tutor for every child, some combination of altered variables might approximate the same effect. He couldn't have guessed one of those variables would be a machine.

Let me just say the point plainly. The way to use LLMs well — for writing, for research, for coding, for almost anything — is to bring to the task a test-driven mindset. Decide what you want. Write down how you'll know you got it. Let the tests accumulate. Let them push back when they fail. View tests as a set of guards that add color in the digital workspace. Revise your beliefs when they surprise you.

A skeptic might say: you're describing what good writers and researchers have always done. True. Good writers have always had, in their head, a running checklist of what the piece has to achieve. They didn't call it TDD. They called it craft. Schön would have called it reflection-in-action. A Symbolics user would have called it Tuesday.

What's changed isn't the method. What's changed is the cost of trying a candidate. Before LLMs, if you wanted to try a radically different opening, you had to write it. That took an hour. You probably didn't try it. Now you can try five openings in five minutes. The tests are still the hard part. But the attempts are almost free.

When the cost of attempts goes to zero, the returns to good tests go up. Anyone who can specify well has a superpower they didn't have three years ago. Anyone who can't will produce mush.

People have been learning by doing for as long as there have been people. Blacksmiths learned by forging. Doctors learn medicine by practicing it. The knowledge is in the loop.

What LLMs do — and this is the thing I keep underestimating — is let you enter this kind of loop in domains where you couldn't before. You can learn Lisp by building an interpreter because the LLM lets you generate candidates fast enough that the feedback loop closes. You can learn a new field conversationally because the LLM keeps you from drowning in tabs. You can learn to write a certain kind of essay by drafting it fifteen times in an afternoon. You can learn, thirty years late, what you thought you already knew.

The loop is old. What's new is that the loop is available for almost anything. Not just for crafts where you already had a master. Not just for subjects where you had years to spend. Not just for people who were lucky enough to sit at a Symbolics in 1988. The things that used to require apprenticeship, or decades of practice, or access to a rare expert or a rare machine — a lot of those can now be entered through the door of a careful prompt and a willingness to run the loop.

This doesn't mean everyone will learn more. Most people will use LLMs for output and keep asking for mush. The loop is available; whether you run it is up to you.

I don't want to oversell this: (I keep telling myself that) the LLM isn't magic. Most of what it does is confidently wrong in small ways, and without tests you'll ship the wrongness. The same meta-analyses that report learning gains also report over-reliance problems that only a learner actively checking can catch.[^6] The method depends on your discipline, not the model's. The better the model gets, the more dangerous it is to work without tests, because the mush will be more plausible.

But with tests — even informal ones, even just a handful of written-down checks — the thing becomes an engine. You put in a question. You get back not an answer but a candidate. The test tells you if it's right. If not, you revise, and in the revising you learn what you didn't know. The pile of tests becomes something close to a specification of the thing you're trying to understand. Which is another way of saying: a coloring of the territory you were trying to map.

That's really what learning is, once you strip it of the trappings of school. The slow accumulation of things you've checked. Books give you a shortcut — someone else did the checking and you're trusting them. Being a user gives you a related shortcut — someone built the thing, and you're trusting the surface. Practice gives you the checks themselves. Now, for the first time, you can practice almost anything, because the cost of trying has dropped so low.

I started this little essay thinking I would write about Lisp. Instead, I ended up writing about learning. Maybe that was the test the essay had to pass — that the Lisp was just the example, and the real subject was the method underneath. Or maybe the real subject was that I finally learned something I'd been carrying around for thirty years without understanding.

If there's something you've always thought you knew but never really got underneath: write a test. Any test. Something small and checkable. Then start the loop.

---

[^1]: Beck, Kent. *Test-Driven Development: By Example*. Addison-Wesley, 2002. Beck has noted he did not invent TDD but rediscovered it from older literature.

[^2]: Peirce, Charles Sanders. *Collected Papers*, vols. 2, 5, 7. For a modern treatment see Douven, "Peirce on Abduction," Stanford Encyclopedia of Philosophy. Peirce placed abduction in the context of discovery rather than justification.

[^3]: Schön, Donald A. *The Reflective Practitioner: How Professionals Think in Action*. Basic Books, 1983. Schön examined five professions — engineering, architecture, management, psychotherapy, and town planning — and argued that skilled practitioners rely less on formal theory than on a tacit, ongoing conversation with the situation they are working in.

[^4]: Kolb, David A. *Experiential Learning: Experience as the Source of Learning and Development*. Prentice Hall, 1984.

[^5]: Bloom, Benjamin S. "The 2 Sigma Problem: The Search for Methods of Group Instruction as Effective as One-to-One Tutoring," *Educational Researcher* 13:6 (1984), 4–16.

[^6]: "Large language models in education: a systematic review of empirical applications, benefits, and challenges," *Computers and Education: Artificial Intelligence*, 2025. See also "A Meta-Analysis of LLM Effects on Students across Qualification, Socialisation, and Subjectification" (2025), synthesizing 133 experimental and quasi-experimental studies from 2022–2025.

[^7]: Kestin, Greg et al. "AI tutoring outperforms in-class active learning: an RCT introducing a novel research-based design in an authentic educational setting," *Scientific Reports*, 2025.
