---
name: voicedock-model-choice
description: Choose the model, voice and setup for a VoiceDock assistant - pipeline or speech-to-speech, which provider, which voice, EU data residency, accents and noisy callers, and when to add a backend model. Use before creating an assistant, when a user asks which model to pick, when callers are misunderstood or talked over, or when an assistant moves to another model.
---

# Choosing the model for a VoiceDock assistant

Model and voice names change. Take them only from the "Valid configuration values" section of `https://doc.voicedock.ai/llms.txt`; names are case-sensitive. This skill helps you pick; the docs give the exact names.

## 1. Start from the callers, not from the model

| Question | Why |
|---|---|
| Where do callers call from: quiet office, home, on the road, on site, speakerphone with others around? | How much the model must cope with noise and other voices |
| Which language(s)? Regional accents or dialects? | Some models hear a strong regional accent as another language and switch to it |
| Must call data stay in the EU? | Limits providers for every component: speech recognition, language model, voice and the post-call analysis model (`https://doc.voicedock.ai/docs/platform/eu-data-sovereignty`) |
| Must wording be exactly controlled (prices, legal text, names spelled back)? | Favours a pipeline |
| Lookups or careful reasoning during the call? | Favours tools, or a backend model on GPT-Live |

## 2. Pipeline or speech-to-speech

- **Pipeline** (separate speech recognition, language model, speech synthesis): most predictable, every call-control feature works, wording fully under your control, each part chosen separately. Slightly slower turn-taking. Needs `stt_config` and `tts_config`.
- **Speech-to-speech** (`openai_live`, `google_realtime`, `xai_realtime`, `elevenlabs_realtime`): lowest latency and the most natural conversation. The voice is `llm_config.voice`; `stt_config` and `tts_config` are ignored. The live transcript is less reliable than a pipeline's: set `postcall_stt_config` for summaries and analysis.

## 3. Rules of thumb

- **Callers in noisy places or on a speakerphone**: GPT-Live (`openai_live`). It decides its own turns, so it can be prompted to ignore background voices and not be interrupted by them. On models with voice activity detection in front, a voice in the room interrupts before the model can judge it.
- **A quiet setting and natural conversation**: compare two speech-to-speech models in the same test series (skill `voicedock-test-and-review`) and let the user listen.
- **Strict wording, many numbers or names spoken back**: a pipeline, with the writing rules from skill `voicedock-prompting`.
- **Regional accents**: test with callers who have them before you choose; write the language rule affirmatively in the prompt.
- **Talking over callers who pause to think** (server-side turn detection): raise the end-of-speech silence threshold (`llm_config.turn_detection.silence_duration_ms`).
- **Lookups during the call**: tools; on GPT-Live the backend model handles tools and reasoning (`https://doc.voicedock.ai/docs/integrations/openai-gpt-live-integration`).

## 4. Voice

- Pick two or three candidate voices for the caller's language with `listVoices` (or the docs), and let the user listen to a test call with each. A voice is judged by a human ear; do not decide it for the user.
- A model's default voice is not a choice: check that gender and accent match the persona's name.
- For EU-only setups, check the voice itself against the EU page, not only the language model.

## 5. Changing the model of a live assistant

- A prompt does not carry over unchanged: rewrite the model-specific blocks (skill `voicedock-prompting`).
- Updating replaces `llm_config` as a whole: send the complete config, never a single field.
- Test the new version as a copy first, then switch the live number over with the user's go.
- If an assistant suddenly sounds different, look for a recent config change first.
