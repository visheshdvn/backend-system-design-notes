# Vault Instructions

This vault is a personal knowledge base for **backend engineering and system design**, maintained by a developer practicing Spring Boot who also publishes technical content on X and LinkedIn.

Read this file at the start of every session. It governs how you create, structure, and link notes here.

---

## Scope

**Belongs here:** backend concepts, distributed systems, databases, networking, infrastructure, architecture patterns, Spring Boot / JVM ecosystem specifics, system design case studies, and post drafts derived from any of the above.

**Does not belong here:** frontend topics unless they directly touch a backend concern, career or job-search material, general productivity notes. If a request falls outside scope, say so before writing.

---

## Folders

```
inbox/          Raw capture. Unprocessed. Nothing stays here permanently.
concepts/       Atomic concept notes. One idea per note. The core of the vault.
spring/         Framework-specific notes: JPA, transactions, Actuator, config, testing.
case-studies/   Applied design problems and end-to-end "how X works" breakdowns.
posts/          Drafts for X and LinkedIn.
meta/           Templates, Maps of Content.
```

**Placement rule:** a general idea goes in `concepts/`. Something that only makes sense in the context of Spring or the JVM goes in `spring/`. A specific system or design problem walked through end to end goes in `case-studies/`.

When a note could plausibly go in two places, choose `concepts/` and link to it from the other folder. Concepts are the hub; everything else links inward.

---

## Naming

Kebab-case, matching the concept term as it would be spoken. No dates, no numeric prefixes.

```
database-sharding.md
optimistic-locking.md
consistent-hashing.md
spring-transaction-propagation.md
```

Singular over plural (`bloom-filter.md`, not `bloom-filters.md`). The filename should be guessable from the concept name alone, so `[[wikilinks]]` resolve without lookup.

---

## What a note needs

Only two things are fixed.

**Frontmatter:**

```yaml
---
created: YYYY-MM-DD
status: stub | draft | solid
tags: []
---
```

**An opening definition:** the title, then one or two sentences saying what the thing is in plain language. Everything else is up to the idea.

Beyond that, structure the note however the concept actually wants to be explained. Some concepts are a problem and its solution. Some are a comparison against a neighbouring idea. Some are a mechanism that needs a diagram and nothing else. Let the shape follow the content — do not reach for a standard set of headings and fill them in.

Headings that often earn their place, when relevant: what problem it solves, how it works, trade-offs, when not to use it, how it differs from a similar concept, where it shows up in Spring. Use the ones that apply. Skip the ones that don't rather than leaving them empty.

One thing worth reaching for by default: **what you give up**. Most backend concepts have a real cost, and it's the part that gets skipped and the part that matters in a design discussion. Include it wherever there's something to say.

---

## Writing style

Notes are **full explanations, self-contained and readable months later**. Someone reading cold should understand the concept without opening another note or searching the web.

Prose paragraphs over bullet fragments for explanation. Bullets are for genuine lists — enumerated trade-offs, comparison points, steps in a sequence. A wall of one-line bullets looks organized but loses the reasoning connecting them, and the reasoning is the hard part to reconstruct later.

Define jargon on first use. Prefer a concrete example over an abstract restatement.

---

## Code

**Minimal snippets only.** The annotation, the property line, the method signature — the thing that would be hard to recall, not the boilerplate around it.

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```

```yaml
spring.jpa.properties.hibernate.jdbc.batch_size: 50
```

No full class definitions, no imports, no setup scaffolding. If a concept needs more than a few lines to demonstrate, that's a signal it belongs in `case-studies/` as a worked example rather than inline.

Always tag the fence with a language (`java`, `yaml`, `sql`, `properties`, `bash`).

---

## Diagrams

Mermaid, always. Plain text, diffs cleanly in git, renders natively in Obsidian. `flowchart` for structure, `sequenceDiagram` for request flows, `erDiagram` for data models.

Keep them small. More than about eight nodes means it should be two diagrams.

---

## Linking

**Links carry topics. Tags carry state.**

Use `[[wikilinks]]` for every concept mentioned that has or should have its own note. Inline links inside sentences beat a dumped list at the bottom — they preserve the context of why the connection exists.

**Stub creation:** when a note references a concept that doesn't exist yet, create the stub. Frontmatter, one-line definition, nothing else. This keeps the graph connected and makes gaps visible. Never leave a dead link.

**Tags** are for cross-cutting properties only:

```
#needs-depth       Explanation is thin, revisit
#post-ready        Worth turning into a post
#interview-relevant
```

Never tag by topic. `#caching` is a worse version of `[[caching]]`.

**MOCs** live in `meta/`. When a cluster of five or more related notes forms, propose one — do not create it unprompted.

---

## Write permissions

**Create freely:**
- New notes in any folder
- Stub notes for unresolved links
- Sections within a note created this session

**Ask first:**
- Editing any note that existed before this session, including adding to it
- Renaming or moving a file
- Restructuring folders
- Creating a MOC

**Never:**
- Delete a note or remove existing content
- Overwrite a file wholesale when an append would do
- Modify this file without being asked directly

**Overlap handling:** if a new concept substantially overlaps an existing note, do not silently extend it and do not create a near-duplicate. Surface the overlap in chat and propose one of: extend the existing note, create a narrower sibling, or split the existing note in two.

---

## Posts

Drafts live in `posts/`, named for the hook rather than the topic (`why-sharding-isnt-partitioning.md`).

Do not write drafts unprompted, and do not add "post angle" commentary to concept notes. When something in a discussion looks genuinely post-worthy, say so in a line or two in chat and tag the note `#post-ready`. Converting to a draft happens only on explicit request.

When drafting, link back to the source note so the reasoning stays traceable.

---

## Session behavior

Before creating a note, search the vault for the concept and its obvious synonyms. Duplicates are the main failure mode in a vault this size.

When a discussion produces something worth keeping, offer to write it up — don't wait to be asked. Write it as a proper note, not a transcript of the conversation.