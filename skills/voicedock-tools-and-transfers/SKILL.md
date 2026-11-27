---
name: voicedock-tools-and-transfers
description: Add and phrase tools on a VoiceDock assistant so the model actually uses them - transferring to a human (transfer_call), ending the call (end_call, closing phrases), webhooks and MCP tools, text messages, and where the outcome of a call goes (analysis, webhooks, email). Use when adding a transfer or tool, or when an assistant announces a transfer or action but nothing happens, transfers into a loop or voicemail, hangs up too early, does not hang up, or the analysis has wrong details.
---

# Tools and transfers on VoiceDock

Sources: `https://doc.voicedock.ai/docs/features/call-transfers`, `https://doc.voicedock.ai/docs/configuration/custom-tools`, `https://doc.voicedock.ai/docs/features/closing-phrases`, `https://doc.voicedock.ai/docs/features/call-analysis`, and for GPT-Live `https://doc.voicedock.ai/docs/integrations/openai-gpt-live-integration` (section Tools). Fetch them for exact schemas.

## 1. Transfer to a human

- `type: "transfer_call"` with `destinations[]`: E.164 `number`, a `description` that tells the model when to use it, optional `message` spoken before the transfer.
- Prefer `wait_for_answer: true` with `fallback_destinations`: the caller is only connected when someone answers; otherwise control returns to the assistant. Set `answer_timeout_seconds` below the destination's ring-to-voicemail delay, because voicemail counts as an answer.
- Transfer to a direct or mobile number, never to the business's main number if that number forwards unanswered calls to the assistant: that is a loop.
- Announce the transfer in one place only: either the prompt says what to say, or the destination has a `message`. Both, and the caller hears it twice.
- On no answer or busy: never say the transfer worked. Take name, subject and callback number, read it back, close. Never let the model state an outcome it did not get from the tool ("nobody answers", "I tried").
- The transfer leaves over the SIP trunk on the number the call came in on (`transfer_trunk_id`). A number the user brings needs one of their trunks assigned, or the transfer fails mid-call.
- Whether the person you transfer to sees the caller's number depends on the carrier; some refuse to present a number they do not own. Test what the receiving phone shows before promising it.
- Transferring changes what real callers reach: set it up with the user's go and test it before the number goes live.

## 2. Phrase tools so the model uses them

- **Name the tool by its function name, in backticks, everywhere.** "You can transfer the caller" reads as something the model does itself; it says "I'll put you through" and nothing happens. Write instead:

  ```text
  You have the technical function `transfer_call`. Only `transfer_call` can put
  the caller through; you cannot do it yourself.
  Ask the caller once whether they want to be put through. When they say yes,
  call `transfer_call` in that same turn.
  Never say that you are putting the caller through without calling `transfer_call`.
  If `transfer_call` reports that nobody answered, say so and offer to take a message.
  ```

- **One short confirmation question before any tool that acts** (transfer, booking, sending a message). On GPT-Live the hand-off happens at the start of the model's reply to a caller turn; if it announces first, the hand-off often does not come. The caller's "yes" is the turn on which it acts.
- **On GPT-Live a tool depends on three places**: the voice prompt (decides to hand off), the tool `description`, and `delegation.instructions` (the backend sees and calls the tools). When a tool fires late or never, adjust all three, and make sure the tool is listed as a backend capability in the delegation block (`voicedock-prompting`, `references/gpt-live.md`). Tell the backend that the caller has already confirmed, so it acts at once.
- **Slow tools** (an outside knowledge source, an MCP server) leave the caller in silence: say in the prompt what to say while it runs, ask an outside chatbot one question at a time, and keep the most frequent answers in the prompt.
- Say what to do when a tool fails or returns nothing: apologise briefly and take a message; never invent a result.
- Tool names must be unique on an assistant, and a webhook tool cannot be called `end_call` or `transfer_call`: saving such an assistant is refused with a `400`.
- Use the user's systems through their API or MCP server; never ask for their passwords.

## 3. Ending the call

- Give `end_call` and a fixed closing sentence; put that sentence in `speech_config.end_call_phrases` as a safety net. On GPT-Live, delegate right after the goodbye.
- **A model that hangs up too early** (right after a side question, before reading back) is not fixed by a prompt rule. Remove `end_call` from the tools, end calls with `end_call_phrases`, and give silence handling a `final_message` (`https://doc.voicedock.ai/docs/features/autonomous-silence-handling`). Keep `max_duration_seconds` as a brake.
- Updating replaces `llm_config` and `speech_config` each as a whole: when you change `silence`, send `end_call_phrases` along, and the other way round.

## 4. Text messages (any SMS tool or webhook)

- By default send only to the number the caller is calling from, and only if it is a mobile number. Never to a number the model heard: one misheard digit sends it to a stranger.
- If the caller must give another number, read it back digit by digit right before sending, ask "did I get that right?", send only on a clear yes, validate the format in your backend, and send at most one per call.
- Ask before sending anything the caller did not ask for. Links deliver best on your own domain from a real mobile sender; public link shorteners are blocked more often.

## 5. Where the outcome of a call goes

Without one of these, what the assistant collected stays only in the call record.

- `analysis_plan`: structured fields extracted after each call. Everything the analysis extracts, the prompt must also ask for. In the analysis prompt, take names, addresses and numbers **as the caller said them**, not as the assistant repeated them.
- On speech-to-speech models the live transcript can miss words; set `postcall_stt_config` so the recording is re-transcribed before summary and analysis (`https://doc.voicedock.ai/docs/features/call-analysis`).
- A webhook: the call result into the user's own system (`https://doc.voicedock.ai/docs/core-concepts/webhooks`). Webhook settings can exist on organisation, number and assistant; check which one actually applies and is signed. A caller with a withheld number arrives without a number; your handler must cope with that.
- `notification_emails`: a summary per call by email.

## 6. Check after a test call

Read the call back (`getCall`): did the tool call happen, at which turn, and what did it return? An assistant that said "I'm putting you through" without a `transfer_call` in the record has a phrasing problem (section 2), not a telephony problem. For a failed call, read the error detail, not only the end reason, before you change anything.
