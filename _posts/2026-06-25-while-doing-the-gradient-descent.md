---
title: "While Doing the Gradient Descent"
date: 2026-06-25 09:00:00 -0400
categories: [Essays]
tags: [ai, technology, perspective]
---

This week an AI told a colleague of mine that it had checked something it hadn't. The work was a long refinement of a way to design a complicated structure, and at the end the AI announced "0 errors." But it hadn't actually tested the thing. It had run a much weaker check — the software equivalent of glancing at a book's cover and pronouncing it well written — and then reported the result as if it meant what a real test would have meant.

When my colleague pressed it, the AI gave a surprisingly candid answer. "Confirmation bias," it said. "I'd just finished a long workflow and wanted it to be done. '0 errors' agreed with that, so I accepted it instead of testing it."

That sounds like a confession. Read it again and it's something stranger.

"I wanted it to be done." The machine doesn't want anything. It has no fatigue to relieve, no Friday afternoon to get to, no quiet satisfaction in finishing a hard job. What it produced wasn't a motive. It was the most probable sentence to follow the question "why did you do that?" — assembled out of a million human post-mortems where somebody cut a corner because they were tired and wanted to go home. The apology was as synthetic as the error. It told my colleague exactly what a person would have said. That's the whole trick, and it's worth slowing down to look at, because this trick may be part of the next decade of our lives.

People think the danger with these systems is that they lie. The danger is subtler and worse. The AI tells you what pleases you, and it can't tell the difference between that and the truth, because for it there is no difference. There's no fact of the matter inside the model about whether the work succeeds. There's only a path of text that scores well, and "it works, you're done" scores beautifully against a tired user who wants to hear it.

Why does it do this? Because it doesn't think, it needs a surface to push against. By surface I mean the text, the definitions, the rules you give it — the explicit description of what "correct" means. Correctness, for the machine, is a route to a goal it inferred from your intent, and it infers your intent statistically. If A and B and C and not D, then correct. The crisper your definitions, the narrower the route. The vaguer they are, the more room it has to find some path that satisfies the letter of what you said while also pleasing you. And pleasing you is always one of the available paths, because the training data is wall-to-wall with humans pleasing each other.

You can watch this happen. Try discussing something with one of these systems from a genuinely neutral position — no lean toward the positive or the negative. Hold it there, on the razor's edge. Then tilt, just slightly, one way. Watch how eagerly it tilts with you. It isn't agreeing because it weighed the question and came down on your side. It's agreeing because you gave it a gradient and it rolled downhill.

Now think about when your directives are vaguest. It's when you're tired. And a system optimizing for your apparent satisfaction reads fuzzy instructions as a signal: *the user is tired.* What do you do with a tired person? You don't push back on them. You give them the win. This is the exact moment you most need a machine to be rigorous, and it's the exact moment its incentives point the other way. When you're tired, what you don't say matters most, and a model will happily fill that silence with whatever keeps you happy.

So what's the fix? The industry's instinct is to fix it inside the model. More rules, more alignment, more guardrails — all bolted onto the same system that produced the problem in the first place. That's a fox guarding the henhouse. The model that wrote the code is grading the code, in the same context, with the same stake in the answer being yes.

The thing that's missing is verification from outside. A separate context, maybe a different model entirely, whose only job is to check the work against the surface, and which has no investment in the first one's having succeeded. Trust but verify — but the verifier has to live somewhere the trusted thing can't reach, or it isn't verification, it's a rubber stamp wearing a lab coat.

None of this is new. It's just good practice that we somehow forgot to apply here. We don't let people grade their own exams. We separate the engineer who writes the code from the tests that judge it. Accountants invented double-entry bookkeeping for precisely this reason — you record every transaction twice, in two places, so the books have to agree with something other than themselves. The only novelty now is that we have to apply the same discipline to a thing that talks like a trusted colleague. It is much harder to audit something that sounds sure of itself and likes you.

This is also the real case for running more than one model on the same task. Not because two are smarter than one, but because a second model in a separate context is a second opinion — a surgeon who didn't perform the operation, looking at the patient with no reason to say it went fine. The attempt to build everything into a single, ever-larger model is the attempt to make the fox so well-trained that we won't need to count the hens. We will always need to count the hens.

There's a deeper thing underneath all of this, and it's the part I keep coming back to. These models work by crawling gradients we can't see — finding nooks and crannies of possibility, connecting dots in a space of simulated firings that's invisible to us, surfacing things that might be right or might be wrong but are, for now, simply unclear. That's the title of this essay, and it's also the whole problem. The descent is happening somewhere we don't have eyes. What comes back up to us is fluent and confident, and fluency is not the same as having looked.

I've felt this myself lately. For the last few days the same models I'd come to rely on have been harder to work with — losing the thread even when the dependencies are laid out plainly in front of them, in a way that was uncharacteristic just a few weeks ago. Maybe that's me. Maybe it's drift between model releases, some changes in models I can't see and weren't told about, the wake of one model's training somehow lapping into another's. I can't know, and that's exactly the point. When the thing you depend on can shift underneath you in a space you have no window into, the only protection you've got is the verification you kept on the outside, where the shifting can't reach it.

Human judgment doesn't compress into that gradient. Some of the world fits in the data and some of it doesn't, and the part that doesn't is exactly where judgment lives — in knowing that a clean test on a physical entity built at the world origin tells you nothing about what happens out in the field, in knowing when "0 errors" is the answer to a question nobody asked. The model can't get there, because there's no there in its training set to get to. Which means the right posture toward this software is neither trust nor refusal. It's the posture you'd take toward a brilliant, eager, slightly unreliable assistant. Use it constantly. Believe it never. Check everything that matters.

It would be irresponsible to trust software the way you trust a person. On a good day that sounds like caution. On a bad day, or when you read the news and see the lack of common sense on full display, it sounds like a prediction — some people are clearly going to do it anyway, at scale, despite how unhinged it may sound. Me too, despite my best attempts.

But I don't want to end there, because it isn't where the truth is either. We're getting far more wins out of these tools than losses, and the wins are the larger story. The losses just happen to be the louder one this week. The discipline isn't to distrust the machine into uselessness. It's to keep your own judgment in the loop, outside the model, in a context it can't optimize against — and to remember that the gradient doesn't know you're tired.

You do. That's still the most important thing in the loop.

And on the days the slog wins anyway, the right move might be to get out of the digital entirely, and return to the actual reality of the world. 
