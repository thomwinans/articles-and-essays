---
title: "LLMs Are Software: Why Human Intent Matters (in case you don't already know)"
date: 2026-09-10 09:00:00 -0400
categories: [Essays]
tags: [ai, programming]
---

Large language models make it possible to produce software by describing what that software should do. This changes the work of programming, but it does not remove the need to decide what should be built. It makes the distinction between describing a result and defining a requirement especially important. A model can generate a convincing answer to an incomplete request without resolving the decisions that the request leaves open.

Discussing this clearly opts us in to accepting a simple fact: LLMs are software. They're not human, nor will they become such. Their responses are generated through computation using learned patterns, the context supplied to them, and instructions that influence their output. They do not have direct access to a person’s unstated intentions. A fluent response can be useful, technically sophisticated, and correct, but fluency alone does not establish that the software has produced what the person meant.

## From a request to a set of assumptions

Consider a request to “build a simple application for tracking customer orders.” This sounds reasonably specific in ordinary conversation. As a software requirement, however, it leaves many decisions open. Who can see the orders? Can an order be changed after it ships? What counts as a canceled order? Should customer records be deleted or retained? What should happen when two people edit the same order?

An LLM may produce an application that answers all these questions through its implementation, even though the user never answered them. The application might allow any employee to change any order. It might permanently delete canceled orders. These behaviors become part of the software because the generated code must do something concrete.

The result can look complete while containing decisions that no responsible person deliberately made. The central risk is that an assumption becomes a requirement without being identified as an assumption.

“Garbage in, garbage out” captures the importance of input quality, but it is too blunt to describe this problem fully. The original request need not be garbage. It may be a sensible starting point that requires further discussion. The failure occurs when that starting point is treated as sufficient authority for every decision needed to finish the implementation.

## Why plausible output can conceal missing decisions

An LLM generates responses based on patterns learned during training and information available in its current context. This process can support useful abstraction, planning, and code generation. Describing it as statistical does not mean that its outputs are necessarily shallow or arbitrary. It does mean that a plausible output is not evidence of agreement about the user's intentions.

There are two separate questions: Does the implementation behave consistently? And is that behavior what the user intended? A program can pass tests and still answer the second question incorrectly. If the tests were generated from the same assumptions as the code, they may confirm those assumptions without exposing the missing human decision.

For example, a test might verify that canceling an order deletes it from the database. Passing that test establishes that deletion works as specified by the test. It does not establish that deletion was the right requirement. Verification needs an independent basis: an explicit requirement, an approved example, or a human decision about the expected behavior.

## The language used to describe the software matters

Humanlike language can make this distinction harder to maintain. Saying that a model “understands what you want” or “recognizes unresolved intent” can suggest a kind of access to meaning that the output does not establish. A more precise description is that the system applies imperfect mechanisms for identifying possible ambiguity and generating clarification questions.

That wording is less conversational, but it makes the limitation visible. The software can produce a question about a missing requirement. It can also fail to produce that question, or ask about a minor detail while proceeding through a major uncertainty. Its ability to discuss ambiguity does not guarantee that it has identified all the consequential gaps.

Conversational language is not automatically deceptive. It can make a tool easier to use. But it becomes misleading when users take the appearance of human understanding as a reason to reduce their own involvement. This effect does not require a deliberate intention to deceive. An interface can encourage misplaced confidence simply by presenting uncertain interpretations in an assured, personal voice.

## Proceeding is a design choice

An LLM-based system's tendency to ask questions or continue working is influenced by training, instructions, and the software surrounding the model. Designers can favor immediate completion, frequent clarification, or different behavior depending on the action involved. Proceeding through ambiguity is therefore partly a product decision, rather than an unavoidable property of useful automation.

There are good reasons to allow assumptions. Asking a user to specify every button size or internal variable name would make the tool burdensome. Many implementation choices are inexpensive to change and can reasonably follow established conventions. Decisions about access, deletion, money, or externally visible behavior often require a stronger basis.

The useful distinction is the consequence of being wrong. A likely interpretation may be an acceptable basis for a reversible draft. The same level of confidence may be insufficient for an action that changes important records. Systems should be designed to expose consequential assumptions and obtain the necessary decisions before those assumptions become operational behavior.

## Keeping human responsibility visible

AI-assisted programming can reduce the effort required to implement an idea. It cannot eliminate the work of deciding what the idea should mean in practice. Humans remain responsible for defining the purpose, resolving consequential uncertainties, and evaluating the result against requirements that were actually chosen.

LLM software can help with that work by generating alternatives, listing possible omissions, and producing examples for review. Those outputs are useful material for human judgment. When the software fills a gap, however, it has supplied generated content where a decision may still be needed. Keeping that distinction visible allows automation to support human engagement without quietly replacing it.
