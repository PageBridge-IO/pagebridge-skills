---
name: sanity-gsc
license: MIT
description: Integrate PageBridge to sync Google Search Console data to Sanity CMS, detect content decay, and manage refresh tasks. Use when setting up GSC + Sanity integration, configuring URL structures, understanding decay signals, running syncs, or monitoring content performance.
---

# Sanity - Google Search Console Skill

## Quick Start

PageBridge syncs your Google Search Console data to Sanity CMS, automatically detects content decay (ranking loss), and recommends refresh tasks.

**Two main use cases:**

1. **Monitor blog posts** - Catch posts losing rankings before traffic drops
2. **Track landing pages** - Ensure use-cases, case studies stay competitive

## Setup in 5 Minutes

### 1. Install Dependencies

```bash
pnpm install
```

### 2. Configure Environment

Create `.env` in the root directory:

```
GOOGLE_SERVICE_ACCOUNT='{"type":"service_account",...}'
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/pagebridge
SANITY_PROJECT_ID=your_project_id
SANITY_DATASET=production
SANITY_TOKEN=your_token
SITE_URL=https://yoursite.com
```

See [setup-guide.md](references/setup-guide.md) for detailed instructions.

### 3. Start Database

```bash
docker compose up -d
pnpm db:push
```

### 4. Create gscSite in Sanity

In Sanity Studio, create a document of type `gscSite` with:

- **GSC Property:** `sc-domain:yoursite.com` (from `pnpm list-sites`)
- **URL Configuration:** Configure your content types

Example for blog + landing pages:

```json
{
  "urlConfigs": [
    {
      "contentType": "post",
      "slugField": "slug",
      "pathPrefix": "/blog/"
    },
    {
      "contentType": "useCase",
      "slugField": "slug",
      "pathPrefix": "/use-cases/"
    }
  ]
}
```

See [url-configuration.md](references/url-configuration.md) for details.

### 5. Run Sync

```bash
# See what will happen
pnpm sync --site sc-domain:yoursite.com --dry-run

# Actually sync
pnpm sync --site sc-domain:yoursite.com
```

Check Sanity for `gscSnapshot` (metrics) and `gscRefreshTask` (decay recommendations).

## Understanding Decay Detection

PageBridge flags content that needs attention based on three signals:

### Position Decay

Your page was ranking well but slipped down results.

- **Trigger:** Position drops 3+ spots in 28 days
- **Action:** Update content to regain ranking

### Low CTR

Page is in top 10 but users rarely click it.

- **Trigger:** CTR < 1% while ranking positions 1-10
- **Action:** Rewrite title and meta description

### Impressions Drop

Your page's visibility in search results decreased significantly.

- **Trigger:** 50%+ drop in impressions over 28 days
- **Action:** Comprehensive content update

**Example:** Blog post at position 5 → position 8 = 🚩 Decay detected → Add to refresh queue

See [decay-detection.md](references/decay-detection.md) for interpreting metrics and acting on signals.

## Common Workflows

### Monitor Blog Posts for Decay

```bash
# Initial setup - run once
pnpm sync --site sc-domain:yoursite.com

# Weekly monitoring
pnpm sync --site sc-domain:yoursite.com

# Schedule with cron (daily at 2 AM)
0 2 * * * cd /path/to/pagebridge && pnpm sync --site sc-domain:yoursite.com
```

Then in Sanity, review the `gscRefreshTask` queue weekly to catch decay early.

### Track Landing Page Performance

Same as above, but configure landing page URL patterns:

```json
{
  "contentType": "useCase",
  "slugField": "slug",
  "pathPrefix": "/use-cases/"
}
```

PageBridge will track impressions, position, and CTR for each use case, flagging any ranking loss.

### Act on Decay Signals

In Sanity `gscRefreshTask`:

1. **High severity decay** → Act within 1-2 weeks
   - Position drop: Update content, add examples, improve readability
   - Low CTR: Rewrite title/meta, add compelling benefits
   - Impressions drop: Comprehensive update with new data/trends

2. **Medium severity** → Act within 1 month
3. **Low severity** → Include in regular content updates

## CLI Reference

**List available GSC properties:**

```bash
pnpm list-sites
```

**Sync with options:**

```bash
pnpm sync --site sc-domain:yoursite.com              # Full sync
pnpm sync --site sc-domain:yoursite.com --dry-run    # Preview only
pnpm sync --site sc-domain:yoursite.com --skip-insights  # Faster
pnpm sync --site sc-domain:yoursite.com --debug      # Debug timing
```

See [cli-reference.md](references/cli-reference.md) for all commands and scheduling options.

## URL Configuration

PageBridge matches GSC URLs to Sanity documents using configurable patterns.

**Key points:**

- Each content type needs a **pathPrefix** (e.g., `/blog/` for blog posts)
- **slugField** must match the field name in your Sanity schema
- Slugs must match the URL path exactly

**Example:** GSC URL `/blog/my-article` → pathPrefix `/blog/` → slug `my-article` → links to post document

See [url-configuration.md](references/url-configuration.md) for multi-locale, custom URL patterns, and validation.

## Sanity Integration

PageBridge uses three schema types:

1. **gscSite** - Configuration document (you create)
2. **gscSnapshot** - Performance metrics (auto-created by PageBridge)
3. **gscRefreshTask** - Refresh recommendations (auto-created when decay detected)

In your content schema, add:

```typescript
{
  name: 'gscSnapshot',
  type: 'reference',
  to: [{ type: 'gscSnapshot' }],
  description: 'Latest GSC metrics'
}
```

See [sanity-integration.md](references/sanity-integration.md) for full schema setup and linking content.

## Troubleshooting

**URL matching failed** → Check [url-configuration.md](references/url-configuration.md) for validation

**"Service account invalid"** → See [setup-guide.md](references/setup-guide.md) for stringifying JSON

**Database connection error** → See [troubleshooting.md](references/troubleshooting.md)

**Sanity write failed** → Check token permissions in [troubleshooting.md](references/troubleshooting.md)

**No refresh tasks created** → Review [decay-detection.md](references/decay-detection.md) for rules and quiet period

See [troubleshooting.md](references/troubleshooting.md) for comprehensive debugging steps.

## Next Steps

1. ✅ Follow the 5-minute setup above
2. ✅ Run first sync: `pnpm sync --site sc-domain:yoursite.com`
3. ✅ Review gscSnapshot and gscRefreshTask in Sanity
4. ✅ Start monitoring decay signals
5. ✅ Schedule daily sync (see [cli-reference.md](references/cli-reference.md))

## Resources

- [Setup Guide](references/setup-guide.md) - Detailed environment and database setup
- [URL Configuration](references/url-configuration.md) - Matching GSC URLs to content
- [Decay Detection](references/decay-detection.md) - Understanding metrics and signals
- [CLI Reference](references/cli-reference.md) - All commands and scheduling
- [Sanity Integration](references/sanity-integration.md) - Schema setup and linking
- [Troubleshooting](references/troubleshooting.md) - Error solutions and debugging

---

_PageBridge syncs Google Search Console data to Sanity CMS for content performance tracking and decay detection. Monitor blog posts and landing pages to catch ranking loss early and maintain competitive visibility._
