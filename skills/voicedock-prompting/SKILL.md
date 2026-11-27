---
name: voicedock-prompting
description: Write or fix the system prompt of a VoiceDock voice assistant for the model it runs on (pipeline, GPT-Live, Gemini Live, Grok Realtime). Use when writing a new prompt, or when an assistant ignores a rule, talks over callers, reacts to background voices, answers in the wrong language, reads phone numbers wrong, skips questions, goes silent, says "one moment" and nothing happens, or does not hang up.
---

# Prompting a VoiceDock voice assistant

A prompt that works on one voice model can be word for word wrong on another, and the failure is silent: the rule reads well and simply never happens. Decide the model first (skill `voicedock-model-choice`), then write for it.

Source of truth: `https://doc.voicedock.ai/docs/guides/prompting-per-model`. Fetch it. This skill adds the order of work and the rules that most often decide whether a phone assistant works. For GPT-Live, also read `references/gpt-live.md`.

## 1. What the prompt can steer

| | Separate speech step | Who decides the turn | Steerable in the prompt |
|---|---|---|---|
| Pipeline (`openai`, `google`, `mistral`, `xai` LLM) | yes | speech settings | wording, spelling, length |
| Gemini Live (`google_realtime`) | no | voice activity detection | persona, rules, tool phrasing, pace in words |
| GPT-Live (`openai_live`) | no | the model itself | all of the above plus interruptions, listening sounds, background voices |
| Grok Realtime (`xai_realtime`) | no | server side | persona, rules, tool phrasing |

Remove every rule the model cannot carry out. A rule that does nothing is worse than no rule, because it looks handled.

## 2. Scope first

A phone assistant that takes calls, answers known questions, collects details and routes is reliable. One that is asked to handle everything invents answers. Keep hard promises (prices, dates, legal or medical advice) with a human, and say so in the prompt.

## 3. Structure

Each block under a plain heading, in this order:

1. **Language**, on its own right after the role: which language, always. Say that the greeting's language says nothing about the caller's language, and that a regional accent is still that language. Write it affirmatively. Never give the model a recovery line that names a language ("that sounded German"); it will say it to callers.
2. **Who you are**: business, role, what you handle, what you never handle. Say that the caller is speaking with an AI assistant and that it answers honestly when asked "am I talking to a computer?".
3. **Greeting**: one short sentence (or `first_message`). Callers hang up during long greetings.
4. **How you speak**: short turns, one question per turn, no gender assumptions (no "Mr/Mrs", formal "you" where the language has it).
5. Model-specific blocks (section 5).
6. **Taking details** (section 4).
7. **Tasks**: one block per task, numbered steps, one action per step.
8. **Facts** (section 6).
9. **Ending the call** (section 4, closing).

## 4. Rules that hold on every model

**Questions and flow**
- One question per turn, and the question is the last thing said. After answering a side question, continue in the same turn with the next open question; otherwise details are never collected.
- Write per step "if the caller has not said this yet", rather than one general "never ask what was already said". Add: "continue with the first question in the list that is still unanswered", and check before the summary that every required detail is there.
- Ask for details one at a time: name, then reference number, then a callback number. Asked together, the caller gives one and waits, while the assistant waits too.
- Avoid answer lines that end with "anything else I can help with?" in the middle of a task: they push the model into closing the call.

**Phone numbers**
- Always ask for a callback number. When a business forwards calls from its own phone system, the caller ID is often the business's own number, and a withheld number arrives with no number at all.
- Ask the caller to say the number digit by digit, and read it back digit by digit, each digit on its own. In languages that say the unit first (Dutch, German: "two-and-thirty"), speech models swap digits when they treat a pair as one number, both when hearing and when reading back.
- Say how many digits a local number has. After a correction, read the whole number again.
- If the model keeps mishearing addresses or names, endless readback makes the call worse. Consider not reading back, let the analysis re-transcribe the recording, and treat an answer that does not fit the question as misheard: ask again, at most twice.
- List names the caller will say (staff, streets, products) with their likely misrecognitions as variants of the right name.

**Lookups and logic**
- Never let the assistant say it will look something up unless a tool or backend actually does. "Let me check that" without anything behind it produces silence.
- Dates, thresholds, conditional offers and safety rules (gas, smoke, fire) are applied inconsistently from a prompt. Compute them in your backend or tool and return the result.
- The platform replaces `{{ now }}` (written with spaces) by the current time; write it exactly like that. Do not ask the model to convert clock times; give examples.

**Silence**
- Do not write a silence policy in the prompt. Use the platform's `autonomous_silence_handling` with fixed sentences in `speech_config.silence` (`messages`, `final_message`). A prompt rule either does nothing or makes the caller hear it twice. See `https://doc.voicedock.ai/docs/features/autonomous-silence-handling`.

**Closing**
- One fixed closing sentence per language. Put it in `speech_config.end_call_phrases` too: the platform hangs up when a turn *ends* on it (`https://doc.voicedock.ai/docs/features/closing-phrases`).
- Add: "After that sentence, say nothing more, even if the caller says goodbye again."
- The goodbye is its own short turn. Do not combine a summary or a refusal with the goodbye: after a hang-up the platform waits only briefly for the last words, so a long final turn is cut off.
- Never embed the closing sentence inside another fixed line used mid-call.

**Native language**
- For a non-English assistant, ask for the language "like a native speaker from <region>", list translated English stock phrases it must avoid (in Dutch for example "goede vraag", "geen zorgen"), and allow listening sounds only in the caller's language.
- Check that the voice matches the persona: a model's default voice is not a choice ("Anna" with a male default voice sounds wrong).

**Nothing visual**: no markdown, lists, URLs, emoji or quotation marks in what the assistant says. Describe where to find something instead of reading out an address. Avoid asking for an email address unless the task needs it; if you must, have it spelled and read back letter by letter, with a way out after two failed attempts.

## 5. Model-specific blocks

**Pipeline**: write for the ear. No digits or symbols in running text, numbers as words, phone numbers digit by digit separated by commas. Commas are the only pause that works; extra dots add pause and artefacts. Pronunciation fixes go in the assistant's pronunciation replacements; recognition of names can be helped with keyword boosting where the speech recognizer offers it.

**Gemini Live**: the instruction is fixed for the whole call, so keep it short and complete. State the language twice, affirmatively. Describe pace in words. On `gemini-3.8-live`, give an explicit rule to hang up only after the caller said goodbye. Interruptions and background voices cannot be fixed in the prompt on this model.

**GPT-Live**: see `references/gpt-live.md`: the Personality, Backchannel, Interruption and Background voices blocks; the delegation block (keep it verbatim); a short voice prompt with knowledge in the backend instructions; and how to avoid "one moment" with nothing behind it.

**Grok Realtime**: persona, rules in execution order, short tool sentences, guardrails last. If it talks over callers who pause to think, raise the end-of-speech silence threshold (`llm_config.turn_detection.silence_duration_ms`) rather than writing a prompt rule.

## 6. Facts: complete beats short when there is no lookup

If the assistant has no tool or backend to look things up, the prompt is all it knows: put every fact a caller can ask in it, one line each (hours, address, contact, delivery, returns, warranty, payment, services, frequently asked questions). Leave out what changes daily (single prices, stock, discount codes) and what is private (bank details, personal mobile numbers); say where to send the caller instead. Where sources disagree, pick one value from the most official source or leave it out.

On GPT-Live with a backend model, keep the voice prompt short and put the knowledge in the backend instructions instead (`references/gpt-live.md`).

Templates: `references/receptionist-template.md`, `references/webshop-template.md`.

## 7. Maintain

After a test round, read the whole prompt again (and the backend instructions, and the text your tools return). A prompt that grew from many small fixes is usually better rewritten around a few steps with one outcome each, then tested against the old version (skill `voicedock-test-and-review`).

## 8. Before it goes live

- Every rule matches what the model can do (section 1).
- Language block on top; short greeting with AI disclosure; one closing sentence also in `end_call_phrases`; nothing said after it.
- Callback number always asked, digit by digit; details one at a time.
- No "let me check" without a lookup; no silence rule in the prompt; logic in code.
- Then test it: skill `voicedock-test-and-review`.
