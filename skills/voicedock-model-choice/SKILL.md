---
name: voicedock-model-choice
description: Choose the model, voice and setup for a VoiceDock assistant - pipeline or speech-to-speech, which provider, which voice, EU data residency, and when to add a backend model. Use before creating an assistant, when a user asks which model to pick, or when an assistant needs to move to another model.
---

# Choosing the model for a VoiceDock assistant

Model and voice names change. Take them only from the "Valid configuration values" section of `https://doc.voicedock.ai/llms.txt`; names are case-sensitive. This skill helps you pick; the docs give the exact names.

## 1. Start from the callers, not from the model

Ask (or state your assumption):

| Question | Why |
|---|---|
| Where do callers call from: quiet office, home, on the road, on site, speakerphone with others around? | Decides how much the model must cope with noise and other voices |
| Which language(s)? Dialects or accents? | Not every voice and recognizer is equally good per language |
| Must call data stay in the EU (health, legal, public sector)? | Limits providers: `https://doc.voicedock.ai/docs/platform/eu-data-sovereignty` |
| Must wording be exactly controlled (prices, legal text, spelling of names)? | Favours a pipeline |
| Does the assistant need lookups or careful reasoning during the call? | Favours a backend model or tools |

## 2. Pipeline or speech-to-speech

- **Pipeline** (separate speech recognition, language model, speech synthesis): most predictable, every call-control feature works, wording fully under your control, and you can choose each part. Slightly slower turn-taking. Needs `stt_config` and `tts_config`.
- **Speech-to-speech** (`openai_live`, `google_realtime`, `xai_realtime`, `elevenlabs_realtime`): lowest latency and the most natural conversation. The voice is set in `llm_config.voice`; `stt_config` and `tts_config` are ignored.

## 3. Rules of thumb

- **Callers in noisy places or on a speakerphone**: GPT-Live (`openai_live`). It decides its own turns, so it can be prompted to ignore background voices and not be interrupted by them. On models with voice activity detection in front, a voice in the room interrupts before the model can judge it.
- **A quiet setting and natural conversation**: any speech-to-speech model works; compare two in a test series (skill `voicedock-test-and-review`).
- **Strict wording, many numbers or names spoken back**: a pipeline, with the writing rules from skill `voicedock-prompting`.
- **Lookups during the call**: tools on the assistant; on GPT-Live a backend model handles tools and reasoning (`https://doc.voicedock.ai/docs/integrations/openai-gpt-live-integration`).
- **EU-only data**: pick from the providers the EU page lists, and check the voice as well, not only the language model.

## 4. Voice

Pick two or three candidate voices for the caller's language with `listVoices` (or the docs), then let the user listen to a test call with each. A voice is judged by a human ear; do not decide it for the user.

## 5. Changing the model of a live assistant

- A prompt does not carry over unchanged: rewrite the model-specific blocks (skill `voicedock-prompting`).
- Updating replaces `llm_config` as a whole: send the complete config, never a single field.
- Test the new version as a copy first, then switch the live number over with the user's go.
