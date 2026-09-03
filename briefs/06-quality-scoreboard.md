# Module Brief — Module 6: The Quality Scoreboard (fsagent, RTCP & MOS)

## Teaching Arc
- **Metaphor:** A **figure-skating judge with instruments**. While the skater (the call) performs, sensors measure every wobble: jitter is how uneven the ice is, packet loss is how many jumps were skipped entirely, and MOS is the final scorecard — one number that says "how did it feel to watch?"
- **Opening hook:** "'The call quality was bad.' — every support ticket ever. Module 6 is about how this system turns that vague complaint into numbers: jitter 14ms, loss 0.8%, MOS 3.6."
- **Key insight:** Voice quality isn't magic — it's measured continuously. Every RTP media stream carries RTCP reports back from the receiver; fsagent reads those reports via FreeSWITCH events, aggregates them per call leg, and exports a final verdict.
- **"Why should I care?":** When someone complains "calls sound robotic," you'll know where the scoreboard lives and how to tell network problems (jitter/loss) from endpoint problems — before escalating anything.

## Content Plan (screens)
1. Hook: turning "it sounds bad" into numbers.
2. The three stats in plain language, each with an everyday analogy:
   - **Jitter** = packets arriving unevenly (a drummer who speeds up and slows down). Measured in ms; <30ms is fine.
   - **Packet loss** = words that never arrive. Even 1–2% is audible as gaps/robot voice.
   - **MOS (Mean Opinion Score)** = 1–5 "how humans rate it", originally from people listening and scoring; now estimated by algorithms from the other two.
3. Code↔English: fsagent's calculator — see the actual struct and aggregation logic.
4. Where scores go: OTLP/gRPC → OpenTelemetry Collector → Prometheus/Grafana dashboards. Per-leg (Unique-ID) vs per-call (SIP Call-ID) correlation — why one call has TWO legs (agent side + customer side) and each can be good while the other is bad!
5. Quiz.

## Code Snippets (pre-extracted)

File: fsagent/pkg/calculator/rtcp.go (lines 24–40):
```go
type RTCPMetrics struct {
	Timestamp     time.Time
	InstanceName  string
	ChannelID     string // Unique-ID for per-leg monitoring
	CorrelationID string // SIP Call-ID for per-call aggregation
	DomainName    string // SIP domain for filtering by tenant/domain
	Direction     string

	Jitter      float64 // Average jitter in milliseconds
	PacketsLost int32   // Total packets lost

	PacketsSent int64 // Total packets sent
	OctetsSent  int64 // Total octets sent
}
```
Plain English: this is the scorecard for one call leg. Note the two IDs: `ChannelID` identifies ONE side of the call; `CorrelationID` ties both sides together into one conversation. `DomainName` means multiple tenant companies share this platform and their metrics stay separable.

File: fsagent/pkg/calculator/rtcp.go (lines 72–88):
```go
func (rc *rtcpCalculator) AggregateMetrics(ctx context.Context, event *connection.FSEvent, direction string, instanceName string) (bool, *RTCPMetrics, error) {
	channelID := event.GetHeader("Unique-ID")
	if channelID == "" {
		return false, nil, fmt.Errorf("RTCP event missing Unique-ID")
	}

	state, err := rc.store.Get(ctx, channelID)
	if err != nil {
		logger.WarnWithFields(map[string]interface{}{
			"channel_id":  channelID,
			"fs_instance": instanceName,
			"direction":   direction,
			"error":       err.Error(),
		}, "Channel state not found for RTCP aggregation")
		return false, nil, fmt.Errorf("failed to get channel state: %w", err)
	}
```
Plain English: every incoming report names its channel; fsagent looks up running state (kept in memory or Redis) and folds the new numbers into a running average. No state found? It refuses to guess and logs loudly — silent wrong data is worse than no data.

## Interactive Elements
- [x] **Interactive visualization** — a "live call" slider or animated bars showing jitter/loss changing and MOS reacting (simple JS-driven bar animation using existing patterns; keep it dependency-free). If too heavy, use pattern cards: three stat cards (Jitter/Loss/MOS) that expand on click.
- [x] **Code↔English translation** — both rtcp.go snippets above.
- [x] **Quiz** — 3 questions:
  Q1 concept: "What does MOS 4.2 mean?" (estimated quality on 1–5 human scale; >4 is good, <3 users complain)
  Q2 debugging: "Customer-side leg shows 0% loss but agent-side leg shows 9% loss. Whose Wi-Fi do you investigate?" (agent's side)
  Q3 code-reading: "Why does AggregateMetrics return an error when it can't find channel state?" (never fabricate metrics; fail loudly instead of exporting garbage averages)

## Reference Files to Read
- `references/content-philosophy.md` → whole file
- `references/gotchas.md` → whole file
- `references/interactive-elements.md` → sections: "Pattern Cards" (for stat cards), "Multiple-Choice Quizzes", "Code ↔ English Translation Blocks"
- `references/design-system.md` → tokens if needed

## Connections
- **Previous module:** Module 5 covered the browser side; here we zoom into the quality monitoring that watches every call from the server side.
- **Next module:** Module 7 — when things break: your detective toolkit (fs_cli, sngrep, logs, dashboards) tying all components together.
- **Tone/style notes:** Teal accent. Glossary tooltips first use per module: RTCP, RTP, jitter, packet loss, MOS, OpenTelemetry, Prometheus, call leg.
