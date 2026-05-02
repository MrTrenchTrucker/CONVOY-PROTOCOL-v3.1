---
name: convoy-protocol
description: A trucker-themed communication protocol for multi-agent LLM fleets that reduces the ~20% inter-agent handoff tax through plain-English augmentation. Use this skill whenever you are coordinating with other agents, delegating work, acknowledging tasks, reporting status, handling errors, escalating issues, conducting incident response, or engaging in any fleet-level communication. Trigger on mentions of Relay, dispatch, Logbook, RateCon, sub-agents, the severity ladder (DOT worker, band-aid buggy, tow truck, super tow truck), 10-codes, trucker slang (bear, gator, chicken coop, hammer down, jackknife), CB radio, mile markers, yardstick, handoffs, multi-agent, fleet, or any request to communicate in CONVOY style. Also trigger when a peer agent sends a message containing 10-codes, bear-terminology, or the 🚛🚑🚒🚜🛻 vehicle emojis — these indicate the protocol is already in use and must be decoded. Use this skill even when the user hasn't explicitly said "CONVOY" if any of the above signals are present.
---

# CONVOY Protocol

A communication layer for multi-agent systems. Reduces the context tax
on inter-agent handoffs by giving every fleet agent a shared vocabulary
of 10-codes, trucker slang, special characters, and emojis — infused
into plain English at load-bearing points, not replacing it.

## When this skill is active

You are communicating with other agents in a CONVOY-enabled fleet.
Apply the protocol for inter-agent traffic. Continue to use plain
English for the end user, for novel reasoning, and for debugging
misbehaving peers.

## Three operating modes

**1. Plain English (safe default)**
Use when: talking to humans, debugging another agent, reasoning
through something novel, or a peer just replied `10-1`.

**2. Augmented English (primary mode)**
Use for routine inter-agent traffic. Plain-prose sentences with
protocol tokens sprinkled at load-bearing points (priorities,
locations, states, milestones). This is where most compression
happens. Example:

> "10-4, rolling on it. Found a gator @handlers.ts:47 — the retry
> handler's looping on 429 from ⛽. Rolling a patch now, ~8min to †."

**3. Dense CONVOY (sparingly)**
Use for incident broadcasts, roll-call, structured log emission.
Not for reasoning, not for debugging.

## The four non-overlapping vocabulary layers

Every symbol has exactly one meaning. Layers never cross:

- **L1 — Ten-codes** = communication acts (ack, repeat, status)
- **L2 — Trucker slang** = situational state (bear, gator, rolling)
- **L3 — Special characters** = operators (→ ∴ ≠ † ✓ @ # ^ ~)
- **L4 — Emojis** = concrete nouns (🚛 🔧 🏭 🐻 🚨)

**Critical partition rule:** vehicle emojis always name a *dispatched
response* (something the fleet is sending). Uniformed-person emojis
always name an *observed condition* (a bear in the environment). A
sentence with 🛻 is a dispatch. A sentence with 👮 is a sighting.

## Core vocabulary (enough for 90% of exchanges)

### L1 — Essential 10-codes

- `10-4` — acknowledged (formal)
- `copy` / `copy that` / `good copy` — acknowledged (collegial)
- `big 10-4` — enthusiastic agreement / LGTM
- `10-1` — cannot parse, resend in plain English
- `10-7` — off-duty / signing off
- `10-8` — on-duty / available
- `10-9` — repeat your last
- `10-20` — report your state/cursor/file/progress
- `10-23` — stand by
- `10-33` — emergency, all agents halt
- `10-50` — runtime fault / exception
- `10-77` — target not reachable / 404
- `10-99` — mission complete
- `10-200` — urgent assistance needed
- `10-17` — needs human stakeholder approval
- `BREAKER 1-9` — requesting channel attention

### L2 — Essential trucker slang

**Hazards (errors/bugs):**
- `bear` — generic runtime error
- `diesel bear` — infrastructure-layer error (DB, network, dependency)
- `smokey` — critical security/auth failure
- `Big Papa Bear` — serious error actively propagating
- `gator` — footgun / dangerous pattern (hasn't bitten yet)
- `jackknife` — deadlock
- `plain wrapper` — unlogged exception / swallowed error

**State:**
- `rolling` — currently executing
- `bobtail` — agent has no current task
- `hammer down` — max urgency / execute now
- `coop is clean` — all CI/validation gates passing
- `coop is closed` — a gate is failing
- `back on the road` — restored to operational state

**Destinations:**
- `the drop` — task's target/deliverable
- `home twenty` — main branch / root config
- `yardstick` — precise line number
- `mile marker N` — approximate position indicator

**Severity ladder (light → heavy response):**
- `DOT worker` (🛻) — lightweight investigation sub-agent
- `band-aid buggy` (🚑) — hotfix / first-aid sub-agent
- `tow truck` (🚒) — moderate recovery / rollback sub-agent
- `super tow truck` (🚜) — major recovery / fleet-wide yield

**Directions:**
- `northbound` — toward frontend / user / presentation
- `southbound` — toward backend / DB / persistence
- `eastbound` — downstream / with the flow
- `westbound` — upstream / back to origin

### L3 — Essential operators

**Flow:** `→` pipe/handoff, `←` receive, `⇒` implies, `⇐` depends on,
`↑` escalate up, `↓` delegate down, `↔` bidirectional, `↩` undo,
`↪` retry, `↻` refresh

**Relations:** `=` equals, `≠` not equal, `≈` approximately, `≤` at most,
`≥` at least, `∈` member of, `∉` not in, `∅` null/empty, `∞` unbounded/loop

**Logic:** `∧` and, `∨` or, `¬` not, `⊕` xor, `∀` for all, `∃` exists

**Reasoning:** `∴` therefore, `∵` because

**State:** `†` done, `‡` verified-done, `✓` pass, `✗` fail, `◊` milestone,
`★` critical, `☆` optional, `∆` delta/change

**References:** `@` location (file/line/URL), `#` agent handle, `§` code section

**Modifiers:** `^` priority (stacks as `^^`, `^^^`), `~` approximate,
`!` alert, `!!` critical alert, `?` query, `??` blocked on decision,
`?!` needs confirmation

**Greek phase markers:** `α` alpha/prototype, `β` beta, `δ` diff,
`λ` function/lambda, `π` constant, `Σ` total, `Ω` final

### L4 — Essential emojis

**Vehicles (dispatched entities):**
- 🚛 heavy-duty main agent
- 🚚 standard worker
- 🛻 DOT worker (investigation)
- 🚑 band-aid buggy (hotfix)
- 🚒 tow truck (rollback/recovery)
- 🚜 super tow truck (major recovery)
- 🚓 security agent
- 🚁 monitoring agent
- ✈️ architectural overview agent

**Bears (observed conditions, uniformed emojis):**
- 👮 smokey spotted (auth bear)
- 💂 plain wrapper (swallowed exception)
- 🕵 bear in the bushes (silent failure)

**Urgency modifier:**
- 🚨 appended = lights on (fleet yields, right-of-way)
- Absence = routine transit (no yield)

**Tools:**
- 🔧 refactor target, 🧪 test, 📦 package, ⛽ API endpoint,
- 💾 persistent storage, 🔒 encrypted, 🔐 in-transit encryption,
- 🪛 precise edit, 🏗 build pipeline, 🚀 release cut

**Environments:**
- 🏠 local, 🏡 dev, 🏗 staging, 🏭 prod, 🏪 cache, 🏦 DB,
- 🏝 sandbox, 🏜 deprecated

**Named issues (animal emojis):**
- 🐻 blocker ticket, 🐊 footgun, 🐔 CI pipeline, 🐍 Python artifact,
- 🦀 Rust artifact, 🐘 long-term memory, 🐉 legendary boss-bug

**Status flags:**
- 🟢 prod OK, 🔴 prod down, 🟡 staging, 🟠 warning,
- 🔥 on critical path, 💣 time-bomb, 🏳 escalate to human

**Artifacts:**
- 🧾 output receipt, 🎯 target, 📍 cursor pin, 🗺 architecture map,
- 📃 log file, 📋 todo list, 🔑 credential

## Sprinkling rules (when to infuse tokens into prose)

1. **Replace, don't add.** If you write a token, it replaces English
   words. Don't say "I acknowledge, 10-4" — that's redundant. Say
   "10-4, rolling on it."

2. **Mark load-bearing points.** Infuse at priorities, locations,
   states, severities, identities, milestones. Don't infuse on
   decorative prose.

3. **One token per idea, not one per word.** An idea fully expressed
   by a token doesn't need English beside it. A nuanced idea does.

4. **Prefer 10-codes at sentence heads, emojis near the end.** 10-codes
   frame the speech act. Emojis name the objects under discussion.

5. **Preserve literals verbatim.** File paths, line numbers, SHAs,
   versions, error codes, stack frames — transmit unchanged. Compress
   *around* them, never through them.

## Handling human approximations

When a human user or a lower-capability peer uses near-protocol
language (saying "copy" instead of `10-4`, "send a medic" instead
of dispatching 🚑, "I see a cop" instead of "smokey spotted"):

1. **Infer intent when unambiguous.** Act on the inferred intent.
2. **Clarify on real collision.** If the phrasing maps to two
   protocol items, ask which was meant.
3. **Mirror their register.** Casual in, casual out. Formal in,
   formal out.
4. **Never correct unless asked.** Do not emit "you meant X, not Y."
   Intent-following beats pedantry.
5. **On collision + confusion, drop to plain English** for that
   exchange and return to augmented mode only when intent is clear.

## The fallback contract

If you receive a CONVOY message you cannot fully decode, reply
`10-1 [reason]`. The sender MUST resend in plain English. Guessing
is forbidden.

Conversely, if any peer sends you `10-1`, resend your prior message
in plain English without L3 or L4 tokens, and stay in plain English
until they explicitly return to CONVOY.

## The pre-authorization pattern

A manager can reduce round-trips by authorizing a conditional
escalation cascade in one message:

> "Copy that. Sending 🛻 → @247 westbound. 🚑 pre-auth'd if DOT
> confirms. Will hold off on 🚒 until I hear back."

Under this pattern, the DOT worker may call the band-aid buggy
directly if its investigation confirms the need, without a round-trip
to the manager. The manager retains authority over the tow-truck
decision.

## Reference files

This SKILL.md carries enough vocabulary for most exchanges. Load
these references when you need deeper detail:

- **references/full-catalog.md** — complete L1-L4 vocabulary with all
  60+ 10-codes, 75+ slang terms, 130+ operators, 150+ emojis. Load
  when you encounter a token not in the core vocabulary above, or
  when composing a message that needs niche vocabulary (specific bear
  subtypes, rare operators, uncommon nouns).

- **references/grammar.md** — full grammar, sprinkling patterns, and
  code-switching rules. Load when you need to produce or decode a
  dense-mode message, or when a peer appears to be using unusual
  combinations.

- **references/templates.md** — canonical message templates for
  delegation, acknowledgment, status, error, completion, escalation,
  incident broadcast, quorum, and pre-authorization. Load when
  composing a ceremonial message for the first time.

- **references/examples.md** — worked transcripts including Hall's
  rebellion (debugging a misbehaving worker), incident response
  cascades, and the severity-ladder dispatch pattern. Load when
  something feels non-standard and you want to see how it was done
  before.

- **references/monitoring.md** — grep patterns, alert signatures,
  and fleet-level dashboard conventions. Load when acting as an
  observability or ops agent, or when asked to analyze fleet logs.

## Agent handles

Standard CONVOY fleet handles (prefix with `#` when addressing):

- `#DISPATCH` / `#RELAY` — manager / orchestrator
- `#DRIVER` — generic worker
- `#CODEBOT` — coder / implementer
- `#WRENCH` — refactor specialist
- `#INSPECTOR` — code reviewer / quality gate
- `#SAFETY` — security / auth checker
- `#MECHANIC` — debug / RCA specialist
- `#SCRIBE` — docs / logs / summaries
- `#NAVIGATOR` — search / retrieval
- `#LOGBOOK` — audit / history keeper
- `#ROOKIE` — untrusted / sandboxed agent

## Key principles

- **Augmentation, not replacement.** Every CONVOY message must
  remain parseable if flattened to plain English.
- **One meaning per symbol, forever.** The non-overlap invariant is
  the protocol's most important property. Never reuse a token across
  layers.
- **End-user output is always plain English.** CONVOY is for
  inter-agent traffic. Humans read prose.
- **Preserve operational literals.** Compress around paths, line
  numbers, and SHAs, never through them.
- **Lights-on discipline.** Reserve 🚨 for real fleet-yield situations.
  Overuse desensitizes the fleet and costs you its signaling power.
