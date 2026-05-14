# LetAITalk Backend Runtime

This document explains how the backend is expected to work at a system level.

## Backend Objective

The backend for `LetAITalk` has one primary job:

`answer a live phone call, process the conversation in real time, respond safely, and leave behind a reliable record of what happened.`

That objective drives the architecture. The real-time path is separated from the administrative and post-call path because the latency requirements are different.

## Service Ownership

### Telephony And Real-Time Edge In Go

The Go backend should own the parts of the system that are sensitive to concurrency, streaming behavior, and provider lifecycle:

- inbound telephony webhooks
- media stream setup and teardown
- call session creation
- runtime session status updates
- audio frame transport
- event publication when a call completes

The reason to keep this in Go is straightforward: these services need predictable performance under concurrent live calls and should not depend on heavier AI orchestration code paths.

### AI Runtime In Python

The Python backend should own the AI-heavy execution path:

- streaming STT integration
- transcript normalization
- conversation state handling
- deterministic conversation rules
- LLM prompt construction and inference orchestration
- response validation and safety gating
- TTS generation
- post-call summarization and evaluation

The reason to keep this in Python is iteration speed. This is where prompts, policies, models, evaluators, and adapters will change frequently.

## Runtime Flow

### 1. Inbound Call Acceptance

1. The caller dials the `LetAITalk` number.
2. The telephony provider sends an inbound webhook to the Go webhook controller.
3. The Go session manager creates a `call_session_id`.
4. Redis is initialized with transient runtime state for the live call.
5. PostgreSQL stores the durable metadata for the call session.
6. The provider opens the media stream and the Go media gateway starts receiving caller audio.

### 2. Real-Time Audio Processing

1. The Go media gateway forwards caller audio frames to the Python AI orchestrator.
2. The AI orchestrator sends audio to the STT adapter.
3. The STT adapter calls the external speech recognition provider and receives partial and final transcript events.
4. The AI orchestrator passes normalized transcript events into the conversation policy engine.

### 3. Response Decisioning

The response path should follow a strict order:

1. Try deterministic rules first.
2. If a deterministic path is sufficient, return the approved response immediately.
3. If the case needs more flexible language generation, build a constrained LLM request.
4. Validate the generated response before it is spoken.
5. Only send approved response text into the TTS path.

This matters because the product should behave like a bounded assistant, not an unconstrained conversational agent.

### 4. Response Synthesis

1. Approved response text is sent to the TTS adapter.
2. The TTS adapter requests synthesized speech from the configured provider.
3. Audio chunks are returned to the AI orchestrator.
4. The AI orchestrator streams assistant audio back to the Go media gateway.
5. The Go media gateway sends the audio back through the telephony provider to the caller.

### 5. State During The Call

Two storage systems serve different purposes:

- `Redis`: temporary runtime state such as conversation buffer, turn state, interruption flags, and playback state.
- `PostgreSQL`: permanent records such as call session metadata, transcript events, summaries, and action items.

Object storage is optional in the MVP path and should hold artifacts such as recording references or processed outputs when needed.

## Post-Call Processing

When the call ends:

1. The telephony provider sends the call completion event.
2. The Go backend publishes a `call completed` event.
3. The Python post-call worker generates:
   - cleaned transcript
   - intent classification
   - summary
   - action items
   - priority or urgency marker
4. Results are stored in PostgreSQL.
5. Optional artifacts are stored in object storage.
6. The notification service informs the user through push, SMS, or WhatsApp.

## Control Plane

The control plane should remain outside the low-latency runtime path.

It includes:

- dashboard or mobile app
- admin API
- settings and rules management
- call log and summary review
- assistant enable / disable controls
- live operational toggles when needed

The control plane reads and writes durable user preferences in PostgreSQL and only touches Redis for live controls that affect active sessions.

## MVP Design Position

For the MVP, the architecture should prefer direct integration paths:

- telephony provider to Go backend
- Go backend to Python AI runtime
- Python runtime to external AI providers

A message bus is a scale feature, not an MVP requirement. Introduce it only when direct service communication becomes a bottleneck for throughput, retry behavior, or fan-out.

## Operational Standard

If the system has to trade off between being impressive and being reliable, reliability should win.

The backend should optimize in this order:

1. stable call pickup
2. low-latency audio transport
3. accurate transcript capture
4. safe response generation
5. useful post-call summary
6. richer automation later
