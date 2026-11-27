---
name: voicedock-voice-assistant
description: Build, change and test a phone or web voice assistant on VoiceDock through its MCP server or REST API. Use when the user wants an AI receptionist, phone agent, voice bot or call handler on VoiceDock, wants to change an existing VoiceDock assistant (prompt, model, voice, tools, transfers), connect a phone number, or test an assistant before real callers reach it.
---

# Build a voice assistant on VoiceDock

VoiceDock runs AI voice assistants that answer and place phone calls and web calls. You configure an assistant as JSON; the platform runs speech recognition, the model, speech synthesis, tools and call control.

Work in this order. Each step names the doc page that is the source of truth; fetch it rather than relying on memory, because model and voice names change.

Related skills in this plugin: `voicedock-model-choice` (step 2), `voicedock-prompting` (step 3), `voicedock-tools-and-transfers` (steps 4 to 6), `voicedock-test-and-review` (step 7).

## 0. Connect

- MCP (preferred): `https://mcp.voicedock.ai/mcp` (the VoiceDock plugin adds it), OAuth sign-in with your VoiceDock account, no key to paste. One tool per API endpoint. Outbound calls, live call control, campaigns and starting test runs are only on the full endpoint `https://mcp.hmsovereign.com/mcp` or the REST API. If the account has more than one organisation, call `listOrganizations` first and pass `organization_id` on every call.
- REST: `https://api.hmsovereign.com/api/v1`, header `Authorization: Bearer <key>`. Keys start with `fl_`. Read the key from an environment variable; never print it.
- Docs index for agents: `https://doc.voicedock.ai/llms.txt`. Its top section lists the rules and every valid provider, model and voice. Read it before writing any config.

## 1. Pin down what the assistant must do

Ask the user, or state your assumption, for each of these. They decide the config.

| Question | Why it matters |
|---|---|
| Language(s) of the callers | Model, voice and speech recognition language |
| Inbound, outbound or both | Outbound needs a number of the user's own with a SIP trunk |
| Which phone number | A managed number from VoiceDock, the user's own over a SIP trunk, or forwarding from their existing number (`references/telephony.md`) |
| Transfers to a human? To which number? | Needs a trunk on the number (step 5) |
| What happens with the outcome | Email, webhook or structured analysis (step 6) |
| Callers in a noisy place (on the road, on site)? | Weighs the model choice |
| Must call data stay in the EU (health, legal, public sector)? | Limits the providers for every component; read `https://doc.voicedock.ai/docs/platform/eu-data-sovereignty` |
| Must callers consent to recording, or may content not be stored? | `recording_consent`, `gdpr_mode`, retention (`references/compliance.md`) |
| What must the assistant never do (prices, advice, promises)? | A narrow scope is more reliable; hard promises stay with a human |

## 2. Choose pipeline or speech-to-speech

Read `https://doc.voicedock.ai/docs/guides/prompting-per-model` first.

- **Pipeline**: `llm_config.provider` is `openai`, `google`, `mistral` or `xai`. Needs `stt_config` and `tts_config`; a request without `tts_config` is rejected. Most predictable, every call-control command works, wording is fully under your control.
- **Speech-to-speech**: `openai_live`, `google_realtime`, `xai_realtime`, `elevenlabs_realtime`. The voice is `llm_config.voice`; `stt_config` and `tts_config` are ignored. Lowest latency and most natural turn-taking.

Take model and voice names only from the "Valid configuration values" section of llms.txt. Names are case-sensitive.

## 3. Write the prompt for that model

The same prompt does not behave the same on every model. Apply the section of the prompting page for the family you chose. The rules that matter most:

- Every model: language rule on top, a short greeting that says it is an AI assistant, one question per turn, details one at a time, always ask for a callback number and take it digit by digit, one fixed closing sentence (also in `speech_config.end_call_phrases`), and nothing said after it.
- Pipeline: write for the ear. No digits or symbols in running text; spell out numbers. Keep answers short.
- Speech-to-speech: describe pace and tone in words; spelling rules do nothing.
- Details, per-model blocks and templates: skill `voicedock-prompting`.

## 4. Create the assistant

- `createAssistant` (or `POST /assistants`) with the full config: `name`, `first_message`, `llm_config` (provider, model, messages, tools; plus `voice` for speech-to-speech), and for a pipeline `stt_config` and `tts_config`.
- Read it back with `getAssistant` and compare it with what you sent.
- **Updating replaces `llm_config` as a whole.** Always send the complete `llm_config`, never just the field you changed, or the prompt, model and tools you left out are gone.
- A change is live on the next call. There is no publish step; `published_at` is dashboard history only.

Built-in tools (transfer, end call, and others) and their parameters: `https://doc.voicedock.ai/docs/configuration/custom-tools` and `https://doc.voicedock.ai/docs/features/call-transfers`. Your own HTTP tools follow the same page.

## 5. Phone number and transfers

- Attach the assistant to a number (`updateNumber` with `assistant_id`). See `https://doc.voicedock.ai/docs/core-concepts/phone-numbers`.
- A transfer leaves over the SIP trunk on the number the call came in on (`transfer_trunk_id`). A managed number has one already. A number the user brings needs one of their SIP trunks assigned, or the transfer fails mid-call.
- Managed numbers cannot place outbound calls.
- Behind forwarding or a phone system the caller ID is often the business's own number: the prompt must ask for a callback number. Place one test call after setting up forwarding to see what comes through (`references/telephony.md`).
- Outbound: voicemail detection works on the first thing the other side says and does not hear the beep; keep `voicemail_message` short and self-contained (`https://doc.voicedock.ai/docs/features/voicemail-detection`).
- Do not attach a number or change a number's routing without the user's go: it changes what real callers reach.

## 6. Decide where the outcome goes

Without one of these, what the assistant collects lives only in the call record.

- `notification_emails`: a summary per call by email.
- A webhook: the call result into the user's own system. `https://doc.voicedock.ai/docs/core-concepts/webhooks`
- `analysis_plan`: structured fields (name, phone number, reason, urgency) extracted after each call. `https://doc.voicedock.ai/docs/features/call-analysis`. On speech-to-speech models set `postcall_stt_config` so the analysis uses a clean re-transcription.

## 7. Test before a real caller does

- Quick check without a number: a web call from the browser. `https://doc.voicedock.ai/docs/features/web-calls`
- Repeatable tests: Arena places real test calls with a persona, background noise, interruptions and per-criterion verdicts. `https://doc.voicedock.ai/docs/features/arena`. It needs a second number of the user's own with a SIP trunk to call from, and it costs credits; tell the user before you start a run.
- Never test by calling the phone numbers of real people, and never on the customer's live number: test a copy (skill `voicedock-test-and-review`).
- After a test, read the call back (`getCall` / `listCalls`): transcript, end reason, tool calls, analysis.

## 8. Report to the user

Say what you built (assistant id, model, voice), what you assumed in step 1, what you did not test, and what the user still has to do (number, trunk, email address, a test call by a human to judge the voice).
