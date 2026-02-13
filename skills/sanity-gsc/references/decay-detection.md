# Decay Detection Guide

## What is Decay Detection?

Decay detection identifies content that is losing ranking and traffic. PageBridge analyzes GSC metrics to flag pages that need refreshing or updating.

## Default Decay Rules

PageBridge monitors three decay signals by default:

### 1. Position Decay

**Trigger:** Average position drops by 3+ positions over 28 days

**What it means:** Your page was ranking well but has slipped down the results. For example, moving from position 5 to position 8+.

**Action:** Update content to regain ranking (refresh information, improve readability, target new related keywords)

**Example:**
- 28 days ago: Position 4
- Today: Position 7+
- **Status:** 🚩 Decaying

### 2. Low CTR (Click-Through Rate)

**Trigger:** CTR < 1% while ranking in top 10

**What it means:** Your page appears in top 10 results but users rarely click it. The title or meta description likely needs improvement.

**Action:** Rewrite title tag and meta description to be more compelling and click-worthy

**Example:**
- Position: 6
- Impressions: 500
- Clicks: 2 (0.4% CTR)
- **Status:** 🚩 Low CTR

### 3. Impressions Drop

**Trigger:** Impressions drop by 50%+ over 28 days

**What it means:** Your page's visibility in search results has significantly decreased. You may have lost rankings across many keywords.

**Action:** Review and update content comprehensively, check for ranking loss on key queries

**Example:**
- 28 days ago: 1000 impressions
- Today: 400 impressions (60% drop)
- **Status:** 🚩 Impressions dropping

## Understanding the Metrics

### CTR (Click-Through Rate)
`CTR = Clicks / Impressions`

- **Good:** 2-5% for competitive keywords, 10%+ for branded keywords
- **Warning:** < 1% indicates title/description needs work
- **Critical:** < 0.5% suggests severe click potential loss

### Average Position
The average ranking position across all queries sending traffic

- **Position 1-3:** Excellent (CTR typically 20-40%)
- **Position 4-10:** Good (CTR typically 2-10%)
- **Position 11+:** Page 2+ (CTR typically < 1%)

### Impressions
How many times your page appears in search results

- Dropping impressions = losing rankings on existing keywords
- Growing impressions = gaining visibility

## Quiet Period

By default, PageBridge ignores content published in the **last 45 days** to avoid flagging new content as decaying.

This prevents false positives on:
- Brand new blog posts (need time to establish rankings)
- Recently launched landing pages
- Recently updated content

You can adjust this in the decay detector configuration if needed.

## Quick Wins

In addition to decay detection, PageBridge identifies **Quick Wins**: keywords ranking in positions 8-20 with high impression volume.

**Why they matter:** These are opportunities to reach page 1 with optimizations:
- Position 8-20 = page 2 of results
- High impressions = significant search interest
- Small ranking boost could reach position 1

**Action:** Update content to target these queries more explicitly

Example:
- Query: "best project management tools"
- Position: 15
- Impressions: 800/month
- Clicks: 8 (1% CTR)
- **Opportunity:** Optimize content for this query → could reach page 1

## Monitoring Blog Posts for Decay

For blog posts, set up monitoring to catch decay early:

1. **Review weekly decay reports** in your Sanity refresh task queue
2. **Set up alerts** for position drops > 5 positions
3. **Track trending queries** - if search volume rises for your keywords, invest in updates
4. **Monitor new competitor content** - if competitors rank higher for your target keywords, refresh your content

## Configuration

Default decay rules are in `packages/core/src/decay-detector.ts`:

```typescript
const DEFAULT_DECAY_RULES = {
  position_decay: {
    threshold: 3,
    days: 28
  },
  low_ctr: {
    threshold: 0.01,
    minPosition: 10
  },
  impressions_drop: {
    threshold: 0.5,
    days: 28
  }
};
```

You can customize these thresholds based on your content strategy.

## Acting on Decay Signals

### For Blog Posts

**Position Decay:**
- Update publish date (freshen the content signal)
- Add new data/statistics
- Improve internal linking
- Consider adding case studies or examples

**Low CTR:**
- Rewrite title to be more compelling
- Improve meta description
- Add power words ("guide", "complete", "2024")
- Consider schema markup for rich snippets

**Impressions Drop:**
- Check if target keywords lost search volume
- Review if competitor content outranked you
- Comprehensive update with new sections
- Add related keywords organically

### For Landing Pages (Use Cases)

**Position Decay:**
- Update case study data/results
- Refresh customer testimonials
- Enhance copy to highlight unique benefits
- Improve CTR with compelling headline

**Low CTR:**
- Test different headline angles
- Add benefit statements to title
- Improve meta description with value prop
- Consider adding "featured snippet" schema

**Impressions Drop:**
- Audit if target keywords changed intent
- Check if search volume declined
- Ensure landing page matches search query
- Consider broader keyword targeting

## Interpreting Decay Reports

In your Sanity Refresh Queue, decay tasks show:

```json
{
  "title": "My Blog Post",
  "decaySignals": {
    "position_decay": {
      "severity": "high",
      "currentPosition": 8,
      "previousPosition": 5,
      "days": 28
    },
    "low_ctr": {
      "severity": "medium",
      "ctr": 0.8,
      "threshold": 1
    }
  }
}
```

- **High severity:** Act within 1-2 weeks
- **Medium severity:** Act within 1 month
- **Low severity:** Monitor and update as part of regular content calendar
