# VoiceDock Agent Skills

Agent Skills that teach AI coding agents (Claude Code, Cursor, Codex, Gemini CLI and others that read `SKILL.md`) how to build voice assistants on [VoiceDock](https://voicedock.ai).

## Install

```bash
npx skills add flireo-bv-org/voicedock-skills
```

Or copy `skills/voicedock-voice-assistant` into your agent's skills folder (for Claude Code: `~/.claude/skills/`).

The skill works best together with the VoiceDock MCP server:

```json
{
  "mcpServers": {
    "voicedock": { "type": "http", "url": "https://mcp.hmsovereign.com/mcp" }
  }
}
```

## Skills

| Skill | What it does |
|---|---|
| `voicedock-voice-assistant` | Build, change and test a phone or web voice assistant: pick pipeline or speech-to-speech, write the prompt for that model, create it, attach a number, set up transfers, route the outcome, and test before real callers. |

## Documentation

- Docs: https://doc.voicedock.ai
- For agents: https://doc.voicedock.ai/llms.txt

## License

MIT
