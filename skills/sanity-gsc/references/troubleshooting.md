# Troubleshooting Guide

## Sync Errors

### "GOOGLE_SERVICE_ACCOUNT is invalid JSON"

**Cause:** Service account credentials not properly formatted

**Solution:**

1. Download service account JSON from Google Cloud Console
2. Use a JSON stringifier (no newlines, escaped quotes):
   ```bash
   # On Mac/Linux
   cat service-account.json | jq -c . | pbcopy

   # On Windows (PowerShell)
   (Get-Content service-account.json | ConvertTo-Json -Compress) | Set-Clipboard
   ```
3. Paste into `.env` as:
   ```
   GOOGLE_SERVICE_ACCOUNT='{"type":"service_account",...}'
   ```

---

### "No GSC properties found"

**Cause:** Service account doesn't have access to GSC properties

**Solution:**

1. Run `pnpm list-sites` to verify properties exist
2. If list is empty:
   - Go to Google Search Console
   - Add the service account email to your property
   - Wait 5 minutes for access to propagate
   - Try again

---

### "URL matching failed - 500+ URLs unmatched"

**Cause:** URL configuration in gscSite doesn't match your actual URL structure

**Solution:**

1. Run sync with `--dry-run`:
   ```bash
   pnpm sync --site sc-domain:yoursite.com --dry-run
   ```

2. Check the output for matched vs unmatched URLs

3. Review your URL configuration:
   ```bash
   # In Sanity, check the gscSite document
   # Verify pathPrefix matches your actual URL structure
   ```

4. Common issues:
   - **Typo in pathPrefix:** `/blog/` vs `/blogs/`
   - **Wrong slugField name:** Is it `slug` or `page_slug`?
   - **Slug doesn't match URL:** URL has `/blog/my-article` but slug is `my_article`

5. Fix and retry

---

### "Error: connect ECONNREFUSED (database)"

**Cause:** PostgreSQL is not running or connection string is wrong

**Solution:**

**Option 1: Start Docker**
```bash
docker compose up -d
```

**Option 2: Verify connection string**
```bash
# Check .env file for DATABASE_URL
DATABASE_URL=postgresql://user:password@localhost:5432/pagebridge

# Test connection
psql postgresql://user:password@localhost:5432/pagebridge
```

**Option 3: Ensure migrations are run**
```bash
pnpm db:push
```

---

### "Error: Sanity API: 401 Unauthorized"

**Cause:** Invalid Sanity token or wrong project/dataset

**Solution:**

1. Verify token has write permissions:
   - Go to Sanity project settings → API tokens
   - Ensure token has "Editor" role or write access
   - Regenerate if uncertain

2. Check environment variables:
   ```bash
   # In .env, verify:
   SANITY_PROJECT_ID=your_actual_id
   SANITY_DATASET=production  # or your dataset
   SANITY_TOKEN=xxxx...
   ```

3. Test with a simple query:
   ```bash
   curl -H "Authorization: Bearer $SANITY_TOKEN" \
     "https://api.sanity.io/v2021-06-07/data/query/$SANITY_PROJECT_ID?query=*[_type=='gscSite']"
   ```

---

## Missing Data Issues

### "Snapshots created but no refresh tasks"

**Cause:** No content is decaying (or decay rules are too strict)

**Solution:**

1. Check if your content should have decay:
   - Is position dropping?
   - Is CTR < 1%?
   - Are impressions down 50%+?

2. Review decay rules:
   - Default quiet period: 45 days (ignores new content)
   - Position decay threshold: 3+ position drop
   - CTR threshold: < 1% while in top 10

3. If content is new, wait 45+ days for decay detection

---

### "URLs match in dry-run but not in actual sync"

**Cause:** Timing issue or Sanity document missing

**Solution:**

1. Verify content exists in Sanity
2. Ensure slug field is filled in
3. Check if content was deleted after dry-run
4. Review Sanity publish/draft status

---

### "Unmatched diagnostics show many URLs"

**Cause:** Content doesn't exist in Sanity for those URLs

**Solution:**

1. Check if pages are in Sanity:
   ```bash
   # Query Sanity for documents
   # In Sanity console or API: *[_type=='post']{slug}
   ```

2. If URLs are for old content that no longer exists, it's expected

3. If URLs are for content that should exist:
   - Add missing content to Sanity
   - Update slug to match URL structure
   - Re-run sync

---

## Performance Issues

### "Sync takes > 5 minutes"

**Cause:** Large dataset or network latency

**Solution:**

**Quick fix:**
```bash
# Skip quick-win analysis to speed up
pnpm sync --site sc-domain:yoursite.com --skip-insights
```

**Optimization:**
```bash
# Run with debug to see timing breakdown
pnpm sync --site sc-domain:yoursite.com --debug
```

Expected times:
- First sync: 2-5 min (full data load)
- Subsequent: 30-60 sec (incremental)

---

### "Database growing too large"

**Cause:** Accumulating historical data

**Solution:**

1. Query analytics are stored per date
2. Monthly storage ≈ (pages × queries) KB
3. To manage size, periodically archive old data:
   ```sql
   -- Keep only last 90 days
   DELETE FROM search_analytics
   WHERE date < NOW() - INTERVAL '90 days'
   ```

---

## Sanity Integration Issues

### "Can't see gscSnapshot in content reference"

**Cause:** Plugin not installed or schema not synced

**Solution:**

1. Install plugin in `sanity.config.ts`:
   ```typescript
   import { pageridgePlugin } from '@pagebridge/sanity-plugin'

   plugins: [pageridgePlugin()]
   ```

2. Restart Sanity:
   ```bash
   pnpm dev
   ```

3. Add reference field to your schema:
   ```typescript
   {
     name: 'gscSnapshot',
     type: 'reference',
     to: [{ type: 'gscSnapshot' }]
   }
   ```

---

### "Refresh task queue is empty"

**Cause:** No decay detected or sync hasn't run

**Solution:**

1. Verify sync completed:
   ```bash
   pnpm sync --site sc-domain:yoursite.com
   ```

2. Check Sanity has the refresh tasks:
   ```bash
   # In Sanity API, query for tasks
   # *[_type=='gscRefreshTask']{title,decaySignals}
   ```

3. If sync ran but no tasks:
   - Content may be new (in quiet period)
   - Content isn't decaying by default rules
   - Content URLs don't match Sanity slugs

---

## Getting Help

### Debugging commands

```bash
# See what would happen without writing
pnpm sync --site sc-domain:yoursite.com --dry-run

# Get detailed timing info
pnpm sync --site sc-domain:yoursite.com --debug

# Check available sites
pnpm list-sites

# Test database
psql $DATABASE_URL -c "SELECT COUNT(*) FROM search_analytics"

# Check Sanity connection
curl -H "Authorization: Bearer $SANITY_TOKEN" \
  "https://api.sanity.io/v2021-06-07/data/query/$SANITY_PROJECT_ID?query=*[_type=='gscSite']"
```

### Collecting diagnostic info

When seeking help, provide:

1. Output from `pnpm sync --site ... --debug`
2. Your URL configuration from gscSite
3. Count of documents in Sanity: `*[_type=='post']{slug}`
4. Count of unmatched URLs: `SELECT COUNT(*) FROM unmatch_diagnostics`
5. Any error messages from logs

### Common resources

- [PageBridge GitHub](https://github.com/PageBridge-IO/pagebridge)
- [Google Search Console Help](https://support.google.com/webmasters)
- [Sanity Documentation](https://www.sanity.io/docs)
- [Drizzle ORM Docs](https://orm.drizzle.team/)
