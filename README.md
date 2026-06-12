# Demox Skill

An [Agent Skill](https://docs.claude.com/en/docs/claude-code/skills) that teaches AI coding agents (Claude Code, Claude Agent SDK, and other skill-compatible runtimes) how to deploy and manage static websites on the [Demox](https://demox.site) platform using the `demox` CLI.

## What it does

When the user asks to deploy a static site to Demox, or to list/inspect/update/delete their Demox-hosted sites, the agent loads this skill and follows the documented workflow — including the things that are easy to get wrong: the browser-based OAuth login that the agent can't complete itself, the interactive delete confirmation, and the 8 MB upload limit.

## Layout

```
skills/
  demox/
    SKILL.md    # the skill: YAML frontmatter (name + description) + instructions
```

## Install

### Claude Code

Copy the skill into your skills directory:

```bash
# Personal (all projects)
cp -r skills/demox ~/.claude/skills/demox

# Or project-scoped
cp -r skills/demox .claude/skills/demox
```

Then invoke it with `/demox` or just let Claude trigger it when you mention deploying to Demox.

### Other runtimes

Any skill-compatible runtime can load `skills/demox/SKILL.md`. Point your skills loader at the `skills/` directory.

## The demox CLI

This skill drives [`@demox-site/cli`](https://www.npmjs.com/package/@demox-site/cli):

```bash
npm install -g @demox-site/cli
demox --help
```

## License

MIT
