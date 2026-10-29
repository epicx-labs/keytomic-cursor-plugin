---
name: content-calendar
description: Plan and read a Keytomic site's content calendar, research keywords, write, read and edit its articles, and generate their hero images. Use when the user asks what articles are planned or scheduled, wants to add, move or edit topics, wants keyword ideas with search volume, wants to start a calendar run or follow one, wants an article written now, wants to read or edit an article, or wants a new hero image for an article.
---

# Content calendar

The content calendar holds a site's planned topics. Each topic has a scheduled day, a primary keyword with its search volume, difficulty and demand trend, and the planning fields: intent, funnel stage, audience, angles and author override. The daily automation writes the article on the scheduled day, on the site's automation days only, and, with automatic publishing on, publishes it.

These tools need the content MCP Scopes. The tools that change data need both `content:read` and `content:write`. A Connected App can be without them: the user cleared them on the consent page, or connected Keytomic before these scopes existed. Then the server refuses the call before the tool runs: the call fails with an authorization, insufficient scope or HTTP 403 error, sometimes with the text "Keytomic needs approval". This is not the `permission_denied` tool error. When a content tool call fails this way, first tell the user to sign in to Keytomic again in the client and approve all the scopes. In Cursor, the user opens **Customize**, finds **keytomic** under the MCP servers, logs out, and signs in again. If the call still fails after the new sign-in, tell the user to open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps, then connect Keytomic again and approve all the scopes. Stop until the user connects again.

## Steps

1. Find the site with `list_sites`. Never show a projectId, a topicId or a runId. Name a topic by its title and its scheduled day.
2. Pick the tool for the request:

   | The user asks for | Tool |
   | --- | --- |
   | What is planned for a day, a week or a month | `list_calendar_topics` with `from` and `to` (UTC, both included) |
   | Every topic, with the unscheduled ones | `list_calendar_topics` without dates. Page with `startRow` and `nextStartRow` |
   | One topic in full | `list_calendar_topics` with `topicId` |
   | How many keyword searches are left | `get_project` with `sections` set to `["keywordSearches"]` |
   | Keyword ideas with volume, difficulty, competition, CPC and trend | `search_keywords` |
   | A change to a topic: day, title, review status, author, planning fields | `update_calendar_topic` |
   | Topics from the keywords of a search | `add_keywords_to_calendar` |
   | A topic from a title the user writes | `create_calendar_topic` |
   | A new content calendar plan | `start_content_calendar` |
   | How a calendar run is going | `get_content_calendar_progress`. Without a `runId`, it reads the run in progress |
   | The article of a topic, written now | `generate_article` |
   | The text of an article | `get_draft` |
   | A change to an article: title, slug, SEO title and description, text | `update_draft` |
   | A new hero image for an article | `generate_hero_image`, with `creativeDirection` and `exclusions` from the user's words |

3. Before each paid tool, tell the user what the call uses and what is left, then get a yes:
   - `search_keywords` and `create_calendar_topic` use 1 of the site's monthly keyword searches each. Read the `keywordSearches` section of `get_project` first.
   - `add_keywords_to_calendar` uses no keyword search. For `add_keywords_to_calendar` and `create_calendar_topic`, the site must have articles left this month, or Article Credits, for the new topics.
   - `generate_hero_image` uses 1 Blog Hero Credit. Keytomic also calls these credits image credits.
   - `start_content_calendar`, `generate_article` and `generate_hero_image`: call the tool without `confirm` first. It returns a plan and starts nothing. Show the plan to the user. Call again with `confirm` set to `true` only after the user agrees. For `generate_article`, add `useArticleCredit` set to `true` when the plan uses an Article Credit.
4. For `search_keywords`, give the seed keyword and the related keywords with their numbers. Keywords with no search volume are for reference only. For `add_keywords_to_calendar`, send each keyword with its `searchVolume`, `difficulty` and `searchTrend` from `search_keywords`, unchanged.
5. When `add_keywords_to_calendar` or `create_calendar_topic` returns collisions, nothing was added. Show the collisions to the user. Call again with `allowCannibalization` set to `true` only after the user agrees.
6. After `start_content_calendar` starts a run, follow it with `get_content_calendar_progress` until it ends. After `generate_article` starts an article, follow the topic with `list_calendar_topics` and its `topicId`, then read the article with `get_draft`.
   - When a `generate_hero_image` call with `confirm` times out, read the article with `get_draft` before you call again. The image can still finish, and a second call uses a second Blog Hero Credit.
   - When `generate_hero_image` returns `generated_not_attached`, the credit is used and the image is in the Image Studio Media Library. Tell the user to attach it in Content Studio.
7. For `update_calendar_topic` and `update_draft`, show the user the exact fields and the new values, then get a yes. Send only the fields that change.
   - `get_draft` gives `canEdit`. When it is false, this connection cannot save edits: the Connected App does not have `content:write`, or the user's role does not give the permission. Tell the user so before an edit.
   - Read the article with `get_draft` just before `update_draft`. `contentMarkdown` replaces the whole text, and the last save wins. Keep each `[[youtube:ID]]` token and each image URL that stays in the article.
   - Tell the user that an edit to a published article does not change the live page until the article is published again.
8. When the workspace can open only some topics for now, the content tools return only those and a `lockedTopicCount`. Say how many topics are locked for now. The billing rule in Errors applies.

## The report

- List the topics by scheduled day. For each: the title, the primary keyword with its volume, the review status, and whether the article is written or published.
- For a calendar run: the status, each stage with its summary, the number of topics it plans, and the message Keytomic shows when it stopped.
- For an article: the title, the primary keyword, the scheduled day, the author, the SEO title and description, the word count, the publication status with the published URL, and the `contentStudioUrl` link to the article in Keytomic.
- After an edit or an add: what changed, and what each paid call used. For what is left, read the `keywordSearches` or `usage` section of `get_project`. The plan of `start_content_calendar`, `generate_article` and `generate_hero_image` gives the counts before the call.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- `topic_not_in_plan`: the topic is locked for now. Tell the user so.
- `topic_has_article`: the topic has an article, or Keytomic writes its article now. The topic cannot change and no new article can start. Read the topic with `list_calendar_topics` and its `topicId`. When the article is ready, offer `get_draft` and `update_draft`.
- `topic_generation_in_progress`: Keytomic writes the article now. Follow it with `list_calendar_topics` and its `topicId`.
- `draft_not_found`: the topic has no article that is ready. Read the topic with `list_calendar_topics` and its `topicId`. When Keytomic writes the article now, follow it. Otherwise, offer `generate_article`.
- `article_slug_taken`: another article of the site uses the slug. Ask the user for a different slug.
- `image_studio_setup_required`, `automation_days_required`, `calendar_run_required`, `brand_profile_review_required`: the site needs a setup step in Keytomic first. Give the user the message, then stop.
- `brand_profile_incomplete`: the brand profile is not complete. Offer to fill the missing fields with `update_brand_profile` after the user's yes. The owner then confirms the review in Keytomic.
- `active_run_exists`: a calendar run is in progress. Follow it with `get_content_calendar_progress`.
- `active_seo_workflow_exists`: an SEO automation run is in progress. Tell the user to start the calendar run after it ends.
- `no_calendar_days_left`: give the user the message, then stop.
- `usage_limit_reached`: the site used all of a Usage Limit, such as its keyword searches, for the window. Give `limitName`, what was used of `limit`, and `resetsAt` when the error has it. Then stop the calls that use it.
- `insufficient_credits`: the site does not have the credits for the action. Give `limitName` and what is left when the error has them, then stop.
- `project_spend_locked`: a Spend Lock stops paid actions on this site until the billing cycle ends. Tell the user to contact Keytomic support, then stop.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
