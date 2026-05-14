# LetAITalk

`LetAITalk` is an AI voice assistant that answers phone calls on behalf of a user, understands why the caller is calling, responds within clear safety boundaries, and produces a clean post-call summary with the next recommended action.

`Echo-Proxy` was the earlier prototype codename in the planning document. The product and repository name moving forward is `LetAITalk`.

## TL;DR

In simple words: if someone calls you, `LetAITalk` can pick up, listen, understand the reason for the call, reply politely, collect useful details, and then tell you what happened.

The first version is not trying to fully replace a human. It should behave like a careful executive assistant:

- answer quickly
- collect the right information
- avoid risky commitments
- block or shorten low-value calls
- send the user a useful summary after the call

## Vision

The product goal is to protect the user's time without pretending to be a fully autonomous human replacement on day one.

The strongest first version of `LetAITalk` is a reliable AI phone layer that:

- answers routine calls
- filters spam and sales calls
- captures recruiter and scheduling details
- escalates uncertain or sensitive conversations
- stores transcripts, summaries, and action items for user review

## Core Use Cases

- `HR / Recruitment`: collect company name, role title, compensation range, work mode, and job description.
- `Sales / Spam`: politely decline, ask to be removed from the list, and end the call quickly.
- `Scheduling`: capture caller identity, meeting purpose, and suggested time slots without auto-confirming.
- `Unknown Calls`: ask clarifying questions, capture callback details, and create a follow-up task for the user.
- `Urgent / Sensitive`: avoid over-automation and escalate to the user immediately.

## Product Principles

- `Useful before autonomous`: the system should solve real call-handling pain before aiming for perfect human imitation.
- `Deterministic before generative`: hard rules and policy checks should shape behavior before the LLM speaks.
- `Real-time first`: phone conversations fail fast if latency is poor, so responsiveness is a core product feature.
- `Auditable by default`: transcripts, summaries, call state, and actions should be observable and reviewable.
- `Safe by construction`: the assistant must not commit, disclose, approve, or impersonate beyond explicit user policy.

## Architecture Summary

At a high level, `LetAITalk` has four layers:

1. `Telephony edge`: receives inbound calls from providers like Twilio, Vonage, or Exotel.
2. `Real-time orchestration`: streams audio through a low-latency Go gateway into the AI pipeline.
3. `AI decisioning`: runs STT, intent detection, rules, LLM generation, and TTS in Python.
4. `Persistence and control`: stores calls, transcripts, summaries, settings, runtime state, and admin controls.

The high-level architecture source is here:

- [docs/architecture/letaitalk-high-level-architecture.puml](/H:/00000_Harsh-Eng/LetAITalk/docs/architecture/letaitalk-high-level-architecture.puml)
- [Rendered High-Level Architecture Preview](https://www.plantuml.com/plantuml/png/XLXVSnkt4N_7fzXnNzAfIQUEdQIUQIQqB6KqbMJ6LEs7A0_WtPA72WTSWJtHRDtltjq3U09rpDWFciFkR__iOl-0_ZcI3jItLf4XYt25jBYyrVOUPh2qcZO-jB08LMC8A-e3Al8Ta8Ur9_8jr2OmtNYdLBmthjD1jx3MrVqs-DxLPzxw03kHBlYnqRNV6RU5ZRQnvDIuqRsb2-_eHhV8bess1R_oRUSTEbhHtYA8RUssjbJHiN6znJy6MEEGzXs23zJuWX6qku_OmVVV5aJdQnJYgvAu2xhxP6fguENBahu8mUzIp7_wxjjNFxm-BvZBvDXR7seCNmEL6HJWnOyNYukp0dXsoCHfVLFOHEFmhu2-qwJNEfvqS8hwkhvVU-nFAPj0lghfnjEnehE_lNtr-iTZn4azRyyNIkcA-7YchSMG3_-C_AbogS2plyxX5YrsZNTxM5GLnWXNUe_X6VnN0VosYD5i7TPmqxTh3F02LfTBE7TrvusZtq77p17mewPby0-6lRvpOfplmb7m5lu2rrWR3IiAg5jkWq71A7QGKWeEzcQ_DQQkqVqUpBQXsKydmFyhmdavXm-exUpMj0YI5dGrdDVC-kON_-SSqoTSDzxVPwyaDuCRcVn85MpfB_oY2NTwFw07HYO8S8XpXJ4Qx-1QExtDQZCjaqRe-GCwWcM_jYOs2Pfe8qafcFeqISXJHyhZ13nrvfWm4NZgKfdhx-Qmk9GaFc28cYJk3xqZoVmtopqrtXqIph3teMemKj3aXrXPj2G9QYmAMDtUmgBM7M72yba0HwQMtffgpvMuDGu5bIa2lBgwFfBdiv0_OEoyYmWVjJNrw4awJPoulLsLIkGiWAMFD1jky9CFzvcNFfM2ux2c-KoXoDyfQ_G_VKm1AGp-EsMLCOpVKrYAGI8RMRoE77Do1ENzlfeF8myMheOLQS9oCDHf63Bi0zOcthahyu1Z_G_mPvAPWNk4bmHk0wv-lRfpgtqaRC5l65VvK0yY6I1IbRSsBTa8pzwl_yNVhDi7zk_E9H6-gWWlO178R7H5SL2HCKf1Sc2IYT98_fo2HZLbLbxFnscsjDfXBl5tEZPhhqFDlbpxjR48YwuRYkFC6dJ4fw4LwjOuM2ml1zPmueDmRZn7OAfKtIiC3wR20LGoCbq26zLE_2xKZfzJq5JjSQ3Vp-7yCs5msd9LZnCzve1N7MBL3FUsTKQKfAXIknxGXswyT1iCw2eSGKmjGH-zOUPgxwZ1Q6A2fQuhjaAfUXhHiShYTB8HJkwF7-PmqLCVs9LAMuIbfYO5_EyUUmPSOuo2VDjppTyik9DUm3_qvbx3YrHgULUaEcIKg4yB66PiEA-rDr0RRMFnw7HfYsR-RBhf1gdWUq8mRYtB7YhMgwOm4PukZZT7Oh1BWDFYHrk49QiKG2jKY4E0wiHgcYmO4MKmOR8P4rELe2INSoS-6ZBQclyWXBnC8cd2YKpHmopMOOWcKZBJ8ccPDqgLOOZ8q_7y9WTp253tjV6m2Rh5g9vWPsbwiyp6XvrqUaOpBpnsWH9CnXON5uTscP6V4Nuc5k1wYXHLWIeiT3f85a2x6ZP6Ef22Th4AfaioWirxWk7EXtR8MLtW06LrHvLHizGWZCwsOOiEmz3-TsxNe8CQ2KDhd8baAWYznGYrP-q4hQQgKIAVzHIXcR6tnwWAG66uud1aCmamRUAaiTnV3Cj-OVq8ArPQQbw6wQvZ4mKA9BDAc1bK-FWmZ9SuZfVHqm9Mw4utMpMzknyW9sjdL2Tvrr7gJpjAifFYcVPKAL5KNOdwmp7WUqgjd4HpHvvkvJnyIeEnGys1JPmi-h0gtrhhVhEHjpprmQLsl7F64ORGTmGRgxTngglipsbPthdHdV429y6F6rX2xiGLRLEBXt7tg-CNAip6nqoUMyClPuk4TJAj1kx4mzYthGxx59WZzgyQOZU4RIna2fUwu2kCKMu_lpvAuAa5AqBoVXpywZlf1NKA9kBoGqtmVMpaLzUrR9fFZQOeBmarS0KsFWCOsmx3yV6YLV56A9u54YqIIVQUfpP_3jRB7MR0PBe6p2qUPIJbN1ai9OhQiV8K9B_Tseolqew8wkbbpUS_oOwKcPgshkQUgbArT3xGawgQudSftHljPhxFLpk5fOk7Bjdbk9LodX26rm3upV5lgFSx5y7wtSmomLNxlFOQZiIox7FL7TfBIcMRHkXpq6CxZkxF5J9HJ3qrcD_wxqvCNFoCLS_OZGcHDTmsE8nF4w6NKZCkicqOsf-FtQ3ekQW7BTc6tWN9bwc8sJLMHjRchDMSc8a_-R-o_2Q_qzCDHl9oxzxPVOf5YkXu7S_LupCU0hQQtG9gK1PngvtWe-z3XQARGazDQTlYbWyf6-hBt-7AE8JqtrPykaRjEC_mHNs1oE7Ly0MkFovdLJ1aAcrXmvSpS6iT6nHsP3BE0kfwVxZoJbF38BQJhAcV-K_Vsly3)
- [docs/architecture/letaitalk-call-runtime-sequence.puml](/H:/00000_Harsh-Eng/LetAITalk/docs/architecture/letaitalk-call-runtime-sequence.puml)
- [Rendered Call Runtime Flow Preview](https://www.plantuml.com/plantuml/png/ZLNVSzD837xtNw4fZpVf_Q0C73Ez8IqjqvaKSgGSBtrHragor_MkJoidXB_-j5uJsm4AdOx7iJxf-wILLg-Y84jJkKAiE88PoVJc3jq3JE36BqFZIxX4v-1zuyLM1DSkx0fi9FYcMX8NHNomlaR62fPe7jQSN2w32moxZHKQsEC6ox2pVWqhT75eAMc5ZPFhuEKjAicKBRgX9_tNa3TqIp7YcgREhZqOya8yG375ElZu1NL1AdNA77OWt0pfsezPuvEdlvqzVtOrCCwHnHfRevUBm2Nnpm2FCdzotVuDAOq4VZnUXlmmqiokQ6OzZO9DfnVFBy--XVfXl8lmwN5T2lWwoelhwVNb0FGso5ZHvPyNPyzUZ17VL7DnDIqAJBctZKPSr7rXuUIE7DMRuFSmvx2rId026E7mkIYMuHESl0dmdj1DxbAF9XhovSb1voYYGg_ADTtxZxJSXF00ly0jbHRX3GhjSA-_5nIZ3HvkqUEwOtmJajSuL18CLrlIztcpT3PkChZ_LQWEK8MTpFbUL3HCRxxCqtSqze1x_uxDXg8mIk2axUvEd_FWh4bIPxDRVVw3pfOTuEvkaJGeIHTY77eUeanQuHy3FsIz-RKNUopcKFjuxnF_hubMdqe5q_bDxFWEg4Aru18ZmKcd0HQ2eicyrqh7Ld-KDTFYxrdh-rhnCzoh11ES8oCs-AZCgcA5Hg9MOVclVjOmWN5DhLFtFXRxDgYBDPWYm89uQqr63mrTlts8IbZu80Hirnk1iFfopEzsFhui9c2xEt2NUoNg9vDg5xiESJfdPcySgQ5gkoWA4rOWZ3xMWKKDBkmc3eMydXWsfGsmGLywwzTjnPE0OndzuNLASa33jXQeEPZ4h_uJA4c8A-jjr290dVj2EyQqOblrLBRdjIPFdFEV72vBsERMoLzZJMGs4FTUDXHj11C2bzPdUo_s_1n-FysBOK7Ir71-NXHvV67oLp-Ty1Aq9cwmLhgrqICKdckksAxQkSHToOjiLLpV9GerJ2WqEenxhrKGfREV2RZhkoWAwsECfA6FE-8snhr7OwYMqI5-HsMeAHtybhh3xC2fA7-SmYmPBeFV4iVSXRCGSe5SUhaE38HQP9FB9Gtx0k3RL1qcywmOAugAxRCvj8bYiuoHIm4zGk734LM7DCcSDckK0aQsoL7eTiBGwSIlh4StwCL2FT49l1vr8JUEm4Q8pKf7pf9F12CjmoInhfMPIiWXgJmqvo4BSf5I3_SM6rljbgbK-wDv9Z_hLyJa3GqoVZndWxvCWq4zfz0d-H6PNedz-FmisS7ZyzV9FrAuRdJL1wEo2NhfA-RBOj0gHvqV6icJDkpWSHUgHpVGB_lm1t7ZpXgFffwdiWylBDgIJvTuKxS5DSFfo0Eg3kq0FUrkbSjGrOuIe9sSxqy7c8oaykiBer_3gOVp-v5JbbRqGFNAox2TA6I1EYtCv3MyTvFMirFeb-dyO6i5vnZ7bOcu9OXDLI7lqsqbofrkdZPFArJ5WMEttfARlX5WlmoPJE0oSJ5rVJtq7Er0TITV3hB8Xr2CGEhq8NPbYXkTyyNjGfyVDoXnMjVTTj8XvS6gphjS_vj89Qm2voDFuvwlntmlTqTyMlGx7vWgj3w2R0XguWfzSehxA5IbzPSJFEqtpYjzQohtFm00)
- [docs/architecture/README.md](/H:/00000_Harsh-Eng/LetAITalk/docs/architecture/README.md)

The `.puml` files are the source of truth. The PlantUML server links are only rendered previews for quick viewing.

## High-Level Call Flow

1. A caller dials the assigned number.
2. The telephony provider sends an inbound webhook to the Go gateway.
3. The gateway creates a `call_session_id` and opens the live media stream.
4. Caller audio is streamed to the AI orchestration layer.
5. STT produces partial and final transcripts.
6. The policy and rules layer decides whether the assistant can answer and how.
7. The LLM generates a bounded response when deterministic rules do not fully handle the case.
8. TTS converts the response into streamed audio.
9. Audio is sent back through the gateway to the telephony provider.
10. After the call, workers produce the transcript cleanup, summary, intent label, follow-up task, and notifications.

## Major System Components

### 1. Go Real-Time Edge

The Go layer should own the infrastructure-heavy and latency-sensitive responsibilities:

- telephony webhook handling
- media stream lifecycle
- WebSocket audio transport
- call session creation and status transitions
- buffering, routing, auth, and rate limiting
- production-grade concurrency handling

Suggested services:

- `gateway-service`
- `call-session-service`
- `admin-api-service`
- `notification-service`

### 2. Python AI Orchestration

The Python layer should own fast-moving AI logic and provider experimentation:

- STT provider integration
- LLM orchestration
- prompt and conversation state management
- policy enforcement and rules
- TTS provider integration
- post-call evaluation and quality analysis

Suggested services:

- `ai-orchestrator-service`
- `stt-adapter-service`
- `tts-adapter-service`
- `conversation-policy-service`
- `evaluation-worker`

### 3. Data and State

- `PostgreSQL`: source of truth for users, rules, call sessions, transcripts, and summaries.
- `Redis`: runtime state for live sessions, interruption flags, and conversation buffers.
- `Object storage`: recordings, audio artifacts, and future exports.

### 4. User Control Surface

- admin dashboard or mobile app
- assistant enable/disable control
- rule editor
- availability settings
- call logs and summaries
- voice profile configuration

## MVP Scope

### Included

- inbound call handling
- real-time transcription
- basic intent classification
- safe scripted or LLM-assisted responses
- transcript storage
- post-call summary generation
- simple admin control surface

### Not Included Initially

- perfect voice cloning
- full mobile experience
- autonomous calendar booking
- CRM-heavy workflow automation
- multilingual expansion
- payment or approval workflows

## Safety Boundaries

The assistant should be explicitly bounded. It can:

- answer basic questions
- collect caller details
- ask clarifying questions
- decline sales calls
- capture scheduling requests
- draft follow-up actions

It must not:

- commit to contracts or legal agreements
- accept job offers unless explicitly allowed
- disclose private user information
- make payments or approvals
- pretend to be the human user without disclosure

Recommended safe opening line:

`Hi, this is an AI assistant answering on behalf of the user. How can I help you?`

## Target Latency

To feel natural on a phone call, the system should aim for:

- `STT partial transcript`: under 500 ms
- `STT final transcript`: under 1.5 seconds after speech ends
- `LLM response start`: under 1 second
- `TTS first audio chunk`: under 1 second
- `Total response delay`: roughly 1.5 to 3 seconds

Anything consistently above 4 seconds will feel slow in a live call.

## Suggested Repository Shape

The repo is currently at an early stage. The following structure is the intended build direction:

```text
LetAITalk/
|-- services/
|   |-- gateway-go/
|   |-- admin-api-go/
|   `-- ai-orchestrator-python/
|-- database/
|   |-- migrations/
|   `-- schema.sql
|-- infra/
|   |-- docker-compose.yml
|   |-- nginx.conf
|   `-- terraform/
|-- mobile-app/
|-- docs/
|   |-- architecture/
|   |   |-- README.md
|   |   `-- letaitalk-high-level-architecture.puml
|   |-- api-contracts.md
|   |-- prompt-design.md
|   `-- safety-policy.md
`-- README.md
```

## Phased Delivery Plan

1. `Phase 0`: product boundaries, scripts, intent set, consent model, summary format.
2. `Phase 1`: telephony webhook, call session creation, media stream bootstrap.
3. `Phase 2`: transcription pipeline and transcript persistence.
4. `Phase 3`: deterministic voice bot with basic TTS replies.
5. `Phase 4`: LLM conversation manager with policy-checked multi-turn flows.
6. `Phase 5`: voice profile and cloned-voice support.
7. `Phase 6`: dashboard or mobile control plane.
8. `Phase 7`: human handoff and urgent escalation.
9. `Phase 8`: production hardening, monitoring, retries, PII controls, and auditability.

## Build Strategy

The right execution order is:

1. make the call pipeline reliable
2. add structured understanding
3. add safe conversational intelligence
4. add personality and voice quality
5. add automation only after the earlier layers are trustworthy

That sequence matters. A polished voice on top of a weak call pipeline is not a product.

## Current Status

This repository currently contains the initial product definition and architecture baseline for `LetAITalk`. The next implementation milestone should be the telephony entry point and live audio session pipeline.
