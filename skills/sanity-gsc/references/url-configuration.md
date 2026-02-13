# URL Configuration Guide

## Overview

PageBridge matches Google Search Console URLs to Sanity documents using configurable URL patterns. This is essential for both tracking landing pages and monitoring blog posts.

## Configuration Format

Configure URL matching in your Sanity `gscSite` document using `urlConfigs`:

```typescript
{
  "urlConfigs": [
    {
      "contentType": "blogPost",
      "slugField": "slug",
      "pathPrefix": "/blog/",
      "description": "Blog posts at /blog/{slug}"
    },
    {
      "contentType": "landingPage",
      "slugField": "slug",
      "pathPrefix": "/use-cases/",
      "description": "Landing pages at /use-cases/{slug}"
    }
  ]
}
```

## Common Patterns

### Blog Posts

For blog posts typically at `/blog/my-article`:

```json
{
  "contentType": "post",
  "slugField": "slug",
  "pathPrefix": "/blog/"
}
```

Make sure your Sanity document has a `slug` field matching the URL path.

### Landing Pages (Use Cases, Case Studies)

For landing pages at `/use-cases/my-use-case`:

```json
{
  "contentType": "useCase",
  "slugField": "slug",
  "pathPrefix": "/use-cases/"
}
```

### Documentation Pages

For docs at `/docs/getting-started`:

```json
{
  "contentType": "docPage",
  "slugField": "slug",
  "pathPrefix": "/docs/"
}
```

### Multi-locale Sites

For localized content (e.g., `/en/blog/article` and `/fr/blog/article`):

```json
[
  {
    "contentType": "blogPost",
    "slugField": "slug",
    "pathPrefix": "/en/blog/"
  },
  {
    "contentType": "blogPostFr",
    "slugField": "slug",
    "pathPrefix": "/fr/blog/"
  }
]
```

### Sites with Custom URL Structures

For complex paths like `/resources/{category}/{slug}`:

```json
{
  "contentType": "resource",
  "slugField": "slug",
  "pathPrefix": "/resources/guides/"
}
```

If your actual URL is `/resources/guides/my-topic`, ensure the slug matches the full remaining path.

## Legacy Format (Deprecated)

Older PageBridge versions used:

```json
{
  "contentTypes": ["blogPost", "page"],
  "slugField": "slug",
  "pathPrefix": "/blog/"
}
```

This still works but is deprecated. **Migrate to the new `urlConfigs` format** for better control over different content types.

## Validation

### Check URL Matching

After configuring, run a dry-run to see which URLs match:

```bash
pnpm sync --site sc-domain:yoursite.com --dry-run
```

Look for:
- ✅ URLs that matched successfully
- ⚠️ URLs that couldn't match (check `unmatch_diagnostics` table)

### Debugging URL Mismatches

If URLs aren't matching:

1. **Verify slug format** - URL slugs must match Sanity document slugs exactly
2. **Check path prefix** - Ensure pathPrefix doesn't have extra slashes
3. **Confirm contentType exists** - The contentType must exist in your Sanity schema
4. **Check slugField name** - Verify the field name in Sanity (usually "slug")

Example: If GSC shows `/blog/my-article` but your Sanity slug is `my-article-2024`, they won't match. Update either the slug or the URL structure.

## When to Update Configuration

- **Adding a new content type** - Add a new entry to `urlConfigs`
- **Changing URL structure** - Update the `pathPrefix` and test with `--dry-run`
- **Migrating content** - Move old URL patterns to new paths in `urlConfigs`
