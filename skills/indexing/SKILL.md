---
name: indexing
description: Check and request the indexing of a site's pages on Google and Bing with Keytomic. Use when the user asks whether their posts or URLs are indexed, why a page is not indexed, when Google or Bing last crawled a page, or wants to submit URLs or sitemaps to Google or Bing.
---

# Indexing

Keytomic reads the indexing status of a site's pages from Google Search Console and Bing Webmaster Tools. It also submits URLs and sitemaps to them.

`get_indexing_status` and `inspect_urls` read Google data with the `gsc:read` MCP Scope and Bing data with the `analytics:read` MCP Scope. An engine whose scope the Connected App does not have gives the status `scope_missing`. `submit_for_indexing` needs the `indexing:write` MCP Scope. A Connected App can be without it: the user cleared it on the consent page, or connected Keytomic before this scope existed. Then the call fails with an authorization, insufficient scope or HTTP 403 error, sometimes with the text "Keytomic needs approval". This is not the `permission_denied` tool error. Tell the user to sign in to Keytomic again and approve all the scopes. In Cursor, the user opens **Customize**, finds **keytomic** under the MCP servers, logs out, and signs in again. If the call still fails, tell the user to open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps, then connect Keytomic again. Stop until the user connects again.

## Steps

1. Find the site with `list_sites`. Never show a projectId or a topicId.
2. Pick the tool for the request:

   | The user asks for | Tool |
   | --- | --- |
   | The indexing status of the posts that Keytomic published | `get_indexing_status`. It reads saved data and makes no Google or Bing call |
   | The saved Google report of one post | `get_indexing_status` with `url` or `topicId` |
   | A live check of URLs on Google and Bing | `inspect_urls`, with up to 50 URLs |
   | A request to index URLs or sitemaps | `submit_for_indexing` |

3. `inspect_urls` uses Google's limit of about 2,000 inspections per property per day. Bing allows only a few calls a minute. Keytomic checks only the first URLs live on Bing, and gives its saved Bing result for the others. Inspect only the URLs that the user asks about.
4. Before `submit_for_indexing`, tell the user what goes to each engine, then get a yes:
   - Google takes sitemaps only. With `urls`, Google gets the preferred sitemap of the site. With `sitemapUrls`, every engine gets those sitemaps. With neither, every engine gets the known sitemaps of the site.
   - Google needs a connected account with Owner or Full permission on the property.
   - Bing takes URLs and sitemaps. Each URL uses the Bing URL quota of the site. A sitemap uses none.
5. Google and Bing read a submit later, so `inspect_urls` shows no change at once. Tell the user so.

## The report

- For a status: the published, indexed and pending numbers, then each post that needs attention, with its Google and Bing status. Bing gives crawl dates only, so a Bing status is `crawled` or `not_indexed`, never `indexed`.
- For a live check: each URL with its Google verdict, coverage and last crawl, and its Bing status.
- For a submit: each engine with its status (`accepted`, `skipped` or `failed`), the reason, and what it sent. For Bing, give the URL quota that is left.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- An engine with the status `not_connected`: the site has not connected it. The owner connects it in Keytomic.
- An engine with the status `scope_missing`: tell the user to sign in to Keytomic again and approve all the scopes.
- A Google entry with `detail.error`: give the user the message. When it has a `connectUrl`, give the link, then stop the calls to Google.
- `gsc_quota_exceeded`: Google rate limits the requests. Inspect the remaining URLs later.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
