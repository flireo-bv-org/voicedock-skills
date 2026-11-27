# GPT-Live (`openai_live`): what to write

GPT-Live decides its own turns, so conversational behaviour really is steered by the prompt. It runs with a backend model: the voice model talks, the backend model calls tools and reasons. Sources: `https://doc.voicedock.ai/docs/guides/prompting-per-model` (GPT-Live) and `https://doc.voicedock.ai/docs/integrations/openai-gpt-live-integration`.

## Blocks under English headings

Keep these headings in English, also in a prompt in another language.

- `Personality`: tone and pace in words.
- `Backchannel policy`: silence while the caller speaks is the default, because anything said then sounds right through them. One short acknowledgement only when the caller pauses inside a long story and waits. When in doubt, stay silent.
- `Interruption policy`: stop the moment the caller interrupts; then continue from what still matters.
- `Background voices`: the conversation partner is only the caller. A TV or people in the room are not. Never react to them or answer their questions. If the caller talks to someone else, wait. If unsure, ask briefly whether they were talking to you.

The model hears the caller continuously, so any "now and then" instruction is carried out constantly. Bound every exception with a named condition and a maximum.

## Delegation

- Put a `Delegation policy` block at the top, in English, with the technical tool names (`transfer_call`, `end_call`, your own tools) as backend capabilities. When you rewrite the prompt for tone or another language, copy this block first and keep it word for word.
- Never forbid delegation or narrow it to a few fixed moments. The voice model says its "one moment" before it decides to delegate; a ban turns that into "one moment" with nothing behind it. To reduce "one moment", remove the reason it delegates (put frequent answers in the voice prompt).
- Add: "Only delegating does something. Never ask the caller to wait without delegating in the same turn. Names, places and prices only from what the backend returned in this call. If the caller speaks again before the backend answered, delegate in that turn."
- Add "Delegate silently" and "say nothing after the closing sentence", or the model narrates the hang-up.
- Every tool that acts must be listed as a backend capability. A policy that says "do not delegate for anything else" without that tool means the tool is never used.

## Short voice prompt, backend as knowledge base

Keep the voice prompt short: persona, conversation flow, the most frequent questions. Put the rest of the company knowledge in `delegation.instructions`. Once `instructions` is set, the backend sees only that text, so also put there: when to transfer and hang up, the rules (short answers, no prices you may not give, "I don't know, a colleague will get back to you"), and the language rule with "translate your answer into the caller's language". A backend that answers in another language drags the voice along. In the backend instructions also write: "The same question again is not a second action; report the result you already have."

## Tools

Name every tool by its function name in backticks and ask one confirmation question before any tool that acts. Details and wording: skill `voicedock-tools-and-transfers`.

## Non-English assistants

An explicit accent line at the top helps ("speak German like a native speaker from <city>, with no American or English accent"), repeated in the target language. Do not combine it with a foreign-accent instruction, which can flip the whole answer into that language.

## Opening and fixed lines

On GPT-Live the first message, an outbound opening line and the silence lines are instructions, not scripts: the model may reword them. If the exact wording matters, test it.
