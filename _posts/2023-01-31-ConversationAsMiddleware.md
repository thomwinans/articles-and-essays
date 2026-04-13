---
title: "Conversation as Middleware"
date: 2023-01-31
categories: [Essays]
tags: [technology, ai]
---

> [Download this essay as PDF](/assets/pdf/ConversationAsMiddleware.pdf)
{: .prompt-info }

# Conversation as Middleware

Here's something that should be obvious but isn't: the most powerful interface between software systems isn't an API. It's a conversation.

I don't mean that metaphorically. I mean that literally, right now, the thing that used to require months of integration work — getting System A to understand what System B is saying, transforming data formats, handling exceptions, mapping between vocabularies — can increasingly be done by talking to a machine in English. And that changes what middleware means.

Let me back up. If you've worked in enterprise software, you know what middleware is. It's the glue. It's the layer that sits between systems that weren't designed to talk to each other and makes them talk to each other anyway. Middleware standardizes data signals across platforms. It transforms formats. It routes messages. It triggers workflows when events happen. It's unglamorous, essential, and historically expensive — both to build and to maintain.

The reason it's expensive is that every integration is bespoke. System A stores dates as Unix timestamps. System B uses ISO 8601. System C has some proprietary format invented by a consultant in 2008 who has since left the company. The middleware developer's job is to write the transformations between all these formats, handle the edge cases, deal with failures gracefully, and keep the whole thing running when either system changes without warning. Which they always do.

So what happens when you throw an LLM at this problem? What does it do when you hand it data in one format and ask it to produce output in another? It just does it. You don't write transformation code. You don't build a mapping table. You describe what you want in natural language, and the model figures out the translation. The date thing? Trivial. But it scales to much harder problems. Translate between these two vocabularies. Map these customer records from CRM format to billing format. Take this unstructured support ticket and extract the structured fields we need for the bug tracking system.

This is middleware. It just doesn't look like middleware because nobody had to write any Java.

What makes LLMs genuinely different from previous approaches to this problem isn't raw capability. NLP tools have existed for decades. What's different is the conversational interface. Previous tools forced you into rigid, single-shot interactions. You submitted a query, got a result, and if the result was wrong, you started over. There was no context. No memory. No ability to say "no, I meant the other kind of customer ID" and have the system understand.

LLMs maintain conversational state. You can refine. You can clarify. You can say "actually, ignore the European records, I only want North American" and the model adjusts without you having to reformulate your entire request. This seems like a small thing if you've never tried to wrangle data at scale. It's a big thing if you have. In practice, data integration is an iterative process. You think you know what you want, you try to get it, you discover your assumptions were wrong, and you adjust. Conversations are the natural interface for iterative work. Forms and APIs are not.

There's a subtlety here that most people miss. When you have a conversation with an LLM about data, you're not just querying. You're shaping. You're saying things like "treat these two fields as equivalent" or "when the status is blank, default to active" or "flag anything that looks like a duplicate but don't delete it." These are business rules. In traditional middleware, they'd be encoded in configuration files or transformation scripts. In a conversational interface, they emerge naturally from dialogue. The business analyst who understands the rules can express them directly, without needing a developer to translate them into code.

What are the implications of this? They go way beyond data integration.

Think about how most enterprise software works today. You interact with it through forms. You click through screens. You fill in fields. You press submit. The form is designed to capture a specific set of data in a specific structure. It accommodates exactly one way of saying what you mean. If you want to say something the form designers didn't anticipate, you're out of luck. You type it into a "notes" field that nobody reads, or you don't say it at all.

Now imagine replacing that form with a conversation. Not a chatbot that follows a decision tree — those are just forms wearing a trench coat. A real conversation, backed by an LLM that understands context and can map what you're saying to the structured data the system needs. You say "I need to update the delivery address for the Henderson order, the one from last Tuesday, to their new warehouse on Fifth Street." The system understands which order you mean, what field needs to change, and what the new value should be. No form. No dropdown. No order number lookup. Just a sentence.

The interesting thing about this isn't the natural language understanding itself. It's that the conversation can accommodate ambiguity. In the form world, ambiguity is an error condition. The system rejects your input or forces you to choose from a list. In the conversation world, ambiguity is a prompt for clarification. The system asks "did you mean the Henderson order for 500 units or the one for 200?" This is how humans handle ambiguity. We ask. It's remarkable that software has taken this long to figure that out.

But here's where it gets really interesting. If conversation is the interface and LLMs are the engine, why does the conversation have to stay at the edge? Traditionally, chatbots and voice assistants live on the surface of the architecture. They're a skin. They capture input and hand it off to the real systems underneath. The LLM never touches the core. But there's no technical reason for this limitation. An LLM can sit in the middle of an architecture, mediating between systems the same way it mediates between humans and systems.

Picture this: System A emits an event — a new customer record was created. The event flows to an LLM-powered middleware layer. The LLM understands the event, transforms the data into the format System B needs, enriches it with information from System C, checks it against business rules expressed in natural language, and routes the result to the appropriate destination. If something is ambiguous or anomalous, it can flag a human for review — in a conversational interface, naturally — and incorporate the feedback into future processing.

This is what the frameworks are reaching toward. Tools like LangChain, LlamaIndex, and their descendants provide the plumbing to connect LLMs to external systems, manage prompts, maintain memory, compose tasks into workflows, and control costs. They're early and rough, but they're pointed in the right direction. They treat the LLM not as a chatbot but as a processing node — something that can be embedded in a pipeline, triggered by events, chained with other operations. They're building the infrastructure for conversational middleware.

The part that excites me most, though, isn't the plumbing. It's the potential for LLMs to interact with knowledge graphs. A knowledge graph captures entities and their relationships in a structured, queryable form. Imagine loading a knowledge graph into an LLM's context — not just the nodes and edges, but the derived relationships, the inference rules, the constraints. Now your conversational middleware doesn't just transform data. It reasons about it. It can say "this new customer record conflicts with an existing relationship in the graph" or "this transaction pattern matches a known fraud signature across three related entities." The combination of conversational fluidity and structured knowledge is extraordinarily powerful. Neither one alone gets you there. Together, they produce something that feels qualitatively different from what came before.

Is this practical today? There's a reasonable objection: LLMs are expensive and slow compared to traditional middleware. A message broker can route millions of events per second. An LLM takes seconds to process a single prompt. This is true, and it means LLM-powered middleware won't replace traditional middleware for high-throughput, well-defined transformations. If you know exactly what the data looks like and exactly what you want to do with it, write the code. It'll be faster and cheaper.

But most integration problems aren't like that. Most integration problems are messy. The data is inconsistent. The formats drift over time. The business rules change quarterly. The exception cases multiply until they outnumber the normal cases. These are the problems where traditional middleware becomes a maintenance nightmare and where a conversational approach starts to look not just viable but superior. You'd rather say "handle the new exception like we handled the last one" than write another hundred lines of transformation logic.

I think what's happening is a blurring of the line between structured and unstructured data. Traditional middleware assumed a clean distinction: structured data on one side, with schemas and types and validation rules, and unstructured data on the other, relegated to document stores and search indexes. LLMs don't respect this boundary. They process natural language and structured data with equal facility. They can read a paragraph and produce JSON. They can read JSON and produce a paragraph. This boundary dissolution is the real revolution, and conversation is the interface through which it happens.

The implications for how we build software are significant. If conversation can serve as middleware — if it can handle data acquisition, transformation, enrichment, routing, and exception management — then the architecture of enterprise applications starts to look different. The rigid, form-driven interfaces soften. The expensive, brittle integration layers become more adaptive. The barrier between "what the business person means" and "what the system does" gets thinner. Not to zero — there will always be hard engineering problems underneath — but thin enough that the translation layer stops being the bottleneck.

We're still early. The frameworks are immature. The models are expensive. The reliability isn't where it needs to be for mission-critical pipelines. But the direction is clear. Conversation is moving from the edge of our architectures toward the core. Not replacing what's there, but augmenting it — handling the messy, ambiguous, constantly-changing parts that traditional middleware was never very good at anyway.

The best middleware, it turns out, isn't the kind that forces every system to speak the same rigid language. It's the kind that's fluent enough to understand what each system is trying to say. And the most natural interface for fluency has always been conversation. We just didn't have machines that could hold up their end until now.
