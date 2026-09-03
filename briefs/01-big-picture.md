# Module Brief — Module 1: The Big Picture

## Teaching Arc
- **Metaphor:** An **airport**. Agents are travelers, the softphone is their boarding pass app, Kamailio is the security checkpoint funneling passengers to the right gates, FreeSWITCH is the terminal where flights (calls) actually depart and land, and FusionPBX is the airport management office.
- **Opening hook:** "Every day in your company, someone clicks a green button in a browser tab, hears a ring, and talks to a customer. That single click crosses seven programs and three protocols in under a second. This course follows that click."
- **Key insight:** A "phone system" is no longer one box — it's a team of specialized programs: one for signaling (who calls whom), one for media (the actual voice), one for management, one for web admin, one for events, one for quality, and two for the browser.
- **"Why should I care?":** As the new person onboarding, your first job is building a mental map. When someone says "calls are dropping," you need to instantly know WHICH program to look at. This module gives you that map.

## Content Plan (screens)
1. Hook screen: the green button. What this platform is (a self-hosted call center built on FreeSWITCH + FusionPBX).
2. The journey of one call, told as an airport story: browser → Kamailio/WSS → FreeSWITCH → PSTN/customer. Animated data-flow.
3. The cast on one page: 7 components with one-line jobs:
   - **pbx** — the terminal (FreeSWITCH + FusionPBX + Lua call logic + Fail2ban)
   - **fsextension** — control tower (Java backend managing agents, queues, dialplans)
   - **fsevents** — flight-tracker wire (Go event forwarder)
   - **fsagent** — weather-and-turbulence sensor (Go quality metrics)
   - **sip.js-worker** — the boarding-pass app (browser softphone)
   - **shared.io** — keeps your app logged in across tabs (shared websocket)
   - **fusionpbx** — airport management office (PHP web admin UI)
4. What's NOT in this repo but exists in production: Kamailio (SIP router/load balancer), carrier SIP trunk.
5. Quiz.

## Code Snippets (pre-extracted)

File: pbx/docker-compose-host.yml (lines 5–19) — the "terminal" is just containers:
```yaml
services:
  postgres:
    image: luongld/postgres:14
    container_name: postgres_140
  pbx:
    image: luongld/pbx:1.10.8
    container_name: pbx_1108
```

File: sip.js-worker/README.md (usage section) — the traveler's boarding pass:
```typescript
sipClient.register({
  uri: 'sip:username@domain.com',
  username: 'username',
  password: 'password',
  displayName: 'Display Name'
}, {
  server: 'wss://domain.com:443',
  secure: true
});
```
Point out `wss://` = secure WebSocket — the browser speaks SIP over a WebSocket, not a phone cable.

File: fusionpbx/index.php (lines 1–20) — the management office entry:
```php
<?php
$version = '0.1';
```
(Keep brief — will be expanded in Module 2.)

## Interactive Elements
- [x] **Data flow animation** (`.flow-animation` with `data-steps`) — actors: Browser (softphone), Kamailio, FreeSWITCH, Customer's Phone. Steps: 1) Agent clicks Call in browser, 2) Browser sends INVITE over WSS to Kamailio, 3) Kamailio routes to a FreeSWITCH node, 4) FreeSWITCH dials out to customer, 5) Voice streams (RTP) directly established, 6) Call ends, BYE sent.
- [x] **Quiz** — 3 questions, multiple-choice:
  Q1 scenario: "Audio is choppy on calls. Which component most likely holds the clues?" (answer: fsagent — it measures voice quality; distractors: shared.io, web frontend, fail2ban)
  Q2 architecture: "Which of these does NOT live inside this repo?" (answer: Kamailio; distractors: FreeSWITCH docker setup, Java backend, Go event forwarder)
  Q3 component ownership: "A teammate proposes adding dialplan logic to fsextension AND Lua scripts. Why is that an architecture smell?" (answer: dialplan decisions should live in one place — fsextension serves the XML, Lua executes it; duplicating logic creates maintenance bugs)
- [x] **Code↔English translation** — use both snippets above.

## Reference Files to Read
- `references/content-philosophy.md` → whole file (short)
- `references/gotchas.md` → whole file
- `references/interactive-elements.md` → sections: "Message Flow / Data Flow Animation", "Multiple-Choice Quizzes", "Code ↔ English Translation Blocks", "Glossary Tooltips"
- `references/design-system.md` → skim tokens only if you need exact class names

## Connections
- **Previous module:** none — this is the opener.
- **Next module:** "The Telephony Engine" will zoom into the pbx directory — FreeSWITCH, FusionPBX, Lua scripts, and Fail2ban.
- **Tone/style notes:** Accent color is teal (`--color-accent`). Actor names used course-wide (keep consistent): **FreeSWITCH**, **FusionPBX**, **fsextension** (Java backend), **fsevents** (Go event forwarder), **fsagent** (Go quality agent), **SipWorker** (browser softphone lib), **SharedIO** (shared websocket lib). Learner is a new engineer onboarding onto a LIVE production system — friendly, practical, zero assumed telephony knowledge.
