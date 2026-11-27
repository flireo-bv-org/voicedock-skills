---
name: voicedock-prompting
description: Write or fix the system prompt of a VoiceDock voice assistant for the model it runs on (pipeline, GPT-Live, Gemini Live, Grok Realtime, ElevenLabs Realtime). Use when writing a new prompt, when an assistant ignores a rule, talks over callers, reacts to background voices, announces a transfer without doing it, goes silent, reads numbers wrong, or does not hang up.
---

# Prompting a VoiceDock voice assistant

A prompt that works on one voice model can be word for word wrong on another, and the failure is silent: the rule reads well and simply never happens. Decide the model first (skill `voicedock-model-choice`), then write for it.

The source of truth is `https://doc.voicedock.ai/docs/guides/prompting-per-model`. Fetch it; this skill adds the order of work and the rules that most often decide whether a phone assistant works.

## 1. Two questions decide what a prompt can steer

| | Separate speech step | Who decides the turn | Steerable in the prompt |
|---|---|---|---|
| Pipeline (`openai`, `google`, `mistral`, `xai` LLM) | yes | speech settings | wording, spelling, length |
| Gemini Live (`google_realtime`) | no | voice activity detection | persona, rules, tool phrasing, pace in words |
| GPT-Live (`openai_live`) | no | the model itself | all of the above plus interruptions, listening sounds, background voices |
| Grok Realtime (`xai_realtime`) | no | server side | persona, rules, tool phrasing |

Remove every rule the model cannot carry out. A rule that does nothing is worse than no rule, because it looks handled.

## 2. Structure

Write in this order, each block under a plain heading:

1. **Language**, first and on its own: which language, always, also when the caller uses a word from another language.
2. **Who you are**: name of the business, role, what you handle and what you never handle.
3. **Greeting**: the exact first sentence (or set `first_message`).
4. **How you speak**: short turns, one question at a time, form of address.
5. Model-specific blocks (section 4).
6. **Taking details**: see section 3.
7. **Tasks**: one block per task, steps numbered, each step one action.
8. **Facts**: everything the assistant may say about the business. See section 5.
9. **Ending the call**: the fixed closing sentence (section 3).

## 3. Rules that hold on every model

- **One question per turn, and the question is the last thing said.** Callers answer the moment they hear a question.
- **Ask for details one at a time**: name, then the reference number, then a callback number. Asking for three things at once makes the caller give one and wait, while the assistant waits too.
- **Read numbers back digit by digit**, each digit on its own ("three two", never "thirty-two"). After a correction, read the whole number again.
- **Never claim to look something up unless a tool or backend actually does.** "Let me check that" without anything behind it produces silence. Without a lookup, answer from the prompt or take a message.
- **One fixed closing sentence**, said as the last turn and nothing after it. Put that exact sentence in `speech_config.end_call_phrases` so the platform hangs up even if the model forgets (`https://doc.voicedock.ai/docs/features/closing-phrases`).
- **Avoid asking for an email address** unless the task needs it; a phone number is easier to say and confirm. If you must, let the caller spell it and read it back letter by letter, with a way out after two failed attempts.
- **Nothing visual**: no markdown, lists, URLs, emoji or quotation marks in what the assistant says.
- **Dates**: the platform replaces `{{ now }}` (with spaces) by the current time. Other spellings stay literal. Do not ask the model to convert clock times; give examples instead.

## 4. Model-specific blocks

**Pipeline**: write for the ear. No digits or symbols in running text, numbers as words, phone numbers digit by digit separated by commas. Commas are the only pause that works; extra dots add pause and artefacts. Pronunciation fixes belong in the assistant's pronunciation replacements, not in the prompt.

**Gemini Live**: the instruction is fixed for the whole call, so keep it short and complete. State the language twice, affirmatively. Describe pace in words. Give an explicit rule to hang up only after the caller said goodbye. Interruptions and background voices cannot be fixed in the prompt on this model.

**GPT-Live**: add these blocks under English headings, also in a prompt in another language:
- `Personality`: tone and pace in words.
- `Backchannel policy`: silence while the caller speaks is the default; one short acknowledgement only when the caller pauses inside a long story and waits. Any "now and then" instruction is carried out constantly on this model, so bound every exception with a condition and a maximum.
- `Interruption policy`: stop the moment the caller interrupts, then continue from what still matters.
- `Background voices`: the conversation partner is only the caller; never react to a TV or people in the room; if unsure, ask briefly whether the caller was talking to you.
- Tools and hand-offs: see skill `voicedock-tools-and-transfers`. In short: name every tool by its function name, ask one confirmation question before any tool that acts, and delegate right after the goodbye.

**Grok Realtime**: like Gemini Live without the language rule: persona, rules in execution order, short tool sentences, guardrails last.

## 5. Facts: complete beats short when there is no lookup

If the assistant has no tool or backend to look things up, the prompt is all it knows. Then put every fact a caller can ask in it: opening hours, address, contact, delivery, returns, warranty, payment, services, frequently asked questions, one line each. Leave out what changes daily (prices of single products, stock, discount codes) and what is private (bank details, personal mobile numbers); tell the assistant where to send the caller for those. Where sources disagree, pick one value from the most official source or leave it out.

A ready template for a webshop or retail customer service line is in `references/webshop-template.md`; a general receptionist template is in `references/receptionist-template.md`.

## 6. Before it goes live

- Every rule matches what the chosen model can do (section 1).
- Language block on top; fixed greeting; fixed closing sentence also in `end_call_phrases`.
- Details asked one at a time; numbers read back digit by digit.
- No "let me check" without a lookup behind it.
- Then test it: skill `voicedock-test-and-review`.
