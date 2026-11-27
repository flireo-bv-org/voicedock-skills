---
name: voicedock-test-and-review
description: Test a VoiceDock assistant before real callers reach it and improve it from the results - web call, Arena test runs with simulated callers, reading calls back, separating assistant faults from caller or platform issues, and comparing two prompts or models. Use after building or changing an assistant, when a user reports a bad call, or when comparing models.
---

# Testing and improving a VoiceDock assistant

## 1. Ways to test

| Way | Good for | Source |
|---|---|---|
| Web call in the browser | A first listen, no number needed | `https://doc.voicedock.ai/docs/features/web-calls` |
| A human calling the number | The voice, the greeting, real background noise | - |
| Arena: simulated callers with a persona, noise and interruptions, verdict per criterion | Repeatable series, regressions after a change | `https://doc.voicedock.ai/docs/features/arena` |

Arena places real test calls: it needs a second number of the user's own with a SIP trunk, and it costs credits. Tell the user what a run costs before you start it. Never test by calling real people.

## 2. Write test cases from the prompt

For each task in the prompt, one ordinary case and one hard case. Always include:

- a caller who gives details in a different order, or corrects a digit;
- a caller who asks something the prompt does not cover;
- a caller who wants a human (if there is a transfer);
- a caller on a speakerphone with another voice in the room;
- a caller who says goodbye abruptly.

Give the simulated caller a rule for silence ("if it goes quiet, ask: is that all right?"), or a silent assistant and a silent caller wait for each other until the time limit.

## 3. Judge in series, not on one call

Speech models vary from call to call. One good call proves nothing and one bad call is not a cause. Run at least six calls per version before you conclude, and compare a change against the same series without it. Count per criterion: did the tool call happen, was the number read back correctly, did it hang up.

## 4. Read the call back

`getCall` / `listCalls`: transcript, end reason, tool calls with their results, analysis, and the recording. Then sort each problem:

- **Assistant**: said something wrong, skipped a step, announced an action without the tool call, read a number wrong. Fix the prompt (skills `voicedock-prompting`, `voicedock-tools-and-transfers`).
- **Caller**: the simulated caller misbehaved or the test case was unclear. Fix the case, not the prompt.
- **Platform or setup**: no audio, a transfer that failed on the trunk, a tool that returned an error. Check the configuration; report it to VoiceDock support if it persists.

Timing: measure pauses in the recording, not in message timestamps; turn boundaries are recorded with a small delay.

## 5. Improve in small steps

Change one thing per round, rerun the same series, and keep the version that scores better. A rule that changes nothing in the results can go.

## 6. Report to the user

What you tested (cases, number of calls), what passed, what failed and why, what you changed, and what a human still has to judge: the voice, and one call on the real number before it goes live.
