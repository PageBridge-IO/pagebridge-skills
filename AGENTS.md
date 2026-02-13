# AGENTS.md

Guidelines for AI agents working in this repository.

## Repository Overview

This repository contains **Agent Skills** for AI agents following the [Agent Skills specification](https://agentskills.io/specification.md). It also serves as a **Claude Code plugin marketplace** via `.claude-plugin/marketplace.json`.

- **Name**: PageBridge Skills
- **GitHub**: [PageBridge-IO/pagebridge-skills](https://github.com/PageBridge-IO/pagebridge-skills)
- **Creator**: PageBridge
- **License**: MIT

## Repository Structure

```
pagebridge-skills/
├── .claude-plugin/
│   └── marketplace.json   # Claude Code plugin marketplace manifest
├── skills/
│   └── sanity-gsc/
│       ├── SKILL.md       # Main skill file
│       ├── assets/        # Configuration examples
│       └── references/    # Detailed documentation
│           ├── setup-guide.md
│           ├── url-configuration.md
│           ├── decay-detection.md
│           ├── cli-reference.md
│           ├── sanity-integration.md
│           └── troubleshooting.md
├── AGENTS.md
├── CLAUDE.md
├── VERSIONS.md
└── LICENSE
```

## Build / Lint / Test Commands

**Not applicable** - This is a content-only repository with no executable code.

Verify manually:

- YAML frontmatter is valid
- `name` field matches directory name exactly
- `name` is 1-64 chars, lowercase alphanumeric and hyphens only
- `description` is 1-1024 characters

## Agent Skills Specification

Skills follow the [Agent Skills spec](https://agentskills.io/specification.md).

### Required Frontmatter

```yaml
---
name: skill-name
description: What this skill does and when to use it. Include trigger phrases.
---
```

### Frontmatter Field Constraints

| Field         | Required | Constraints                                                    |
| ------------- | -------- | -------------------------------------------------------------- |
| `name`        | Yes      | 1-64 chars, lowercase `a-z`, numbers, hyphens. Must match dir. |
| `description` | Yes      | 1-1024 chars. Describe what it does and when to use it.        |
| `license`     | No       | License name (default: MIT)                                    |
| `metadata`    | No       | Key-value pairs (author, version, etc.)                        |

### Name Field Rules

- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- No consecutive hyphens (`--`)
- Must match parent directory name exactly

**Valid**: `sanity-gsc`, `pagebridge-slack`, `webhook-sync`
**Invalid**: `PageBridge-GSC`, `-sanity`, `sanity--gsc`

### Skill Directory Structure

```
skills/skill-name/
├── SKILL.md        # Required - main instructions (<500 lines)
├── references/     # Optional - detailed docs loaded on demand
├── scripts/        # Optional - executable code
└── assets/         # Optional - templates, data files
```

## Writing Style Guidelines

### Structure

- Keep `SKILL.md` under 500 lines (move details to `references/`)
- Use H2 (`##`) for main sections, H3 (`###`) for subsections
- Use bullet points and numbered lists liberally
- Short paragraphs (2-4 sentences max)

### Tone

- Direct and instructional
- Second person ("You are helping integrate PageBridge with Sanity...")
- Professional but approachable

### Formatting

- Bold (`**text**`) for key terms
- Code blocks for examples and templates
- Tables for reference data
- No excessive emojis

### Clarity Principles

- Clarity over cleverness
- Specific over vague
- Active voice over passive
- One idea per section

### Description Field Best Practices

The `description` is critical for skill discovery. Include:

1. What the skill does
2. When to use it (trigger phrases)
3. Related skills for scope boundaries

## Claude Code Plugin

This repo serves as a plugin marketplace. The manifest at `.claude-plugin/marketplace.json` lists all skills for installation.

See [Claude Code plugins documentation](https://code.claude.com/docs/en/plugins.md) for details.

## Git Workflow

### Branch Naming

- New skills: `feature/skill-name`
- Improvements: `fix/skill-name-description`
- Documentation: `docs/description`

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat: add pagebridge-slack skill`
- `fix: improve setup guide in sanity-gsc skill`
- `docs: update README`

### Pull Request Checklist

- [ ] `name` matches directory name exactly
- [ ] `name` follows naming rules (lowercase, hyphens, no `--`)
- [ ] `description` is 1-1024 chars with trigger phrases
- [ ] `SKILL.md` is under 500 lines
- [ ] No sensitive data or credentials

## Checking for Updates

When using any skill from this repository:

1. **Once per session**, on first skill use, check for updates:
   - Fetch `VERSIONS.md` from GitHub: https://raw.githubusercontent.com/PageBridge-IO/pagebridge-skills/main/VERSIONS.md
   - Compare versions against local skill files

2. **Only prompt if meaningful**:
   - 2 or more skills have updates, OR
   - Any skill has a major version bump (e.g., 1.x to 2.x)

3. **Non-blocking notification** at end of response:

   ```
   ---
   Skills update available: X PageBridge skills have updates.
   Say "update skills" to update automatically, or run `git pull` in your pagebridge folder.
   ```

4. **If user says "update skills"**:
   - Run `git pull` in the pagebridge directory
   - Confirm what was updated

## Skill Summary

| Skill          | Purpose                                                                           |
| -------------- | ------------------------------------------------------------------------------- |
| `sanity-gsc`   | Integrate PageBridge with Sanity CMS for GSC data sync and content decay detection |

## Key Concepts (sanity-gsc)

The skill teaches agents how to:

1. Set up Google Service Account authentication and database configuration
2. Configure URL patterns to match GSC URLs to Sanity document slugs
3. Understand and interpret decay detection signals (position drops, low CTR, impressions loss)
4. Run sync operations with options for dry-run, filtering, and performance tuning
5. Create and configure gscSite, gscSnapshot, and gscRefreshTask schema types
6. Monitor content performance and act on decay recommendations systematically
7. Troubleshoot common issues with URL matching, database, and permissions
