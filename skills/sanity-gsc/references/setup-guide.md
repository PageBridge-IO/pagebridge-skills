# PageBridge Setup Guide

## Prerequisites

- **Node.js 18+** and **pnpm**
- **PostgreSQL database** (can use Docker)
- **Google Search Console property** with service account credentials
- **Sanity CMS project** with API token
- Basic understanding of your website's URL structure

## Step 1: Clone and Install

```bash
git clone https://github.com/PageBridge-IO/pagebridge.git
cd pagebridge
pnpm install
```

## Step 2: Set Up PostgreSQL

### Using Docker

```bash
docker compose up -d
```

This starts PostgreSQL on `localhost:5432` (default credentials in docker-compose.yml).

### Manual Setup

Create a PostgreSQL database and note the connection string:
```
postgresql://username:password@localhost:5432/pagebridge
```

## Step 3: Configure Environment Variables

Create a `.env` file in the root directory:

```bash
# Google Service Account (export as JSON from Google Cloud Console, then stringify)
GOOGLE_SERVICE_ACCOUNT='{"type":"service_account","project_id":"...","private_key":"..."}'

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/pagebridge

# Sanity
SANITY_PROJECT_ID=your_project_id
SANITY_DATASET=production
SANITY_TOKEN=your_api_token

# Site Configuration
SITE_URL=https://yoursite.com
```

### Getting Google Service Account

1. Go to Google Cloud Console
2. Create a service account with "Google Search Console API" access
3. Download the JSON key
4. Stringify the JSON (remove newlines, escape quotes) for the environment variable

### Getting Sanity Credentials

1. Go to your Sanity project settings
2. Create an API token with write permissions
3. Note your project ID and dataset name

## Step 4: Set Up Database Schema

```bash
pnpm db:push
```

This creates the necessary tables: `search_analytics`, `query_analytics`, `sync_log`, `page_index_status`, and `unmatch_diagnostics`.

## Step 5: Verify Installation

```bash
# Test the CLI
pnpm sync --help

# Test Sanity connection
pnpm sync --site sc-domain:yoursite.com --dry-run
```

## Troubleshooting Setup

**"Invalid service account"** - Ensure JSON is properly stringified (no line breaks)

**"Database connection failed"** - Verify DATABASE_URL is correct and PostgreSQL is running

**"Sanity auth failed"** - Check SANITY_TOKEN has write permissions to your dataset

**"GSC property not found"** - Verify your property exists and the service account has access
