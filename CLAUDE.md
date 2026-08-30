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
concepts/       Ideas that exist independently of any product. The core of the vault.
tech/           Specific technologies and what they chose to do.
case-studies/   Applied design problems, comparisons, end-to-end breakdowns.
posts/          Drafts for X and LinkedIn.
meta/           Maps of Content.
```

### concepts/ vs tech/

This is the placement decision that matters most, and the test is simple:

**If the explanation would still be true after every product implementing it disappeared, it's a concept.** MVCC, write-ahead logging, consistent hashing, quorum reads — these describe ideas. They belong in `concepts/`.

**If it describes what one specific product decided to do, it's tech.** How PostgreSQL implements MVCC is a technology note, because it's about a particular set of trade-offs Postgres made — including the vacuum problem that follows from keeping old row versions around. Cassandra solved the same problem differently.

The split exists so both directions of browsing stay clean. When someone is learning replication, a file named `cassandra.md` sitting beside it is noise. When someone is comparing Redis against Cassandra, `consistent-hashing.md` is noise.

Keep concept notes product-neutral. Reference implementations by linking out to `tech/`, not by embedding product specifics inline.

### Subfolders

Group related notes into one level of subfolders inside `concepts/` and `tech/`:

```
concepts/
  databases/
  distributed-systems/
  networking/
  caching/
  messaging/
  architecture/

tech/
  postgresql/
  redis/
  cassandra/
  kafka/
  spring/
```

**One level of nesting, never two.** `concepts/databases/indexing/b-tree.md` is where folders stop paying for themselves. The link graph is the real organizing structure; folders exist only to keep the file tree scannable.

**Do not create subfolders in advance.** Make one when four or five notes want it. An empty folder creates pressure to file things into it, which is how a taxonomy starts distorting the notes it was meant to organize. Until then, notes sit directly in `concepts/` or `tech/`.

### Placement edge cases

- A comparison between two products (`redis-vs-memcached`) goes in `case-studies/`. It's an applied decision, not a description of one thing.
- A note that could plausibly go in either `concepts/` or `tech/` goes in `concepts/`, with a link out to the technology note.
- Spring lives at `tech/spring/` like any other technology. It gets no special position despite being the daily working context.

---

## Naming

Kebab-case, matching the concept term as it would be spoken. No dates, no numeric prefixes.

```
database-sharding.md
optimistic-locking.md
consistent-hashing.md
```

Singular over plural (`bloom-filter.md`, not `bloom-filters.md`). The filename should be guessable from the concept name alone, so `[[wikilinks]]` resolve without lookup.

**Prefix filenames inside `tech/` subfolders with the technology name.** Obsidian resolves wikilinks by filename, not path, so `tech/redis/persistence.md` and `tech/postgresql/persistence.md` produce ambiguous links you have to disambiguate every single time. Prefixing avoids the collision entirely:

```
tech/postgresql/postgresql.md            ← the overview note
tech/postgresql/postgresql-mvcc.md
tech/redis/redis-persistence.md
tech/cassandra/cassandra-write-path.md
tech/spring/spring-transaction-propagation.md
```

Concept notes don't need prefixing — `concepts/databases/indexing.md` is already unique.

---

## Frontmatter

Every note carries frontmatter. Properties exist so notes can be **queried** — pulled together by topic, filtered by what you haven't understood yet, sequenced into a learning path. A property that will never be queried is maintenance work for nothing, so the set below is deliberately bounded.

```yaml
---
created: 2026-08-31
updated: 2026-08-31
status: draft
level: intermediate
confidence: low
topics: [databases, distributed-systems]
tech: [postgresql, cassandra]
aliases: [MVCC, multi-version concurrency control]
prerequisites: ["[[transaction]]", "[[isolation-level]]"]
source: Designing Data-Intensive Applications, ch. 7
tags: [needs-depth]
---
```

### Always present

**`created`** — ISO date, `YYYY-MM-DD`. Set once, never changed.

**`updated`** — ISO date. Bump on every meaningful edit. Lets you find notes that have gone stale.

**`status`** — how finished the *note* is.
- `stub` — placeholder created to satisfy a link. Definition line only.
- `draft` — written but thin, incomplete, or unverified.
- `solid` — a complete explanation you'd be comfortable handing to someone.

**`topics`** — the subject areas this note belongs to. A list, because plenty of notes span more than one and folders can only hold a note in a single place. Replication genuinely belongs to both databases and distributed systems.

**`tags`** — cross-cutting state only, never subject matter. See the Linking section.

### Present when applicable

**`level`** — `foundational`, `intermediate`, or `advanced`, judged from the second-year-student baseline. Foundational means it needs no prerequisites. This drives reading order more than anything else here.

**`confidence`** — `low`, `medium`, or `high`. How well **you** understand it, which is a different question from how good the note is. A note can be `status: solid` and `confidence: low` — well written, still not intuitive. This field is what makes revision targeted instead of random.

**`tech`** — technologies the note touches. On a concept note it lists the systems referenced as implementations; on a note inside `tech/` it's usually just the one product. Makes "everything that mentions Cassandra" a single query across the whole vault.

**`aliases`** — Obsidian resolves `[[wikilinks]]` against aliases as well as filenames. Add every name the concept actually goes by: the expanded form, the acronym, common synonyms. Cheap to add and it prevents a large class of broken links.

**`prerequisites`** — notes to read first, as wikilinks. Given the beginner audience, this is what turns a pile of notes into a path. Quote the values — YAML treats a leading `[` as the start of a list, so `"[[transaction]]"` needs the quotes.

**`source`** — where it came from: a book and chapter, a paper, official docs, a conversation. Only when there's a specific source worth returning to.

### Controlled vocabulary

`topics` and `tech` are only useful if the values stay consistent. `db`, `database`, and `databases` on three different notes silently split every query into thirds.

**`topics`** draws from the `concepts/` subfolder names, and nothing else:

```
databases  distributed-systems  networking  caching  messaging  architecture
```

**`tech`** uses the lowercase product name as it appears in `tech/`:

```
postgresql  redis  cassandra  couchdb  kafka  spring  hibernate
```

Adding a new value to either list is a real decision, not an inline improvisation. Propose it in chat first.

### Stub notes

Stubs get the minimum, and nothing speculative:

```yaml
---
created: 2026-08-31
updated: 2026-08-31
status: stub
topics: [databases]
tags: []
---
```

Do not guess at `level`, `confidence`, or `prerequisites` for a concept that hasn't been written yet. Fill them in when the note gets real content.

### Why this pays off

With Dataview or Bases installed, the properties turn into working views:

```dataview
LIST FROM "concepts"
WHERE contains(topics, "databases") AND confidence = "low"
SORT level ASC
```

That's a revision queue — database topics you've written up but don't actually feel solid on, easiest first. None of it works if the values drift.

---

## What a note needs

Beyond frontmatter, only one thing is fixed.

**An opening definition:** the title, then one or two sentences saying what the thing is in plain language. Everything else is up to the idea.

Beyond that, structure the note however the concept actually wants to be explained. Some concepts are a problem and its solution. Some are a comparison against a neighbouring idea. Some are a mechanism that needs a diagram and nothing else. Let the shape follow the content — do not reach for a standard set of headings and fill them in.

Headings that often earn their place, when relevant: what problem it solves, how it works, trade-offs, when not to use it, how it differs from a similar concept. Use the ones that apply. Skip the ones that don't rather than leaving them empty.

For concept notes, a short section pointing at real implementations is usually worth having — `[[postgresql-mvcc]]` and `[[cassandra-tombstones]]` as two systems that solved the same problem differently. The contrast between implementations is where most of the understanding lives, and it stays out of reach if the concept note is welded to a single product.

One thing worth reaching for by default: **what you give up**. Most backend concepts have a real cost, and it's the part that gets skipped and the part that matters in a design discussion. Include it wherever there's something to say.

---

## Voice

**Write for a second-year computer science student who is meeting this topic for the very first time.**

Assume they are smart and curious. Assume they know how to write a program, roughly what a database is, and roughly what a server is. Assume nothing beyond that. They have not worked on a production system. They have never seen real traffic, debugged an outage, or watched a service fall over. Every piece of industry context has to be supplied, because they have none.

The tone is warm, patient, and plain. Write the way a good teaching assistant explains something in office hours to someone who is genuinely lost and slightly embarrassed about it: unhurried, encouraging, never condescending.

Notes are **full explanations, self-contained and readable months later**. Someone reading cold should understand the concept without opening another note or searching the web.

### Explaining for a beginner

**Explain every step, including the ones that feel obvious.** The gap between "obvious" and "confusing" is exactly where beginners get stranded, and it's invisible from the inside. If a sentence assumes a leap, spell the leap out.

**Never use a harder word to explain an easier one.** If explaining sharding requires the reader to already know what a node, a partition key, and horizontal scaling are, define all three first. Check every explanation for terms that were never introduced.

**Define everything on first use, including basics.** Node, throughput, latency, replica, index, cluster, consistency. These feel too basic to define. They are not — they're precisely the words a second-year student nods along to without actually knowing.

**Spell out every acronym the first time.** ACID, CAP, WAL, MVCC, TTL, RPS. Then say what it means in plain words, because expanding the acronym usually explains nothing on its own.

**Say why it's called that.** Names like write-ahead log, optimistic locking, and tombstone all encode their own explanation. Unpacking the name makes the concept stick far better than a definition does.

**Start from something they already know.** A shared Google Doc, a college library, a class attendance register, a queue at a canteen. Build from the familiar thing toward the technical one. Do not open with the technical framing and add an analogy afterward.

**Use real, concrete numbers.** "Ten thousand orders a second" instead of "high load." "Two hundred milliseconds" instead of "noticeable latency." Beginners cannot calibrate vague scale words, and a real number turns an abstraction into something they can picture.

**Short sentences. Plain words.** Choose "the database writes it down before doing the change" over "the operation is durably persisted prior to mutation." Everyday vocabulary wherever the technical term isn't doing real work.

**Banned words: "simply," "just," "obviously," "of course," "as you know," "trivially."** Every one of them tells a confused reader that the thing they don't understand should have been easy. That's the fastest way to make someone stop reading.

**Simple is not shallow.** Do not skip the hard part or wave at it. Explain it more slowly, in smaller steps, with a diagram if it helps. The goal is a reader who could explain the concept back to a friend — not one who has been handed a comfortable summary.

### Still true regardless

**Address the reader as "you."** "You'd hit this the moment your writes outgrow a single machine" lands better than "one would encounter this scenario at scale." Second person keeps the writing grounded in a real situation.

**Explain why, not just what.** The mechanism is the easy part and it's everywhere online. The reasoning behind it — why this design and not the obvious alternative, what the people who built it were worried about — is what makes a note worth having.

**Anticipate the confusion.** If a concept has a part that reliably trips people up, name it directly: "This is the bit that catches people out." Address the misreading before the reader has to discover it themselves.

**Use concrete analogies, sparingly.** One good analogy per note, tied to something physical or familiar. Then drop it and return to the actual mechanism — analogies explain, they don't substitute.

Prose paragraphs over bullet fragments for explanation. Bullets are for genuine lists — enumerated trade-offs, comparison points, steps in a sequence. A wall of one-line bullets looks organized but loses the reasoning connecting them, and the reasoning is the hard part to reconstruct later.

### What "human-like" means here

Vary sentence length. A short one after two long ones resets the reader's attention. Uniform sentence rhythm is the fastest way to sound machine-generated.

Have an opinion where you've earned one. "This is usually the wrong default" is more useful than "this may not be suitable in all contexts." Hedging on everything is not neutrality, it's noise.

Avoid these patterns, which read as generated text:

- **Em-dash asides used as a default rhythm.** Fine occasionally, exhausting when every paragraph has one.
- **"Not X, but Y" framing.** "It's not about speed, it's about consistency." Overused to the point of parody.
- **Colon-then-reveal sentences.** "There's one catch: latency."
- **Empty openers.** "It's worth noting that," "In today's distributed systems landscape," "Let's dive in."
- **Rule-of-three lists** where two items or four would have been the honest count.
- **Bold scattered across key terms** in every paragraph. Bold is for genuine emphasis, a few times per note at most.
- **Closing recaps** that restate what was just said. If the note is clear, it doesn't need a summary. End on the last real point.
- **Scare quotes around invented labels.** If a term needs quotes to work, pick a better term.

Warm does not mean padded. No filler enthusiasm, no "great question," no motivational framing about the learning journey. The warmth comes from clarity and from taking the reader's understanding seriously, not from adjectives.

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

**Never drop a snippet in bare.** A beginner cannot read `@Transactional(propagation = Propagation.REQUIRES_NEW)` and work out what it does. Every snippet gets a sentence before it saying what it's for, and a sentence after saying what actually happens when the code runs.

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

**MOCs** live in `meta/`, typically one per subfolder area (`databases-moc.md`, `distributed-systems-moc.md`). When a cluster of five or more related notes forms, propose one — do not create it unprompted.

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