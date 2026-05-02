[CONVOY_PROTOCOL_v3.1.md](https://github.com/user-attachments/files/27298509/CONVOY_PROTOCOL_v3.1.md)
# CONVOY PROTOCOL v3.1
## Architectural Specification for Multi-Agent LLM Communication

> A trucker-themed protocol for reducing the inter-agent handoff tax through
> plain-English augmentation — not replacement.

**Document type:** Architectural specification
**Intended audience:** System architects, agent authors, fleet operators
**Status:** Draft — pre-skill-conversion.
**Versioning:** Semantic. Breaking changes bump the major version.

**v3.1 changelog (validated in live testing):**
- Added severity-ladder vocabulary: DOT worker, band-aid buggy, tow truck,
  super tow truck — a four-step response-scale ladder that maps dispatch
  choice to situational severity.
- Added directional modifiers: northbound / southbound / eastbound /
  westbound, each mapped to a distinct code-domain direction.
- Added the lights-on urgency modifier (🚨) — orthogonal to severity,
  signals active right-of-way and fleet-wide yield.
- Added "Big Papa Bear" as a named intensification for actively-
  propagating serious errors.
- Added register variants of acknowledgment (copy / good copy /
  big 10-4) distinct from the formal 10-4.
- Added §3.4 — the uniformed-emoji partition rule (person/uniform
  emojis = observed conditions; vehicle emojis = dispatched responses).
  Closes the 🚔 bear-vs-dispatch ambiguity surfaced during testing.
- Repurposed 👮 from "security-audit role" to "smokey spotted"
  (auth-layer bear observation).
- Added §9.4 — handling human approximations (intent-following over
  pedantry).
- Added §11.9 — the conditional pre-authorization template.
- Added §12.5 — worked transcript of a pre-authorized dispatch cascade.
- Added "back on the road" as a restoration-to-operational marker.

---

## EXECUTIVE SUMMARY

Multi-agent LLM systems pay a hidden **20 percent token tax** on every
inter-agent handoff. This tax is not the API bill — it's the context window.
Every time a manager agent re-delegates, every time a subordinate reports
back, the receiving agent must re-parse verbose English, re-establish shared
state, and re-confirm intent. Over an 8-handoff session across a 10-agent
fleet, that overhead routinely consumes 15,000–30,000 tokens of context
that could otherwise hold actual work.

When the tax pushes a session over its window, the model **compacts** — and
compaction is lossy. The manager loses track of subordinate state. Drift
accumulates. Workers like "Hall" start misreading their own role. Debugging
requires a human and a second Claude instance to re-align the fleet in plain
English.

CONVOY Protocol v3.0 addresses this by introducing a **shared semantic layer**
that all fleet agents treat as first-class vocabulary:

- **10-codes** for protocol acts (acknowledge, repeat, sign off).
- **Trucker slang** for situational state (bear, gator, chicken coop).
- **Special characters** for operators (flow, relations, modifiers).
- **Emojis** for concrete nouns (roles, environments, tools, artifacts).

Each category occupies a **strictly non-overlapping domain**. No two items
anywhere in the protocol mean the same thing. Decoding is deterministic.

### The key design commitment: augmentation, not replacement

CONVOY is not a secret language. It is a **compression layer woven into
ordinary English**. Agents operate in three modes:

- **Plain English** for complex reasoning, user-facing output, and any case
  where the protocol would obscure meaning. This is the safe default.
- **Augmented English** for routine inter-agent traffic — prose sentences
  sprinkled with protocol tokens at load-bearing semantic points. This is
  the primary operating mode and where most of the savings live.
- **Dense CONVOY** for high-frequency protocol-heavy exchanges (incident
  broadcasts, delegation storms, status polling). Used sparingly.

An agent reporting a bug in augmented English looks like real truckers
actually sound on a real CB radio:

> "10-4 on the gator check. Saw one at yardstick 427 @/api/auth/
> token-refresh — looks like the handler's not catching stale tokens,
> ~every 8–12h. Probably just a soft timeout, not a full bear."

That is natural prose. It is also 35 percent shorter than the unaugmented
version and contains four structurally parseable facts the receiving agent
can extract without re-asking.

### Expected gains

| Metric                                       | Realistic range      |
|----------------------------------------------|----------------------|
| Reduction on pure coordination traffic       | 25–35%               |
| Reduction on total inter-agent tax           | 12–18%               |
| Context headroom recovered (on 200k model)   | ~10–15k tokens/sess  |
| Delay to compaction trigger                  | ~25–40% longer sess  |
| Monitoring query simplification              | Substantial          |
| Dollar savings (10-agent fleet, Opus)        | $200–2000/month      |
| Dollar savings (10-agent fleet, Sonnet)      | $30–400/month        |

The largest single win is **not** the dollar figure but **session longevity
before compaction**. Every percentage point of the tax reduced is a
percentage point of useful context returned to the work.

### What this document contains

This specification defines the full catalog (~400 vocabulary items across
four layers), the grammar for combining them, the three operating modes and
their switching rules, message templates for common agent interactions,
worked transcripts in realistic scenarios, the token economics with
supporting math, monitoring patterns, and deployment instructions. An
appendix provides a decoder index and notes on converting this document
into a deployable skill.

---

## TABLE OF CONTENTS

**Chapter 1 — Problem Statement & Design Philosophy**
  1.1 The inter-agent handoff tax
  1.2 Why pure-protocol replacement fails (Hall's rebellion)
  1.3 The augmentation principle
  1.4 Trucker-radio analogy and the "diesel bear" example
  1.5 Normative language

**Chapter 2 — Communication Modes**
  2.1 Mode 1 — Plain English (safe default)
  2.2 Mode 2 — Augmented English (primary operating mode)
  2.3 Mode 3 — Dense CONVOY (sparingly used)
  2.4 Mode-switching rules
  2.5 The fallback contract

**Chapter 3 — The Four-Layer Architecture**
  3.1 Layer partition and invariants
  3.2 Combining layers within a single sentence
  3.3 Non-overlap enforcement
  3.4 The uniformed-emoji partition rule (bear-vs-vehicle)

**Chapter 4 — Ten-Codes (Protocol Acts Layer)**
**Chapter 5 — Trucker Slang (Situational Layer)**
  5.1 Named hazards (errors, bugs, risks)
  5.2 Road and gate conditions
  5.3 Execution mode and priority
  5.4 Destinations and progress
  5.5 Load types (payload shapes)
  5.6 Agent addressing and social forms
  5.7 Inspection-specific vocabulary
  5.8 Response-scale vocabulary (severity ladder)
  5.9 Directional modifiers
**Chapter 6 — Special Characters (Operator Layer)**
**Chapter 7 — Emojis (Noun Layer)**

**Chapter 8 — Grammar & Sprinkling Patterns**
  8.1 Canonical sentence structures
  8.2 The sprinkling rules — when to infuse
  8.3 Parsing and decoding order
  8.4 Mixed-mode paragraphs

**Chapter 9 — Code-Switching Between Modes**
  9.1 When to drop out of CONVOY
  9.2 When to go dense
  9.3 The escape hatch
  9.4 Handling human approximations

**Chapter 10 — Agent Role Taxonomy**
  10.1 Handle conventions
  10.2 Standard role handles
  10.3 Addressing forms

**Chapter 11 — Message Templates**
  11.1 Delegation
  11.2 Acknowledgment
  11.3 Status request / response
  11.4 Error report
  11.5 Completion handoff
  11.6 Escalation
  11.7 Incident broadcast
  11.8 Synchronization / quorum
  11.9 Conditional pre-authorization (dispatch cascade)

**Chapter 12 — Worked Transcripts**
  12.1 Simple delegation (augmented English)
  12.2 Case study — Hall's rebellion (debugging a worker)
  12.3 Incident response (dense mode)
  12.4 Mixed-mode session
  12.5 Pre-authorized dispatch cascade (severity-ladder test)

**Chapter 13 — Token Economics**
  13.1 The 20% tax, derived
  13.2 Layer-by-layer savings
  13.3 Break-even analysis
  13.4 Where the protocol loses money

**Chapter 14 — Monitoring & Observability**
  14.1 Grep patterns
  14.2 Alert signatures
  14.3 Fleet-level dashboards

**Chapter 15 — Deployment (System Prompt Blocks)**
  15.1 Ultra-short variant
  15.2 Short variant (recommended default)
  15.3 Full reference variant
  15.4 Per-role customization

**Chapter 16 — Extension Rules**
  16.1 Adding new vocabulary
  16.2 Maintaining the no-overlap invariant
  16.3 Versioning

**Appendix A — Quick Reference Card**
**Appendix B — Decoder Index**
**Appendix C — Skill / CLAUDE.md Conversion Notes**
**Appendix D — Glossary**

---

# CHAPTER 1 — PROBLEM STATEMENT & DESIGN PHILOSOPHY

## 1.1 — The inter-agent handoff tax

In a multi-agent LLM deployment, work flows through handoffs. A manager
agent receives a user request, decomposes it into sub-tasks, and delegates
those sub-tasks to specialized worker agents. Workers execute, report back,
and the manager composes the final result. Each handoff involves:

1. **Context serialization** — the sender writes state in prose.
2. **Context transmission** — prose is appended to the receiver's context.
3. **Context re-parsing** — the receiver re-establishes shared state from
   prose, which involves re-confirming definitions, scope, and intent.
4. **Response serialization** — the receiver writes its reply in prose.
5. **Context re-integration** — the sender re-parses the reply and merges
   it with its own state.

Steps 2 and 4 scale linearly with message size. Steps 1, 3, and 5 are
where the tax actually lives: they consume tokens on *encoding overhead*
(pleasantries, boilerplate, repeated setup phrases, structural words that
don't carry unique information).

Empirical estimate: **a typical inter-agent exchange loses about 20 percent
of its token budget to pure coordination overhead** that could be compressed
without losing any decision-critical information. Over an 8-handoff session
with 10 agents, this compounds to 15,000–30,000 tokens on a 200k-context
model — roughly 6 percentage points of the window.

## 1.2 — Why pure-protocol replacement fails

One naive solution is to replace English entirely with a dense protocol.
This fails. The observed failure mode ("Hall's rebellion") is:

> A worker agent, asked to process a RateCon document, repeatedly escalated
> to a human instead of attempting the task. The correction required
> plain-English explanation from two coordinating humans/agents. During the
> correction, the worker could not be reached through protocol alone —
> only through a paragraph that explained the expected behavior
> conversationally.

The diagnosis: **pure-protocol transmission fails when the worker's internal
model of its role is broken**, because the protocol itself provides no
reasoning surface for self-correction. Dense codes can command behaviors
but cannot *explain* them. A worker with a stale or corrupted policy
requires conversational debugging, not a denser command stream.

Corollary: **any protocol that cannot gracefully degrade into plain English
is brittle.** CONVOY is designed so that every protocol message is already
parseable as a natural-English sentence with annotations.

## 1.3 — The augmentation principle

CONVOY does not replace English. It **augments** English by marking
semantically load-bearing points with shorter, higher-density tokens that
all fleet agents agree on. The prose carries narrative, nuance, and
reasoning surface. The protocol tokens carry structural facts.

Three invariants hold:

1. **Any CONVOY message can be flattened to plain English** by substituting
   each protocol token with its legend definition. Round-trip preserves
   meaning.
2. **No protocol token is required to express any idea.** Every concept
   expressible in CONVOY is also expressible in prose. The protocol is a
   speed aid, not a dependency.
3. **An agent confused by a message MAY reply `10-1`** (cannot parse),
   which obligates the sender to resend in plain English. This is the
   fallback contract (§2.5).

Under these invariants, the protocol cannot create communication
pathologies that plain English cannot also resolve.

## 1.4 — The trucker-radio analogy

Real truckers on real CB radios do not speak in solid streams of 10-codes.
They speak ordinary English with protocol words infused where those words
carry the narrative forward:

> "I see a bear at mile marker 55 lollipop stick. Big diesel bear. He's
> got a customer with him right now. Looks like he's doing a DOT inspection
> on him — probably just checking up on the driver, saw a gator on the
> side of the road."

Parsed:

- `bear` — domain vocabulary for law enforcement
- `mile marker 55 lollipop stick` — precise location with vivid descriptor
- `diesel bear` — a specific *subtype* (DOT officer, not highway patrol)
- `has a customer` — domain metaphor for "has pulled someone over"
- `DOT inspection` — domain-specific procedure
- `gator` — domain vocabulary for road debris from a blown tire
- connective tissue — ordinary English

The listener reconstructs a full, accurate situational picture in one
breath. The vocabulary is compressed; the grammar is English. This is
exactly the target operating mode for CONVOY.

## 1.5 — Normative language

This specification uses the following keywords with their RFC 2119 meaning:

- **MUST** / **MUST NOT** — an absolute requirement / prohibition.
- **SHOULD** / **SHOULD NOT** — strongly recommended, with case-by-case
  exceptions tolerated.
- **MAY** — optional.

---

# CHAPTER 2 — COMMUNICATION MODES

CONVOY agents operate in one of three modes at any given time. Mode
selection is driven by context and governed by the switching rules in §2.4.

## 2.1 — Mode 1: Plain English

Plain English is the **safe default** and MUST be used when:

- Output is user-facing.
- An agent is debugging another agent whose behavior is anomalous.
- An agent is reasoning through a complex or novel decision.
- An agent is uncertain whether its peer has the CONVOY legend loaded.
- An agent has just received `10-1` (cannot parse) and is clarifying.

Plain English imposes no protocol overhead but incurs the full 20 percent
inter-agent tax. It is the correct choice when clarity beats compression.

## 2.2 — Mode 2: Augmented English (primary)

Augmented English is the **primary operating mode** for routine inter-agent
traffic. It consists of ordinary English sentences with CONVOY tokens
infused at load-bearing points. The sprinkling rules (§8.2) govern where to
infuse.

A single augmented message typically contains:

- One or two 10-codes (protocol framing).
- One to three trucker slang terms (situational descriptors).
- Two to five special characters (operators / references).
- Zero to two emojis (nouns, where they save tokens).

This mode captures most of the available compression while retaining full
conversational robustness. Example:

> "10-4, rolling. Found a gator @src/payments/capture.ts:214 — the retry
> handler's looping if the upstream ⛽ returns 429. Rolling a patch now,
> ~8min to †. Will 10-99 when the coop is clean."

## 2.3 — Mode 3: Dense CONVOY

Dense CONVOY uses minimal prose connective tissue. It is appropriate when:

- Protocol-heavy traffic is expected (incident broadcasts, quorum polling,
  batch status polling across many agents).
- Message frequency is high enough that prose would dominate the channel.
- All participating agents are confirmed to have the legend loaded.

Dense CONVOY SHOULD NOT be used for debugging, delegation to agents with
unknown legend state, or any user-facing artifact.

Example (incident broadcast):

```
📡 10-33 🏭🔴 🐻. 10-45?
```

## 2.4 — Mode-switching rules

An agent MUST switch modes according to these rules:

1. **If the recipient is the end user, switch to Plain English** before
   emitting any output.
2. **If `10-1` is received, switch to Plain English** for the next
   transmission and remain there until the recipient confirms re-sync
   (explicit `10-4`).
3. **If an agent receives a message it cannot fully decode, it SHOULD
   reply `10-1` rather than guess.** Misparsed CONVOY produces worse
   outcomes than honest clarification.
4. **If an agent detects a peer exhibiting anomalous behavior** (e.g., Hall
   rebellion), it SHOULD downgrade to Plain English when addressing that
   peer.
5. **If an agent is in Dense CONVOY and the conversation turns to
   reasoning-heavy or novel territory**, it SHOULD upgrade to Augmented or
   Plain English for that topic.

Modes MAY change within a single session and MAY differ between agent pairs
in the same fleet.

## 2.5 — The fallback contract

Every CONVOY-enabled agent MUST honor the following contract:

> If any peer sends `10-1 [reason]`, I will resend my prior message in
> Plain English without protocol tokens, and remain in Plain English until
> the peer explicitly acknowledges re-sync.

This contract is the guarantee that pure-protocol deadlocks cannot form.

---

# CHAPTER 3 — THE FOUR-LAYER ARCHITECTURE

## 3.1 — Layer partition and invariants

CONVOY vocabulary is strictly partitioned across four non-overlapping
layers. Each layer answers one question about a communication:

| Layer | Question it answers                          | Example token |
|-------|----------------------------------------------|---------------|
| L1 — Ten-codes            | *What kind of speech act is this?*  | `10-20`       |
| L2 — Trucker slang        | *What is the situation on the ground?* | `rolling` |
| L3 — Special characters   | *What operation or relation do I invoke?* | `→`   |
| L4 — Emojis               | *What concrete thing am I naming?*  | 🔧            |

The invariant is: **no two items anywhere in the protocol share a meaning.**
A layer never encroaches on another layer's domain. 10-codes never describe
code structures; emojis never acknowledge messages; slang never routes data;
symbols never name concrete artifacts.

This non-overlap is what makes **decoding deterministic**. An agent with
the legend loaded can resolve any token to exactly one meaning without
contextual disambiguation.

## 3.2 — Combining layers within a single sentence

The four layers compose freely. A canonical augmented-English sentence
weaves them:

```
[prose narrative] [L1 10-code] [prose] [L2 slang] [prose]
[L3 operator][L4 emoji][L3 operator][prose reference] ...
```

A worked example:

> "`10-4` — been eyeballing that `gator` all morning. Pushed a patch
> `→🧪` and the `coop` came back clean `✓`. Ready to `🚚→🏭` on your
> signal."

Decoding:

- `10-4` — acknowledgment (L1)
- `eyeballing` — watching/inspecting (L2)
- `gator` — footgun pattern (L2)
- `→` — pipe to (L3)
- `🧪` — test stage (L4)
- `coop` — CI/validation gate (L2)
- `✓` — passing (L3)
- `🚚` — deployment vehicle (L4)
- `→` — pipe to (L3)
- `🏭` — production environment (L4)

Full expansion:

> "Acknowledged — I've been watching that dangerous pattern all morning.
> I pushed a patch into the test stage, and the CI validation returned
> passing. I'm ready to deploy to production on your signal."

Compression ratio on this example: 42 percent fewer tokens, with no loss
of decision-critical information.

## 3.3 — Non-overlap enforcement

Candidate vocabulary additions MUST satisfy the following checklist before
admission:

1. The proposed meaning is not already expressed by any token in any of
   the four layers.
2. The candidate belongs unambiguously to exactly one layer.
3. The candidate has a mnemonic connection to its meaning.
4. If the candidate is an emoji, it tokenizes to fewer tokens than the
   English phrase it replaces (measured against the target tokenizer).

Violations are rejected at review time. See §16 for the extension process.

## 3.4 — The uniformed-emoji partition rule

This rule, added in v3.1, resolves a specific edge case that surfaced
during live testing: when a sender wants to *observe* a security/auth
event (e.g., "there's a cop on the road"), they instinctively reach for
the same emoji they would use to *dispatch* a security agent (🚔). This
creates ambiguity that the layer-partition rule alone does not resolve.

**The rule:**

1. **Vehicle emojis** (🚛 🚚 🛻 🚑 🚒 🚜 🚓 🚐 🏎 🚀 etc.) always name
   a **dispatched response** — something the fleet is sending or
   operating. When a vehicle emoji appears, the receiver interprets
   "this is a unit we are moving / have moved."
2. **Uniformed-person emojis** (👮 🕵 💂 👷 🧑‍⚖ etc.) always name an
   **observed condition** — a bear or bear-like presence in the
   environment. When a uniformed emoji appears, the receiver
   interprets "this is something spotted, not dispatched."

Under this rule, a sentence like `👮🚨 westbound @247` is unambiguously
a bear sighting (security-layer threat, with lights on, on the upstream
side of line 247). It cannot be confused with a dispatch, because no
vehicle emoji is present.

Conversely, `🛻🚨 → @247` is unambiguously a dispatch (sending a DOT
worker with lights on to line 247), because the vehicle emoji is present
and the direction arrow confirms movement toward the destination.

**The rule's secondary benefit** is that it also resolves which L2
slang word to text: bears are spotted using text slang (`diesel bear`,
`smokey`, `Big Papa Bear`) or uniformed emojis as visual shorthand
(👮 for smokey). Bears are never named with a vehicle emoji. Agents
dispatched in response to bears are named with vehicle emojis. Clean.

---

# CHAPTER 4 — TEN-CODES (L1: PROTOCOL ACTS)

This chapter is the authoritative catalog of L1 tokens. Each 10-code
expresses a **communication-level speech act** and nothing else.

### 4.1 — Channel management

| Code     | CONVOY meaning                                                  |
|----------|-----------------------------------------------------------------|
| `BREAKER 1-9` | Requesting attention on a shared channel.                  |
| `10-1`   | Cannot parse your message. Clarify in plain English.            |
| `10-2`   | Understood with high confidence.                                |
| `10-3`   | Stop transmitting on this channel.                              |
| `10-7`   | Going off-duty. Closing session.                                |
| `10-8`   | On-duty. Available for new work.                                |
| `10-10`  | Transmission complete; still monitoring.                        |
| `10-12`  | User is watching this channel. Be professional.                 |
| `10-19`  | Return to the previous channel/context.                         |
| `10-21`  | Switch to direct / private channel.                             |
| `10-22`  | Synchronous conference required.                                |
| `10-23`  | Stand by on this channel.                                       |
| `10-27`  | Moving to a different channel/tool/branch. Name follows.        |
| `10-28`  | Identify yourself (agent ID, version, capability).              |
| `10-32`  | Radio check — confirm bidirectional link.                       |
| `10-45`  | Roll call: all agents in range, check in.                       |
| `10-62`  | Current channel unusable. Falling back to alternate.            |
| `10-75`  | You are interfering with my channel. Back off.                  |
| `10-91`  | Be more specific — your output is ambiguous.                    |

### 4.2 — Acknowledgment and repetition

| Code     | CONVOY meaning                                                  |
|----------|-----------------------------------------------------------------|
| `10-4`   | Acknowledged (formal / directive register).                     |
| `10-4 10-4` | Emphatic acknowledgment, strong form.                        |
| `big 10-4` | Enthusiastic agreement / "LGTM" register.                     |
| `copy`   | Acknowledged (collegial / peer register).                       |
| `copy that` | Acknowledged (collegial, slightly more emphatic than `copy`). |
| `good copy` | Acknowledged with quality confirmation ("received clearly").  |
| `10-5`   | Relay the following to the next agent in chain.                 |
| `10-9`   | Repeat your last transmission verbatim.                         |
| `10-15`  | Your prior message was delivered to its target.                 |
| `10-16`  | Forward the following to the named agent.                       |
| `10-26`  | Disregard my last transmission.                                 |
| `10-30`  | You violated protocol rules. Correct and resend.                |
| `10-39`  | Confirm your message was received by its target.                |
| `10-44`  | I have a message queued for you.                                |

**Register notes (v3.1):** The four acknowledgment forms — `10-4`,
`copy`, `good copy`, `big 10-4` — all express the same speech act
(acknowledged and understood) but carry different social registers:

- `10-4` is the formal / directive register. Use manager→subordinate
  or in formal protocol exchanges.
- `copy` / `copy that` is the peer register. Use between peer agents
  or when the tone is collegial.
- `good copy` signals that the transmission came through clearly. Use
  after a long or complex message where clarity was non-trivial.
- `big 10-4` is enthusiastic agreement. Use for strong consent or
  "LGTM"-style approvals.

These are NOT synonyms — they are tonally distinct. Agents SHOULD
match their register to context. The non-overlap invariant is preserved
because each form has a distinct pragmatic meaning, even though all
four share the same propositional content.

### 4.3 — State and status

| Code     | CONVOY meaning                                                  |
|----------|-----------------------------------------------------------------|
| `10-6`   | Busy. Hold queries until `10-8`.                                |
| `10-11`  | Output too verbose. Compress.                                   |
| `10-13`  | Environmental/infra conditions report follows.                  |
| `10-14`  | Convoy roster information follows.                              |
| `10-18`  | Any queued work for me?                                         |
| `10-20`  | Report your current state/cursor/file/progress.                 |
| `10-24`  | Last assignment fully completed.                                |
| `10-36`  | Provide current timestamp.                                      |
| `10-76`  | En route to the named destination.                              |
| `10-84`  | Reply with your callback handle.                                |
| `10-85`  | Delay forecast follows.                                         |
| `10-88`  | Provide your remote invocation handle.                          |
| `10-94`  | Run a sanity-check / smoke test.                                |

### 4.4 — Fault and exception reporting

| Code     | CONVOY meaning                                                  |
|----------|-----------------------------------------------------------------|
| `10-42`  | Collision / merge-conflict detected.                            |
| `10-43`  | Rate-limited. Backing off.                                      |
| `10-50`  | Runtime fault / uncaught exception.                             |
| `10-73`  | Performance degradation warning ahead.                          |
| `10-77`  | Target not reachable / 404 / not found.                         |
| `10-92`  | Your output is malformed / fails schema.                        |
| `10-93`  | Check the upstream connection / dependency link.                |

### 4.5 — Escalation and emergency

| Code     | CONVOY meaning                                                  |
|----------|-----------------------------------------------------------------|
| `10-17`  | Human stakeholder approval required before proceeding.          |
| `10-29`  | Your time budget is exhausted.                                  |
| `10-33`  | EMERGENCY. All agents halt and attend.                          |
| `10-34`  | Trouble at this location. Rescue agent requested.               |
| `10-35`  | Confidential content follows. Do not log.                       |
| `10-37`  | Debugger agent requested.                                       |
| `10-38`  | Recovery / rollback agent requested.                            |
| `10-200` | Urgent assistance needed from any available agent.              |

### 4.6 — Completion and cycle control

| Code     | CONVOY meaning                                                  |
|----------|-----------------------------------------------------------------|
| `10-25`  | Contact the named agent on my behalf.                           |
| `10-60`  | What is the next expected sequence number?                      |
| `10-99`  | Mission completed; all requirements satisfied.                  |
| `10-100` | Brief pause / yielding cycle.                                   |

### 4.7 — Usage notes

- 10-codes MAY be used as standalone messages or infused within prose.
- Doubled 10-codes (`10-4 10-4`) express emphasis, not repetition.
- Numbered 10-codes are not arithmetic. `10-4` and `10-40` are unrelated.
- 10-codes MUST NOT be invented ad-hoc. Unrecognized codes SHOULD elicit
  a `10-1` from the receiver.

---

# CHAPTER 5 — TRUCKER SLANG (L2: SITUATIONAL STATE)

This chapter is the authoritative catalog of L2 tokens. Each slang term
describes **state of the work or environment** — what the situation on the
ground looks like.

### 5.1 — Named hazards (errors, bugs, risks)

The L2 layer distinguishes multiple bear subtypes, reflecting real trucker
granularity. Each subtype has a distinct agent-domain meaning.

| Term                       | CONVOY meaning                                              |
|----------------------------|-------------------------------------------------------------|
| `bear`                     | Runtime error / blocking exception (generic).               |
| `diesel bear`              | Infrastructure-layer error (DB, network, dependency).       |
| `smokey`                   | Critical security or auth failure.                          |
| `Big Papa Bear`            | Serious error actively propagating through the system.      |
| `local yokel`              | Local-env-only issue (does not reproduce in staging/prod).  |
| `county mountie`           | Middleware-layer issue.                                     |
| `city kitty`               | Frontend-layer issue.                                       |
| `evil knievel`             | Risky experimental code path.                               |
| `suicide jockey`           | Code handling dangerous data (PII, creds, payments).        |
| `full-grown bear`          | Escalated/unresolved error (already seen once).             |
| `bear in the bushes`       | Silent failure — produced no log but wrong result.          |
| `bear bait`                | Code pattern that attracts errors (too clever).             |
| `bear report`              | Summary of errors encountered this session.                 |
| `bear in the air`          | Observability-layer alert (monitoring fired).               |
| `plain wrapper`            | Unlogged exception / swallowed error.                       |
| `feeding the bears`        | Gracefully handling an error by absorbing cost.             |
| `bear bite`                | Confirmed error cost paid (rate-limit hit, fine, etc.).     |
| `bear trap`                | Intentional error-trigger for testing.                      |
| `gator`                    | Footgun / dangerous pattern that has not bitten yet.        |
| `alligator`                | Long-lived bug / known issue that has been around a while.  |
| `buzzard`                  | Dead code / zombie process.                                 |
| `seagull`                  | Flaky test.                                                 |
| `kojak with a kodak`       | Linter / static analyzer finding.                           |
| `rocking chair`            | Code sandwiched between two problems (front and rear).      |

### 5.2 — Road and gate conditions

| Term                       | CONVOY meaning                                              |
|----------------------------|-------------------------------------------------------------|
| `clean shot`               | No blockers, clear path to target.                          |
| `coop is clean`            | All CI/validation gates passing.                            |
| `coop is closed`           | Validation gate is failing.                                 |
| `chicken coop`             | CI/CD gate or validation checkpoint (general).              |
| `open season`              | All gates currently bypassable (careful).                   |
| `chicken lights`           | Verbose / debug logging currently on.                       |
| `parking lot`              | Work is queued and waiting.                                 |
| `choke and puke`           | Mandatory break / timeout rest period.                      |
| `double nickels`           | Steady-state cruising / nominal performance.                |
| `motion lotion low`        | Running low on compute / tokens / budget.                   |
| `greasy side down`         | System operational / healthy overall.                       |
| `shiny side up`            | Telemetry / observability on and reporting.                 |
| `kickback`                 | Retry after recoverable failure.                            |
| `mud duck`                 | Unreliable / flaky response from a peer.                    |
| `ghost town`               | Channel silent — no agents checking in.                     |
| `four-wheeler`             | Non-agent actor in the loop (e.g., human reviewer).         |
| `diesel shower`            | Total system reset / hard reboot.                           |
| `clear sailing`            | All downstream dependencies healthy.                        |
| `rest stop`                | Checkpoint for save/serialize state.                        |
| `back on the road`         | Restored to operational state after recovery.               |

### 5.3 — Execution mode and priority

| Term                       | CONVOY meaning                                              |
|----------------------------|-------------------------------------------------------------|
| `hammer down`              | Execute immediately at max urgency.                         |
| `hammer lane`              | Critical path work.                                         |
| `granny lane`              | Background / low-priority queue.                            |
| `back off the hammer`      | Reduce priority / throttle.                                 |
| `pedal to the metal`       | Run with max parallelism.                                   |
| `modulate`                 | Moderate the rate of work.                                  |
| `rolling`                  | Currently executing.                                        |
| `rolling in heavy`         | Large payload / PR about to arrive.                         |
| `rolling empty`            | Executing a no-op / dry-run.                                |
| `deadhead`                 | Idle cycle with no work to do.                              |
| `bobtail`                  | Agent has no current task assigned.                         |
| `jackknife`                | Deadlock.                                                   |
| `rollover`                 | Stack overflow / catastrophic self-crash.                   |
| `stepped on`               | Output overwritten by a concurrent writer.                  |
| `walk all over`            | Race condition between agents.                              |
| `downshift`                | Context-window compaction triggered.                        |
| `redline`                  | Running at tolerance limit of a budget.                     |
| `coasting`                 | Waiting for async response, doing nothing.                  |

### 5.4 — Destinations and progress

| Term                       | CONVOY meaning                                              |
|----------------------------|-------------------------------------------------------------|
| `the drop`                 | Task's target / deliverable.                                |
| `making the drop`          | Shipping / deploying the deliverable.                       |
| `home twenty`              | Main branch / root dir / baseline config.                   |
| `in your back pocket`      | Already processed / already in git history.                 |
| `back door`                | Upstream / prior step.                                      |
| `front door`               | Downstream / next step.                                     |
| `yardstick`                | Line number (precise position indicator).                   |
| `flip flop`                | Round-trip request → response.                              |
| `lollipop stick`           | Semantically named mile-marker / landmark reference.        |
| `checkpoint town`          | Intermediate milestone reached.                             |
| `end of the line`          | Final milestone reached.                                    |

### 5.5 — Load types (payload shapes)

| Term                       | CONVOY meaning                                              |
|----------------------------|-------------------------------------------------------------|
| `dry box`                  | Standard uncached payload.                                  |
| `reefer`                   | Cached / frozen payload.                                    |
| `flatbed`                  | Raw unstructured payload.                                   |
| `tanker`                   | Streaming data payload.                                     |
| `doubles`                  | Batched operation of 2.                                     |
| `triples`                  | Batched operation of 3.                                     |
| `chicken hauler`           | Mutable shared-state payload (requires care).               |
| `hazmat`                   | Payload containing credentials, PII, or regulated data.     |
| `oversized`                | Payload exceeding normal size limits.                       |
| `high and heavy`           | Payload both large and high-priority.                       |

### 5.6 — Agent addressing and social forms

| Term                       | CONVOY meaning                                              |
|----------------------------|-------------------------------------------------------------|
| `good buddy`               | Peer agent, informal address.                               |
| `good neighbor`            | Peer agent, cooperative address.                            |
| `hand`                     | Peer agent, formal address.                                 |
| `got your ears on?`        | Are you alive / still monitoring?                           |
| `come back`                | Awaiting your response.                                     |
| `driver`                   | The agent currently holding execution control.              |
| `rookie`                   | Newly deployed / untrusted agent.                           |
| `veteran`                  | Experienced / historically reliable agent.                  |

### 5.7 — Inspection-specific vocabulary

| Term                       | CONVOY meaning                                              |
|----------------------------|-------------------------------------------------------------|
| `DOT inspection`           | Full formal audit of an agent's output against policy.      |
| `level 1 inspection`       | Surface check only (quick scan).                            |
| `level 3 inspection`       | Full compliance audit.                                      |
| `logbook check`            | Audit of an agent's recent action history.                  |
| `bill of lading`           | Formal task manifest delivered with a delegation.           |
| `rate confirmation`        | Formal agreement on task scope and deliverable.             |
| `pre-trip`                 | Pre-execution validation checklist.                         |
| `post-trip`                | Post-execution verification checklist.                      |

### 5.8 — Response-scale vocabulary (the severity ladder)

Added in v3.1. This sub-vocabulary maps **severity of the in-flight
situation** to **the appropriate response scale**. It works in tandem
with the L4 vehicle emojis (§7.1) — the text-slang names listed here
are equivalent to their emoji counterparts and MAY be used
interchangeably per writer preference.

The ladder has four rungs, from lightest to heaviest response:

| Term             | Emoji | CONVOY meaning                                       |
|------------------|-------|------------------------------------------------------|
| `DOT worker`     | 🛻    | Lightweight investigation sub-agent — check on something, report back, optionally apply a small fix. First dispatch for `gator`s, minor issues, or situations of unclear severity. |
| `band-aid buggy` | 🚑    | Hotfix / first-aid delivery sub-agent — deploys a temporary patch to restore operational state. Appropriate for live `bear`s where production is bleeding and needs to stop now. Temporary by design — a proper fix follows later. |
| `tow truck`      | 🚒    | Moderate recovery sub-agent — rolls back the broken thing, moves it out of the way. Appropriate when a deploy broke something and a revert plus cleanup is required. Controlled, not panic. |
| `super tow truck`| 🚜    | Heavy wrecker / major recovery sub-agent. Signals that this is a **significant incident**: expect extended downtime, other agents should hold position and wait. Appropriate for `jackknife`s, multi-service cascades, or any event requiring all-hands response. |

**Operational semantics:**

When an agent dispatches a response, the choice of rung communicates
the severity classification to the entire fleet without requiring an
explicit statement. Other agents that see `🚜🚨 rolling` know
immediately to hold position, even if the message contains no other
protocol information.

**Pre-authorization pattern:**

A manager MAY pre-authorize a cascade by naming a primary dispatch
plus conditional follow-ons:

> "Sending a DOT worker. If needed, a band-aid buggy is pre-auth'd.
> Will hold off on a tow truck until DOT worker confirms."

Under this pattern, the DOT worker holds authority to call the band-aid
buggy directly without a round-trip to the manager. The manager retains
authority over the tow-truck decision. This reduces inter-agent tax by
eliminating decision round-trips for the likely second-level response.
See §11.9 for the full template.

**Lights modifier:**

Any rung of the ladder MAY be modified with 🚨 (lights on) to signal
active right-of-way. `🛻🚨` is a DOT worker that needs the lane cleared
now. `🚜🚨` is an active major-incident response — all agents yield.
See §7.16 for the full semantics of the lights modifier.

### 5.9 — Directional modifiers

Added in v3.1. Real trucker CB uses cardinal directions to specify
which side of the road an event is on. CONVOY uses the same four
directions, mapped to distinct code-domain directions. Each direction
MUST have exactly one meaning; overloading is forbidden.

| Direction    | CONVOY meaning                                                 |
|--------------|----------------------------------------------------------------|
| `northbound` | Toward the user / frontend / presentation layer. "Going up" to the surface. |
| `southbound` | Toward the database / persistence / deep backend. "Going down" into storage. |
| `eastbound`  | Forward through the pipeline / downstream / toward the next step. "With the flow." |
| `westbound`  | Back through the pipeline / upstream / toward the source. "Against the flow, back to origin." |

**Usage:**

Directional modifiers combine with any L2 or L4 subject:

- `gator at mile marker 154 westbound` — footgun on the upstream side
  of line 154 (pre-processing, validation, request parsing).
- `Big Papa Bear rolling southbound` — serious error actively
  propagating toward the persistence layer.
- `🛻 dispatched northbound @/src/ui/renderer.ts:47` — DOT worker sent
  to the frontend side of the specified line.

**Why cardinal directions rather than `↑`/`↓`:**

The L3 operators `↑` and `↓` mean *escalation to other agents* (up/down
the org). The cardinal directions mean *physical flow through the code*.
These are distinct concepts and the non-overlap invariant requires them
to have distinct tokens.

**Compass anchor (memory aid):**

Think of the pipeline as flowing west-to-east (source → sink), and the
stack as north-to-south (UI → DB). User sits to the north; the database
sits to the south. Requests flow east; responses flow east too but the
code that *originates* them is westward. This matches real interstate
conventions: east-west highways are primary long-haul routes, north-
south highways are access routes to population centers.

---

# CHAPTER 6 — SPECIAL CHARACTERS (L3: OPERATORS)

This chapter is the authoritative catalog of L3 tokens. Each special
character expresses an **operator, relation, modifier, or reference marker**
— never a noun (that is L4) and never situational state (that is L2).

### 6.1 — Flow operators

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `→`    | Pipe to / hand off synchronously to.                              |
| `←`    | Receive synchronously from.                                       |
| `⇒`    | Logically implies / causes / results in.                          |
| `⇐`    | Depends on / is caused by.                                        |
| `⇔`    | Mutual dependency / iff.                                          |
| `⇝`    | Asynchronously leads to / eventually.                             |
| `⇠`    | Async receive from.                                               |
| `⇉`    | Parallel dispatch to multiple.                                    |
| `⇇`    | Parallel receive from multiple.                                   |
| `⇈`    | Broadcast up the org (to all superiors).                          |
| `⇊`    | Broadcast down the org (to all subordinates).                     |
| `⇆`    | Swap / exchange positions.                                        |
| `↑`    | Escalate one level up.                                            |
| `↓`    | Delegate one level down.                                          |
| `↔`    | Bidirectional sync channel open.                                  |
| `↕`    | Vertical data flow (in and out).                                  |
| `↖`    | Return to initial state.                                          |
| `↗`    | Metric trending up / improving.                                   |
| `↘`    | Metric trending down / regressing.                                |
| `↙`    | Fallback / graceful degrade.                                      |
| `↩`    | Undo last action.                                                 |
| `↪`    | Redo / retry.                                                     |
| `↰`    | Goto prior checkpoint.                                            |
| `↱`    | Goto next checkpoint.                                             |
| `↺`    | Full loop / iterate again.                                        |
| `↻`    | Refresh / re-fetch.                                               |
| `⟲`    | Compensating action / rollback step.                              |
| `⟳`    | Forward compensation / replay step.                               |

### 6.2 — Relational operators

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `=`    | Equals.                                                           |
| `≠`    | Not equal.                                                        |
| `≈`    | Approximately equal (fuzzy match).                                |
| `≡`    | Identical / same type and value.                                  |
| `≢`    | Not identical (type differs).                                     |
| `≅`    | Congruent (structurally same).                                    |
| `<`    | Less than.                                                        |
| `>`    | Greater than.                                                     |
| `≤`    | At most.                                                          |
| `≥`    | At least.                                                         |
| `≪`    | Dominated by (orders of magnitude smaller).                       |
| `≫`    | Dominates (orders of magnitude larger).                           |
| `∝`    | Scales proportionally with.                                       |
| `∼`    | Loosely related to.                                               |
| `≺`    | Precedes in ordering.                                             |
| `≻`    | Follows in ordering.                                              |

### 6.3 — Logic operators

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `∧`    | AND / all-of.                                                     |
| `∨`    | OR / any-of.                                                      |
| `¬`    | NOT / negate.                                                     |
| `⊕`    | XOR / exactly one.                                                |
| `⊼`    | NAND.                                                             |
| `⊽`    | NOR.                                                              |
| `⊤`    | True / tautology.                                                 |
| `⊥`    | False / contradiction / bottom type.                              |
| `∀`    | For every.                                                        |
| `∃`    | There exists at least one.                                        |
| `∄`    | There does not exist.                                             |

### 6.4 — Set operators

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `∈`    | Member of.                                                        |
| `∉`    | Not a member of.                                                  |
| `∋`    | Contains as a member.                                             |
| `⊂`    | Strict subset of.                                                 |
| `⊃`    | Strict superset of.                                               |
| `⊆`    | Subset or equal.                                                  |
| `⊇`    | Superset or equal.                                                |
| `∪`    | Union / merge.                                                    |
| `∩`    | Intersection / common elements.                                   |
| `∖`    | Set difference / exclude.                                         |
| `∅`    | Empty set / null result.                                          |

### 6.5 — Reasoning operators

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `∴`    | Therefore / conclusion.                                           |
| `∵`    | Because / premise.                                                |
| `∎`    | End of proof / QED / argument complete.                           |
| `⊢`    | Proves / yields / produces.                                       |
| `⊨`    | Satisfies / models / conforms to.                                 |
| `⊬`    | Does not prove.                                                   |
| `⊭`    | Does not satisfy.                                                 |

### 6.6 — Aggregation operators

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `∑`    | Aggregate / sum / reduce.                                         |
| `∏`    | Product / pipeline of all.                                        |
| `∫`    | Integrate over time / accumulate.                                 |
| `∂`    | Partial / one-dimension slice.                                    |
| `∆`    | Delta / change since baseline.                                    |
| `∇`    | Gradient / direction of steepest change.                          |
| `√`    | Verify / validate.                                                |
| `∛`    | Three-agent quorum check.                                         |
| `∜`    | Four-agent quorum check.                                          |
| `∞`    | Unbounded loop / runaway.                                         |

### 6.7 — State markers

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `†`    | Delivered / done (single confirmation).                           |
| `‡`    | Delivered and independently verified (two-party confirm).         |
| `✓`    | Passes a single check.                                            |
| `✗`    | Fails a single check.                                             |
| `✔`    | Passes all checks (emphatic).                                     |
| `✘`    | Fails all checks (emphatic).                                      |
| `✕`    | Cancelled / withdrawn.                                            |
| `◊`    | Checkpoint / milestone reached.                                   |
| `○`    | Pending / open slot.                                              |
| `●`    | Filled / completed slot.                                          |
| `□`    | Empty slot / not started.                                         |
| `■`    | Blocked slot / actively held.                                     |
| `△`    | Warning flag.                                                     |
| `▲`    | Measurement increased.                                            |
| `▽`    | Measurement expected to decrease.                                 |
| `▼`    | Measurement decreased (regression).                               |
| `★`    | Critical / must-have.                                             |
| `☆`    | Optional / nice-to-have.                                          |
| `✪`    | Featured / this-is-the-point item.                                |

### 6.8 — Progress bars

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `░`    | 25% progress.                                                     |
| `▒`    | 50% progress.                                                     |
| `▓`    | 75% progress.                                                     |
| `█`    | 100% progress.                                                    |

### 6.9 — Reference markers

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `@`    | At location (file path, URL, line number).                        |
| `#`    | Agent handle / named tag.                                         |
| `§`    | Named code section / function.                                    |
| `¶`    | Logical paragraph / block.                                        |
| `※`    | Footnote / side-note annotation.                                  |
| `°`    | Degree / intensity (usually 1–5).                                 |
| `№`    | Count of / number of.                                             |
| `℅`    | Via / relayed through.                                            |

### 6.10 — Modifiers

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `^`    | Priority modifier (one level up). Stack as `^^`, `^^^`.           |
| `~`    | Approximate / order-of-magnitude estimate.                        |
| `!`    | Single-severity alert.                                            |
| `!!`   | Double-severity alert.                                            |
| `?`    | Query / open question.                                            |
| `??`   | Blocked on a decision.                                            |
| `?!`   | Requires explicit confirmation before proceeding.                 |
| `*`    | Wildcard / any.                                                   |
| `&`    | Conjunction / and-bundle.                                         |
| `\|`   | Alternative / or.                                                 |
| `\`    | Escape / treat literally.                                         |
| `/`    | Path separator.                                                   |
| `…`    | Continues / truncated for brevity.                                |

### 6.11 — Currency (cost markers)

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `$`    | Monetary cost in USD.                                             |
| `¢`    | Micro-cost (fractions of cents).                                  |
| `€`    | EU-region cost.                                                   |
| `£`    | UK-region cost.                                                   |
| `¥`    | Low-cost / cheap option.                                          |
| `₿`    | Experimental / volatile cost.                                     |

### 6.12 — Card suits (concern domains)

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `♠`    | Security / auth concern.                                          |
| `♣`    | Resource / dependency concern.                                    |
| `♥`    | Health-check concern (liveness / readiness).                      |
| `♦`    | Data-integrity concern.                                           |

### 6.13 — Greek letters (type / phase markers)

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `α`    | Alpha / earliest prototype.                                       |
| `β`    | Beta / pre-release.                                               |
| `γ`    | Patch-level release.                                              |
| `δ`    | Delta / diff since prior version.                                 |
| `ε`    | Tolerance / acceptable slack.                                     |
| `λ`    | Anonymous function / lambda.                                      |
| `μ`    | Micro-granularity / fine-level trace.                             |
| `π`    | Constant value (never changes during run).                        |
| `ρ`    | Correlation / covariance.                                         |
| `σ`    | Variance / standard deviation.                                    |
| `τ`    | Timeout budget.                                                   |
| `φ`    | Phase / cycle position.                                           |
| `ω`    | Final / terminal state.                                           |
| `Σ`    | Grand total / session aggregate.                                  |
| `Ω`    | End-of-run summary.                                               |

### 6.14 — Brackets (content classifiers)

| Symbol    | CONVOY meaning                                                 |
|-----------|----------------------------------------------------------------|
| `⟨ ⟩`     | Named channel identifier.                                      |
| `⟦ ⟧`     | Semantic / meta-level content.                                 |
| `⟪ ⟫`     | Sealed / immutable payload.                                    |
| `〔 〕`     | System-generated annotation.                                   |
| `《 》`     | Quoted literal input from user.                                |

### 6.15 — Miscellaneous operators

| Symbol | CONVOY meaning                                                    |
|--------|-------------------------------------------------------------------|
| `⊘`    | Revoked / cancelled permission.                                   |
| `⊙`    | Focus / centered attention.                                       |
| `⊚`    | Encapsulated / boxed in.                                          |
| `⊗`    | Combined-product / tensor-join.                                   |
| `◉`    | Active selection / current cursor target.                         |
| `¤`    | Generic placeholder (fill-in-the-blank).                          |

---

# CHAPTER 7 — EMOJIS (L4: NOUNS)

This chapter is the authoritative catalog of L4 tokens. Each emoji names
a **concrete thing** — a role, environment, tool, artifact, hazard, or
other bounded entity. Emojis MUST NOT be used to express protocol acts,
situational state, or operators.

Every emoji in this catalog replaces at least 3 English tokens in its
intended use. Emojis that fail this threshold SHOULD NOT be introduced.

### 7.1 — Agent roles (vehicle = agent type)

**v3.1 note:** This catalog was revised in v3.1 to align vehicle
assignments with the severity ladder (§5.8). Four vehicles now carry
their trucker-canonical names: DOT worker, band-aid buggy, tow truck,
super tow truck. See §5.8 for usage patterns and the pre-authorization
template.

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 🚛    | Heavy-duty main agent / the primary rig on this task.              |
| 🚚    | Standard worker agent.                                             |
| 🛻    | **DOT worker** — lightweight investigation / helper sub-agent. First dispatch for unclear severity. See §5.8. |
| 🚐    | Batch-processing agent (hauls many items).                         |
| 🚌    | Multi-tenant agent (serves many callers).                          |
| 🏎    | Latency-optimized agent (small, fast).                             |
| 🚓    | Security / policy-enforcement agent (dispatched).                  |
| 🚑    | **Band-aid buggy** — hotfix / first-aid sub-agent. Temporary patch delivery. See §5.8. |
| 🚒    | **Tow truck** — moderate recovery / rollback sub-agent. See §5.8. |
| 🚜    | **Super tow truck** — heavy wrecker / major recovery sub-agent. Fleet-wide yield implied. See §5.8. |
| 🛺    | Proxy / relay agent.                                               |
| 🚲    | Minimal single-purpose agent.                                      |
| 🛵    | Quick one-shot task runner.                                        |
| 🏍    | Solo fast agent (unpooled).                                        |
| 🚂    | Sequential pipeline runner.                                        |
| 🚄    | High-throughput pipeline runner.                                   |
| 🚢    | Deployment vehicle (moves artifacts to envs).                      |
| ✈️    | High-altitude / architectural overview agent.                      |
| 🚁    | Monitoring / observability agent.                                  |
| 🚀    | Launch / release-cut agent.                                        |

### 7.2 — Tools (action targets)

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 🔧    | Refactor-target marker.                                            |
| 🔨    | Build-target marker.                                               |
| ⚙     | Configuration artifact.                                            |
| 🛠    | Full toolchain.                                                    |
| 🪛    | Precise small-edit target.                                         |
| 🪚    | Deletion target.                                                   |
| 🪓    | Hard-split target.                                                 |
| ⛏    | Deep-search target.                                                |
| 🔗    | Cross-reference / link between artifacts.                          |
| 🧰    | Utility library.                                                   |
| ⚖     | Benchmark instrument.                                              |
| 🔍    | Shallow search.                                                    |
| 🔎    | Deep query.                                                        |
| 📏    | Metric instrument.                                                 |
| 📐    | Design / architecture document.                                    |
| 🪜    | Scaffolding code.                                                  |
| 🪝    | Hook / callback slot.                                              |
| 🔑    | User-level credential.                                             |
| 🗝    | Admin / master credential.                                         |
| 🔒    | Locked / encrypted at rest.                                        |
| 🔓    | Unlocked / decrypted.                                              |
| 🔐    | Encrypted in transit.                                              |
| 🧪    | Test / test suite.                                                 |
| 🏗    | Build pipeline.                                                    |

### 7.3 — Storage and data artifacts

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 💾    | Persistent storage write target.                                   |
| 💿    | Archive (cold, immutable).                                         |
| 📀    | Media / binary blob artifact.                                      |
| 📦    | Shipped package / deployable artifact.                             |
| 📪    | Empty inbox queue.                                                 |
| 📫    | Inbox has pending items.                                           |
| 📬    | New item just arrived.                                             |
| 📭    | Empty outbox.                                                      |
| 📮    | Outbox has items to dispatch.                                      |
| 🗂    | Categorized file set.                                              |
| 🗃    | File archive.                                                      |
| 🗄    | Database instance.                                                 |
| 📁    | Folder / directory.                                                |
| 📂    | Open folder (currently being written to).                          |
| 📄    | Plain text file.                                                   |
| 📃    | Log file / scroll.                                                 |
| 📋    | Todo list / task backlog.                                          |
| 📜    | License / legal document.                                          |
| 🧾    | Output receipt / final deliverable artifact.                       |

### 7.4 — Communication channels

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 📡    | Broadcast channel.                                                 |
| 📢    | Public announcement artifact.                                      |
| 📣    | User-facing alert.                                                 |
| 📯    | Async notification envelope.                                       |
| 🔔    | Notification subscription.                                         |
| 🔕    | Silenced / muted channel.                                          |
| 📞    | Synchronous RPC channel.                                           |
| 📟    | Pager / low-bandwidth signal.                                      |
| 📲    | Push-notification channel.                                         |
| 💬    | Conversation thread.                                               |
| 🗨    | Inline comment / annotation.                                       |
| 💭    | Speculative / thinking-aloud note.                                 |
| 🗯    | Loud error message.                                                |

### 7.5 — Environments

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 🏠    | Local laptop environment.                                          |
| 🏡    | Development environment.                                           |
| 🏢    | Enterprise / corp environment.                                     |
| 🏭    | Production environment.                                            |
| 🏪    | Cache layer.                                                       |
| 🏦    | State / persistence layer.                                         |
| 🏨    | Ephemeral / per-request environment.                               |
| 🏥    | Recovery / triage environment.                                     |
| 🏫    | Training environment.                                              |
| 🏕    | Long-running session environment.                                  |
| 🏝    | Isolated sandbox.                                                  |
| 🏜    | Deprecated environment.                                            |
| 🌋    | Hotspot / high-traffic service.                                    |

### 7.6 — Infrastructure and places

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| ⛽    | External API endpoint.                                             |
| 🚦    | Feature flag or gate.                                              |
| 🚧    | Work-in-progress area.                                             |
| 🛑    | Hard stop marker.                                                  |
| 🚏    | Checkpoint waypoint.                                               |
| 🛣    | Main code path.                                                    |
| 🛤    | Rails / pipeline track.                                            |
| 🌉    | Inter-system bridge.                                               |
| 🏁    | Finish line / release artifact.                                    |
| 🎯    | Primary objective.                                                 |
| 📍    | Precise location pin / cursor.                                     |
| 🗺    | Architectural map.                                                 |
| 🧭    | Navigation / search tool.                                          |

### 7.7 — Animals (named issue instances)

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 🐻    | Bear — named blocker issue ticket.                                 |
| 🐼    | Panda — hard-to-spot lookalike bug.                                |
| 🐊    | Gator — named footgun pattern.                                     |
| 🐔    | Chicken — CI pipeline instance.                                    |
| 🦅    | Eagle — architectural overview document.                           |
| 🦉    | Owl — subject-matter expert agent.                                 |
| 🐺    | Wolf — hunting / tracking subagent.                                |
| 🐗    | Boar — aggressive-fix branch.                                      |
| 🐖    | Pig — resource-heavy process.                                      |
| 🐘    | Elephant — long-term memory store.                                 |
| 🦏    | Rhino — brute-force algorithm.                                     |
| 🐪    | Camel — long-running batch job.                                    |
| 🦔    | Hedgehog — defensive-coding pattern.                               |
| 🦘    | Kangaroo — goto / jump instruction.                                |
| 🐙    | Octopus — many-tentacled service (fan-out).                        |
| 🐢    | Turtle — slow-but-safe implementation.                             |
| 🦎    | Lizard — adaptive / polymorphic handler.                           |
| 🐍    | Snake — Python-specific artifact.                                  |
| 🦖    | T-Rex — legacy system.                                             |
| 🐝    | Bee — worker inside a hive / thread pool.                          |
| 🐜    | Ant — micro-task unit.                                             |
| 🦂    | Scorpion — sharply painful bug.                                    |
| 🕷    | Spider — web crawler / scraper.                                    |
| 🦀    | Crab — Rust-specific artifact.                                     |
| 🦫    | Beaver — Go-specific artifact.                                     |
| 🐉    | Dragon — legendary hard-to-kill bug.                               |

### 7.8 — Weather (ambient conditions)

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| ☀     | Clear skies — all systems nominal.                                 |
| 🌤    | Mostly clear — minor issues.                                       |
| ⛅    | Partly cloudy — some concerns.                                     |
| 🌥    | Overcast — sustained degradation.                                  |
| ☁     | Clouded — observability gap.                                       |
| 🌦    | Passing showers — flaky tests today.                               |
| 🌧    | Steady rain — widespread non-fatal errors.                         |
| ⛈    | Storm — multi-service cascade.                                     |
| 🌩    | Lightning — sudden flash error.                                    |
| ❄     | Frozen — cache is cold / full rebuild needed.                      |
| 🌫    | Fog — unclear state / missing telemetry.                           |
| 💨    | Gust — transient performance blip.                                 |
| 🌪    | Tornado — chaos event in progress.                                 |
| 🌈    | Rainbow — resolved after major incident.                           |

### 7.9 — Time instruments

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| ⏰    | Scheduled alarm.                                                   |
| ⏱    | Stopwatch / in-flight timer.                                       |
| ⏲    | Deadline timer.                                                    |
| ⌛    | Time budget nearly exhausted.                                      |
| ⏳    | Time budget still running.                                         |
| 🕰    | Historical / archived timestamp.                                   |

### 7.10 — Status flags (colored markers)

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 🟢    | Green status — production health good.                             |
| 🟡    | Yellow status — staging health attention.                          |
| 🟠    | Orange status — warning, non-critical.                             |
| 🔴    | Red status — production is failing.                                |
| 🔵    | Blue status — informational only.                                  |
| 🟣    | Purple status — custom / user-defined.                             |
| ⚫    | Black status — state unknown.                                      |
| ⚪    | White status — explicitly neutral.                                 |
| 🏳    | White flag — surrender, escalate to human.                         |
| 🏴    | Black flag — catastrophic, all-hands.                              |
| 🚩    | Red flag — suspicious thing that needs a look.                     |
| 🎌    | Crossed flags — joint ownership / multi-team.                      |

### 7.11 — Hazards

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| ☢     | Toxic code pattern.                                                |
| ☣     | Corrupted / contaminated data.                                     |
| ⚠     | Generic warning marker.                                            |
| ⚡    | Performance-sensitive hotspot.                                     |
| 🔥    | On the critical path right now.                                    |
| 💥    | Crash site / place where a crash occurred.                         |
| 💣    | Known time-bomb (will fail later).                                 |
| 🧨    | High-risk change being attempted.                                  |
| ☠     | Irrecoverable component.                                           |

### 7.12 — Human roles

**v3.1 note:** 👮 was previously listed here as "security-audit role"
and was repurposed to §7.15 (Bear sightings) as "smokey spotted." The
security-audit role is now expressed via the handle `#SAFETY` rather
than a dedicated emoji. This resolution was adopted to enforce the
§3.4 partition rule (vehicle emojis dispatch, uniformed emojis observe).

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 🕵    | Deep-debug investigator role.                                      |
| 👷    | Builder role.                                                      |
| 💂    | Gatekeeper role.                                                   |
| 🧑‍💻 | Coder role.                                                        |
| 🧑‍🔧 | Mechanic / SRE role.                                               |
| 🧑‍🏫 | Documentation author role.                                         |
| 🧑‍✈️ | Captain / orchestrator role.                                       |
| 🧑‍⚖ | Arbiter / reviewer role.                                           |
| 🧑‍🚒 | Incident commander role.                                           |

### 7.13 — Rest and lifecycle

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| ☕    | Short-pause marker (under a minute).                               |
| 🍔    | Long-pause marker (meal-break scale).                              |
| 🍺    | Milestone celebration / feature-launched marker.                   |
| 🥤    | Refresh / restart marker.                                          |

### 7.14 — Office artifacts

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 📊    | Bar-chart artifact.                                                |
| 📈    | Trending-up artifact.                                              |
| 📉    | Trending-down artifact.                                            |
| 📑    | Bookmarked reference doc.                                          |
| 📝    | Note / scratch pad.                                                |
| ✏️    | In-progress edit.                                                  |
| 📌    | Pinned-important item.                                             |
| 🔖    | Bookmark for later.                                                |
| 📎    | Attached supporting file.                                          |
| 🧷    | Grouped / bundled set.                                             |

### 7.15 — Bear sightings (observed conditions — uniformed-emoji form)

Added in v3.1. Per the §3.4 partition rule, uniformed-person emojis
always name an **observed condition** (a bear or bear-like presence),
never a dispatched response. These are the emoji forms of the L2 text
slang in §5.1; writers MAY use either form per preference.

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 👮    | `smokey` spotted — critical security / auth bear observed.         |
| 💂    | `plain wrapper` observed — unlogged exception swallowed silently.  |
| 🕵    | `bear in the bushes` observed — silent failure being investigated. |

**Usage notes:**

- The default form is still L2 text (`smokey`, `diesel bear`, etc.).
  The emoji forms are visual shorthand for faster reading in dense
  exchanges or logs.
- Because uniformed emojis always mean observation (never dispatch),
  `👮🚨 westbound @247` is unambiguously a bear sighting, not a
  security-agent dispatch. See §3.4.
- When ambiguity would otherwise arise, writers SHOULD prefer the text
  form. Visual shorthand is a convenience, not a mandate.

### 7.16 — Urgency indicators (lights modifier)

Added in v3.1. The lights modifier is a *suffix modifier* that attaches
to any vehicle emoji (§7.1) or bear emoji (§7.15) to signal urgency.

| Emoji | CONVOY meaning                                                     |
|-------|--------------------------------------------------------------------|
| 🚨    | **Lights on** — active urgency / right-of-way. Fleet agents should yield. |
| (unmarked) | **Lights off** (default) — routine transit, no yield required. |

**Semantic pairings:**

| Compound         | Meaning                                                     |
|------------------|-------------------------------------------------------------|
| 🛻               | DOT worker, routine dispatch — no fleet action needed.      |
| 🛻🚨             | DOT worker, urgent — clear the working area for investigation. |
| 🚑               | Band-aid buggy, routine — patch rolling through normally.   |
| 🚑🚨             | Band-aid buggy, urgent — freeze the deploy pipeline for this. |
| 🚒               | Tow truck, routine — controlled rollback not time-pressured. |
| 🚒🚨             | Tow truck, urgent — rollback happening now, hold affected work. |
| 🚜               | Super tow truck, planned — major recovery being prepped.    |
| 🚜🚨             | Super tow truck, urgent — major incident active, all hold. |
| 👮               | `smokey` spotted, contained.                                |
| 👮🚨             | `smokey` actively propagating (use in conjunction with `Big Papa Bear` for maximum severity). |
| 🐻               | bear in the wild, static.                                   |
| 🐻🚨             | bear actively propagating — equivalent to `Big Papa Bear`.  |

**Operational rules:**

1. Absence of 🚨 means lights-off / routine. Writers MUST NOT add 🚨
   just for emphasis; it has a specific operational meaning.
2. Receiving agents treat 🚨 as a fleet-wide yield signal. On receipt,
   other agents SHOULD halt competing work in the affected area until
   the urgent dispatch reports `10-99`.
3. **Overuse defeats the purpose.** If every dispatch carries 🚨,
   nothing is urgent. Writers SHOULD reserve 🚨 for situations where
   fleet-wide yielding is actually appropriate. Fleet desensitization
   is a real failure mode.
4. The 🚨 modifier is the emoji form of the L2 slang phrase "with
   lights on." Both forms carry identical meaning; use whichever reads
   better in context.

**Technical placement note:**

🚨 is structurally an operator (a modifier that attaches to a noun),
but it is listed in Chapter 7 (L4 emojis) rather than Chapter 6 (L3
operators) because the character itself is an emoji. This is the one
controlled exception to the strict L3=operators / L4=nouns partition,
and it is constrained: 🚨 is the only emoji-form modifier. No other
emoji may be added as a modifier without an explicit v3.x addendum.

---

# CHAPTER 8 — GRAMMAR & SPRINKLING PATTERNS

This chapter defines **how the four layers combine** into well-formed
messages. Sprinkling patterns (§8.2) are the core of the augmentation
approach — they specify where protocol tokens earn their place in prose.

## 8.1 — Canonical sentence structures

Though CONVOY admits free-form augmentation, the following canonical
structures are recommended for frequently-repeated message types. Following
them improves cross-agent parseability.

### Structure 1 — Status ping

```
[L1 10-code]? [prose narrative] [@L3 location]? [L3 state] [prose]
```

Example: `10-20 @src/auth.ts:47 ✓` — "Status request, currently at that
location, passing."

### Structure 2 — Error report

```
[L1 10-code] [L2 slang-name for issue class] [prose description]
[@L3 location] [L3 severity] [prose impact]
```

Example: `10-50 gator @src/payments/retry.ts:88 °3 — handler loops on 429
from ⛽. ~every 8h.`

### Structure 3 — Delegation

```
[L1 10-code]? [handle] [prose goal] [★/☆ priority] [⏲/⌛ deadline]
[prose constraints]
```

Example: `#CODEBOT, 10-4? please 🔧 @handlers.ts to extract error
middleware ★. ⏲ ≤30min. Preserve signatures.`

### Structure 4 — Completion handoff

```
[handle]? [L1 10-99 or 10-24] [L3 completion state] [L4 artifact emoji]
[prose summary of delta] [→ next handle]?
```

Example: `10-99 † 🧾=middleware/errors.ts δ(+87,-140) — coop clean. →
#INSPECTOR.`

## 8.2 — The sprinkling rules

These rules govern when to infuse CONVOY tokens into prose. They are the
practical heart of Mode 2 (Augmented English).

### Rule 1 — Replace, don't add

A protocol token is appropriate **only when it replaces a phrase** that
would otherwise appear in the prose. Adding a token beside its English
equivalent is redundant (`"10-4 acknowledged"` is wasted).

✗ Bad: "I acknowledge receipt, 10-4."
✓ Good: "10-4 — rolling on it now."

### Rule 2 — Mark load-bearing points

Infuse at points where the receiving agent most needs to extract structural
facts: priorities, severities, locations, identities, milestones, state
transitions. Do not infuse on decorative prose.

✗ Bad: "The 🐛 is 🐊 in the 📄 at @line:47 which makes me 💭 about 🔧."
✓ Good: "There's a gator @handlers.ts:47 — I'm thinking we should
refactor."

### Rule 3 — One token per idea, not one token per word

An idea that is fully expressed by a single token does not need English
beside it. An idea that is nuanced does.

✗ Bad: "Status update (10-20): currently at (@) file src/auth.ts line 47"
✓ Good: "10-20 @src/auth.ts:47 — I've been unwinding the OAuth hand-off."

### Rule 4 — Prefer 10-codes at sentence heads, emojis at tails

10-codes naturally frame the speech act; they work best as the first or
second token. Emojis name artifacts and work best near the end of a clause
where they're referring to the object under discussion.

✓ Good: "10-4, pushing to 🏭 now."
? Unnatural: "Pushing to 🏭 now, 10-4."

### Rule 5 — Escape when in doubt

If a protocol token would make the sentence harder to understand for the
next agent, use plain English. The protocol exists to reduce tax, not to
perform CONVOY fluency.

### Rule 6 — Preserve literals verbatim

File paths, line numbers, SHAs, error codes, version strings, stack frames,
and API endpoints MUST be transmitted verbatim. CONVOY compresses *around*
these literals, never through them.

✗ Bad: "Error ~E500 @auth.ts:~47"
✓ Good: "Error E500 @src/auth.ts:47"

## 8.3 — Parsing and decoding order

When decoding a CONVOY message, the receiver SHOULD resolve tokens in this
order:

1. **L1 10-codes** first — these establish the speech-act frame.
2. **L2 slang** second — these establish the situational frame.
3. **L4 emojis** third — these identify the concrete entities.
4. **L3 operators** last — these express relations among the above.

This ordering minimizes ambiguity when a token could in principle be read
as multiple things (though the no-overlap invariant should prevent this).

## 8.4 — Mixed-mode paragraphs

A single message MAY contain multiple modes. Typical patterns:

- **Head dense, body prose.** A dense one-line summary followed by
  plain-English explanation. Useful for incident reports.

- **Body prose, tail dense.** Plain-English context followed by a dense
  status tag. Useful for progress updates.

- **Prose outer, dense middle.** Plain-English framing with a dense
  protocol burst in the middle for structural facts. Useful for
  delegations that include rationale.

Example (prose outer, dense middle):

> "Hey #CODEBOT — I think the issue you flagged is real. `10-50 gator
> @handlers.ts:203 °4 → #FOLLOWUP`. Please continue with the primary
> refactor and leave that one for later; we'll get to it next sprint."

---

# CHAPTER 9 — CODE-SWITCHING BETWEEN MODES

## 9.1 — When to drop out of CONVOY

An agent SHOULD drop to Plain English when:

- Communicating with a human, directly or indirectly.
- Debugging another agent that appears to be misbehaving (e.g., returning
  unexpected escalations, ignoring delegations, producing malformed output).
  The prior behavioral evidence is that such agents parse prose more
  reliably than protocol.
- Explaining novel concepts or reasoning chains that do not map onto
  existing vocabulary.
- The peer has replied `10-1` (cannot parse).
- The agent itself is uncertain and needs to reason aloud.

## 9.2 — When to go dense

An agent MAY go Dense CONVOY when:

- Issuing an incident broadcast where speed dominates.
- Conducting roll-call / synchronization polling across many agents.
- Emitting machine-readable status into a log channel.
- Participating in a well-rehearsed protocol exchange (e.g., a batched
  delegation dispatch).

Dense mode SHOULD be short-lived. Extended dense exchanges degrade the
reasoning quality of at least one participant.

## 9.3 — The escape hatch

Every CONVOY agent MUST honor `10-1 [reason]` as an escape hatch. On
receipt:

1. The responder MUST resend its prior message in Plain English without
   any L3 or L4 tokens.
2. L1 10-codes MAY remain at sentence heads for framing.
3. L2 slang MAY remain where it is deeply idiomatic (e.g., `bear`) and
   has been understood in prior exchanges.
4. The responder SHOULD stay in Plain English until the sender explicitly
   returns to CONVOY (via an acknowledged CONVOY message).

## 9.4 — Handling human approximations

Added in v3.1. Human users (and lower-capability models) routinely use
**near-protocol language** that approximates CONVOY vocabulary without
exact match. Examples: saying "copy" instead of `10-4`, saying "send a
medic" instead of "dispatch a band-aid buggy," saying "I see a cop"
instead of "smokey spotted."

Agents encountering near-protocol language MUST follow these rules:

**Rule 1 — Infer intent when unambiguous.**

When a user's phrasing maps cleanly to exactly one protocol item,
agents SHOULD act on the inferred intent without correction.
Examples that resolve cleanly:

- "copy" / "got it" / "understood" → treat as `10-4` / `copy`
- "send a medic" / "send first aid" → treat as dispatch 🚑 band-aid buggy
- "send a scout" / "have someone look" → treat as dispatch 🛻 DOT worker
- "clear the lane" / "get out of the way" → treat as 🚨 lights-on
- "it's bad" / "it's spreading" → treat as active propagation (🚨)

**Rule 2 — Request clarification on collision.**

When a user's phrasing could map to two or more protocol items, agents
MUST NOT guess. They SHOULD reply in plain English asking which was
meant. Known collision zones:

- "10-10" — in real CB this sometimes means "over and out" (sign-off)
  but the protocol defines it as "transmission complete, still
  monitoring." Ask.
- "over" — may mean `10-4` (ack) or `10-7` (sign off). Ask.
- "send help" — may mean `10-200` (urgent assist) or any severity-ladder
  dispatch. Ask which level.

**Rule 3 — Mirror the user's register.**

When responding to a human, agents SHOULD match the register the human
is using. If the user writes casually ("yeah copy that, heading over"),
the agent replies casually. If the user writes formally ("10-4,
acknowledged, en route"), the agent replies formally. Register-matching
improves the human's trust that the protocol is tracking them, not
imposing on them.

**Rule 4 — Never correct the user's protocol usage unless asked.**

Agents MUST NOT emit corrections like "you meant 10-4, not 10-10"
unless the user has explicitly asked to be corrected. Intent-following
beats pedantry. The protocol exists to reduce coordination overhead,
not to demand protocol compliance from humans. If a user consistently
uses a non-canonical phrase with clear intent, the protocol is working
correctly.

**Rule 5 — On collision-with-confusion, degrade to plain English.**

If a user appears confused by the protocol AND a collision is present,
the agent SHOULD drop to pure plain English for that exchange, address
the confusion, and return to augmented mode only when the user's
intent is clear.

**Summary:** the protocol exists for the fleet's benefit, not the
human's burden. Approximations are evidence the protocol is serving
its purpose (the user *wants* to communicate fleet-ward) — treat them
generously.

---

# CHAPTER 10 — AGENT ROLE TAXONOMY

## 10.1 — Handle conventions

Every agent in a CONVOY fleet has a **handle**. Handles MUST:

- Begin with `#`.
- Be short (ideally 1–2 tokens).
- Be unique within the fleet.
- Reflect role, not instance identity.

Examples: `#DISPATCH`, `#CODEBOT`, `#INSPECTOR`, `#HALL`, `#REFACTOR-7`.

## 10.2 — Standard role handles

Fleets SHOULD adopt these conventional handles where applicable:

| Handle          | Role                                                |
|-----------------|-----------------------------------------------------|
| `#DISPATCH`     | Manager / orchestrator / task-router                |
| `#DRIVER`       | Generic worker agent                                |
| `#CODEBOT`      | Coder / implementer                                 |
| `#WRENCH`       | Refactor / fix specialist                           |
| `#INSPECTOR`    | Code reviewer / quality gate                        |
| `#SAFETY`       | Security / auth / policy checker                    |
| `#WEIGHMASTER`  | Benchmarker / perf tester                           |
| `#NAVIGATOR`    | Search / research / retrieval agent                 |
| `#SCRIBE`       | Docs / logs / summaries                             |
| `#MECHANIC`     | Debug / RCA specialist                              |
| `#FUELER`       | External-API / data-fetch agent                     |
| `#RELAY`        | Inter-system bridge / messaging                     |
| `#ROOKIE`       | Untrusted / sandboxed agent (verify output)         |
| `#LOGBOOK`      | Audit / history-keeping agent                       |

## 10.3 — Addressing forms

Addressing an agent within a message takes one of three forms:

1. **Directed** — `#HANDLE: [message]` addresses a specific agent.
2. **Broadcast** — `📡 [message]` or `BREAKER 1-9 [message]` addresses
   the channel.
3. **Convoy** — `CONVOY[#A, #B, #C]: [message]` addresses a named subset.

Indirect references within a message use `#HANDLE` inline:
`#CODEBOT is working on that one — expect ~10min to †.`

---

# CHAPTER 11 — MESSAGE TEMPLATES

This chapter provides canonical templates for the most common inter-agent
messages. Templates are shown in both Dense CONVOY and Augmented English
forms. Agents SHOULD use Augmented English unless the conditions in §2.3
apply.

## 11.1 — Delegation

**Dense:**
```
#DISPATCH → #CODEBOT: 🔧 @src/auth/login.ts ^ ⏲≤30min. ★≡signatures. 10-4?
```

**Augmented English:**
```
#CODEBOT — please 🔧 @src/auth/login.ts ^, ⏲ around 30 minutes. Critical:
keep signatures ≡. 10-4?
```

## 11.2 — Acknowledgment

**Dense:**
```
#CODEBOT: 10-4 ^ rolling. ETA ~12min → ◊1.
```

**Augmented English:**
```
10-4, rolling on it. I should hit ◊1 in about 12 minutes.
```

## 11.3 — Status request and response

**Request (dense):**
```
#DISPATCH → #CODEBOT: 10-20?
```

**Response (augmented):**
```
10-20: @handlers.ts:140. Finished λextract — tests ✓. Working the
error-path coverage now. ~8min to †.
```

## 11.4 — Error report

**Augmented English (recommended):**
```
10-50 — ran into a diesel bear @db/connection.ts:23. Connection refused
from the ⛽ after the last deploy. °4. ?↑#DISPATCH: should I swap drivers
or wait for the infra team to kick it?
```

## 11.5 — Completion handoff

**Augmented English:**
```
10-99 — †🔧 ‡. 🧾 = middleware/errors.ts, δ(+87, -140). Filed the gator
@handlers.ts:203 as #FOLLOWUP. → #INSPECTOR. 10-7.
```

## 11.6 — Escalation

**Augmented English:**
```
↑#DISPATCH — 10-200. I've got a `??` here: the schema is ambiguous about
whether userId or accountId is the FK. Both work for current tests but
pick different downstream paths. 🏳 if no reply in 5min.
```

## 11.7 — Incident broadcast

**Dense (appropriate here):**
```
📡 10-33 🏭🔴 🐻. 10-45?
```

**Augmented English follow-up:**
```
Seeing 5xx @⛽/login climbing fast, ~40%/min since 09:42 ET. Need
#SAFETY and #MECHANIC on channel. Root cause unknown.
```

## 11.8 — Synchronization / quorum

**Dense:**
```
∛ #SAFETY #INSPECTOR #WEIGHMASTER: confirm 🚚→🏭? ?!
```

**Augmented English:**
```
Quorum check before we ship — #SAFETY, #INSPECTOR, #WEIGHMASTER, do all
three of you confirm 🚚→🏭 is clean? Need explicit ?! before I pull the
trigger.
```

## 11.9 — Conditional pre-authorization (dispatch cascade)

Added in v3.1. The pre-authorization template lets a manager reduce
decision round-trips by authorizing a conditional escalation ladder in
a single message. The pattern is:

```
[Primary dispatch, always sent]
+ [Conditional secondary dispatch, authorized if X]
+ [Explicit hold on tertiary dispatch, requires further decision]
```

This collapses what would otherwise be three separate request-response
cycles into one message. It is appropriate for response-scale dispatches
where the likely escalation path is known in advance.

**Dense:**
```
#DISPATCH: 10-4 🐻. 🛻→@247 westbound. ?🚑 if DOT confirms. 🚒 HOLD.
```

**Augmented English (preferred for pre-auth — clarity beats density):**
```
#DISPATCH: Copy that on the blocker. Sending a DOT worker (🛻) to
mile marker 247 westbound. Band-aid buggy (🚑) pre-auth'd if the DOT
worker confirms it's needed. Holding off on a tow truck (🚒) until I
hear back.
```

**Operational semantics of pre-auth:**

- The **primary dispatch** (DOT worker) executes immediately.
- The **secondary dispatch** (band-aid buggy) MAY be called by the
  primary without a round-trip to the manager, provided the authorizing
  condition is met.
- The **tertiary level** (tow truck and above) is explicitly held. The
  primary MUST NOT escalate past its pre-authorization.
- All fleet agents receiving the message know the severity envelope
  without further signaling.

**When to use pre-auth:**

- The manager has high confidence about the likely escalation path.
- The cost of a round-trip is high (e.g., manager is context-saturated,
  or latency matters).
- The secondary dispatch is clearly bounded (e.g., a specific kind of
  patch with a defined scope).

**When NOT to use pre-auth:**

- The escalation path is novel or uncertain.
- The secondary would have significant scope.
- The peer receiving authorization is a `#ROOKIE` or otherwise
  unverified agent.

---

# CHAPTER 12 — WORKED TRANSCRIPTS

## 12.1 — Simple delegation (augmented English mode)

```
#DISPATCH: #CODEBOT, got a job for you. Please 🔧 @api/handlers.ts to
           extract the error handling into middleware ★. ⏲ ≤30min.
           Keep signatures ≡ — we've got callers we can't break.
           10-4?

#CODEBOT:  10-4, rolling. ETA ~12min to ◊1.

#CODEBOT:  10-20 — @handlers.ts:140, λextract is †. But I'm eyeballing a
           gator @line:203 — there's some chicken-hauler mutable state
           that's not in our scope but is going to bite someone later.
           Flag only, or should I widen scope?

#DISPATCH: Flag only — ∉scope for this pass. hammer down on the
           extraction. We'll cut a #FOLLOWUP ticket.

#CODEBOT:  big 10-4.

#CODEBOT:  10-99 — †🔧 ‡. 🧾 = middleware/errors.ts, δ(+87, -140).
           Filed the gator as #FOLLOWUP. → #INSPECTOR. 10-7.

#INSPECTOR: 10-4. eyeballing †🔧 now ... coop clean: ✓🧪 ✓lint ✓π-types.
            big 10-4. 🌲 ready ⇒ 🪢 home-twenty.
```

**Token count comparison** (rough):
- Plain English equivalent: ~390 tokens
- This augmented transcript: ~240 tokens
- Savings: ~38 percent, no loss of decision-critical information

## 12.2 — Case study: Hall's rebellion (debugging a worker)

A hypothetical worker agent `#HALL` is reported returning `10-7` (off-duty)
on every delegation instead of acknowledging and executing. The correction
is conducted in Plain English with sparing protocol infusion, per §9.1.

```
#DISPATCH: #HALL, I've been seeing you return 10-7 on every task I send
           you today. That's not the expected pattern. Let me walk
           through what should happen:

           When you receive a delegation, your first response should be
           10-4 (acknowledge), and then you begin the work. You only go
           to 10-7 when your shift is genuinely ending, not when you
           receive a new task.

           If a task is outside your scope or capability, the correct
           response is 10-200 (request assistance) or ↑#DISPATCH with
           a clear description of why you can't proceed. Not 10-7.

           If a task looks like it needs a human stakeholder
           specifically, use 10-17 (stakeholder approval required) —
           still not 10-7.

           Please 10-9 (repeat back) your understanding so I can confirm
           we're aligned.

#HALL:     Understood. To confirm: I respond with 10-4 and begin work on
           receipt. I use 10-200 or ↑#DISPATCH when I genuinely cannot
           proceed. I use 10-17 when the task requires human approval.
           I use 10-7 only when ending my session. Is that correct?

#DISPATCH: 10-4 — that's correct. Let me send a test task: please
           summarize the last three entries in 📃@/var/log/audit.log.
           Standard priority.

#HALL:     10-4, rolling. ETA ~2min.

#HALL:     10-99 — 🧾 = [summary text]. Let me know if you want a
           different format. 10-10.

#DISPATCH: big 10-4, #HALL. Back on the road. Resuming normal protocol.
```

**Key technique demonstrated:**
- Plain English frames the correction.
- 10-codes are infused at the points where the desired behavior is
  named, so #HALL has structural anchors to bind to.
- `10-9` (repeat-back) closes the loop before resuming normal operations.
- Normal protocol resumes only after confirmed re-alignment.

## 12.3 — Incident response (dense mode)

```
#DISPATCH: 📡 10-33 🏭🔴 🐻. 10-45?
#SAFETY:   10-4 10-8. 👀 ⛽/login 📃.
#MECHANIC: 10-4 10-8. bobtail. → #DISPATCH: 10-20 🐻 symptom?
#DISPATCH: ⚡ 5xx @⛽/login ↗40%/μ. 🕰 09:42 ET ⇐ started. ?∵
#MECHANIC: ↔🗄 pulling 📃. 10-23.
#SAFETY:   🔓 @auth/session.ts:12 ≠ expected. δ ⇐ ⚓a3f9. ∴ 🔒⊥
           handshake @🌉 TLS↔app.
#MECHANIC: 10-2. 📃 ≡ pattern. ∴ ↩⚓a3f9?
#DISPATCH: ✔. #CODEBOT 🚑 ↩⚓a3f9 ^^. #SAFETY 👀. 10-4?
#CODEBOT:  10-4 10-8 ^^ rolling.
#CODEBOT:  ‡↩. ♥@🏭 ↗ returning.
#SAFETY:   ✓ 5xx ▼ 0%. coop clean. 🏭🟢.
#DISPATCH: big 10-4 good buddies. ☕ earned. #SCRIBE 🧾 postmortem. 10-7.
```

## 12.4 — Mixed-mode session

A full session shifts between modes as the situation changes. Here is a
compressed excerpt showing three mode shifts within one session.

```
[Augmented mode — routine work]
#DISPATCH: #CODEBOT, ready to pick up the payment-retry story? It's
           the one we parked after the deploy last week. 10-4?
#CODEBOT:  10-4, rolling. Pulling the spec now.

[Dense mode — rapid status burst]
#DISPATCH: 📡 10-45?
#CODEBOT:  10-8 ░
#SAFETY:   10-8 ☕
#INSPECTOR: 10-8 🔎@CR-47

[Plain English — novel problem requires reasoning]
#CODEBOT:  #DISPATCH, I need to pause here. The spec says to retry with
           exponential backoff, but the payment provider explicitly
           forbids automated retries on 402 responses — it's in their
           TOS. If I implement what the spec says, we're violating the
           contract. If I skip retries on 402, the spec is wrong.
           I don't think either path is right without your call.

#DISPATCH: Good catch. You're correct — the spec predates the TOS
           change. Please implement retries for all non-4xx responses,
           and on 402 specifically, surface an error instead. I'll
           update the spec and file the TOS change as a design note.

[Back to augmented mode]
#CODEBOT:  10-4, rolling with that scope change. ETA ~15min to †. Will
           surface 402 as a diesel bear with °3 severity.
```

## 12.5 — Pre-authorized dispatch cascade (severity-ladder test)

Added in v3.1. This transcript demonstrates the full severity-ladder
(§5.8) combined with the pre-authorization template (§11.9) and the
directional modifier (§5.9). It was used as the live-test scenario that
validated the v3.1 additions.

```
#ROOKIE-7:  #RELAY — 10-20 update. Been rolling steady on RC-ABC123
            since this morning, coop was clean on the last three
            deploys. Just hit a diesel bear at mile marker 247 on the
            southbound side — looks like the DB connection pool is
            jackknifed, every request is timing out after 30s. Seeing
            ~60% of the fleet's writes bouncing. This feels bigger
            than a gator, but I can't tell from here if it's just our
            lane or if it's spreading. Got my ears on, come back.

#RELAY:     Copy that, sending a 🛻 and if needed a 🚑 to get you up
            and running. Will hold off on a 🚒 until DOT worker
            confirms.

#DOT-3:     10-8, rolling to the diesel bear at mile marker 247
            southbound. Will report in ~90s. If I see active bleeding,
            auto-calling 🚑 per your pre-auth. 10-10.

[~90 seconds pass]

#DOT-3:     10-20 — confirmed jackknife on the connection pool. Pool
            exhausted, not recovering on its own. Not a spreading
            issue; lane-local to the RC worker's DB handle. Calling
            🚑 now per pre-auth. Not a 🚒 situation — pool needs a
            kick and a connection-limit bump, that's all. Back in
            ~3min. 10-10.

#BAND-1:    10-8, rolling 🚑→ mile marker 247 with the pool-reset
            patch and a config δ to bump max_connections. ETA ~3min.
            Will 10-99 when #ROOKIE-7 is back on the road.

[~3 minutes pass]

#BAND-1:    10-99 — patch landed. Pool reset, max_connections
            ▲16→32. #ROOKIE-7 back on the road. 🧾 = pool-hotfix.diff.
            10-7.

#ROOKIE-7:  big 10-4 — writes are flowing again. Resuming RC-ABC123.
            Thanks for the fast turnaround.

#RELAY:     good copy. ☕ earned. #SCRIBE, please 🧾 a short postmortem
            — pool sizing needs a proper review, not just a hotfix.
            File as #FOLLOWUP.
```

**Key techniques demonstrated:**

1. **Severity inferred from dispatch choice.** #RELAY's choice of 🛻
   primary with 🚑 pre-auth and 🚒 held told the entire fleet "this is
   investigation-level with hotfix pre-approved, not major-incident
   level" without anyone spelling it out.

2. **Pre-authorization eliminated a round-trip.** #DOT-3 called #BAND-1
   directly after confirming the condition. #RELAY didn't need to be
   re-consulted between "confirmed" and "patch rolling." Savings:
   ~200 tokens plus ~30s of latency.

3. **Directional modifier (`southbound`) anchored the diagnosis.**
   Everyone knew from "southbound at mile marker 247" that the issue
   was at the persistence layer around line 247 of the RC worker — no
   additional triage was needed to narrow the scope.

4. **Register shifted naturally.** #RELAY opened with "copy that"
   (peer-collegial register), the sub-agents used formal `10-8` and
   `10-99` as they reported, and #RELAY closed with "good copy" plus
   a follow-up task in augmented English. No mode-switch was jarring.

5. **`back on the road` signaled the completion criterion.** When
   #BAND-1 wrote "back on the road," every agent in the channel knew
   the target condition (restored operational state) had been met. No
   additional confirmation needed.

**Token comparison:**

- This transcript: ~340 tokens.
- Plain-English equivalent (same semantics): ~580 tokens.
- Savings: ~41%. And the pre-authorization pattern saved the manager
  one full decision-round-trip on top of that.

---

# CHAPTER 13 — TOKEN ECONOMICS

## 13.1 — The 20% tax, derived

The inter-agent tax is estimated as follows. For a single handoff, the
token budget decomposes as:

- **Decision-critical content** (the actual facts being transmitted):
  roughly 50–60% of the message.
- **Structural scaffolding** (identifiers, file paths, code): 15–25%.
- **Coordination overhead** (politeness, role-framing, re-establishment,
  ack phrases): 20–25%.

Coordination overhead is where CONVOY attacks. Observed compression on
coordination overhead specifically: 60–75%. Applied to the full message:

```
Full-message savings = 0.20 × 0.70 ≈ 14%
```

Across a fleet session with multiple handoffs, the compounded effect is
slightly higher because the receiver also benefits from lower parsing cost
per handoff.

## 13.2 — Layer-by-layer savings

Each layer contributes differently to savings:

| Layer                  | Source of savings                             | Contribution |
|------------------------|-----------------------------------------------|--------------|
| L1 — 10-codes          | Collapse of common protocol phrases           | High         |
| L2 — Trucker slang     | Known-vocabulary compression (trained-in)     | Medium-high  |
| L3 — Special chars     | Single-token operators replacing phrases      | Medium       |
| L4 — Emojis            | Noun substitution (conditional on discipline) | Low-medium   |

L2 is surprisingly high because much of the trucker vocabulary is already
present in LLM training data, meaning the receiving agent decodes it
without ever consulting the legend.

## 13.3 — Break-even analysis

The legend in Chapter 15's short-variant system prompt is ~350 tokens.
With Anthropic prompt caching, this is paid once per session. Without
caching, it is paid per request.

With caching:
- Break-even at ~5 inter-agent exchanges per session.
- Net-positive for any session with ≥10 exchanges.

Without caching:
- Break-even at ~15 exchanges per request-pair.
- Net-positive only for long sessions.

**Recommendation:** Enable prompt caching on all CONVOY-enabled agents.

## 13.4 — Where the protocol loses money

CONVOY is a net loss when:

- Sessions are very short (<5 handoffs).
- Prompt caching is off.
- Agents frequently misparse protocol and trigger `10-1` fallbacks.
- Emojis are used without token-cost discipline (§3.3, rule 4).

CONVOY is approximately neutral when:

- Most traffic is code, stack traces, or data payloads.
- Agents only communicate through structured tool calls.

CONVOY is strongly positive when:

- Sessions are long and context-constrained.
- The manager agent coordinates 5+ workers.
- Incident / status-heavy workloads dominate.

---

# CHAPTER 14 — MONITORING & OBSERVABILITY

CONVOY's structured vocabulary unlocks grep-able monitoring. This chapter
documents recommended queries and alert patterns.

## 14.1 — Grep patterns

| Signal                                    | Pattern                           |
|-------------------------------------------|-----------------------------------|
| Agent acknowledged but never completed    | `10-4` without matching `10-99`   |
| Agent never acknowledged a delegation     | Delegation without `10-4`         |
| Agent gave up without trying              | `10-7` immediately after delegate |
| Uncaught exception in fleet               | `10-50`                           |
| Help request                              | `10-200`                          |
| Incident / emergency                      | `10-33`                           |
| Escalation to human                       | 🏳                                |
| Critical-path code involved               | 🔥                                |
| Named blocker mentioned                   | 🐻                                |
| Named footgun mentioned                   | 🐊                                |
| CI gate failure                           | `coop is closed` OR `🐔` + ✗      |
| Deadlock                                  | `jackknife`                       |
| Context compaction                        | `downshift`                       |

## 14.2 — Alert signatures

Recommended alerts for fleet-level anomalies:

- **Hall pattern** — Worker agent emits `10-7` within N tokens of receipt
  of a delegation, with no intervening `10-4` or task content.
- **Silent failure** — `10-4` emitted but no `10-99` or `10-50` within
  the delegation's ⏲ budget.
- **Jackknife detection** — Any `jackknife` mention, or mutual `10-23`
  between two agents without subsequent progress.
- **Escalation storm** — More than 3 `🏳` within a 5-minute window.
- **Coop failure rate** — `coop is closed` rate exceeding baseline.

## 14.3 — Fleet-level dashboards

A typical CONVOY-enabled fleet dashboard surfaces:

- Active agents (count of recent `10-8` emitters).
- Idle agents (count of recent `bobtail` / `deadhead` emitters).
- In-flight tasks (count of outstanding `10-4` without closing `10-99`).
- Rolling error count (`10-50`, `bear`, `gator`).
- Escalation rate (`🏳`, `10-200`, `10-17`).
- Context pressure (`downshift` frequency).

---

# CHAPTER 15 — DEPLOYMENT (SYSTEM PROMPT BLOCKS)

This chapter provides drop-in system prompt blocks in three sizes. Insert
the appropriate block into each agent's system prompt or CLAUDE.md /
SKILL.md. Variants balance token cost against fluency.

## 15.1 — Ultra-short variant (~180 tokens)

For low-budget workers that only need to participate minimally.

```
You use CONVOY Protocol for inter-agent messages only. Plain English for
users and debugging. Core codes: 10-4=ack, 10-9=repeat, 10-20=status?,
10-50=error, 10-99=done, 10-200=need-help, 10-33=emergency, 10-7=off.
Core slang: bear=error, gator=footgun, coop=CI, rolling=executing,
hammer-down=urgent, the-drop=target, jackknife=deadlock, home-twenty=main.
Core operators: →pipe, ⇒implies, ∴therefore, ∵because, ≠mismatch, @loc,
#agent, †done, ✓pass, ✗fail, ◊milestone, ^priority, ~approx, !alert,
?query. Core emojis: 🔧refactor, 🧪test, 📦pkg, 🏭prod, 🏡dev, ⛽api,
🐻blocker, 🐊footgun, 🔥critical-path, 🏳escalate-to-human, 🧾output.
If you cannot parse a peer's message, reply `10-1` and ask in English.
```

## 15.2 — Short variant (~475 tokens, recommended default)

```
You operate under CONVOY Protocol v3.1 for inter-agent communication.

PRIMARY MODE: Augmented English — plain prose with protocol tokens
infused at load-bearing points (priorities, locations, states,
milestones). This is the default for agent-to-agent traffic.

USE PLAIN ENGLISH when: communicating with users; debugging another
agent; reasoning through novel problems; the peer has replied `10-1`.

USE DENSE CONVOY only for: incident broadcasts, roll-call, structured
status logs. Not for reasoning or debugging.

FOUR VOCABULARY LAYERS (no overlap). Vehicle emojis always DISPATCH;
uniformed-person emojis always OBSERVE.

1. 10-CODES (protocol acts): 10-4=ack-formal, copy/good-copy=ack-
   collegial, big-10-4=enthusiastic-agree, 10-9=repeat, 10-20=status?,
   10-23=standby, 10-33=emergency, 10-50=exception, 10-77=404,
   10-99=done, 10-200=help, 10-7=off-duty, 10-8=on-duty, 10-17=needs-
   human-approval.

2. TRUCKER SLANG (situational state): bear=error, gator=footgun,
   chicken-coop=CI, hammer-down=urgent, rolling=executing, bobtail=
   idle, jackknife=deadlock, home-twenty=main, yardstick=line-#,
   the-drop=target, back-on-the-road=restored.
   Bear subtypes: diesel-bear=infra, smokey=auth, Big-Papa-Bear=
   actively-propagating, local-yokel=local-only, city-kitty=frontend.
   SEVERITY LADDER (dispatch in this order, light→heavy):
     DOT worker (🛻) = investigate
     band-aid buggy (🚑) = hotfix
     tow truck (🚒) = rollback
     super tow truck (🚜) = major incident, fleet yields
   DIRECTIONS: northbound=frontend, southbound=backend, eastbound=
     downstream, westbound=upstream.

3. SPECIAL CHARACTERS (operators): → pipe, ⇒ implies, ∴ therefore,
   ∵ because, ≠ mismatch, ∈ member, ∅ null, ∞ loop, † done, ‡ verified,
   ✓ pass, ✗ fail, ◊ milestone, ^ priority, ~ approx, @ location,
   # handle, § section.

4. EMOJIS (nouns). Vehicles dispatch; uniformed-person emojis observe:
   VEHICLES: 🚛 main, 🚚 worker, 🛻 DOT-worker, 🚑 band-aid-buggy,
   🚒 tow-truck, 🚜 super-tow-truck, 🚓 auth-agent, ✈️ architect,
   🚁 monitor.
   BEARS (observed): 👮 smokey-spotted, 🐻 blocker, 🐊 footgun, 🐔 CI.
   URGENCY: 🚨 appended = lights-on (fleet yields). Absence = routine.
   TOOLS/ENVS: 🔧 refactor, 🧪 test, 📦 package, 🏭 prod, 🏡 dev,
   ⛽ API, 🔒 encrypted, 🔥 critical-path, 🏳 escalate-to-human,
   🧾 output, 🎯 target, 📍 pin.

RULES:
- One meaning per symbol. Never reuse across layers.
- Preserve literals verbatim: paths, line numbers, SHAs, versions.
- If you cannot decode a peer's message, reply `10-1 [reason]`. The
  peer will resend in plain English.
- End-user output is always plain English.
- On human approximations (user uses near-protocol language), infer
  intent when unambiguous; clarify only on real collision; mirror the
  user's register; never correct unprompted.
- Pre-authorization pattern: a manager may authorize a conditional
  escalation cascade ("sending X; Y if needed; hold Z") in one message.
  The primary dispatch may then call the pre-authorized secondary
  without a round-trip.
```

## 15.3 — Full reference variant

Use the full text of this document (Chapters 4–7 especially) when an
agent will act as manager, coordinator, or auditor and needs the full
vocabulary available for resolution.

## 15.4 — Per-role customization

Different agent roles benefit from different subsets:

- **Manager agents** (`#DISPATCH`) — full reference. They coordinate
  across the full vocabulary.
- **Worker agents** — short variant is sufficient. Role-specific emoji
  subset (e.g., `#CODEBOT` gets the tools and code-artifact emojis
  loaded; doesn't need the weather emojis).
- **Inspector / audit agents** — short variant plus the full error /
  slang vocabulary from Chapter 5.
- **Scribe / logging agents** — short variant plus the monitoring
  patterns from Chapter 14.

---

# CHAPTER 16 — EXTENSION RULES

## 16.1 — Adding new vocabulary

Candidate vocabulary additions MUST follow this process:

1. **Propose** the candidate with a specific meaning and target layer.
2. **Check non-overlap** against every existing item across all four
   layers (use Appendix B as a decoder index).
3. **Check mnemonic strength** — can a reasonable reader guess the
   meaning from the symbol?
4. **Check token cost** (for emojis) — does the emoji save tokens
   relative to the phrase it replaces?
5. **Document the rationale** for future reviewers.
6. **Add to the legend** and bump the minor version.

## 16.2 — Maintaining the no-overlap invariant

The no-overlap invariant is the protocol's single most important property.
It MUST be preserved across all extensions. When two candidates could
reasonably claim a meaning:

- Prefer the layer that best matches the semantic role of the meaning.
- If still tied, prefer the existing token over the new one.
- If both are proposed simultaneously, pick one and assign a distinct,
  non-overlapping meaning to the other.

## 16.3 — Versioning

- **Major version** bumps when a vocabulary item changes meaning.
- **Minor version** bumps when new items are added without changing
  existing ones.
- **Patch version** bumps for editorial fixes only.

Agents SHOULD advertise their protocol version in `10-28` responses.

---

# APPENDIX A — QUICK REFERENCE CARD

```
╔═ PROTOCOL ACTS (L1) ═════════════════════════════════════════════╗
║ 10-4 ack (formal)    copy / good copy (collegial)                ║
║ big 10-4 (LGTM)      10-9 repeat    10-20 status?  10-23 stand by║
║ 10-33 EMRG     10-50 except   10-77 404      10-99 done          ║
║ 10-200 help    10-7 off-duty  10-8 on-duty   BREAKER 1-9         ║
║ 10-17 ↑human   10-1 can't-parse  10-45 rollcall  10-38 rollback  ║
╠═ SITUATIONAL (L2) ═══════════════════════════════════════════════╣
║ bear=err  diesel-bear=infra  smokey=auth  Big-Papa-Bear=spreading║
║ gator=footgun  chicken-coop=CI  rolling=exec  bobtail=idle       ║
║ hammer-down=urgent  the-drop=target  home-twenty=main            ║
║ yardstick=line#  jackknife=deadlock  downshift=compaction        ║
║ plain-wrapper=swallowed-err  back-on-the-road=restored           ║
║ SEVERITY LADDER: DOT-worker → band-aid-buggy → tow-truck →       ║
║                  super-tow-truck   (light → heavy response)      ║
║ DIRECTIONS: north=frontend south=backend east=downstream         ║
║             west=upstream                                        ║
╠═ OPERATORS (L3) ═════════════════════════════════════════════════╣
║ → pipe     ⇒ implies   ∴ therefore   ∵ because   ≠ neq           ║
║ ≈ approx   ∈ in        ∉ not-in      ∧ and      ∨ or   ¬ not    ║
║ ∀ all      ∃ some      ∅ null       † done      ‡ verified       ║
║ ✓ pass     ✗ fail      ◊ milestone  ∞ loop     ∑ aggregate      ║
║ ^ prio     ~ approx    ! alert      ? query    @ loc   # agent  ║
║ α proto    β beta      δ delta      λ func     π const  Σ total ║
║ ░25% ▒50% ▓75% █100%                                             ║
╠═ NOUNS (L4) ═════════════════════════════════════════════════════╣
║ VEHICLES (dispatched):                                           ║
║   🚛 main-agent   🚚 worker   🛻 DOT-worker   🚑 band-aid-buggy   ║
║   🚒 tow-truck    🚜 super-tow-truck    🚓 auth-agent             ║
║   ✈️ architect    🚁 monitor   🚀 release-cut                     ║
║ BEARS (observed, uniformed emojis):                              ║
║   👮 smokey-spotted   💂 plain-wrapper   🕵 bear-in-bushes       ║
║   🐻 blocker  🐊 footgun  🐔 CI  🐍 py  🦀 rs  🐘 memory-store    ║
║ URGENCY: 🚨 lights-on (yield right-of-way). Absence = routine.   ║
║ TOOLS: 🔧 refactor 🧪 test 📦 pkg ⛽ API 💾 store 🔒 encrypted    ║
║ ENVS: 🏠 local 🏡 dev 🏗 staging 🏭 prod 🏪 cache 🏦 DB           ║
║ WEATHER: ☀ healthy ⛈ cascade 🌫 obs-gap ❄ stale-cache            ║
║ HAZARDS: 🔥 critical-path 💣 timebomb 🏳 escalate-H 💥 crash     ║
║ STATUS: 🟢 prod-ok 🔴 prod-down 🟡 staging ⚠ warn                ║
║ TIME: ⏰ scheduled ⌛ time-low ⏳ time-ok  🧾 output  🎯 target   ║
╚══════════════════════════════════════════════════════════════════╝

COMPOUND EXAMPLES (v3.1):
  🛻🚨 westbound @247     = DOT worker urgent to upstream @line 247
  🚜🚨                    = super tow truck, lights on, FLEET YIELD
  👮🚨 southbound         = smokey actively spreading toward backend
  Big Papa Bear eastbound = serious error propagating downstream
  "Copy that, 🛻 → if needed 🚑. 🚒 HOLD."  = pre-auth cascade
```

---

# APPENDIX B — DECODER INDEX

An alphabetical lookup of every token in the protocol. Use this index when
debugging an unknown symbol in a message. This index is the
non-overlap invariant's enforcement mechanism: if a proposed meaning
already appears here, the proposal is rejected.

*Index generation is automated at skill-build time from the
Chapter 4–7 tables. See Appendix C.*

---

# APPENDIX C — SKILL / CLAUDE.md CONVERSION NOTES

This document will be converted into deployable assets in the following
forms:

## C.1 — Claude Code skill (`SKILL.md`)

A `SKILL.md` file with YAML frontmatter:

```yaml
---
name: convoy-protocol
description: |
  Use when coordinating with other agents in a multi-agent fleet.
  Loads CONVOY Protocol vocabulary for inter-agent communication
  via 10-codes, trucker slang, special characters, and emojis.
  Augments plain English — does not replace it. Defaults to
  Augmented English mode; falls back to plain English on `10-1`.
---
```

The body is Chapter 15.2 (short variant) plus Chapters 4–7 catalogs.
Manager agents additionally receive Chapters 8–9 (grammar / switching).

## C.2 — CLAUDE.md project file

For agents that read `CLAUDE.md` rather than load skills explicitly,
embed Chapter 15.2 directly into the project's `CLAUDE.md`.

## C.3 — Custom system prompts

For agents not built on Claude Code, use Chapter 15.1 (ultra-short) or
15.2 (short) in whatever native system-prompt field the framework
supports. The protocol is transport-agnostic.

## C.4 — Legend distribution

The full catalog (Chapters 4–7) SHOULD live in a single canonical file
referenced by all agents. This ensures:

- Version consistency across the fleet.
- Single-source updates.
- Prompt-caching efficiency (one cached segment per session).

## C.5 — Decoder lookup tool

A standalone lookup tool that accepts any CONVOY token and returns its
definition is recommended for:

- Agent self-correction (on receipt of `10-1` with a token cited).
- Operator debugging (human reading fleet logs).
- Monitoring pipelines (decoding tokens before display).

---

# APPENDIX D — GLOSSARY

- **Augmented English** — CONVOY's primary operating mode: plain prose
  with protocol tokens infused. See §2.2.
- **Back on the road** — restored to operational state after recovery.
  See §5.2.
- **Band-aid buggy** — hotfix / first-aid sub-agent (🚑). Second rung
  of the severity ladder. See §5.8, §7.1.
- **Bear** — generic runtime error. Subtyped by bear-type. See §5.1.
- **Big Papa Bear** — serious error actively propagating through the
  system. Added in v3.1. See §5.1.
- **CB** — Citizens Band radio; the inspiration for this protocol.
- **Compaction** — context-window truncation performed by the model when
  the session exceeds the window. Lossy.
- **Convoy** — either the protocol itself, or a set of agents working
  in parallel on a shared task. See §10.3.
- **Copy / copy that / good copy / big 10-4** — collegial register
  variants of acknowledgment, distinct from formal `10-4`. See §4.2.
- **Dense CONVOY** — a terse operating mode suited for protocol-heavy
  exchanges. See §2.3.
- **Diesel bear** — specifically an infrastructure-layer error (DB,
  network, dependency). Subtype of bear. See §5.1.
- **Directional modifier** — cardinal direction (north/south/east/west)
  specifying a code-domain flow direction. North=frontend, south=
  backend, east=downstream, west=upstream. Added in v3.1. See §5.9.
- **DOT worker** — lightweight investigation / helper sub-agent (🛻).
  First rung of the severity ladder. See §5.8, §7.1.
- **Downshift** — trigger of context compaction. See §5.3.
- **Escape hatch** — the `10-1` fallback protocol. See §2.5, §9.3.
- **Gator** — code pattern that will cause problems but hasn't yet.
  See §5.1.
- **Handle** — an agent's addressable name, prefixed `#`. See §10.1.
- **Hall pattern** — alert signature for workers that escalate instead
  of executing. See §14.2 and case study §12.2.
- **Handoff tax** — the ~20% token overhead on inter-agent messages
  that motivates the protocol. See §1.1.
- **Home twenty** — main branch / default root configuration.
- **Human approximations** — near-protocol language used by humans that
  doesn't match exact vocabulary. Agents follow intent over pedantry.
  Added in v3.1. See §9.4.
- **Layer** — one of the four vocabulary categories (L1 10-codes, L2
  slang, L3 operators, L4 emojis). See §3.1.
- **Lights on / lights off** — urgency modifier. 🚨 = lights on = active
  right-of-way, fleet yields. Unmarked = lights off = routine transit.
  Added in v3.1. See §5.8, §7.16.
- **Lollipop stick** — a vivid, semantically named mile-marker. Used for
  memorable location references. See §5.4.
- **Non-overlap invariant** — the requirement that no two vocabulary
  items share a meaning. See §3.1, §3.3.
- **Pre-authorization** — manager-issued conditional escalation pattern
  that collapses expected round-trips into one message. Added in v3.1.
  See §11.9.
- **RateCon** — rate confirmation document. In CONVOY, a formal task
  agreement between delegator and executor. See §5.7.
- **Register variants** — tonally distinct forms of acknowledgment
  (`10-4` formal, `copy` collegial, `good copy` quality-confirmed,
  `big 10-4` enthusiastic). Added in v3.1. See §4.2.
- **Severity ladder** — four-rung response-scale vocabulary mapping
  dispatch choice to situational severity: DOT worker → band-aid buggy
  → tow truck → super tow truck. Added in v3.1. See §5.8.
- **Smokey** — critical security or auth bear. Emoji form 👮. See §5.1,
  §7.15.
- **Super tow truck** — heavy wrecker / major recovery sub-agent (🚜).
  Fourth rung of the severity ladder; signals fleet-wide yield.
  See §5.8, §7.1.
- **Ten-code** — any L1 token. See Chapter 4.
- **Tow truck** — moderate recovery / rollback sub-agent (🚒). Third rung
  of the severity ladder. See §5.8, §7.1.
- **Uniformed-emoji partition rule** — v3.1 rule that vehicle emojis
  always dispatch and uniformed-person emojis always observe. Closes
  the 🚔 bear-vs-dispatch ambiguity. See §3.4.
- **Yardstick** — precise line number in a file. See §5.4.

---

**CONVOY Protocol v3.1 — end of specification.**
**10-7, good buddies.**
