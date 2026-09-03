# Module Brief — Module 2: The Telephony Engine (pbx, FusionPBX, Lua, Fail2ban)

## Teaching Arc
- **Metaphor:** A **train station**. FreeSWITCH is the railway itself — the tracks, signals, and scheduling that make trains (calls) run. FusionPBX is the stationmaster's office — a web dashboard where you configure routes, set timetables (dialplans), and manage passengers (agents). Lua scripts are the local dispatchers — custom logic that decides which train goes where when the standard schedule doesn't cover it. Fail2ban is the security guard — watching for suspicious behavior and banning troublemakers.
- **Opening hook:** "Module 1 showed you the whole airport from above. Now let's walk into the terminal and meet the engine that actually makes calls happen — plus the admin panel, the custom scripts, and the security system that keep it running."
- **Key insight:** The pbx directory isn't just FreeSWITCH — it's a layered system: Docker containers orchestrate everything, FusionPBX provides a PHP web UI for administration, Lua scripts handle custom call logic, and Fail2ban provides brute-force protection. Understanding these layers tells you WHERE to look when something breaks.
- **"Why should I care?":** When someone says "the PBX is down," you need to know: is it the FreeSWITCH process? The FusionPBX web UI? A Lua script crashing? Fail2ban blocking legitimate traffic? Each layer has different symptoms and different fixes.

## Content Plan (screens)
1. Hook: the pbx directory is not one thing — it's four things working together.
2. FreeSWITCH: the telephony engine. Docker containers run a patched version with custom modules (mod_callcenter, mod_avmd). Show the Dockerfile and docker-compose.
3. FusionPBX: the admin web UI. PHP application with its own MVC architecture, PostgreSQL database, and nginx. Show index.php entry point and login flow.
4. Lua scripts: the custom dispatchers. bridge_to_agent.lua queries the database to check agent availability before bridging. queue_cascade.lua tries multiple queues in sequence. Show the Lua code.
5. Fail2ban: the security guard. Monitors FreeSWITCH and FusionPBX logs for failed authentication attempts. Shows the jail config and Dockerfile.
6. Quiz.

## Code Snippets (pre-extracted)

File: pbx/docker-compose-host.yml (lines 5–19):
```yaml
services:
  postgres:
    image: luongld/postgres:14
    container_name: postgres_140
  pbx:
    image: luongld/pbx:1.10.8
    container_name: pbx_1108
```
Plain English: two containers — one for the database, one for FreeSWITCH itself. The `pbx` image is custom-built with patched modules.

File: fusionpbx/index.php (lines 27–30):
```php
//includes files
require_once __DIR__ . "/resources/require.php";

//if logged in, redirect to login destination
```
Plain English: every page request starts here — FusionPBX loads its framework (database connections, auth checks, routing) before doing anything else. This is the front door of the admin panel.

File: pbx/fs/resources/lua/bridge_to_agent.lua (lines 1–15):
```lua
local agent_uuid = argv[1] or ''
local dialplan_id = argv[2] or ''

if not session or not session:ready() or agent_uuid == '' then
    return false
end

local api = freeswitch.API()
local Database = require "resources.functions.database"
local switch = Database.new("switch")

if not switch:connected() then
    freeswitch.consoleLog('WARN', '[bridge_to_agent] Database not connected!\n')
    return
end
```
Plain English: when FreeSWITCH needs to connect a call to a specific agent, it runs this Lua script. First check: is the call still alive? Is there an agent ID? Then open a database connection. If the database is down, log a warning and bail — never try to bridge a call without knowing the agent's status.

File: pbx/fs/resources/lua/bridge_to_agent.lua (lines 18–36):
```lua
switch:query("select agt.name name, agt.contact contact, agt.status status, agt.state state \
    from agents agt \
    inner join  sip_registrations sreg on split_part(agt.contact, 'user/', 2) = (sreg.sip_username || '@' || sreg.sip_realm) \
    where agt.name = :agent_uuid", {
    agent_uuid = agent_uuid
}, function(row)
    local agent_status = row.status or ''
    local agent_state = row.state or ''
    ...
    -- Check agent available trước khi dial
    if agent_status ~= 'Available' or agent_state ~= 'Waiting' then
        freeswitch.consoleLog("DEBUG", "[bridge_to_agent] Agent " .. agent_uuid .. " không sẵn sàng, status=" .. agent_status .. " state=" .. agent_state .. "\n")
        return
    end
```
Plain English: the script joins the agents table with SIP registrations to verify the agent is both registered (their phone is connected) AND available (not on another call). If either condition fails, it logs why and stops — no wasted ring attempts.

File: pbx/fs/resources/lua/queue_cascade.lua (lines 1–15):
```lua
local args = argv[1] or ""
local dialplan_id = argv[2] or ""

if args == "" then
    freeswitch.consoleLog("ERR", "[queue_cascade] Không có queue_ids được truyền vào\n")
    return
end

local queue_ids = {}
for id in string.gmatch(args, "([^,]+)") do
    id = id:match("^%s*(.-)%s*$") -- trim
    if id ~= "" then
        table.insert(queue_ids, id)
    end
end
```
Plain English: this script implements cascade routing — try multiple queues in order. It receives a comma-separated list of queue IDs, trims whitespace, and builds an ordered list. If queue #1 has no available agents, it falls through to queue #2, then #3, etc.

File: pbx/f2b/Dockerfile (lines 1–10, 45–50):
```dockerfile
ARG FAIL2BAN_VERSION="0.11.2"

FROM alpine:3.15

ARG FAIL2BAN_VERSION
RUN apk --update --no-cache add \
    bash \
    curl \
    grep \
    ipset \
    iptables \
...
ENTRYPOINT [ "/entrypoint.sh" ]
CMD [ "fail2ban-server", "-f", "-x", "-v", "start" ]
```
Plain English: Fail2ban runs in its own Alpine Linux container. It installs iptables (the Linux firewall) and watches log files for patterns like "failed password" — when it spots too many failures from one IP, it adds a firewall rule to block that IP entirely.

File: pbx/f2b/resources/jail.local (lines 12–23):
```ini
[freeswitch]
enabled  = true
port     = 5060:5091
protocol = all
filter   = freeswitch
logpath  = /var/log/freeswitch/freeswitch.log
action   = iptables-allports[name=freeswitch, protocol=all]
maxretry = 10
findtime = 60
bantime  = 3600
```
Plain English: the Fail2ban jail for FreeSWITCH SIP traffic. If an IP fails authentication 10 times within 60 seconds, it gets banned from ALL ports for 1 hour. This stops brute-force attacks on SIP accounts.

## Interactive Elements
- [x] **Architecture diagram** — layered view of the pbx directory: Docker containers at bottom, FreeSWITCH engine in middle, FusionPBX admin UI on top, Lua scripts as plugins, Fail2ban as a sidecar. Use the interactive architecture diagram pattern.
- [x] **Code↔English translation** — Lua bridge_to_agent.lua snippet (the query + availability check), Lua queue_cascade.lua snippet (the argument parsing), Fail2ban jail.local snippet.
- [x] **Quiz** — 3 questions:
  Q1 scenario: "You notice Fail2ban has banned an IP address that belongs to a legitimate remote agent. What do you check?" (the jail.local maxretry/findtime settings, or the agent's authentication failures — maybe they're typing the wrong password)
  Q2 architecture: "FusionPBX shows 'Database Connection Failed' in the web UI. Which container is the problem likely in?" (the postgres container, since FusionPBX connects to it)
  Q3 Lua debugging: "A call is supposed to cascade to queue #2 but doesn't. bridge_to_agent.lua logs 'Agent không sẵn sàng'. What's wrong?" (the agent in queue #1 isn't in Available/Waiting state, so the cascade never proceeds to queue #2)

## Reference Files to Read
- `references/content-philosophy.md` → whole file
- `references/gotchas.md` → whole file
- `references/interactive-elements.md` → sections: "Interactive Architecture Diagram", "Code ↔ English Translation Blocks", "Multiple-Choice Quizzes"
- `references/design-system.md` → tokens if needed

## Connections
- **Previous module:** Module 1 gave the airport map; this module walks into the terminal and meets the engine.
- **Next module:** "The Control Tower" zooms into fsextension — the Java backend that manages agents, queues, and serves dialplans to FreeSWITCH.
- **Tone/style notes:** Teal accent. Actor names: FreeSWITCH, FusionPBX, Fail2ban, Lua scripts. Vietnamese comments in Lua/TypeScript are part of the codebase — show them as-is, explain in English. Glossary tooltips first use per module: Docker, container, MVC, PHP, Lua, iptables, jail, dialplan, mod_callcenter, ESL.
