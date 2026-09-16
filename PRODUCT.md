# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Users

**Primary:** an engineer meeting this platform for the first time — originally a new hire onboarding onto the call-center team, with zero assumed telephony knowledge. They are reading linearly, in a browser tab, alongside the real source repos.

**Secondary (confirmed):** outside readers. The course is intended to be shareable beyond the team — candidates, partners, and engineers at other companies. This sets a floor on assumed context: no internal shorthand, no "ask the team."

**Job to be done:** build a correct mental map of a live production call-center platform — which program does what, and which one to look at when something misbehaves.

## Product Purpose

A self-contained, browser-based course that teaches the architecture of a self-hosted call center platform by following a single call from the moment an agent clicks the green button to the moment the call ends.

Seven modules, read in order:

1. The Big Picture
2. The Telephony Engine
3. The Control Tower
4. The Event Wire
5. Browser Integration
6. The Quality Scoreboard
7. When Things Break

**Success means two things, both required:** the learner can trace the full path from click to conversation through every layer, *and* can act on it — given a symptom, name the component to inspect.

## Positioning

**The whole stack in one narrative.** Most available material covers one layer at a time — a FreeSWITCH guide, SIP.js docs, a Kafka primer. This course follows one call across all of them: browser softphone → SIP over WebSocket → telephony engine → Java backend → event wire → quality metrics. The connective tissue between layers is the product, not any individual layer.

## Operating Context

- **Read in a browser**, scroll-based and linear; modules run in numeric order.
- **Assembled from parts.** `bash build.sh` concatenates `_base.html` + `modules/*.html` (version-sorted) + `_footer.html` into `index.html`. Edit the module files, never `index.html` directly.
- **Per-module briefs** live in `briefs/*.md`: teaching arc, metaphor, screen-by-screen content plan, pre-extracted code snippets, and planned interactive elements. They are the authoring spec.
- **The source of truth is the real code.** The teaching material is derived from the production repositories in the sibling directory — `pbx`, `fsextension`, `fsevents`, `fsagent`, `fusionpbx`, `shared.io`, `sip.js-worker`, `channelgw`.
- **The course is expanding.** The existing seven modules are stable; additional modules are added over time.

## Capabilities and Constraints

**Delivered in the seven modules:**

- 42 multiple-choice quiz questions (3 per module, with feedback and reset)
- 4 step-through data-flow animations (`data-steps`)
- 164 glossary tooltips (`class="term"`)
- Code ↔ English translation blocks pairing real source excerpts with plain-language readings

**Technical constraints:**

- Static HTML/CSS/JS. No backend, no database, no build tool beyond `bash build.sh`.
- `index.html` is a generated artifact — source files are `_base.html`, `modules/*.html`, `_footer.html`.
- The only network dependency is the Google Fonts CDN. Everything else must work from disk.
- Navigation uses ARIA roles (`progressbar`, `tablist`, labelled buttons); keyboard and screen-reader access is expected to keep working.
- **Layout independence:** the course is served as a standalone page at a URL root. It must not assume a host page's chrome, and it must not be embedded in a frame.

**Publishing constraint (confirmed, unresolved):**

- The course repository is currently **private**, and it is not yet published. `github.io` is the target host, not the current state.
- **Sanitization is deferred, not waived.** Before the course goes public, the parts backed by private or internal code must be resolved: Docker image tags, repo links to non-public repositories, and any internal hostnames. Until that happens, the course stays private.
- **Never publish, in any form:** anything derived from `channelgw` — it lives on `source.mpt.com.vn` (MPT internal, `omicx/mcredit`) and is not public. It is currently absent from the course and must stay that way.

**Terminology, fixed course-wide.** Use these names exactly as written, in prose and in code:

| Name | Role |
|---|---|
| **FreeSWITCH** | the media/signalling engine where calls actually happen |
| **FusionPBX** | the web admin UI for the PBX |
| **fsextension** | the Java/Spring Boot backend — agents, queues, dialplans |
| **fsevents** | the Go event forwarder |
| **fsagent** | the Go quality-metrics agent |
| **SipWorker** | the browser softphone library |
| **SharedIO** | the shared-websocket library keeping the app logged in across tabs |

Kamailio (SIP router) and the carrier SIP trunk exist in production but are deliberately **not** part of this repository.

## Brand Commitments

- **Title:** *Call Center Platform: From Click to Conversation*. Used as the page title and the navigation title.
- **Voice:** friendly, practical, and concrete. Metaphor-driven explanations (airport, control tower, event wire, scoreboard) carry the architecture; jargon is introduced only alongside a plain-language equivalent.
- **Component naming is binding** — see the terminology table above. A module that renames a component breaks the course.

## Evidence on Hand

- **Real source repositories** in `/home/hoangtu34/Documents/code-repos/call-center/`: `pbx`, `fsextension`, `fsevents`, `fsagent`, `fusionpbx`, `shared.io`, `sip.js-worker`, `channelgw`. Code excerpts in the course are drawn from these.
- **Per-module briefs** in `briefs/01-big-picture.md` … `briefs/07-when-things-break.md`, including pre-extracted, line-referenced snippets.
- **Public visibility today:** `luongdev/fsagent`, `luongdev/shared.io`, `luongdev/sip.js-worker`, `luongdev/fsevents` are public. `luongdev/pbx` and `luongdev/fsextension` are **not** public (404).

**Absent — must not be fabricated:** testimonials, customer names, adoption numbers, benchmarks, pricing, licensing claims, or deployment details for anyone else's installation.

## Product Principles

1. **One call, one narrative.** Never teach a component in isolation from the call it serves. Every layer earns its place by what it does to the call.
2. **Grounded in shipped code.** Real excerpts from real repositories, not a hypothetical architecture. If the code and the lesson disagree, the lesson is wrong.
3. **Answer "where do I look?", not just "what is this?".** Comprehension and diagnosis are both required outcomes. A module that only explains has not finished.
4. **Metaphor first, then mechanism.** The airport, the control tower, the wire, the scoreboard — each is a load-bearing bridge to the real system, never decoration.
5. **Private until sanitized.** Nothing from private or internal code goes public until the sanitization pass is done, and `channelgw` never appears at all.

## Accessibility & Inclusion

The course already ships ARIA scaffolding on the navigation (`progressbar` with live `aria-valuenow`, a labelled `tablist` of module dots) and uses semantic headings per screen. Quiz interaction and tooltips must remain keyboard-reachable and screen-reader legible. Assume readers may have no telephony background and may be reading in a second language — expand acronyms on first use.
