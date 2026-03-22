# lightweight-subagent-dev-workflow

A pragmatic Codex skill for implementation-oriented coding tasks.

It gives Codex a lightweight end-to-end workflow for feature work, bug fixes, refactors, and test updates. The main agent keeps ownership of requirements, planning, integration, verification, and final acceptance. Subagents are available, but only when complexity or risk justifies them.

## Files

- `SKILL.md`: the workflow itself
- `agents/openai.yaml`: UI metadata and implicit invocation policy

## Install

Clone or copy this folder into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
cd ~/.codex/skills
git clone https://github.com/pixelsama/lightweight-subagent-dev-workflow.git
```

Restart Codex if the skill does not appear immediately.

## License

MIT
