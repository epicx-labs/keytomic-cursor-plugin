---
name: reddit-opportunities
description: Find Reddit threads where a site can answer, with Keytomic Reddit Opportunities, and write reply drafts. Use when the user asks for Reddit threads to answer, Reddit marketing, a reply to a Reddit thread, the topics or subreddits that Keytomic scans, or wants to mark a thread done or hide it.
---

# Reddit Opportunities

Keytomic scans Reddit each day for threads about a site's topics and keeps the threads that fit, the same as the Reddit Opportunities page. Nothing posts to Reddit: the user copies a reply draft and posts it.

The read tools need the `visibility:read` MCP Scope. The tools that change data or start work need both `visibility:read` and `visibility:write`. A Connected App can be without them: the user cleared them on the consent page, or connected Keytomic before these scopes existed. Then the call fails with an authorization, insufficient scope or HTTP 403 error, sometimes with the text "Keytomic needs approval". This is not the `permission_denied` tool error. Tell the user to sign in to Keytomic again and approve all the scopes. In Cursor, the user opens **Customize**, finds **keytomic** under the MCP servers, logs out, and signs in again. If the call still fails, tell the user to open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps, then connect Keytomic again. Stop until the user connects again.

## Steps

1. Find the site with `list_sites`. Never show a projectId or an opportunityId. Name a thread by its title and its subreddit.
2. Pick the tool for the request:

   | The user asks for | Tool |
   | --- | --- |
   | The topics, the subreddits, the Agent Mode and the scan times | `get_reddit_settings` |
   | The threads to answer | `list_reddit_threads` with `view` set to `open` |
   | The threads that left the open list | `list_reddit_threads` with `view` set to `history` |
   | New topics, subreddits or Agent Mode | `update_reddit_settings` |
   | A scan now | `scan_reddit` |
   | A reply draft for a thread | `generate_reddit_reply` |
   | Mark a thread done, hide it, or restore it | `set_reddit_opportunity_status` |

3. For `update_reddit_settings`, read `get_reddit_settings` first. The save replaces the settings: send every value. Show the user the new settings, then get a yes. New topics make one AI call. The next daily scan uses the new settings.
4. `scan_reddit` starts a scan in the background. A site can start one scan each UTC day, and Keytomic pays for it. Get a yes first. Read the new threads later with `list_reddit_threads`.
5. `generate_reddit_reply` makes one AI call and adds a new reply draft to an open thread. Get a yes first. The draft follows the Agent Mode of the site and the Brand Mention Safety of the thread.

## The report

- For each open thread: the title, the subreddit, the relevance score (0 to 100), why it fits, the Brand Mention Safety, and the newest reply draft.
- For a reply draft: the text, and a reminder that the user copies it from the Reddit Opportunities page and posts it.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- A `manual_check_used` result: the site used its scan of today. Give `checkNowAvailableAt`, then stop.
- `opportunity_not_found`: the thread is not on the open list. Read `list_reddit_threads` again.
- A `reply_failed` result: the reply broke the brand rules of the thread, and Keytomic saved no draft. Tell the user so.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
