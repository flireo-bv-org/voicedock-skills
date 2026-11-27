# Telephony: numbers, trunks and forwarding

Sources: `https://doc.voicedock.ai/docs/core-concepts/phone-numbers`, `https://doc.voicedock.ai/docs/features/call-transfers` (Using SIP Trunks).

## Getting calls to the assistant

| Situation | Route |
|---|---|
| A new number is fine | A managed number from VoiceDock (requested in the dashboard, assigned by VoiceDock). Inbound only; it cannot place outbound calls. |
| The business keeps its own number and its provider offers a SIP trunk | Connect the trunk to VoiceDock and point the number at the assistant. Needed for outbound calls and transfers from your own number. |
| The business keeps its own number, no SIP trunk available | Forward the existing number to the assistant's number: always, when busy, when not answered, or on a schedule. Many hosted business phone systems only offer forwarding. The forwarding minutes are charged by their own provider. |

- Inbound SIP is accepted from the provider's fixed addresses. A provider that can only register a phone or extension cannot send calls directly; use forwarding, or ask VoiceDock support.
- If you connect your own carrier account, check where it anchors media. A media region on another continent adds delay in both directions, and a transferred call pays it twice. Pick a region close to the platform (Europe).
- Trunks can use TLS (`transport: tls`, port 5061).

## Caller ID

- Behind forwarding or a phone system, the call often arrives with the business's own number as caller ID. The assistant then does not know who is calling: always ask for a callback number in the prompt, and after setting up forwarding place one test call to see which number comes through.
- A withheld number arrives without a number (`null`). Tools and webhooks must handle that.
- On a transfer, whether the destination sees the caller's number depends on the carrier.

## Before switching a live number

Changing which assistant a number points to, or its trunk, changes what real callers reach. Do it with the user's go, after testing a copy (skill `voicedock-test-and-review`), and check afterwards with one real call.
