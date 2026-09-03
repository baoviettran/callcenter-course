# Module Brief — Module 4: The Event Wire (Everything That Happens, Gets Reported)

## Teaching Arc
- **Metaphor:** A **news agency wire service**. FreeSWITCH is the newsroom where everything actually happens; ESL is the newsroom's door where reporters stand; fsevents is the wire service stamping and re-broadcasting bulletins; Kafka is the newspaper distribution system; fsextension's database is the archive where history is written; fsagent is the specialized sports desk that only cares about performance stats.
- **Opening hook:** "The call you followed in Module 3 didn't just happen — it was ANNOUNCED, dozens of times: channel created, agent answered, bridged, held, hung up. Every announcement travels a wire. This module is about that wire."
- **Key insight:** The control plane never guesses what calls are doing — it listens to a firehose of events. FreeSWITCH emits ESL events → fsevents forwards them → consumers (fsextension via Kafka, HTTP consumers) record and react. Lose this wire and your dashboards/CDRs go blind even though calls still work.
- **"Why should I care?":** "Calls work but CDRs/reports are missing" is one of the most common production mysteries. After this module you'll know exactly which hop lost the message.

## Content Plan (screens)
1. Hook: every call is narrated in real time.
2. What ESL is: Event Socket Library — FreeSWITCH opens a socket and shouts structured events over it. Show the Go client connecting.
3. Group chat animation: the wire as a group chat between FS Newsroom (FreeSWITCH), WireService (fsevents), Archive (fsextension), SportsDesk (fsagent). Messages: CHANNEL_CREATE → "call started", CHANNEL_ANSWER → "agent picked up", CALL_UPDATE/hold events, CHANNEL_HANGUP_COMPLETE with hangup_cause.
4. Downstream paths: fsevents → HTTP endpoints with retries; fsextension consumes Kafka (Avro-encoded) into PostgreSQL tables (CDRs, active calls, agent history); fsagent subscribes for RTCP events instead.
5. Quiz.

## Code Snippets (pre-extracted)

File: fsevents/internal/esl/client.go (lines 47–66):
```go
func NewClientWithBuffer(config *config.ESLConfig, bufferSize int, logger *zap.Logger) *Client {
	ctx, cancel := context.WithCancel(context.Background())

	return &Client{
		config: config,
		logger: logger.Named("esl"),
		events: make(chan *types.Event, bufferSize), // Configurable buffer for events
		stats: &ConnectionStats{
			ConnectedAt: time.Now(),
		},
		ctx:    ctx,
		cancel: cancel,
	}
}
```
Plain English: `events` is a buffered channel — a waiting room with `bufferSize` chairs. If the newsroom shouts faster than reporters can write, extra bulletins queue up here instead of being lost (until it's full). `ConnectionStats` tracks how many events arrived vs processed — that difference is your backlog alarm.

File: fsevents/internal/esl/client.go (lines 37–45):
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
Plain English: the health report card of the whole wire. `ReconnectAttempts > 0` recently? FreeSWITCH restarted or network blipped — expect an event gap.

File: fsextension FreeswitchController.java (lines 77–88) — the reverse direction: commands going INTO telephony:
```java
@PostMapping("/dialed-number")
public UUID createDialedNumber(
        @RequestHeader("Domain-Name") String domainName,
        @RequestAttribute("Domain-Id") String domainId,
        @RequestBody CreateDialedNumberCommand command) {

    if (command.getDomainId() == null) {
        command = command.domainId(domainId);
    }

    return bus.execute(command).getId();
}
```
Plain English: events flow OUT of FreeSWITCH on the wire; but management actions flow IN through APIs like this ("register this dialed number"). Two directions, two doors.

## Interactive Elements
- [x] **Group Chat Animation** (REQUIRED — mandatory element) — chat container with id, actors: **FS Newsroom** (FreeSWITCH), **WireService** (fsevents), **Archive** (fsextension), **SportsDesk** (fsagent). Flow: FS posts "CHANNEL_CREATE — new call from 1001"; WireService replies "forwarded ✓"; Archive: "CDR row created"; FS posts "CHANNEL_ANSWER"; SportsDesk: "subscribed to RTCP for this leg…"; FS posts "HANGUP_COMPLETE cause=NORMAL_CLEARING"; Archive: "call duration saved".
- [x] **Code↔English translation** — both esl/client.go snippets + the Java controller snippet.
- [x] **Quiz** — 2 questions:
  Q1 debugging scenario: "Calls connect fine, but yesterday 14:00–14:20 has no records anywhere. First thing to check?" (fsevents ConnectionStats/reconnects around 14:00 — event wire gap)
  Q2 concept: "What's the buffered channel in the ESL client for?" (absorb bursts so events aren't dropped instantly when processing lags)

## Reference Files to Read
- `references/content-philosophy.md` → whole file
- `references/gotchas.md` → whole file
- `references/interactive-elements.md` → sections: "Group Chat Animation" (critical — follow exactly, needs id attributes on containers), "Multiple-Choice Quizzes", "Code ↔ English Translation Blocks"
- `references/design-system.md` → tokens if needed

## Connections
- **Previous module:** Module 3 followed the call's signaling/media; this module covers the parallel narration stream those same steps generate.
- **Next module:** Module 5 covers the browser side — how sip.js-worker and shared.io keep calls alive across tabs.
- **Tone/style notes:** Teal accent. Actor names consistent. Glossary tooltips first use per module: ESL, event, channel, Kafka, Avro, CDR, goroutine/channel (Go terms).
