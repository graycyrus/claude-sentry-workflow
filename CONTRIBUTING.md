# Contributing to claude-sentry-workflow

Thanks for your interest in contributing! This project provides AI-assisted Sentry bug triage workflows for Claude Code.

## How to contribute

### Improving existing skills

The skills live in `skills/<name>/SKILL.md`. Each is a self-contained Claude Code skill file with YAML frontmatter and markdown instructions.

When improving a skill:
- Keep it generic — no hardcoded org names, repo names, or project slugs
- The workflow should work with any Sentry organization and any GitHub repo
- Maintain the "ask before assuming" philosophy
- Test with different Sentry project types if possible

### Adding a new skill

1. Create `skills/<skill-name>/SKILL.md`
2. Add YAML frontmatter with `description`, `allowed-tools`, and optionally `argument-hint`
3. Add corresponding documentation in `docs/`
4. Update the skill table in `README.md`
5. Add the skill name to `install.sh` `SKILLS_LIST`

### Updating docs

The `docs/` folder contains detailed reference documentation. Keep these in sync with the skills.

## Guidelines

- **Keep it generic.** Skills should work with any Sentry org and any GitHub repo.
- **Auto-detect, don't configure.** Use Sentry MCP tools and `gh` to detect context at runtime.
- **Ask, don't assume.** Always get user confirmation before creating issues.
- **Scrub sensitive data.** Never include PII, tokens, or internal URLs in issue bodies.

## Reporting issues

If a skill doesn't work as expected, open an issue with:
- What Sentry setup you're using
- What happened vs. what you expected
- The skill command you ran

## Code of conduct

Be kind, be helpful, be constructive.
