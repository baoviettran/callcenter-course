# Module Brief — Module 7: When Things Break (Your Detective Toolkit)

## Teaching Arc
- **Metaphor:** A **crime-scene investigation**. Every incident leaves evidence in different rooms: SIP envelopes in packet captures (`sngrep`), the PBX's own diary (`fs_cli`), bulletins on the wire (fsevents stats/Kafka lag), scorecards on the dashboard (fsagent/Prometheus), and the written record (fsextension's PostgreSQL). The detective's skill is knowing WHICH room to enter first based on the symptom.
- **Opening hook:** "It's 9:05 AM. Slack: 'phones are down??'. Take a breath — by the end of this module you'll have a decision tree for exactly this moment."
- **Key insight:** Debugging telephony = narrowing layers. Is it registration? Signaling? Media? Events? Each layer has a dedicated tool, and the components you met each leave fingerprints.
- **"Why should I care?":** This is the module that turns everything before it into job skills. On your first incident, you'll move through these steps instead of freezing.

## Content Plan (screens)
1. Hook: the 9:05 AM Slack message. Introduce the symptom-first mindset.
2. The detective's toolkit — one card per tool:
   - `fs_cli` — FreeSWITCH's interactive console: watch live events (`show channels`, event monitoring).
   - `sngrep` — sees actual SIP envelopes flying between hosts; answers "did the INVITE even arrive?"
   - fsevents `/metrics` + health endpoint on :9090 — is the wire connected? ReconnectAttempts climbing?
   - Kafka consumer lag / fsextension logs — is the archive keeping up?
   - Prometheus/Grafana from fsagent — was quality degrading BEFORE the outage (jitter creeping up)?
   - FusionPBX web UI — check agent/queue configuration, extension status.
   - Kamailio logs — did calls even get routed to FreeSWITCH?
3. The symptom→first-room decision tree (interactive):
   - "Can't register / phone shows offline" → Kamailio logs, then sngrep for REGISTER
   - "Rings but no answer" → fs_cli, dialplan, xml_curl response from fsextension
   - "Connected but no audio / one-way audio" → SDP/NAT, media path, sngrep for RTP endpoints
   - "Calls fine but records missing" → event wire: fsevents stats → Kafka → DB
   - "Sounds terrible" → fsagent dashboards, per-leg comparison
   - "Can't add agent to queue" → FusionPBX or fsextension management domain
4. Code↔English: ConnectionStats again as the health fingerprint (brief reuse) OR a sample `fs_cli` session transcript.
5. Final quiz + course wrap-up: what to do next week (set up local sandbox with pbx/docker-compose, follow one real call end-to-end).

## Code Snippets (pre-extracted)

File: fsevents/internal/esl/client.go (lines 37–45) — reuse as the "health fingerprint":
```go
type ConnectionStats struct {
	ConnectedAt       time.Time
	EventsReceived    uint64
	EventsProcessed   uint64
	ReconnectAttempts uint64
	LastError         error
	mu                sync.RWMutex
}
```
Plain English: if Received >> Processed, backlog is growing (consumers too slow). If ReconnectAttempts climbs, the wire keeps dropping — expect data gaps in every downstream system.

Illustrative fs_cli transcript (protocol/tool output, not repo source):
```
freeswitch@pbx> show channels count
0 total, 0 active

freeswitch@pbx> version
FreeSWITCH Version 1.10.8 (git abc123)
```

## Interactive Elements
- [x] **Interactive decision tree** — clickable symptoms revealing "first room to search + why". Implement as pattern cards or an accordion of 7 scenarios (use existing patterns; keep JS-free where possible via details/summary or main.js-supported classes).
- [x] **Code↔English translation** — ConnectionStats snippet + fs_cli transcript.
- [x] **Quiz (final exam feel)** — 4 questions mixing all modules:
  Q1: "All agents show offline. First check?" (registration path — Kamailio/sngrep REGISTER)
  Q2: "Audio robotic since this morning's network change." (fsagent jitter/loss graphs)
  Q3: "CDRs stopped at 03:00 during a deploy." (fsevents reconnect stats / Kafka consumers after deploy)
  Q4: "New agent can't be added to a queue." (FusionPBX or fsextension — management domain)
- [x] **Wrap-up checklist card** — onboarding next steps (sandbox via pbx/docker-compose.yml, learn `fs_cli` and `sngrep`, read fsextension Kafka topics, bookmark Grafana boards).

## Reference Files to Read
- `references/content-philosophy.md` → whole file
- `references/gotchas.md` → whole file
- `references/interactive-elements.md` → sections: "Pattern Cards", "Multiple-Choice Quizzes", "Code ↔ English Translation Blocks"
- `references/design-system.md` → tokens if needed

## Connections
- **Previous module:** Module 6 showed where quality numbers come from; here those dashboards become evidence in investigations.
- **Next module:** none — final module. End with encouragement + the practical next-steps checklist.
- **Tone/style notes:** Teal accent. This is the graduation module — slightly more energizing tone. Glossary tooltips first use per module: fs_cli, sngrep, packet capture, consumer lag, registration.
