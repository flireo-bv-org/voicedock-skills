# Privacy and compliance checklist

Not legal advice: the business that runs the assistant decides what applies to it, with its own counsel. Sources: `https://doc.voicedock.ai/docs/features/privacy-compliance`, `https://doc.voicedock.ai/docs/platform/eu-data-sovereignty`, and the VoiceDock Trust Center.

## Questions to ask before building

| Question | What to configure |
|---|---|
| Must callers know they speak with an AI? (In the EU the AI Act has transparency duties for AI systems that talk to people.) | Say "AI assistant" or "digital assistant" in the greeting, and answer honestly when asked. Test it. |
| Must callers consent before the call is recorded or processed? | `recording_consent` (the caller presses a key; inbound phone calls). |
| May call content be stored on the platform at all? | `gdpr_mode`: no transcript or recording kept; summary and analysis go only to your webhook. |
| How long may recordings and transcripts be kept? | Retention per organisation; delete a single recording when a caller objects. |
| Must all processing stay in the EU (health, legal, public sector)? | Pick EU options for every component: speech recognition, language model, voice, and the post-call analysis model. A voice counts too. Check the EU page for which options qualify. |
| Does the caller share special categories of data (health, for example)? | Keep the assistant's task narrow (take a message, route), avoid asking for more than needed, and check the processing agreement. |

## What not to promise

- EU data residency is not the same as zero retention: providers can keep data for a limited time for abuse monitoring or caching. Only promise what the Trust Center states for your setup.
- Do not let the assistant give medical, legal or financial advice; route to a human.
