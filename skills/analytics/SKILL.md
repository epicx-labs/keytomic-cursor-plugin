---
name: analytics
description: Report a site's Google Analytics 4 traffic and events from Keytomic. Use when the user asks about users, sessions, new users, bounce rate, traffic sources, top pages, countries, traffic from AI assistants or from organic search, or GA4 events and key events.
---

# Analytics

Keytomic reads the Google Analytics 4 property that the owner connected to the site, the same data as the Analytics page in Keytomic.

These tools need the `analytics:read` MCP Scope. A Connected App can be without it: the user cleared it on the consent page, or connected Keytomic before this scope existed. Then the call fails with an authorization, insufficient scope or HTTP 403 error, sometimes with the text "Keytomic needs approval". This is not the `permission_denied` tool error. Tell the user to sign in to Keytomic again and approve all the scopes. In Cursor, the user opens **Customize**, finds **keytomic** under the MCP servers, logs out, and signs in again. If the call still fails, tell the user to open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps, then connect Keytomic again. Stop until the user connects again.

## Steps

1. Find the site with `list_sites`. Never show a projectId or a GA4 property id. The `integrations` section of `get_project` tells whether Google Analytics is connected.
2. Pick the tool for the question:

   | The user asks about | Tool |
   | --- | --- |
   | Users, sessions, new users, bounce rate, traffic sources, top pages, top countries | `get_analytics_performance` |
   | Events, conversions, key events | `list_analytics_events`. Use `eventName` for one event, and `limit` for more than 100 |

3. Set the period with `startDate` and `endDate` (YYYY-MM-DD). Without them, the period is the 28 days that end yesterday.
4. Use the filters when the question asks for them: `trafficSegment` set to `organic` (Organic Search) or `llm` (sessions from AI assistants such as ChatGPT, Perplexity and Gemini), `country` (an ISO code such as `US`), and `device` (`desktop`, `mobile` or `tablet`).

## The report

- Name the period and the filters.
- For `get_analytics_performance`, give each number with its change against the previous period of the same length. `bounceRate` and `share` are fractions from 0 to 1: show them as percentages.
- `get_analytics_performance` results can be up to 6 hours old. Give `generatedAt` when the user needs current data.
- For events: give `eventCount`, `totalUsers` and `keyEvents`. GA4 counts key events only from the day that the owner marked the event as a key event, so `keyEvents` can be lower than `eventCount`.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- `ga4_not_connected`, `ga4_reconnect_required` or `ga4_permission_denied`: the site has no working Google Analytics connection. Give the user the `connectUrl`, then stop.
- `ga4_quota_exceeded`: Google limits the requests for now. Wait `retryAfterSeconds`, then call again.
- `invalid_date_range`: `startDate` is after `endDate`. Correct the dates.
- `ga4_invalid_request`: Google refused the request. Give the user the message, then stop.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
