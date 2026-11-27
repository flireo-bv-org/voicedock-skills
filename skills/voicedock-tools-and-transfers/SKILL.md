---
name: voicedock-tools-and-transfers
description: Add and phrase tools on a VoiceDock assistant so the model actually uses them - transferring to a human (transfer_call), ending the call (end_call), webhooks and MCP tools, and the outcome of a call (analysis, webhooks, email). Use when adding a transfer or tool, or when an assistant announces a transfer or action but nothing happens, transfers to voicemail, or does not hang up.
---

# Tools and transfers on VoiceDock

Sources: `https://doc.voicedock.ai/docs/features/call-transfers`, `https://doc.voicedock.ai/docs/configuration/custom-tools`, and for GPT-Live `https://doc.voicedock.ai/docs/integrations/openai-gpt-live-integration` (section Tools). Fetch them for exact schemas.

## 1. Transfer to a human

Config (in `llm_config.tools`):

- `type: "transfer_call"` with `destinations[]`: `number` in E.164, a `description` that tells the model when to use it, optional `message` spoken before the transfer.
- Prefer `wait_for_answer: true` with `fallback_destinations`: the caller is only connected when someone answers; otherwise control returns to the assistant, which can take a message. Set `answer_timeout_seconds` below the destination's ring-to-voicemail delay, because voicemail counts as an answer.
- The transfer leaves over the SIP trunk on the number the call came in on (`transfer_trunk_id`). A number the user brings needs one of their SIP trunks assigned, or the transfer fails mid-call.
- Transferring changes what real callers reach: set it up with the user's go and test it before the number goes live.

## 2. Phrase tools so the model uses them

These rules come from repeated test series; they decide most "it said it would, but nothing happened" cases.

- **Name the tool by its function name, in backticks, everywhere.** "You can transfer the caller" reads as something the model does itself; it says "I'll put you through" and nothing happens. Write instead:

  ```text
  You have the technical function `transfer_call`. Only `transfer_call` can put
  the caller through; you cannot do it yourself.
  Ask the caller once whether they want to be put through. When they say yes,
  call `transfer_call` in that same turn.
  Never say that you are putting the caller through without calling `transfer_call`.
  If `transfer_call` reports that nobody answered, say so and offer to take a message.
  ```

- **One short confirmation question before any tool that acts** (transfer, booking, sending a message). On GPT-Live the hand-off happens at the start of the model's reply to a caller turn; if it announces first, the hand-off often does not come. The caller's "yes" is the turn on which it acts. Only "one moment" may come before the call.
- **Never let the model state an outcome it did not get** ("nobody answers", "I tried"): only report what the tool returned.
- **On GPT-Live, tools run on the backend model.** Put each acting tool in the delegation block of the prompt and in `delegation.instructions`, and tell the backend that the caller has already confirmed, so it acts at once.
- **Ending the call**: give `end_call`, delegate right after the goodbye (GPT-Live), and put the fixed closing sentence in `speech_config.end_call_phrases` as a safety net.

## 3. Your own tools

- Webhook tools and MCP servers: describe in the tool `description` exactly when to call it and what to say while it runs. Keep parameters few and named in plain words.
- Say in the prompt what to do when a tool fails or returns nothing: apologise briefly and take a message; never invent a result.
- Use the user's own systems through their API or MCP server; do not ask for their passwords.

## 4. Where the outcome of a call goes

Without one of these, what the assistant collected stays only in the call record.

- `analysis_plan`: structured fields extracted after each call (name, phone number, reason, urgency). Everything the analysis extracts, the prompt must also ask for. `https://doc.voicedock.ai/docs/features/call-analysis`
- A webhook: the call result into the user's own system. `https://doc.voicedock.ai/docs/core-concepts/webhooks`
- `notification_emails`: a summary per call by email.

## 5. Check after a test call

Read the call back (`getCall`): did the tool call happen, at which turn, and what did it return? An assistant that said "I'm putting you through" without a `transfer_call` in the record has a phrasing problem (section 2), not a telephony problem.
