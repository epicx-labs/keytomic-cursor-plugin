---
name: backlinks
description: Read a site's backlinks and backlink gaps from Keytomic and set the competitors of the gap list. Use when the user asks which sites link to them, about anchor text, broken or new backlinks, link authority or spam, which sites link to a competitor but not to them, link building or outreach, or wants to change their backlink competitors.
---

# Backlinks

Keytomic keeps a backlink report for each site, the same data as the Backlinks page: the backlinks that point to the site, and the backlink gaps, which are pages that link to a competitor but not to the site.

`get_backlinks` needs the `visibility:read` MCP Scope. `set_backlink_competitors` needs both `visibility:read` and `visibility:write`. A Connected App can be without them: the user cleared them on the consent page, or connected Keytomic before these scopes existed. Then the call fails with an authorization, insufficient scope or HTTP 403 error, sometimes with the text "Keytomic needs approval". This is not the `permission_denied` tool error. Tell the user to sign in to Keytomic again and approve all the scopes. In Cursor, the user opens **Customize**, finds **keytomic** under the MCP servers, logs out, and signs in again. If the call still fails, tell the user to open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps, then connect Keytomic again. Stop until the user connects again.

## Steps

1. Find the site with `list_sites`. Never show a projectId.
2. Read the report with `get_backlinks`. It reads saved data only and spends no credits. `fetchedAt` tells when Keytomic read the data.
3. When no report is saved, give the user the `backlinksUrl`. The Backlinks page reads the data when the user opens it.
4. To change the competitors of the gap list:
   - Read the current list in `competitorDomains` of `get_backlinks`.
   - Show the user the full new list, then get a yes. `set_backlink_competitors` replaces the list: send every domain that the list must have. An empty list clears it. A list has at most 10 domains.
   - Tell the user that the gaps of a new competitor show after the next view of the Backlinks page.

## The report

- The backlinks: the referring page, the anchor text, the authority (0 to 100), dofollow or nofollow, the spam score, and whether the link is broken or new.
- The gaps: the page that links to a competitor, and its outreach draft.
- Say when the data was read (`fetchedAt`).

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- `invalid_domains`: a value is not a domain. Ask the user for a domain such as `rival.com`.
- `too_many_competitors`: the list has more than 10 domains. Ask the user which to keep.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
