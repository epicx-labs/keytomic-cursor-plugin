---
name: ai-visibility
description: Track how AI platforms such as ChatGPT mention a brand, with Keytomic AI Visibility. Use when the user asks whether AI search or AI assistants mention their brand, about their AI visibility or share of voice against competitors, which sources AI answers cite, the prompts that Keytomic tracks, or wants to set up AI Visibility, change its prompts or settings, or run a check.
---

# AI Visibility

Keytomic asks AI platforms the questions that buyers ask (the prompts). It reports how the answers mention the brand and its competitors, the same as the AI Visibility page. The setup researches the site and saves Suggested prompts. Active prompts go into each check.

The read tools need the `visibility:read` MCP Scope. The tools that change data or start work need both `visibility:read` and `visibility:write`. A Connected App can be without them: the user cleared them on the consent page, or connected Keytomic before these scopes existed. Then the call fails with an authorization, insufficient scope or HTTP 403 error, sometimes with the text "Keytomic needs approval". This is not the `permission_denied` tool error. Tell the user to sign in to Keytomic again and approve all the scopes. In Cursor, the user opens **Customize**, finds **keytomic** under the MCP servers, logs out, and signs in again. If the call still fails, tell the user to open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps, then connect Keytomic again. Stop until the user connects again.

## Steps

1. Find the site with `list_sites`. Never show a projectId, a promptId or a checkId.
2. Read the state with `get_visibility`. Without `sections`, it returns `overview`: the settings, the limits of the plan, the setup status and the next scheduled check. Add `check`, `prompts` or `suggestions` in `sections` when the request needs them.
3. Pick the tool for the request:

   | The user asks for | Tool |
   | --- | --- |
   | The results: metrics, trend, AI platform scores, competitor share, sources, answers | `get_visibility_report` for a date range |
   | The answers to one prompt | `get_visibility_report` with `promptId` |
   | The setup | `start_visibility_setup` |
   | Prompts to track | `add_visibility_prompts` |
   | A change to the text or the state of a prompt | `update_visibility_prompt` |
   | The AI platforms to check, or the run days | `update_visibility_settings` |
   | A check now | `start_visibility_check` |

4. `start_visibility_setup` researches the site and saves Suggested prompts. Keytomic pays for it. It makes no prompt Active and starts no check. Follow it with the `overview` and `suggestions` sections of `get_visibility`.
5. For `add_visibility_prompts`, `update_visibility_prompt` and `update_visibility_settings`, show the user the change, then get a yes. `update_visibility_settings` replaces the settings: send every value. A text edit of an Active prompt can make a new prompt with a new `promptId`: use the `promptId` of the result after it.
6. `start_visibility_check` uses one observation for each Active prompt on each AI platform. Call it without `confirm` first. It returns a plan and starts nothing. Show the plan to the user: the observations the check needs and the observations left this month. Call again with `confirm` set to `true` only after the user agrees. When the schedule runs the checks, a user can start only the first check of a site.
7. A check runs in the background. Read its results later with `get_visibility` and `get_visibility_report`.

## The report

- Name the date range and the AI platforms.
- Give the brand's visibility and its trend, the score on each AI platform, the competitor share, and the sources that the answers cite.
- For a prompt: its text, how many answers mention the brand, and samples of the answers.
- Give the `visibilityUrl` link to the AI Visibility page.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- `feature_disabled`: AI Visibility is not available. Tell the user so, then stop.
- `check_running`: a check runs now. Prompts cannot change until it ends.
- `active_prompt_limit_reached`, `saved_prompt_limit_reached`, `tier_limit_exceeded`: the change is above a limit of the plan. Give the user the limit from the `overview` section, then stop.
- `setup_branded_limit_exceeded`: the call adds too many prompts that name the brand. Ask the user which to keep.
- `prompt_not_found`, `prompt_revision_conflict`: read the prompts again with `get_visibility`, then try again.
- `prompt_duplicate`: the market already has this text.
- `manual_check_not_allowed`: the schedule runs the checks of this site. Give the user `nextRunAt`.
- `insufficient_capacity`: the site does not have the observations for the check. Give what is left, then stop.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
