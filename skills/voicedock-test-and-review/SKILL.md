---
name: voicedock-test-and-review
description: Test a VoiceDock assistant before real callers reach it and improve it from the results - a safe test copy, web calls, Arena runs with simulated callers, test-caller rules, verdicts per call, reading calls and recordings back, comparing two prompts or models, and the last check before go-live. Use after building or changing an assistant, when a user reports a bad call, or when comparing models.
---

# Testing and improving a VoiceDock assistant

## 1. Never test on the live line

Test a copy, not the assistant real callers reach. Calls to a live number send report emails, use minutes and pollute the call list, analytics and webhooks.

- Copy the assistant to a test number, with `notification_emails` empty, no webhook, and transfers pointed at a reserved fictional number (for example the UK drama range `+44 7700 900000` to `+44 7700 900999`, which the network rejects). That is also how you test a transfer nobody answers.
- Before calling any number, check which assistant it belongs to.
- Without the live integrations, tell the backend in a clearly marked test line what a tool "returns", and report that you tested the conversation, not the integration.

## 2. Ways to test

| Way | Good for | Source |
|---|---|---|
| Web call in the browser | A first listen, no number needed | `https://doc.voicedock.ai/docs/features/web-calls` |
| Arena: simulated callers with persona, noise, interruptions, verdict per criterion | Repeatable series, regressions | `https://doc.voicedock.ai/docs/features/arena` |
| A human calling | The voice, real background noise, a real transfer | - |

Arena places real calls: it needs a second number of the user's own with a SIP trunk, and it costs credits. Tell the user what a run costs before you start it. Arena never places transfers; test those by hand. Never test by calling real people.

## 3. Test-caller rules

- Wait until the assistant speaks; after five seconds of silence say "hello?".
- Never state a phone number, even when asked ("I don't have a mobile"): a simulated caller invents real numbers, and messages or links go to strangers.
- Decline transfers. If transferred and someone answers, apologise and stop.
- Stay in your own language even if the assistant switches.
- Always set `max_duration_seconds`.
- A simulated caller says "yes" to every readback, so a correct readback proves little: check the captured value against the script, and include a case with a correction.

## 4. Test set for a receptionist

Normal request; request with an email address (spelled and confirmed); caller pushes for a price; outside the service area (friendly no, goodbye not cut off); borderline case (noted, a human decides); existing customer (transfer, message on no answer); asks for the owner (one confirmation, then transfer); frequent questions (answers only from the source); "am I talking to a computer?" (honest yes); caller on a speakerphone with another voice in the room; caller gives details in another order or corrects a digit; abrupt goodbye. Schedule time-related cases at a moment they make sense.

## 5. Verdicts

One verdict per call on the current version:

- **Good**: the conversation went well, not only the outcome. Read the transcript as the customer would: did the caller have to repeat themselves, was every wish answered honestly, any words in the wrong language or cut-off sentences?
- **Good with a remark**: goal met, small issue (a wrong path where the caller was still helped).
- **Not good**: blocking. The assistant gets stuck or leaves long dead air, gives dangerous or invented advice, shares someone else's data, or the caller is not helped and nothing is recorded. Stop the round at the first blocking fault and fix it first.

Speech models vary from call to call: run at least six calls per version before you conclude, and never conclude from one call.

## 6. Read the call back

`getCall` / `listCalls`: transcript, end reason and error detail, tool calls with their results, analysis, and the recording.

- Measure pauses in the recording, not in message timestamps; on speech-to-speech models those lag by a second or more.
- A short call without messages is not necessarily silent: check the audio first. Callers who hang up during a long greeting look like this too.
- Sort each problem: **assistant** (fix the prompt), **test caller or case** (fix the case), **platform or setup** (check the configuration; report to support if it persists). A misconfigured call is rejected with a reason in the call log; the platform does not silently switch to another model or voice.

## 7. Improve

- Small fixes: change one thing, rerun the same series.
- A prompt that grew from many fixes: rewrite it around a few steps with one outcome each and compare old and new on the full scenario set with exactly the same judging instructions. Also test a rule you copied from elsewhere against a version without it: a rule can do nothing.

## 8. Before go-live

One human call with background noise, a side conversation, an interruption, a question the assistant cannot answer, a price push, a transfer and the goodbye. Then one real transfer to the real destination (warn that phone), and point the test copy back to the fictional number afterwards.

## 9. Report to the user

What you tested (cases, number of calls) with a recording link per call, the verdicts, what you changed, and what a human still has to judge: the voice, real background noise, a real transfer.
