# Sanity Integration Guide

## Overview

PageBridge integrates with Sanity CMS through:
- **gscSite** schema - Configuration document linking your GSC property
- **gscSnapshot** schema - Performance metrics snapshot linked to your content
- **gscRefreshTask** schema - Content refresh recommendations with decay signals

## Setting Up Sanity

### 1. Install the PageBridge Plugin

In your Sanity studio `sanity.config.ts`:

```typescript
import { defineConfig } from 'sanity'
import { pageridgePlugin } from '@pagebridge/sanity-plugin'

export default defineConfig({
  // ... other config
  plugins: [
    pageridgePlugin(),
  ],
})
```

### 2. Create a gscSite Document

In your Sanity studio, create a new document of type `gscSite`:

**Document Settings:**
- Title: "My Site"
- GSC Property ID: `sc-domain:yoursite.com` (from `pnpm list-sites`)
- URL Configuration: Set up your content types

**URL Configuration Example (for blog + landing pages):**

```json
[
  {
    "contentType": "post",
    "slugField": "slug",
    "pathPrefix": "/blog/",
    "description": "Blog posts"
  },
  {
    "contentType": "useCase",
    "slugField": "slug",
    "pathPrefix": "/use-cases/",
    "description": "Use case landing pages"
  }
]
```

Save the document. PageBridge will reference this when syncing.

### 3. Link Content to gscSnapshot

Your blog posts and landing pages should have a reference field to `gscSnapshot`:

In your `post` schema:

```typescript
{
  name: 'gscSnapshot',
  title: 'Performance Metrics',
  type: 'reference',
  to: [{ type: 'gscSnapshot' }],
  description: 'Latest GSC performance data'
}
```

In your `useCase` schema:

```typescript
{
  name: 'gscSnapshot',
  title: 'Performance Metrics',
  type: 'reference',
  to: [{ type: 'gscSnapshot' }],
  description: 'Latest GSC performance data'
}
```

### 4. View gscRefreshTask Documents

After running `pnpm sync`, navigate to the `gscRefreshTask` tab in Sanity. You'll see:

- **Title** - Content that needs refreshing
- **Decay Signals** - Why it needs refresh (position drop, low CTR, etc.)
- **Current Metrics** - Latest GSC data
- **Status** - Mark as "pending", "in-progress", or "completed"

## gscSnapshot Structure

When PageBridge syncs, it creates `gscSnapshot` documents with this structure:

```typescript
{
  _type: 'gscSnapshot',
  _id: 'gsc-snapshot-123',

  // Link to content
  linkedDocument: {
    _ref: 'post-abc123',
    _type: 'reference'
  },

  // Metrics
  clicks: 45,
  impressions: 1200,
  ctr: 0.0375,
  position: 6.2,

  // Time window (28 days)
  startDate: '2024-01-15',
  endDate: '2024-02-12',

  // Top queries driving traffic
  topQueries: [
    {
      _key: 'key123',
      query: 'how to use pagebridge',
      clicks: 25,
      impressions: 600,
      ctr: 0.0417,
      position: 4.5
    }
  ],

  // Quick win opportunities (position 8-20)
  quickWinQueries: [
    {
      _key: 'key456',
      query: 'content decay detection',
      clicks: 2,
      impressions: 120,
      ctr: 0.0167,
      position: 15
    }
  ]
}
```

### Using Snapshots in Your Content

Display performance metrics in your blog post/landing page:

```typescript
// In a React component
export function PerformanceCard({ document }) {
  const snapshot = useDocumentStore(store =>
    store.snapshot(document.gscSnapshot._ref)
  )

  return (
    <div>
      <p>Position: {snapshot.position}</p>
      <p>CTR: {(snapshot.ctr * 100).toFixed(1)}%</p>
      <p>Monthly Clicks: {snapshot.clicks}</p>
    </div>
  )
}
```

## gscRefreshTask Structure

After decay detection, PageBridge creates refresh tasks:

```typescript
{
  _type: 'gscRefreshTask',
  _id: 'task-refresh-123',

  // Content reference
  linkedDocument: {
    _ref: 'post-abc123'
  },

  // Why refresh is needed
  decaySignals: {
    position_decay: {
      severity: 'high',
      description: 'Position dropped from 5 to 8 in 28 days',
      currentPosition: 8,
      previousPosition: 5
    },
    low_ctr: {
      severity: 'medium',
      description: 'CTR is 0.8%, target is > 1%',
      ctr: 0.008
    }
  },

  // Current status
  status: 'pending', // pending | in-progress | completed

  // When created
  createdAt: '2024-02-12T10:30:00Z',

  // Notes
  notes: 'Update content to regain ranking and improve click appeal'
}
```

### Working with Refresh Tasks

1. **Review** - Check the Sanity refresh task queue in the plugin UI
2. **Prioritize** - High severity tasks first
3. **Update** - Edit your content (blog post or landing page)
4. **Mark Complete** - Set status to "completed"

## Linking to Content

PageBridge automatically links `gscSnapshot` and `gscRefreshTask` to your content based on URL matching.

**This requires:**

1. **gscSite document** with correct URL configuration
2. **Content slug field** matching the URL structure
3. **Content reference field** to gscSnapshot (in your content schema)

**Example matching:**

GSC URL: `https://yoursite.com/blog/my-article`
→ Matches path prefix: `/blog/`
→ Slug: `my-article`
→ Links to: `post` document with `slug: "my-article"`

## Troubleshooting Sanity Integration

**"No snapshots created"**
- Verify gscSite document exists with correct GSC property ID
- Check URL configuration matches your content paths
- Run sync with `--dry-run` to see matching results

**"Snapshots not linked to content"**
- Check content slug matches the URL path exactly
- Verify slugField name in URL config matches your schema
- Look in `unmatch_diagnostics` table for which URLs failed

**"Refresh tasks not showing"**
- Ensure gscRefreshTask schema is installed with the plugin
- Check your content has decay signals
- Verify sync completed successfully

**"Can't reference gscSnapshot in content"**
- Ensure plugin is installed and enabled
- Add reference field to your content schema
- Reload Sanity studio after plugin installation
