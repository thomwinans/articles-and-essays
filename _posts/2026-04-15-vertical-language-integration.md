---
title: "Vertical Languages"
date: 2026-04-15 09:00:00 -0400
categories: [Essays]
tags: [ai, programming]
---

[This is a working paper.]

People talk about Swift as if Apple's great achievement was the language itself — the features, the syntax, the type system. Closures. Optionals. Value types. Protocols. Undoubtedly it is a great achievement. But I think portraying it without acknowledging the still greater achievement misrepresents.

The most important design decision in Swift wasn't any of its features. It was the decision to keep speaking fluent Objective-C.

This sounds backwards at first. But think about what would have happened if Apple had shipped all the same features and made them incompatible with Cocoa. The App Store in 2015 would still have been an Objective-C app store. Swift would be a curiosity some blog posts about on Tuesdays. The features were the reason developers noticed. The compatibility was the reason they could actually use it.

I want to argue that this — let's call it *vertical language integration* — is the single most underrated idea in language design. I also want to argue that it's about to matter more than ever, because of LLMs. But to get there we have to look carefully at what Apple actually did, and at what most other language designers did instead.

Swift didn't appear out of nowhere in 2014. Chris Lattner started LLVM as a research project at the University of Illinois around 2000, joined Apple in 2005, and spent years building Clang as a modern replacement for GCC's stagnating Objective-C frontend.[^1] By the time Swift started in the summer of 2010, Apple's compiler team already owned the toolchain end to end. They could add a new language on top of Clang without asking anyone's permission. The thing that parsed Objective-C was already theirs.

This is the necessary condition for everything else. Swift and Objective-C share a frontend, an optimizer, a backend, and — by design — a memory model. Automatic Reference Counting, the feature that got rid of manual `retain` and `release` in Objective-C, landed in Clang in 2011. Three years before Swift shipped. Why? Because Lattner's team needed it first, for a language they hadn't announced yet. Here's how Lattner himself described it:

> "ARC came to Objective-C as part of Swift driving it, because we needed safety and memory management and we wanted to be able to talk to Objective-C frameworks... There was a reason: it was to make it so that when Swift came on the scene that we could do interoperability."[^2]

Read that again. They were modifying Objective-C *to prepare its successor*. Object literals, modules, ARC — all of it was groundwork. By the time Swift landed, the old language had been quietly reshaped to meet it.

And then the interop itself was absurdly clean. Bridging headers let a single `#import` line expose an entire Cocoa API to Swift.[^3] Generated headers went the other way, letting Objective-C call Swift.[^4] The Objective-C runtime kept working — `objc_msgSend`, key-value observing, the responder chain, all of it. Swift's runtime sat alongside, not on top. `NSString` became `String` and back again, transparently. You could pass a value across the boundary without thinking about it.

What this meant in practice was that on the day Swift shipped, every iOS developer already had a working standard library. Cocoa, Foundation, UIKit, AVFoundation, CoreData. Years of ecosystem. All of it available on launch day, because the new language hadn't replaced the old. It had climbed on top of it.

Apple has kept climbing. Swift-C++ interop became an official workgroup in 2022, and by 2023 the two languages could call each other both directions.[^5] Embedded Swift, announced at WWDC 2024, strips out reflection and existentials so the language can run on microcontrollers.[^6] Swift keeps extending downward into lower-level hosts, never replacing them. The pattern is consistent. Start from the top. Reach down. Never demand that the old ecosystem die.

Now compare this to what everyone else has been doing.

Flutter went the other way. Google's Flutter team asked the operating system for a single window and painted everything inside it themselves, first with Skia and now with Impeller.[^7] This gave Flutter beautiful consistency across platforms. It also meant they had to rebuild, from scratch, every scrollbar, every text field, every keyboard animation, every platform convention users had ever internalized. Eight years in, the ecosystem is real. But it's still catching up on things that iOS developers get for free by inheriting. Flutter chose horizontal replacement, and the bill came due in ecosystem time.

Xamarin went halfway and died. Microsoft's bet on bridging .NET into mobile worked, technically. Adoption never quite caught on. When React Native and Flutter arrived, Xamarin got squeezed from above and below, and Microsoft ended Xamarin support on May 1, 2024.[^8] Its successor MAUI inherited the brand damage.[^9] A bridge is not the same as vertical integration. Bridges wear out.

Rust is the hardest case, because Rust is a genuinely better language than C++ by almost any technical measure. But Rust started at the bottom instead of at the top, and climbing up into an existing C++ codebase turns out to be grueling work. `bindgen` autogenerates FFI but requires deep knowledge of C++ ABI; a function returning `std::unique_ptr` can segfault because its ABI differs from a raw pointer.[^10] The `cxx` crate bridges high-level concepts but requires hand-written bridge modules.[^11] The Chromium team had to invent `autocxx` on top of `cxx` because, in their words, "Chrome doesn't want to specify a cxx::bridge section for every API" — and they note that without a better solution, Rust is confined to "leaf nodes," "calling into question whether the costs of an extra language are justified."[^12] Firefox's Oxidation project took nearly a decade to meaningfully Rustify.[^13] Rust's problem isn't its design. Rust's problem is that it has no Apple — no single owner of a compiler stack patient enough to spend five years preparing the host to receive it.

The positive counterparts tell the same story in reverse. Kotlin is the cleanest example. JetBrains designed it to compile to JVM bytecode with near-trivial two-way interop — Java calls Kotlin, Kotlin calls Java, a team can migrate one file at a time.[^14] Google made Kotlin a first-class Android language in 2017 and Kotlin-first in 2019. In the Snyk JVM Ecosystem Report for 2018, Kotlin jumped from 11.4% to 28.8% of JVM language share in a single year, while Scala slipped from 28.4% to 21.6%.[^15] Kotlin ate Scala's lunch. Not because Kotlin was a more elegant language — Scala is arguably more elegant — but because calling Kotlin from Java was easy, and calling Scala from Java wasn't. That one asymmetry flipped the market.

Julia is another quiet win. Julia was pitched as the solution to the "two-language problem" — prototype in Python, rewrite hot loops in C. In practice what made Julia useful fast was `PyCall.jl` and `PythonCall.jl`, which let Julia import Python modules with automatic type conversion.[^16] Julia didn't have to reinvent SciPy. It stood on top of it.

There's an old Richard Gabriel essay called "Worse Is Better," written around 1989, that keeps echoing through computing.[^17] Gabriel's thesis was that Unix and C beat MIT-style "Right Thing" systems because they preserved simplicity at the cost of correctness and completeness. They were easier to port. Easier to adapt. Easier to grow into. The ugly, compromised system spread. The beautiful one didn't.

Vertical integration is a specific flavor of this. The new language is deliberately *worse* than a clean-break design would be, because it has to keep speaking the old language's protocols, respecting the old memory model, running on the old runtime. That compromise is what lets it win. A language that refuses to compromise on purity gets exactly the audience that cares about purity, which is smaller than its designers think. A language that compromises on purity inherits the whole installed base of its host, and can use that base as leverage to introduce its better ideas one at a time.

Lattner has said this was never just a technical argument. It was an institutional one. The executive objection inside Apple was, in his own telling, "Swift is risky. This could fragment the community. It might not be any good. You all seem well-intentioned, but this is actually hard — look at all the languages that have come and failed before you."[^2] The interop story wasn't a nicety. It was the political precondition for the project existing at all. A Swift that couldn't call Objective-C wouldn't have shipped.

This is where most language efforts actually die. Not on the technical merits, but on the question: can I introduce this without burning everyone's existing code to the ground? If the answer is no, the project doesn't happen, or it happens and fails quietly. If the answer is yes, the project still might fail. But at least it has a chance.

So far, none of this is new. People have known vertical integration works for decades. Why bring it up now?

Because LLMs change the economics.

The classic objection to vertical integration has always been that it's expensive. Building and maintaining FFI glue is miserable work. Someone has to write the bridging headers. Someone has to sort out reference counting across the boundary. Someone has to keep both sides of every translated type in sync, forever. The reason most new languages don't do what Swift did is that doing what Swift did costs a fortune, and most language teams can't afford it.

LLMs make this cheap.

If an LLM can fluently generate code in a new high-level language *and* drop down to the host language when the high language is deficient, the traditional "standard library coverage" problem mostly evaporates. The model writes the glue on demand. Missing a binding? The model synthesizes it. Need to drop to C for a hot loop? The model writes the C and the glue. The per-call cost of interop approaches zero, which means the marginal cost of *not* having a mature standard library approaches zero too — as long as there's a mature host to fall through to.

This isn't hypothetical. Mojo, the language Chris Lattner and Tim Davis started at Modular in 2023, is the explicit bet on exactly this. Mojo is designed as a strict superset of Python. Existing Python code runs as Mojo. Mojo can drop down to MLIR and LLVM for systems-level performance on CPUs, GPUs, TPUs, and custom accelerators.[^18] The design goal is transparently the Swift-Objective-C lesson reapplied one layer up the stack. Inherit the entire Python ecosystem. Add a lower tier for performance. Let developers mix modes in the same file.

Jeremy Howard called Mojo "the biggest programming language advance in decades" on the day it launched.[^19] Whether or not that holds up, notice the form of Howard's argument. Python is the ergonomic top tier. C++ and CUDA are the performant bottom tier. The "two-language problem" in machine learning is only solvable if one language can credibly occupy both tiers — which is not a bridge, it's vertical integration. Mojo is the bet.

Here's the part I think most people aren't seeing yet.

In an LLM world, the top tier of a new language can be much weirder than was historically practical. Much more opinionated. Much more domain-specific. A language doesn't need a decade of Stack Overflow to become usable anymore. The model reads the spec and a small corpus of hand-written examples, and does the rest. That's a new kind of freedom, and it changes what's worth building.

But there's a catch, and it's the whole point. The bottom tier — the host-language drop-through — is effectively free to *use*, because the LLM writes the glue. But it's only free if the interop itself is clean. If types don't round-trip, if memory models disagree, if calling conventions don't line up, then the LLM spends most of its time debugging its own glue, and the experience degrades to something worse than just writing in the host directly.

So the language that wins the LLM era won't be the one that's easiest for an LLM to write in isolation. It'll be the one whose host-language drop-through is cleanest. Because an LLM is going to spend half its time writing FFI, and the friction of that FFI will dominate everything else.

This is the Swift lesson, sharpened. Apple spent five years quietly reshaping Objective-C to be ready for Swift, and then shipped a language that speaks its predecessor perfectly. Most language projects don't have that kind of patience, or that kind of institutional control. They should try to get some. The language most people will end up writing in five years is the one whose designers, today, are doing the boring, compounding, expensive work of making it speak fluent Python, or fluent C, or fluent whatever-is-already-there.

The best new language in 2030 will be half an old one. That will turn out to have been the whole point.

---

## References

[^1]: Background on Chris Lattner and LLVM: [Chris Lattner — Wikipedia](https://en.wikipedia.org/wiki/Chris_Lattner). Recent interview retrospective: Pragmatic Engineer, ["From Swift to Mojo and high-performance AI Engineering with Chris Lattner"](https://newsletter.pragmaticengineer.com/p/from-swift-to-mojo-and-high-performance). Original AppleInsider reporting on Swift's origins: ["Apple's top-secret Swift language grew from work to sustain Objective-C, which it now aims to replace"](https://appleinsider.com/articles/14/06/04/apples-top-secret-swift-language-grew-from-work-to-sustain-objective-c-which-it-now-aims-to-replace), June 4, 2014.

[^2]: Chris Lattner, interviewed by Paul Hudson on *Hacking with Swift*, June 18, 2020: ["Could Apple have improved Objective-C instead of making Swift?"](https://www.hackingwithswift.com/interviews/chris-lattner-could-apple-have-improved-objective-c-instead-of-making-swift). Both the ARC quote and the "Swift is risky" recollection are from this interview.

[^3]: Apple Developer Documentation, ["Importing Objective-C into Swift"](https://developer.apple.com/documentation/swift/importing-objective-c-into-swift).

[^4]: Apple Developer Documentation, ["Importing Swift into Objective-C"](https://developer.apple.com/documentation/swift/importing-swift-into-objective-c).

[^5]: Swift.org, ["C++ Interoperability Workgroup"](https://www.swift.org/cxx-interop-workgroup/); Swift Forums, ["Swift and C++ Interoperability Workgroup Announcement"](https://forums.swift.org/t/swift-and-c-interoperability-workgroup-announcement/54998).

[^6]: Swift.org, ["Get started with Embedded Swift"](https://www.swift.org/get-started/embedded/); InfoQ, ["Apple Goes Embedded with Embedded Swift"](https://www.infoq.com/news/2024/07/embedded-swift/), July 2024.

[^7]: Flutter, ["Architectural overview"](https://docs.flutter.dev/resources/architectural-overview). For background on the Skia-to-Impeller transition, see Ayaan Haider, ["Impeller vs Skia: How Flutter's New Renderer Changes Everything"](https://medium.com/@ayaanhaider.dev/impeller-vs-skia-how-flutters-new-renderer-changes-everything-189ee5102bef) — a solo-author overview, fine for color only.

[^8]: InfoWorld, ["Xamarin Forms reaches end of life: What should you do?"](https://www.infoworld.com/article/2336827/xamarin-forms-reaches-end-of-life-what-should-you-do.html).

[^9]: In The Pocket, ["Xamarin is dead, long live MAUI?"](https://www.inthepocket.com/blog/xamarin-is-dead-long-live-maui).

[^10]: eshard, ["A tour of Rust/C++ interoperability"](https://eshard.com/posts/Rust-Cxx-interop).

[^11]: cxx.rs, ["Context"](https://cxx.rs/context.html).

[^12]: Chromium project, ["Rust and C++ interoperability"](https://www.chromium.org/Home/chromium-security/memory-safety/rust-and-c-interoperability/).

[^13]: Manish Goregaokar, ["Integrating Rust and C++ in Firefox"](https://manishearth.github.io/blog/2021/02/22/integrating-rust-and-c-plus-plus-in-firefox/).

[^14]: Kotlin, ["Calling Java from Kotlin"](https://kotlinlang.org/docs/java-interop.html); Android Developers, ["Kotlin-Java interop guide"](https://developer.android.com/kotlin/interop).

[^15]: Snyk, ["JVM Ecosystem Report 2018"](https://snyk.io/blog/jvm-ecosystem-report-2018/) (5,160 respondents): Scala fell from 28.4% to 21.6% of JVM language share year over year; Kotlin rose from 11.4% to 28.8%. See also Snyk, ["Kotlin overtakes Scala and Clojure, becoming the 2nd most popular language on the JVM ecosystem"](https://snyk.io/blog/kotlin-overtakes-scala-and-clojure-to-become-the-2nd-most-popular-language-on-the-jvm/). On Scala's interop asymmetry specifically, see John De Goes, ["Scala Resurrection"](https://degoes.net/articles/scala-resurrection). Background: [Scala (programming language) — Wikipedia](https://en.wikipedia.org/wiki/Scala_(programming_language)).

[^16]: [JuliaPy/PyCall.jl](https://github.com/JuliaPy/PyCall.jl); arXiv, ["Bridging Julia and Python"](https://arxiv.org/html/2404.18170v1).

[^17]: Richard P. Gabriel, ["Worse Is Better"](https://www.dreamsongs.com/WorseIsBetter.html), 1989–1991. See also [*Worse is better* — Wikipedia](https://en.wikipedia.org/wiki/Worse_is_better).

[^18]: [Mojo (programming language) — Wikipedia](https://en.wikipedia.org/wiki/Mojo_(programming_language)); Modular, ["Why Mojo?"](https://docs.modular.com/mojo/why-mojo/); Modular, ["Developer Voices: Deep Dive with Chris Lattner on Mojo"](https://www.modular.com/blog/developer-voices-deep-dive-with-chris-lattner-on-mojo).

[^19]: Jeremy Howard, ["Mojo may be the biggest programming language advance in decades"](https://www.fast.ai/posts/2023-05-03-mojo-launch.html), fast.ai, May 3, 2023.

## Further reading

These sources informed the essay but are not cited inline.

Developers Slashdot (2017). ["Slashdot's Interview With Swift Creator Chris Lattner."](https://developers.slashdot.org/story/17/01/23/085232/slashdots-interview-with-swift-creator-chris-lattner)

ATP Podcast Episode 205. ["Chris Lattner Interview Transcript."](https://atp.fm/205-chris-lattner-interview-transcript)

Apple historical transitions — background on Apple's long pattern of bridged language/OS migrations: [Carbon (API) — Wikipedia](https://en.wikipedia.org/wiki/Carbon_(API)), [NeXTSTEP — Wikipedia](https://en.wikipedia.org/wiki/NeXTSTEP).
