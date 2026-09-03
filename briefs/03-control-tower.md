# Module Brief — Module 3: The Control Tower (fsextension, Java/Spring Boot)

## Teaching Arc
- **Metaphor:** An **air traffic control tower**. FreeSWITCH is the runway where planes actually land, but fsextension is the tower — it decides which planes land where, keeps the flight plan database, pushes updated schedules to the runway, and records every landing in the logbook. Without the tower, planes still fly — but nobody knows who landed, where, or when.
- **Opening hook:** "Module 2 showed you the engine that makes calls happen. But who DECIDES how calls are routed? Who keeps the list of agents, queues, and extensions? Who remembers every call after it ends? That's fsextension — the Java brain behind the telephony brawn."
- **Key insight:** fsextension is the management layer that sits between humans (admins configuring the system) and FreeSWITCH (the engine executing calls). It uses CQRS (separating commands from queries), Kafka (event streaming), SFTP (file delivery), and Flyway (database migrations) — each pattern solves a specific problem.
- **"Why should I care?":** When someone says "I added a new queue but calls aren't routing to it," the problem is almost certainly in fsextension — it's the component that serves dialplan XML to FreeSWITCH. Understanding its architecture means you know exactly where to look.

## Content Plan (screens)
1. Hook: the Java brain behind the telephony brawn.
2. Spring Boot overview: the framework that builds the backend. Show the main application entry point and the FreeswitchController.
3. CQRS pattern: commands go IN, queries come OUT. Show the `bus.execute(command)` pattern and why it matters.
4. Kafka integration: the event streaming backbone. Show KafkaConfiguration and JsonCdrConsumer — how call records flow from FreeSWITCH into the database.
5. SFTP provisioning: how fsextension pushes dialplan XML and recordings to FreeSWITCH. Mention the SFTP service.
6. Flyway migrations: how the database schema evolves safely. Show the migrations directory.
7. Quiz.

## Code Snippets (pre-extracted)

File: fsextension/src/com/metechvn/fsextension/controller/FreeswitchController.java (lines 59–68):
```java
@PostMapping("/script")
public Boolean createScript(
        @RequestBody GenerateScriptCommand cmd,
        @RequestHeader("Domain-Name") String domainName) throws JsonProcessingException {

    cmd = cmd.domainName(domainName);
    log.debug("createScript > Script with id: {} data: {}", cmd.getUniqueId(), objectMapper.writeValueAsString(cmd));

    return bus.execute(cmd);
}
```
Plain English: when someone POSTs a new dialplan script, stamp it with the tenant's domain name and hand it to the command bus. The `bus.execute(command)` is CQRS — commands go IN through this door, queries come OUT through another. Never mixed.

File: fsextension/src/com/metechvn/fsextension/controller/FreeswitchController.java (lines 77–88):
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
Plain English: registering a new phone number (dialed number) follows the same pattern — the HTTP request carries the command, the domain is extracted from headers (multi-tenant!), and the command bus handles the rest.

File: fsextension/src/com/metechvn/fsextension/config/KafkaConfiguration.java (lines 34–50):
```java
@Configuration
@Conditional(kafkaCondition.class)
public class KafkaConfiguration {

    private final List<NewTopic> queueTopics;
    private final KafkaProperties kafkaProperties;
    private final Map<String, Object> producerProperties;
    private final Map<String, Object> consumerProperties;
    private final String schemaRegistryUrl;

    private final Logger log = LoggerFactory.getLogger(this.getClass());

    public KafkaConfiguration(
            KafkaProperties kafkaProperties,
            @Value("${app.call_recording_topic:call_recording}") String callRecordingTopic,
            @Value("${app.call_aggregated_topic:call_aggregated}") String callAggregatedTopic,
```
Plain English: this class configures Kafka — the system that carries event messages between components. Notice `@Conditional(kafkaCondition.class)` — Kafka is optional. If it's not running, fsextension still works, just without event streaming. The topics (call_recording, call_aggregated) are named channels where specific types of events flow.

File: fsextension/src/com/metechvn/fsextension/domain/consumers/JsonCdrConsumer.java (lines 1–27):
```java
@Component
@Conditional(kafkaCondition.class)
@ConditionalOnProperty(value = "app.json_cdr_queued", havingValue = "true")
public class JsonCdrConsumer {

    private final Bus bus;
    private final KafkaTemplate<Object, Object> kafkaTemplate;
```
Plain English: this consumer listens on the `json_cdr` Kafka topic for call detail records (CDRs). It's gated by TWO conditions: Kafka must be available AND the config flag `app.json_cdr_queued` must be true. Double-gating means the system degrades gracefully — if Kafka is down, CDRs still get processed another way.

## Interactive Elements
- [x] **Data flow animation** (`.flow-animation` with `data-steps`) — actors: Admin (web UI), fsextension, Kafka, PostgreSQL, FreeSWITCH. Steps: 1) Admin creates a new queue via REST API, 2) fsextension stores in PostgreSQL, 3) fsextension generates dialplan XML, 4) fsextension pushes XML to FreeSWITCH via SFTP/mod_xml_curl, 5) FreeSWITCH loads new dialplan, 6) Call events flow back through Kafka.
- [x] **Code↔English translation** — FreeswitchController createScript snippet, KafkaConfiguration snippet, JsonCdrConsumer snippet.
- [x] **Quiz** — 3 questions:
  Q1 architecture: "Why does fsextension use CQRS (separate command and query buses) instead of regular method calls?" (commands change state and need validation/logging; queries are read-only and can be cached; mixing them makes it hard to scale reads vs writes independently)
  Q2 debugging: "A new dialplan was saved in fsextension but FreeSWITCH doesn't use it. What do you check?" (the SFTP connection — did fsextension successfully push the XML to FreeSWITCH? Check SFTP logs and FreeSWITCH's xml_curl response)
  Q3 Kafka: "The json_cdr_consumer has @ConditionalOnProperty. What happens if you set app.json_cdr_queued=false?" (CDRs stop flowing through Kafka — but they still get processed by the direct JSON CDR file import path; the system degrades gracefully)

## Reference Files to Read
- `references/content-philosophy.md` → whole file
- `references/gotchas.md` → whole file
- `references/interactive-elements.md` → sections: "Message Flow / Data Flow Animation", "Code ↔ English Translation Blocks", "Multiple-Choice Quizzes"
- `references/design-system.md` → tokens if needed

## Connections
- **Previous module:** Module 2 showed the engine (FreeSWITCH) and admin UI (FusionPBX); this module shows the brain that manages everything.
- **Next module:** "The Event Wire" covers what happens AFTER/DURING the call — how events flow out of FreeSWITCH through fsevents and Kafka.
- **Tone/style notes:** Teal accent. Actor names: fsextension, FreeSWITCH, Kafka, PostgreSQL. Glossary tooltips first use per module: Spring Boot, CQRS, Kafka, topic, CDR, SFTP, Flyway, migration, REST API, multi-tenant, conditional.
