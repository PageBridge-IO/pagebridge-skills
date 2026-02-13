# CLI Reference Guide

## Overview

PageBridge CLI provides two main commands: `sync` and `list-sites`.

## sync Command

Syncs Google Search Console data to Sanity CMS, detects decay, and creates refresh tasks.

### Basic Usage

```bash
pnpm sync --site sc-domain:yoursite.com
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `--site` | GSC property to sync (required) | `sc-domain:example.com` or `sc-https://example.com` |
| `--dry-run` | Show what would be synced without writing | `--dry-run` |
| `--debug` | Show timing and debug information | `--debug` |
| `--skip-insights` | Skip quick-win analysis (faster) | `--skip-insights` |

### Common Patterns

**First-time sync (see what will happen):**
```bash
pnpm sync --site sc-domain:yoursite.com --dry-run
```

**Full sync with decay detection:**
```bash
pnpm sync --site sc-domain:yoursite.com
```

**Quick sync (skip analysis):**
```bash
pnpm sync --site sc-domain:yoursite.com --skip-insights
```

**Debug sync issues:**
```bash
pnpm sync --site sc-domain:yoursite.com --debug
```

### Output Indicators

After running, check the results:

- ✅ **Success** - Data synced, decay detected, refresh tasks created
- ⚠️ **Warnings** - Some URLs didn't match (check logs)
- ❌ **Error** - Sync failed (check database and Sanity connection)

### What Sync Does

1. **Fetches GSC data** - 28 days of search analytics
2. **Matches URLs** - Links GSC URLs to Sanity documents
3. **Stores metrics** - Saves to PostgreSQL
4. **Detects decay** - Identifies decaying content
5. **Creates tasks** - Adds refresh tasks to Sanity
6. **Finds quick wins** - Identifies ranking opportunities

### Typical Sync Time

- **First sync:** 2-5 minutes (initial data load)
- **Subsequent syncs:** 30-60 seconds (incremental updates)

## list-sites Command

Shows all Google Search Console properties your service account can access.

### Usage

```bash
pnpm list-sites
```

### Example Output

```
Available GSC Properties:
  sc-domain:example.com
  sc-https://example.com/en/
  sc-https://example.com/fr/
```

Use any of these values with `--site` flag in the sync command.

## Running Sync on a Schedule

### Using cron (Linux/Mac)

```bash
# Sync daily at 2 AM
0 2 * * * cd /path/to/pagebridge && pnpm sync --site sc-domain:yoursite.com
```

### Using Task Scheduler (Windows)

1. Create a batch file `sync.bat`:
```batch
cd F:\Code\pagebridge\oss
pnpm sync --site sc-domain:yoursite.com
```

2. Schedule it in Task Scheduler to run daily

### Using GitHub Actions

Create `.github/workflows/pagebridge-sync.yml`:

```yaml
name: PageBridge Sync
on:
  schedule:
    - cron: '0 2 * * *'

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm sync --site sc-domain:${{ secrets.GSC_PROPERTY }}
        env:
          GOOGLE_SERVICE_ACCOUNT: ${{ secrets.GOOGLE_SERVICE_ACCOUNT }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          SANITY_PROJECT_ID: ${{ secrets.SANITY_PROJECT_ID }}
          SANITY_DATASET: ${{ secrets.SANITY_DATASET }}
          SANITY_TOKEN: ${{ secrets.SANITY_TOKEN }}
```

## Environment Variables

PageBridge CLI requires these environment variables (in `.env` or shell):

- `GOOGLE_SERVICE_ACCOUNT` - Stringified JSON service account key
- `DATABASE_URL` - PostgreSQL connection string
- `SANITY_PROJECT_ID` - Your Sanity project ID
- `SANITY_DATASET` - Dataset name (usually "production")
- `SANITY_TOKEN` - API token with write permissions
- `SITE_URL` - Base URL of your website

## Troubleshooting Sync

**"Service account invalid"**
```bash
# Verify service account JSON is properly stringified (no newlines)
# Ensure it has Google Search Console API access
```

**"No GSC properties found"**
```bash
# Run list-sites to verify properties exist
pnpm list-sites

# If empty, ensure service account is added to GSC property
```

**"URL matching failed"**
```bash
# Check URL configuration in Sanity gscSite document
# Run with --dry-run to see which URLs matched
pnpm sync --site sc-domain:yoursite.com --dry-run
```

**"Sanity write failed"**
```bash
# Verify SANITY_TOKEN has write permissions
# Check SANITY_DATASET matches your config
# Ensure gscSite and gscSnapshot schemas exist
```

**"Database connection failed"**
```bash
# Verify DATABASE_URL is correct
# Ensure PostgreSQL is running
# Check database migrations: pnpm db:push
```

## Multiple Sites

To sync multiple properties:

```bash
# Sync site A
pnpm sync --site sc-domain:siteA.com

# Sync site B
pnpm sync --site sc-domain:siteB.com
```

Or schedule separate cron jobs for each site.
