# Module Brief — Module 5: Browser Integration (sip.js-worker + shared.io)

## Teaching Arc
- **Metaphor:** A **co-working space**. The browser is the building, each tab is a desk, and the SharedWorker is the building's shared utilities — one internet connection, one phone line, one receptionist. Instead of every desk having its own phone line (expensive, wasteful), there's one SIP line shared across all desks. When someone leaves a desk (closes a tab), the call doesn't die — it stays alive on the shared line.
- **Opening hook:** "You know that feeling when you accidentally close a browser tab during a call and the call drops? This system fixes that. The softphone runs in a SharedWorker — a background thread that survives tab reloads. And there's a second SharedWorker that keeps your WebSocket connection alive across all tabs. Let's see how both work."
- **Key insight:** Browsers weren't designed for phone calls. sip.js-worker wraps SIP signaling (the phone protocol) over WebSocket and runs it in a SharedWorker so calls survive tab navigation. shared.io does the same for the data connection — one Socket.IO link shared by every tab. Both use the same pattern: background thread + PostMessage IPC.
- **"Why should I care?":** If you're building a browser-based agent UI, understanding SharedWorker patterns means you can build features that survive tab switches — call state, notifications, real-time updates — without duplicating connections in every tab.

## Content Plan (screens)
1. Hook: closing a tab shouldn't kill a call.
2. The SharedWorker pattern: what it is (a background thread shared across tabs), why it matters (one SIP registration, not N), how tabs communicate with it (PostMessage).
3. sip.js-worker deep dive: SIP signaling over WebSocket, WebRTC for media, the LenientTransport (patched for real-world quirks). Show the worker entry point and sip-core setup.
4. shared.io deep dive: Socket.IO over SharedWorker, ACK-based request/response, notification fan-out, long-polling fallback. Show the worker entry point and socket-manager.
5. How they work together: sip.js-worker handles the call (ringing, talking, hangup), shared.io handles everything else (agent status updates, queue notifications, CDR displays).
6. Quiz.

## Code Snippets (pre-extracted)

File: sip.js-worker/src/worker/index.ts (lines 1–21):
```typescript
/**
 * Worker entry point
 */

import { MessageBroker } from './message-broker';
import { TabManager } from './tab-manager';
import { SipCore, SipCoreOptions } from './sip-core';
import { WorkerState } from './worker-state';
import { SipWorker, VERSION } from '../common/types';

// Khởi tạo WorkerState
const workerState = new WorkerState();

// Khởi tạo MessageBroker
const messageBroker = new MessageBroker();

// Set WorkerState reference cho MessageBroker
messageBroker.setWorkerState(workerState);

// Biến lưu trữ SipCore
let sipCore: SipCore | null = null;
```
Plain English: the SharedWorker starts by creating three managers: WorkerState (keeps track of what's happening), MessageBroker (routes messages between tabs), and SipCore (the actual SIP phone engine). Notice SipCore starts as `null` — it's created when the first tab sends a "register" message.

File: sip.js-worker/src/worker/index.ts (lines 24–39):
```typescript
const tabManager = new TabManager(messageBroker, workerState, {
  onTabClosedWithCall: async (callId: string, reason: string) => {
    if (sipCore) {
      console.log(`Auto-hanging up call ${callId} due to: ${reason}`);
      await sipCore.hangupCall(callId);
    } else {
      console.error(`Cannot hangup call ${callId}: SipCore not initialized`);
    }
  },
  onAudioContextStateChanged: (hasRunningAudioContext: boolean, previousState: boolean) => {
    if (sipCore) {
      console.log(`AudioContext state changed: ${previousState} → ${hasRunningAudioContext}`);
      sipCore.handleAudioContextStateChange(hasRunningAudioContext, previousState);
    }
  }
});
```
Plain English: the TabManager watches for tabs closing. If a tab closes while on a call, it auto-hangs-up (nobody's listening anymore). It also tracks the AudioContext — browsers suspend audio when no user interaction happens, so the worker needs to know when audio state changes.

File: sip.js-worker/src/worker/sip-core.ts (lines 1–27):
```typescript
/**
 * SipCore - Lớp xử lý SIP signaling
 */

import { SipWorker } from '../common/types';
import { MessageBroker } from './message-broker';
import { TabManager } from './tab-manager';
import {
  createWorkerSessionDescriptionHandlerFactory,
  WorkerSessionDescriptionHandlerOptions,
} from './worker-session-description-handler';
import {
  UserAgent,
  UserAgentOptions,
  Registerer,
  RegistererState,
  Inviter,
  Invitation,
  Session,
  SessionState,
  Web,
  InviterOptions
} from 'sip.js';
import { v7 as uuidv7 } from 'uuid';
import { WorkerState } from './worker-state';
import { LenientTransport } from './lenient-transport';
```
Plain English: SipCore is the SIP phone engine. It imports from `sip.js` (the SIP library) — UserAgent (the phone identity), Registerer (tells the server "I'm online"), Inviter (starts a call), Invitation (receives a call). The `LenientTransport` is a custom patch that fixes real-world WebSocket quirks that the standard library doesn't handle.

File: shared.io/src/worker/index.ts (lines 1–44):
```typescript
/**
 * Shared Worker Entry Point
 *
 * This module is the entry point for the Shared Worker, handling connections from clients
 * and coordinating messages.
 */

import { MessageType, WorkerMessage } from '../common/types';
import { createMessageRouter, IMessageRouter } from './message-router';
import { createSocketManager, ISocketManager } from './socket-manager';
import { createAckManager, IAckManager } from './ack-manager';
import { createNotificationManager, INotificationManager } from './notification';
import { createLongPollingManager, ILongPollingManager } from './long-polling';

// Initialize managers
const messageRouter: IMessageRouter = createMessageRouter();
const ackManager: IAckManager = createAckManager();
const notificationManager: INotificationManager = createNotificationManager(messageRouter);
const longPollingManager: ILongPollingManager = createLongPollingManager(messageRouter);
const socketManager: ISocketManager = createSocketManager(
  messageRouter,
  ackManager,
  notificationManager,
  longPollingManager
);

/**
 * Initialize the Shared Worker
 */
export function initializeWorker(): void {
  try {
    // Listen for connection events from clients
    self.addEventListener('connect', (event: MessageEvent) => {
      handleConnect(event.ports[0]);
    });

    console.log('Socket.IO Shared Worker initialized');
  } catch (error) {
    console.error('Could not initialize Shared Worker:', error);
  }
}
```
Plain English: shared.io's SharedWorker creates four managers: MessageRouter (directs messages), AckManager (tracks request/response pairs), NotificationManager (broadcasts events to all tabs), and LongPollingManager (fallback for browsers without SharedWorker support). When a tab connects, it sends a MessagePort — the worker uses this private channel to talk to that specific tab.

File: shared.io/src/worker/socket-manager.ts (lines 22–32):
```typescript
export interface ISocketManager {
  connect(url: string, options: SharedSocketIOOptions, message: WorkerMessage): void;
  disconnect(message: WorkerMessage): void;
  emit(eventName: string, args: any[], message: WorkerMessage): void;
  emitWithAck(eventName: string, args: any[], message: WorkerMessage): Promise<any>;
  on(eventName: string, message: WorkerMessage): void;
  off(eventName: string, message: WorkerMessage): void;
  once(eventName: string, message: WorkerMessage): void;
  getConnectionStatus(): ConnectionStatus;
}
```
Plain English: the SocketManager interface — it's the gateway between the SharedWorker and the Socket.IO server. Every tab can call `connect`, `emit`, `on`, etc., but they all go through this single manager. `emitWithAck` is the request/response pattern: send a message and wait for the server to acknowledge it.

## Interactive Elements
- [x] **Interactive architecture diagram** — show the browser tab landscape: multiple tabs at top, two SharedWorkers in the middle (sip.js-worker, shared.io), server connections at bottom (WebSocket to FreeSWITCH, Socket.IO to backend). Tabs communicate with workers via PostMessage arrows.
- [x] **Code↔English translation** — sip.js-worker index.ts (the initialization), sip-core.ts (the imports showing what SIP.js provides), shared.io index.ts (the manager initialization), socket-manager.ts (the interface).
- [x] **Quiz** — 3 questions:
  Q1 scenario: "An agent has 3 browser tabs open. How many SIP registrations does the system see?" (ONE — the SharedWorker registers once, all tabs share that registration)
  Q2 debugging: "shared.io shows 'long-polling mode' instead of WebSocket. Why?" (the browser doesn't support SharedWorker — shared.io falls back to long-polling for compatibility)
  Q3 architecture: "Why put SIP signaling in a SharedWorker instead of directly in the page?" (calls survive tab reloads; one SIP registration for all tabs; audio state is managed centrally; tab crashes don't kill calls)

## Reference Files to Read
- `references/content-philosophy.md` → whole file
- `references/gotchas.md` → whole file
- `references/interactive-elements.md` → sections: "Interactive Architecture Diagram", "Code ↔ English Translation Blocks", "Multiple-Choice Quizzes"
- `references/design-system.md` → tokens if needed

## Connections
- **Previous module:** Module 4 covered the event wire (server-side); this module covers the browser side — how calls are actually made and how the UI stays in sync.
- **Next module:** "The Quality Scoreboard" zooms into fsagent — how voice quality is measured from the server side using RTCP reports.
- **Tone/style notes:** Teal accent. Actor names: SipWorker, SharedIO. Vietnamese comments in TypeScript are part of the codebase — show as-is, explain in English. Glossary tooltips first use per module: SharedWorker, WebWorker, PostMessage, IPC, SIP, WebSocket, WebRTC, Socket.IO, ACK, long-polling, AudioContext.
