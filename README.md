# PageBridge Skills for Claude Code

A collection of AI agent skills for content performance monitoring and optimization. Built for developers who want Claude Code (or similar AI coding assistants) to help with integrating Google Search Console data, tracking content decay, and managing content refresh tasks with Sanity CMS.

Built by [PageBridge](https://pagebridge.io). PageBridge syncs Google Search Console data to Sanity CMS, automatically detects content decay (ranking loss), and recommends content refresh tasks.

**Contributions welcome!** Found a way to improve a skill or have a new one to add? [Open a PR](#contributing).

## What are Skills?

Skills are markdown files that give AI agents specialized knowledge and workflows for specific tasks. When you add these to your project, Claude Code can recognize when you're working on content performance monitoring and integration tasks, and apply the right patterns and best practices.

## Available Skills

<!-- SKILLS:START -->
| Skill | Description |
|-------|-------------|
| [sanity-gsc](skills/sanity-gsc/) | Integrate PageBridge to sync Google Search Console data to Sanity CMS, detect content decay, and manage refresh tasks.... |
<!-- SKILLS:END -->

## Installation

### Option 1: CLI Install (Recommended)

Use [npx skills](https://github.com/vercel-labs/skills) to install skills directly:

```bash
# Install all skills
npx skills add PageBridge-IO/pagebridge-skills

# Install specific skills
npx skills add PageBridge-IO/pagebridge-skills --skill sanity-gsc

# List available skills
npx skills add PageBridge-IO/pagebridge-skills --list
```

You may also use `pnpm`.

```bash
pnpm dlx skills add PageBridge-IO/pagebridge-skills
```

This automatically installs to your `.claude/skills/` directory.

### Option 2: Claude Code Plugin

Install via Claude Code's built-in plugin system:

```bash
# Add the marketplace
/plugin marketplace add PageBridge-IO/pagebridge-skills

# Install all PageBridge skills
/plugin install pagebridge
```

### Option 3: Clone and Copy

Clone the entire repo and copy the skills folder:

```bash
git clone https://github.com/PageBridge-IO/pagebridge-skills.git
cp -r pagebridge-skills/skills/* .claude/skills/
```

### Option 4: Git Submodule

Add as a submodule for easy updates:

```bash
git submodule add https://github.com/PageBridge-IO/pagebridge-skills.git .claude/pagebridge-skills
```

Then reference skills from `.claude/pagebridge-skills/skills/`.

### Option 5: Fork and Customize

1. Fork this repository
2. Customize skills for your specific needs
3. Clone your fork into your projects

### Option 6: SkillKit (Multi-Agent)

Use [SkillKit](https://github.com/rohitg00/skillkit) to install skills across multiple AI agents (Claude Code, Cursor, Copilot, etc.):

```bash
# Install all skills
npx skillkit install PageBridge-IO/pagebridge-skills

# Install specific skills
npx skillkit install PageBridge-IO/pagebridge-skills --skill sanity-gsc

# List available skills
npx skillkit install PageBridge-IO/pagebridge-skills --list
```

## Usage

Once installed, just ask Claude Code to help with PageBridge tasks:

```
"Set up PageBridge with Sanity CMS"
→ Uses sanity-gsc skill

"Configure GSC URL matching for my blog"
→ Uses sanity-gsc skill

"Monitor blog posts for ranking decay"
→ Uses sanity-gsc skill

"Create refresh task recommendations"
→ Uses sanity-gsc skill
```

You can also invoke skills directly:

```
/sanity-gsc
```

## What the Skills Cover

### sanity-gsc

- **Setup** — Environment configuration, database setup, Google Service Account authentication
- **URL Configuration** — Matching GSC URLs to Sanity documents, multi-locale patterns, custom paths
- **Decay Detection** — Position drops, low CTR, impressions loss, severity calculation
- **Sync Operations** — Running full syncs, dry-runs, scheduling with cron, performance optimization
- **Sanity Integration** — Schema setup, gscSite configuration, gscSnapshot metrics, gscRefreshTask recommendations
- **Content Monitoring** — Tracking blog posts, landing pages, use cases with GSC metrics
- **Troubleshooting** — Database issues, URL matching problems, token permissions, sync failures
- **CLI Reference** — All commands, options, automation, and monitoring workflows

## Planned Skills

- `pagebridge-analytics` — Content performance analytics and reporting
- `pagebridge-slack` — Slack notifications for decay detection and refresh tasks
- `pagebridge-webhooks` — Custom webhook integrations for external systems

## Contributing

Found a way to improve a skill? Have a new skill to suggest? PRs and issues welcome!

### Adding a New Skill

1. Create a directory under `skills/` with your skill name (lowercase, hyphens only)
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`)
3. Keep the main file under 500 lines — use `references/` for detailed docs
4. Run `node .github/scripts/sync-skills.js` to update marketplace.json and README

See [AGENTS.md](AGENTS.md) for full guidelines on skill structure and writing style.

## License

[MIT](LICENSE) — Use these however you want.
