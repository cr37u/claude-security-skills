# Claude Security Skills

Three [Claude Code](https://claude.com/claude-code) skills for finding, fixing, and vetting security issues — adapted from OpenAI's [codex-security](https://github.com/openai/codex-security) plugin and rewritten to run entirely on Claude Code's own tools. No external scanner, MCP server, or OpenAI API key required.

| Skill | What it does |
|---|---|
| [`security-audit`](skills/security-audit/SKILL.md) | Audits a repo, folder, PR, commit, branch diff, or uncommitted changes. Builds a threat model, runs an independent baseline auditor subagent plus focused investigation, validates every candidate against source, grades severity, and writes `report.md`, `threat-model.md`, and `findings.json`. Modes: `repo`, `diff`, `deep`. |
| [`security-fix`](skills/security-fix/SKILL.md) | Fixes one finding with the smallest repo-native patch at the shared enforcement boundary, adds a regression test, challenges its own patch for bypasses, and verifies. Outcome: `fixed`, `no_change`, or `blocked`. |
| [`skill-safety-check`](skills/skill-safety-check/SKILL.md) | Read-only pre-install review of a downloaded skill or plugin — hooks, `.mcp.json` servers, `` !`cmd` `` dynamic context, `allowed-tools`, exfiltration, obfuscation. Verdict: `BLOCK`, `REVIEW`, `INCOMPLETE`, or `PASS`. |

## Install

Personal (all projects):

```bash
git clone https://github.com/<you>/claude-security-skills.git
mkdir -p ~/.claude/skills
cp -r claude-security-skills/skills/* ~/.claude/skills/
```

Per project: copy the folders into `<project>/.claude/skills/` instead.

## Usage

In Claude Code:

- `run a security audit on this repo`
- `security-audit my uncommitted changes` / `audit PR #42` / `deep security scan of src/api`
- `fix finding F-001 from the security audit`
- `run skill-safety-check on /tmp/review/some-skill`

Audit output goes to `.claude/security-audits/<date>-<mode>-<target>/` — consider adding that path to `.gitignore`.

## What was left out from codex-security

The managed scan state/workbench, Codex Security Cloud lookups, and Linear/Jira/GitHub ticket publishing — all depend on OpenAI's MCP server and cloud services.

## License

Apache License 2.0. Methodology adapted from [openai/codex-security](https://github.com/openai/codex-security) (Apache-2.0); see [NOTICE](NOTICE).
