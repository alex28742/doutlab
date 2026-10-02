# DoutLab

> Public architecture and product showcase for DoutLab. Production source code is maintained in a
> private repository.

DoutLab is an AI-native platform for turning raw, high-volume information — starting with news — into
structured, trustworthy intelligence. It began as a single AI News Aggregator and has grown into a
reusable foundation of domains and infrastructure intended to support multiple products over time.

This document is the canonical source for DoutLab's public GitHub presence. It describes the current
architecture honestly: what exists today, what is still direction rather than delivery, and why the
production codebase itself stays private.

## Not just a news aggregator

The original idea — collect news, summarize it with AI, show it in one feed — is still true, but it is
no longer the whole picture. What used to be one monolithic pipeline has been decomposed into
independent domains (source acquisition, editorial feed composition, article lifecycle, AI agent
identity and execution, orchestration) sitting behind a small set of reusable platform infrastructure
modules. DOUT / NEWS — the public news-reading product ([news.doutlab.com](https://news.doutlab.com)) —
is the main product currently composing that foundation, not the foundation itself. See
[docs/evolution.md](docs/evolution.md) for how that separation happened and why.

## What DoutLab actually is today

Three ownership tiers, consistently applied across the codebase:

- **Products** — concrete, user-facing systems that compose the domains and platform capabilities
  below into one coherent experience. Today: **DOUT / NEWS**, the public AI-assisted news reading
  product and the main current consumer of the shared domain/platform stack, and the DoutLab corporate
  site, a separate product surface that owns the shared visual shell other products extend but does
  not itself depend on the domain/platform stack.
- **Domains** — reusable business capability and policy, independent of any one product's UI:
  **Sources**, **Feed**, **Articles**, **Intelligence** (Agent Identity + Intelligence Execution), and
  **Scenario** (orchestration). Each owns one clear responsibility and exposes it as a stable
  capability, not as something another part of the system reaches into directly.
- **Platform** — reusable technical infrastructure that is neither product-specific nor a business
  domain: **Core LLM** (provider/model/credential/pricing infrastructure for LLM execution), a reusable
  staff **Entity Editor** framework, the shared **Staff Console** presentation shell internal operators
  use, **Retention / Cleanup** (reusable cleanup execution/scheduling infrastructure, while each
  resource's owning domain retains its own deletion-eligibility policy), and **System Observability**
  (a narrow, current diagnostic/system-status and historical-run inspection capability — not a general
  observability platform).

The normal dependency direction is **product → domain → platform**. Products consume domain and
platform capabilities; domains do not depend on any one product's presentation; platform infrastructure
does not depend on business policy. See [docs/architecture.md](docs/architecture.md) for how these
pieces compose and why the boundaries are drawn where they are.

## The domains, briefly

- **Sources** owns the source catalog, source health/recovery, and RSS/feed acquisition — turning a
  configured source into a durable, auditable batch of newly discovered articles.
- **Feed** owns the editorial boundary: what a reading experience actually consists of — source and
  topic membership, demand/selection, and the renderer-neutral ordering a product then displays. A
  `FeedProfile` is a durable information space, not a particular screen.
- **Articles** owns the lifecycle of one article from extracted content, through AI enrichment
  application, to the editorial decision that makes it reader-visible — three independently understood
  stages (content processing, application, publication) sharing one identity and one provenance
  pattern.
- **Intelligence** owns what an AI agent *is* (Agent Identity — background, character, analytical
  stance, voice, and how they compose into one coherent identity) separately from *how it executes a
  task* (Intelligence Execution — the action it performs, the context it is given, and the structured,
  auditable result it produces).
- **Scenario** is the orchestration layer: it connects a trigger, a sequence of registered domain
  capabilities, and a durable execution history into one repeatable, auditable run. It does not
  reimplement what any domain does — it calls each one as an opaque, versioned capability and lets that
  domain own its own internals.

**Core LLM** is the shared provider/model execution infrastructure underneath the parts of the system
that actually need to call a language model directly — today, that is **Intelligence**, which consumes
the Core LLM platform contract to execute an agent's task. Normalizing how any provider's model is
called, and tracking what that call costs, is kept entirely separate from what an agent is or what it
is asked to do. Other domains do not call Core LLM themselves; where they consume AI-derived output
(for example, Articles applying an Intelligence Result), they do so through their own accepted
domain-to-domain boundary, not by becoming Core LLM dependents.

## Why the production source stays private

DoutLab's production repository contains real operational configuration, in-progress product and
revenue decisions, and a large volume of implementation detail that is only meaningful with full
project context. Publishing the source directly would expose more than it would explain. This public
repository exists instead to communicate the architecture — its boundaries, its reasoning, and how it
is expected to grow — to an audience that will never need, and should not need, the production
internals to evaluate it.

## Current architecture vs. future direction

Everything described in [docs/architecture.md](docs/architecture.md) and the domain summaries above reflects
**current, implemented** structure, grounded in the project's own internal engineering documentation.
Some ideas that appear in DoutLab's own public-facing marketing copy or internal planning are
deliberately **not** claimed here as implemented, including:

- a dedicated recommendation engine;
- a public mobile API or native mobile applications;
- user-owned or operator-distilled custom agents;
- a durable, retrievable long-term "agent memory" domain;
- a formal entitlement/subscription (tiering) architecture;
- additional products beyond DOUT / NEWS and the corporate site.

These are real directions the project is working toward, not features already shipped. See
[docs/evolution.md](docs/evolution.md) for the historical arc and the current-vs-future distinction in
more detail.

## Learn more

- [docs/architecture.md](docs/architecture.md) — how the platform is organized conceptually, and why
  those boundaries matter as it grows.
- [docs/evolution.md](docs/evolution.md) — how DoutLab got here, from a single AI News Aggregator to
  its current foundation for multiple future products.
