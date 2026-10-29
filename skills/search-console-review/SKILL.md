---
name: search-console-review
description: Answer questions about a site's Google Search performance from Keytomic's Search Console tools. Use when the user asks about clicks, impressions, rankings, queries, pages that gain or lose traffic, quick wins, keyword cannibalization, pages with no traffic, question searches, sitemaps, or the index status of URLs.
---

# Search Console review

Keytomic reads the site's connected Google Search Console property. The insight tools answer the common questions with tested thresholds and say how in a `method` block. The raw tools return Google's data as it is.

## Steps

1. Find the site with `list_sites`. The result says whether Search Console is connected. When it is not, give the user the connect link from the tool and stop. Never show a projectId.
2. Start with `get_search_performance` for the period the user names, or the default period. It gives clicks, impressions, CTR and average position, the change against the comparison period, and the top queries and pages.
3. Pick the `kind` of `get_search_insight` for the question. Each kind takes `projectId` and its own inputs, such as a period, or `dimension` set to `query` or `page`. The tool description lists the inputs of each kind:

   | The user asks about | `kind` |
   | --- | --- |
   | The best queries or pages | `top_rankings` (set `sortBy` to `impressions` for visibility) |
   | What is new in search | `new_rankings` |
   | What gains | `improving` |
   | What loses, and why | `declining` (each page carries a likely cause: position, demand or CTR loss) |
   | Quick wins just below the top | `low_hanging_fruits` |
   | Pages that get seen but not clicked | `underperforming` |
   | Several pages that compete for one query | `cannibalization` |
   | Sitemap pages with no search traffic | `zombie_pages` |
   | What people ask | `question_queries` |

4. Use the raw tools when no insight kind answers:
   - `search_analytics_query` for any dimensions and filters.
   - `list_sitemaps` for the sitemaps of the property (with `sitemapUrl` for one sitemap), and `list_sitemap_urls` for the URLs in a sitemap.
   - `inspect_urls` for the index status of up to 50 URLs. Google allows about 2,000 inspections per property per day.
   - `get_project` with `sections` set to `["property"]` for the property and the permission level.
5. To submit URLs or sitemaps to Google and Bing, use the indexing skill.

## The report

- Name the period and the comparison period. Say when the data is not final for the latest days.
- Give each row with its numbers: clicks, impressions, CTR and position. Say what changed, and by how much.
- Give the method in one sentence: the thresholds and the rule from the `method` block. Make no claim that the data does not support. A position has no change when a period had no impressions.
- End with the actions, one per finding. Say which page or query each action is for.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- `gsc_quota_exceeded`: Google rate limits the Search Console requests. Wait `retryAfterSeconds` when the error has it, then call the tool again. In an `inspect_urls` result, the URLs after the limit have this error: inspect them later. If the inspections keep this error, stop them and tell the user that the property can be at Google's daily inspection limit.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
