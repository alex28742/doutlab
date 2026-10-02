# Evolution

This document describes how DoutLab's architecture arrived at its current shape, and why the
boundaries between its parts exist where they do. It is a history of *why*, not a changelog of
*what shipped when* — no dates or milestone sequencing are claimed beyond what the project's own
engineering documentation supports, and historical state, current architecture, and future direction
are kept explicitly distinct throughout.

## Where it started: AI News Aggregator

The project began as an AI News Aggregator with a straightforward premise: collect news from RSS
sources, process it with AI, and present readers with a single, AI-assisted feed. "AI News Aggregator"
remains the project's internal engineering identity today — it was not renamed — but the *public*
identity it presents to readers later became **DOUT / NEWS**, a deliberate separation between the
project's internal name and its public product brand that reflects a broader shift: AI became internal
technology and product capability, not the headline the public product leads with.

In its earliest shape, acquiring a source, extracting an article, enriching it with AI, and deciding
what to publish were steps of one fairly direct pipeline. That was a reasonable starting point for an
MVP, and it is also the shape that eventually had to change.

## Structured ingestion and processing

As the system matured, "fetch some RSS and run AI on it" grew real structure: source health tracking
and automatic recovery so a broken source does not need manual attention forever, deduplication so the
same story is not processed twice, and durable, auditable records of what was actually ingested and
processed rather than transient in-memory state. This was the first step away from "a script that
does news things" toward a system whose own operations could be inspected, trusted, and reasoned about
after the fact.

## Why Sources, Feed, and Articles became separate

Once ingestion, editorial composition, and article lifecycle were each doing real, independent work,
keeping them as one undifferentiated concept started costing more than it saved. Three distinct
questions had emerged, and each deserved its own owner:

- **Sources** answers "where does content come from, and is that source healthy?" — acquisition,
  health, and recovery, independent of any editorial decision about what to do with the result.
- **Feed** answers "what does a given reading experience actually consist of?" — the editorial
  boundary of sources, topics, and selection, independent of how that content was acquired or what
  happens to an individual article afterward.
- **Articles** answers "what is the lifecycle of one piece of content?" — from extracted text, through
  AI enrichment, to the editorial decision that makes it reader-visible.

Separating these meant a change to how a source is acquired no longer risks the editorial feed logic,
and a change to editorial feed composition no longer risks how an individual article's AI enrichment is
applied. Each boundary tracks a genuinely different kind of change.

## Why Agent Identity and Intelligence Execution became separate

A similar pressure showed up on the AI side. Early on, "what an AI agent is" and "what task it performs
right now" were not clearly distinguished — identity and execution evolved together because nothing
yet required them not to. That stopped scaling once the project needed to reuse the same identity
across different tasks, and to improve how a task executes without touching what any agent actually is.
Splitting the two let an agent's character, background, analytical stance, and voice be defined once
and reused everywhere that agent appears, while the mechanics of running a task — the structured
context given to it, the structured result it produces, provenance, and auditability — evolved as its
own concern underneath.

## Why orchestration became its own layer: Scenario

The earliest pipeline directly coupled trigger handling, AI execution, article mutation, and
publication in one path. That coupling worked while the system did one thing, but it became a real
constraint once the project needed to compose new automated behavior safely: a pipeline that already
knows how to mutate an article and decide what gets published is a pipeline that is hard to extend
without risking the parts that already work, and hard to reason about independently of the one task it
was originally built for.

The resolution was to introduce Scenario as a dedicated orchestration boundary — one that connects a
trigger, an ordered sequence of already-trusted domain capabilities, and a durable execution history,
while deliberately knowing nothing about how any invoked capability works internally. Each domain
exposes its own capability as a stable, versioned, opaque call; Scenario's only job is sequencing and
auditability. This is why Scenario is described as an orchestration layer rather than a generic
workflow engine: it was built to compose *existing, independently correct* capabilities safely, not to
become a place where new business logic accumulates.

## Reusable platform capabilities emerge

As more of the system needed to call a language model, more of it needed a consistent staff editing
experience, and more of it needed a shared internal operator shell, those concerns were extracted into
their own platform infrastructure — Core LLM, the Entity Editor framework, and the Staff Console shell —
rather than being solved again inside whichever domain or product needed them first. This is the
architectural signal that the project had become more than one product's implementation detail: shared
infrastructure is only worth extracting once more than one thing genuinely depends on it.

## From one product to a platform

With acquisition, editorial composition, article lifecycle, AI identity, AI execution, and
orchestration each standing as their own domain — and language-model infrastructure, staff editing, and
the operator shell standing as their own reusable platform modules — what had been one application
became a foundation multiple products can be built on. DOUT / NEWS, the public news-reading product, is
the first and currently the primary product on that foundation; the DoutLab corporate site is a second,
independent product sharing the same underlying conventions. This is the current state of the platform:
**DoutLab** names the platform and company; DOUT / NEWS is one product built on it, not a synonym for
it.

## Current state vs. future direction

Everything above describes architecture that exists today. The natural next step this evolution points
toward — more products sharing the same domains and platform, a public API surface, and deeper
personalization — is real direction the project is working toward, not something already built. In
particular, a dedicated recommendation engine, a public mobile API or native applications, a durable
long-term agent-memory domain, user-owned or distilled custom agents, and a formal entitlement/tiering
architecture are all future direction, evidenced by the project's own roadmap, not current
architecture. See [../README.md](../README.md) and [architecture.md](architecture.md) for what is
actually implemented today.
