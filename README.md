# dotclaude

Claude Code boilerplate with a two-layer AI workflow: Claude Pro for reasoning, OpenCode Go for execution.

## Setup

```bash
git clone https://github.com/motcondicontrinh1/dotclaude.git /tmp/dotclaude
cd your-project
mkdir -p .claude
cp /tmp/dotclaude/settings.json .claude/
cp -r /tmp/dotclaude/{rules,skills,agents,hooks} .claude/
cp /tmp/dotclaude/CLAUDE.md ./
cp /tmp/dotclaude/DESIGN.md ./
chmod +x .claude/hooks/*.sh
echo "CLAUDE.local.md" >> .gitignore
rm -rf /tmp/dotclaude
```

Restart Claude Code, then run `/setupdotclaude` to auto-customize for your stack.

## What's Inside

| Path | Purpose |
|---|---|
| `CLAUDE.md` | Two-layer workflow rules, model assignment, spec format, deploy rules |
| `DESIGN.md` | UI/design source of truth — all agents read this before visual changes |
| `rules/` | Code quality, testing, security, frontend, database, error handling |
| `skills/` | `/debug-fix`, `/ship`, `/hotfix`, `/pr-review`, `/tdd`, `/explain`, `/refactor` |
| `agents/` | `@code-reviewer`, `@security-reviewer`, `@performance-reviewer`, `@frontend-designer`, `@doc-reviewer` |
| `hooks/` | Format-on-save, secret scanning, dangerous command blocking, auto-test |

## License

MIT
