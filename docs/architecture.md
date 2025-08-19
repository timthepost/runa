## Runa Architecture Overview

Runa is a micro kernel inspired platform for deploying and coordinating local
large language model experiences in a modular, private, and resilient fashion.

This document outlines the system architecture and how its components interact
at both a runtime and systems level.

Name resulted from needing "A runtime" and "Arun" just didn't roll off the
tongue so well.

### Component Breakdown

#### `gilgul`

- Runa's service manager and lifecycle orchestrator.
- Written in Rust, launching and supervising model services.
- Designed for resilience and health-aware orchestration.
- Currently in development (repo).

#### `splinter`

- Shared-memory message bus and KV store.
- Message routing layer with decay, observability, and component-level stats.
- Serves as the central coordination plane for all components.
- Currently in development (repo).
- Current focus:
  - Janitorial (automating TS bindings, doc gen)
  - Instrumentation and observability tools (a scope)
  - Encryption (payload), keys-by-hash scheme (plain text keys get hashed)
    before being saved, keys show up as one-way hashes a secret is required to
    create, and/or, All keys are UUIDs FTW! scheme. Could be on a per-bus basis
    and part of the metadata. This is needed for `coven`and `alethia` (see
    below).

#### `tieto`

- RAG server with cosine similarity over JSONL documents.
- Euclidean distance also calculated to narrow or filter results.
- Modes: Retrieval-only or full-completion (prompt + model eval).
- Frontmatter-aware with query filtering (`=`, `<=`, `>=`, `in`).
- Embedding support via CPU-friendly `llama.cpp` variant.
- Currently in development (repo).
- Current focus:
  - Proper-izing as a class and library
  - Adding binary indexing with goals of eventually supporting:
    - Vector quantization (e.g. IVF, HNSW, product quantization)
    - K-means clustering (coarse buckets)
    - Approximate nearest neighbor (ANN) search

#### `threadwell`

- Shoelace + SortableJS + Deno Universal Kanban Frontend For Runa implemented
  purely in HTML components + JavaScript with Deno for splinter message bus
  integration.
- Is being designed to handle prompt, personality and other data to use
  highly-capable 8B quantized models in very versatile ways, from corporate SOP
  agents to personal assistants to translators to transcribers to researchers:
  cards provide a great base for many useful things to begin in familiar, easy
  JS and HTML.
- Also provides Oak-based APIs to enable integrations with things like slack,
  discord, notion and other things that can take advantage of an API.
- Needed something that could "thread well" (as in be a front end for many
  things simultaneously) and the name stuck.
- Currently in development (repo link)
- Current focus:
  - Oak back-ends
  - Simple turn-based chat plugin

#### `tokenWhisper`

- Designed to run fast: inline in any LLM pipeline, or as a sidecar monitor.
- Token-level policy enforcement and monitoring.
- Uses a Token Impact Matrix to detect violations and guide control flows.
- Detects pattern-based violations (e.g., jailbreaks, policy circumvention).
- Can escalate to further checks or redirect response generation.
- Currently in design / research (docs link).

#### `coven`

- Secrets manager
- Trusts gilgul only; knows it because it starts gilgul (required to use Coven).
- gilgul enforces policy, coven decrypts on-the-fly and provides response.
- Uses separate bus shared only with gilgul.
- Gilgul monitors coven; multiple options on failure are possible.
- Currently in design / research (docs link).

#### `snarf`

- Passive personality mapper.
- Learns over time then adapts based on changes.
- Designing toward better safety for neurodivergent users.
- Currently in design / research (docs link).

#### `alethia`

- Greek for “truth,” also the act of unforgetting or bringing into light.
- Encrypted version of Tieto + indexing
- Can hold a very fast index of every book in a public library in just under
  256mb; decrypts and retrieves based on index matches.
- Decrypts matches on-demand and transmits them over a dedicated bus
- Specifically written for local LLMs to have an eidetic memory of every
  interaction they've had with their user + linked feedback from it that can be
  used to re-train or re-orient context.
- Is also a very capable document store, with planned addition of:
- 100% local, 100% encrypted, as easy to snapshot and rotate as Tieto.
- Available under dual license on request.
- Currently in design / research (docs link) with proof-of-concept emerging very
  soon!

## Inter-Component Protocol

See splinter base protocol docs for the current envelope protocol. Of course, it
is subject to change as encryption is increasingly supported.

## Deployment Targets

Designed with the idea of being the only consequential processes using a common
minimum Linux/GNU installation with some custom packages installed on top, but
can also be installed on top of a modern Debian/Fedora based system.

- **Minimum hardware**: Intel i3, 6GB RAM
- **OS**: Debian or Fedora (or derivatives)
- **Isolation options**: Xen is recommended (PV), QEMU/KVM work well, even a
  chroot can help. But it can also be used over any compatible OS as just
  another part of the OS without an air gap.

## Development Principles

There are some things that guide decisions about how Runa works, what future
needs are going to be, and what should be prioritized. Here they are:

- **Privacy-first**: No cloud unless you add one, no calls home or telemetry
  unless you add it, no ads ever.
- **Auditability**: Logs, bus messages, token-level context.
- **Modularity**: Replace any component. Nothing is tightly coupled.
- **Self-healing**: System should diagnose and repair itself whenever possible.

But the biggest one is last: _**Do. No. Harm.**_ We don't present illusions of
safety we can't back up with data. We have to optimize for the most vulnerable
of our users with something like Runa as safety and privacy are so very
important.

[Kim Crayton's Four Guiding Principles][1] help us make sure that we're staying
on course, and on vision:

- Tech is Not Neutral, Nor is it Apolitical,
- Intention without Accurately Informed Strategy is Chaos,
- Lack of Inclusion is a Risk/Crisis Management Issue,
- Prioritize the Most Vulnerable.

The test is, be able to explain how we embrace each one of these principles in
how we build and govern Runa. This seems like overkill right now for such a
small project, but Tim (the author) has worked on massive things and has seen
the value of having this kind of guard rail in place from the beginning to point
to as things pick up momentum. Trying to do it later can easily go sideways, and
often does.

---

[1]: https://kimcrayton.com/guiding-principles/
