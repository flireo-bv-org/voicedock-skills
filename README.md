# VoiceDock for Claude

A plugin that lets Claude build, test and improve phone and web voice assistants on [VoiceDock](https://voicedock.ai). It connects Claude to your VoiceDock account through the VoiceDock MCP server and adds skills with what we learned building voice assistants: how to prompt each voice model, which model to pick, how to make tools and transfers actually happen, and how to test before real callers do.

## Install

**Claude Code**

```
/plugin marketplace add flireo-bv-org/voicedock-skills
/plugin install voicedock@voicedock
```

Then run `/mcp` and sign in with your VoiceDock account.

**Claude (web and desktop)**

Customize, Plugins, Add marketplace, and paste `https://github.com/flireo-bv-org/voicedock-skills`. Install VoiceDock, then connect it under Connectors and sign in with your VoiceDock account.

**Other agents (Cursor, Codex, Gemini CLI)**

```bash
npx skills add flireo-bv-org/voicedock-skills
```

and add the MCP server `https://mcp.voicedock.ai/mcp` in your agent's MCP settings.

## What's inside

| Skill | What Claude can do with it |
|---|---|
| `voicedock-voice-assistant` | Build or change an assistant end to end: requirements, model, prompt, number, transfers, outcome, test |
| `voicedock-prompting` | Write a prompt for the model it runs on, with templates for a receptionist and a webshop customer service line |
| `voicedock-model-choice` | Pick pipeline or speech-to-speech, provider, voice and EU data residency |
| `voicedock-tools-and-transfers` | Transfers, ending the call, own tools, and where the outcome of a call goes |
| `voicedock-test-and-review` | Test with web calls and Arena, read calls back, and improve in small steps |

The MCP server is `https://mcp.voicedock.ai/mcp`. It signs in with your own VoiceDock account and only reaches the organisations you belong to. Claude asks before any action that changes your account.

## Documentation

- Docs: https://doc.voicedock.ai
- For agents: https://doc.voicedock.ai/llms.txt

## License

MIT
