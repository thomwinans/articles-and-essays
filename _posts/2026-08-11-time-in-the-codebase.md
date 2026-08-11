---
title: "Time in the Codebase"
date: 2026-08-11 09:00:00 -0400
categories: [Essays]
tags: [ai, programming, testing]
---

Build a language runtime on top of a managed platform and you get two surfaces instead of one. There's the language itself — in a mature one, close to a thousand names in the public interface, most with behavior that varies by argument type, by keyword, by declaration, by whether the code was interpreted or compiled, and by the ambient state of a dozen dynamic variables. Every one of those axes is a place a defect can hide. Underneath sits the host platform's own semantics for characters, numbers, exceptions, and threads, and the two surfaces don't agree at every point.

The obvious response is more tests. That response is correct and insufficient, and working out why it's insufficient changes two things you thought you understood: what a test is for, and what a bug is.

Start with the division of labor, because the intuitive one is backwards.

The tempting way to use a language model here is as an authority. What does this function return when you hand it that combination of flags? The model will answer fluently and will sometimes be wrong. A wrong answer in conversation costs nothing. A wrong answer that becomes an assertion in a test suite is worse than no test at all, because you'll then modify the implementation to satisfy a hallucination. You'll have encoded a fiction into the one artifact whose whole purpose is to hold you to the truth.

So invert it. The model proposes cases; a reference implementation adjudicates results. Generate candidate expressions, run them under an oracle that actually executes code, record what comes back — the value, or the error type if it fails — and assert that your implementation matches. The model's contribution becomes case selection, which it's genuinely good at, and the oracle role stays with something that runs.

The general form of the rule is: never let a model occupy the position of ground truth when a cheaper ground truth exists. Testing just makes the rule concrete, because in testing the cheaper ground truth is a program you can run.

We're good at writing tests that show the thing works. We write them right after building the thing, while we still know how it's supposed to be used. That knowledge is exactly what makes the test weak. It traces the path the author had in mind. The path is real. It's also one of an unbounded family.

The negative side is worse, and worse in a way that resists the usual remedy. You can't enumerate the ways a function can be misused, because the space of wrong inputs isn't another path. It's infinite in a different direction. Adding negative cases one at a time isn't slow progress toward coverage. It isn't progress toward coverage at all.

Two moves help, and neither one is "write more cases."

The first is to collapse the negative space into a property. Instead of enumerating bad inputs, state something universal that has to hold across all of them. For a language hosted on a managed runtime, the strongest such property is that for any input whatsoever, evaluation ends in a value of the language or an error of a documented type, and no host-runtime exception ever escapes to the user. A null-reference exception surfacing from the evaluator is a bug no matter how deranged the input was. That property needs no oracle at all. A fuzzer feeding random programs tests the entire negative space against it at once, and it targets exactly the failures that a conformance suite written decades ago, for implementations written in C, will never check.

The second is to respect the specification's modal taxonomy. A good specification doesn't offer a binary between "works" and "errors." It distinguishes at least four things: an error is signaled, an error should be signaled, the consequences are undefined, and it is an error but nobody is required to detect it. A generated negative test that demands a type error where the standard says the consequences are undefined produces a permanent false failure. Worse, it tempts you into adding a runtime check whose cost you'll carry forever.

So every negative test should carry the specification section and the literal normative phrase, tagged by category. Tests in the undefined bucket then stop being conformance assertions and become something more useful — documentation of a choice the implementation has made. Without that, the choices drift, and they drift silently.

There's a comforting story that says the coverage problem solves itself. Build enough with the system and the untested paths get walked.

Partly true. But the person building with the system is the person who knows where the soft spots are, and that knowledge operates below conscious attention. You reach for the function you trust. You avoid the argument combination that blew up last month. You write the idiom that works. The traversal concentrates in the region that's already hardened, and the dormant defects stay dormant because your hands learned to route around them. Organic use is a random walk with a drift toward the safe.

It's also a detector for loud bugs only. Use finds the failures that announce themselves within a few frames of the corruption. The dangerous class is quiet: a name table entry aliased wrong, a shared structure mutated where a copy was intended, a return value silently truncated, a hash table left unrehashed after the hash basis of one of its keys changed. None of those throw. They wait until something three layers away reads the corrupted state, at which point the backtrace describes the victim and says nothing about the assailant. More usage doesn't convert quiet bugs into loud ones. It accumulates more sites at which the quiet damage has already occurred.

Both problems share a fix, and it isn't more tests. It's making every path self-checking, so that traversal becomes detection regardless of what the traversing code was trying to accomplish. An implementation that owns its own function factories, macro machinery, and loader has unusual leverage here. A paranoid mode can validate structural invariants on entry to and exit from every builtin — name and namespace coherence, type tags on cells, discipline on how multiple return values are passed, handler and unwind stack depths matching dynamic extent. It runs at a crawl. It also runs the whole conformance suite and every application you've built on top of it. And detection stops depending on some test happening to assert the exact thing that broke.

Alongside that, differential testing against yourself: interpreted against compiled, checks off against checks on, fully expanded then run against run directly. Same input, three paths, and the results have to agree. No external oracle required, and it explores exactly the region where one implementation's internal paths diverge from each other.

Which brings up the use of a model that's easiest to miss and probably the most valuable.

Not "generate a test for this function." Instead: here's the test corpus and here's the source tree — what regions of the practice does nobody here touch?

That's a statistical query rather than an inferential one, and it plays to what a language model actually is. It has read thousands of test suites. It knows the shape of the distribution. Asking it to diff your suite against that distribution and report the holes is asking it to do the one thing its architecture is built for.

The answer lands hardest when it names a technique rather than a case, because a technique is a generator. "You have no tests that re-run an expression after redefining a function it calls" is worth more than a hundred generated assertions. It opens a whole territory. What happens to a call that was inlined? To a method after the signature of the operation it specializes changes? To a record accessor after the record is redefined? To a compiled closure over a macro that has since been altered? Specifications are deliberately vague across most of that region, which is precisely why nobody tests it and precisely why it rots without anyone noticing.

For a hosted implementation there's a sharper version, because the corpus the model has read is overwhelmingly tests for natively compiled implementations, and their absences are structural rather than accidental. A native implementation has no reason to test its interaction with a garbage collector it doesn't own, or class-loading order, or exception unwinding across a host boundary, or weak-reference semantics under a foreign collector. So the productive query is a two-way diff. What does the language's testing tradition test that you don't, and what does the managed-runtime testing tradition test that neither of them does?

The caution is the failure mode of the same technique. A model will also produce plausible absences that aren't absences — practices imported from other language cultures that don't map, or gaps that are gaps because the thing genuinely doesn't apply. Treat the output as a list of territories to examine, ordered by your own judgment about whether the architecture makes each one load-bearing. The statistics tell you where the practice is thin. Only you know which thin spots sit over something structural.

And the best of these nudges don't stay in the test suite. If nobody tests redefinition during execution and you go look, you may conclude that the system needs an explicit invalidation protocol rather than an accident. A feature falls out of a coverage gap. The property about host exceptions never escaping isn't really a test either. It's an architectural commitment about the boundary, which you then design toward.

All of which is prologue to the thing that reorganizes the rest.

Bootstrapping a language in itself requires that a minimal version of some function exist before the machinery for the complete version is available. A stripped-down macro facility runs in the loader until the real one can be defined. A hand-rolled record facility supports the records the type system itself requires, until the general one arrives. This is ordinary. Every self-hosting system does it, and three-stage compiler bootstraps are the canonical case.

The mind bug is what happens when someone reads the result statically. Two functions with overlapping extension look like duplication. The inference is locally sound — on the inputs both accept, they do behave identically — and globally wrong, because what distinguishes them isn't behavior. It's what infrastructure each one presupposes. The distinguishing fact is temporal, and a static reading has no way to represent it.

A compiler bootstrap hides this, because its stages live in separate build directories and then get thrown away. A system that keeps every stage alive in one running process doesn't. Every stage coexists in one namespace, indefinitely. A build-system fact becomes a permanent comprehension hazard — for a static analyzer, for a dead-code detector, for a model reading the tree, and for you in six months.

There's a second case that looks identical and isn't, and it matters more. A wrapper that absorbs runtime dynamism is not an earlier stage of the primitive it wraps. A primitive array accessor may assume a contiguous array, a known element type, an index already in bounds. The full accessor has to handle arrays that are views onto other arrays, arrays with a movable end, arrays that can be resized, and the possibility that the array was resized between the type check and the access. Both are correct. Both are permanent. The primitive stays fast, and the wrapper is where dynamism gets paid for. These never collapse into one function.

Whereas a bootstrap pair should eventually collapse, or at minimum the early definition should become unreachable once the system is built.

So there are two categories with identical appearance under a static reading. One is distinguished by its position in the build order, the other by the breadth of its contract. One is temporary and should end up unreachable; the other is permanent and should stay layered. Confusing them is how you either delete something structural or preserve dead scaffolding for years out of uncertainty.

The fix is to stop requiring the distinction to be inferred. Make it declared. Each such function records its category and what it presupposes. That's documentation, and — this is the part that connects back to everything above — it's also testable.

If the presuppositions are declared, then at the end of a build you can assert that nothing in the user-visible namespace still resolves to a stage-zero definition. If something does, either staging didn't complete or something captured a reference it shouldn't have: a closure or an inlined call that froze the primitive where the wrapper was intended. That failure would be silent. It would show up only on the dynamic paths the primitive doesn't handle. It's exactly the quiet, dormant shape described earlier, and without declared stages there's nothing to test against.

You can assert something else too. The presupposition graph has to be acyclic at build time, even though the call graph after the system is up may well be cyclic, and the build order has to be a valid topological sort of it. That catches bootstrap breakage at its origin rather than far downstream, when something receives a half-initialized value instead of an error.

So the problem looked like coverage — surface too large, tests too few. Three things displace that.

A test's value isn't proportional to the number of cases. The high-value artifacts are the ones that hold across unbounded input: universal properties, self-checking invariants, differential agreement between an implementation's own paths. Cases are the low-leverage end of the instrument.

The best use of a model in this loop is statistical, not inferential. Not "what does this return" but "what does the world test that we don't." Absence detection, delivered as techniques rather than cases, sorted by your own judgment of structural relevance. The moment a model occupies the oracle position, it becomes a liability with excellent grammar.

And a bug is not a defect at a point. It's a latent relation that becomes a defect at a time. Dormancy — the gap between corruption and manifestation — is what makes quiet bugs dangerous, and dormancy is a temporal property. Bootstrap staging is the same insight in constructive form: two definitions aren't duplicates when they occupy different moments. Both are failures of a static reading to represent time.

Once you see it, the synthesis isn't subtle. Nearly everything worth testing in a self-hosting system is a claim about when rather than what, and nearly everything that makes such a system hard to read is time flattened out of the picture. The remedy runs in both directions and it's the same remedy: make the temporal structure explicit, declared, and checkable, so that what would otherwise take inference becomes something a machine can verify.

A codebase is a still photograph of something that only exists in motion. Most of what we call reading one is guessing at the motion. The guessing is where the bugs live, and there's no rule saying you have to guess.
