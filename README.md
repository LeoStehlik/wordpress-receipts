# WordPress Receipts

OpenClaw skill for publishing WordPress posts with receipts.

It exists for one boring reason: a scheduled agent job is not done because the cron says `ok`. It is done when the WordPress API returns a post ID and the public URL returns HTTP 200.

## Install

```bash
openclaw skills install wordpress-receipts
```

## What It Does

- publishes Markdown content through the WordPress REST API
- parses env files explicitly, including application passwords with spaces
- avoids duplicate posts by checking recent published titles
- verifies API status and public page status
- gives cron jobs a deterministic post-run proof check

## Required Env

```text
WORDPRESS_API_BASE=https://example.com/wp-json/wp/v2
WORDPRESS_USERNAME=publisher-user
WORDPRESS_APPLICATION_PASSWORD=application password may contain spaces
WORDPRESS_AUTHOR_ID=123
WORDPRESS_DEFAULT_CATEGORY_ID=1
```

Do not commit real env files.

## Publish

```bash
node scripts/wp-publish-receipt.mjs \
  --env-file .env.wordpress \
  --title "My Post" \
  --content-file draft.md \
  --author 123 \
  --category 1
```

Output:

```json
{
  "ok": true,
  "action": "published",
  "id": 456,
  "status": "publish",
  "link": "https://example.com/my-post/",
  "publicStatus": 200
}
```

## Verify

```bash
node scripts/wp-verify-receipt.mjs --env-file .env.wordpress --date 2026-07-01
node scripts/wp-verify-receipt.mjs --env-file .env.wordpress --id 456
node scripts/wp-verify-receipt.mjs --url https://example.com/my-post/
```

## Cron Pattern

1. Main cron publishes the post.
2. A second cron runs shortly after the publish window.
3. The second cron runs `wp-verify-receipt.mjs`.
4. If verification fails, alert the human with the exact missing receipt.

That turns "the job sounded fine" into "the artifact exists".

## Privacy

This repository contains no private domains, paths, tokens, chat IDs, post IDs, or workspace names. The scripts are env-driven and generic.

## License

MIT
