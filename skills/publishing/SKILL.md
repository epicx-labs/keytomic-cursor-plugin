---
name: publishing
description: Publish a Keytomic article to the site's CMS and check where it was published. Use when the user wants to publish an article, send it to WordPress, Webflow, Shopify, Framer, Contentful, Notion, GoHighLevel, a webhook or an API destination as a draft or live, or asks whether or where an article was published.
---

# Publishing

Keytomic publishes the article of a content calendar topic to a publishing destination of the site: a CMS, a webhook or an API destination that the owner connected in Keytomic. A destination takes the article as a draft or live.

`publish_draft` needs the `content:publish` and `content:read` MCP Scopes. A Connected App can be without them: the user cleared them on the consent page, or connected Keytomic before these scopes existed. Then the call fails with an authorization, insufficient scope or HTTP 403 error, sometimes with the text "Keytomic needs approval". This is not the `permission_denied` tool error. Tell the user to sign in to Keytomic again and approve all the scopes. In Cursor, the user opens **Customize**, finds **keytomic** under the MCP servers, logs out, and signs in again. If the call still fails, tell the user to open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps, then connect Keytomic again. Stop until the user connects again.

## Steps

1. Find the site with `list_sites`. Never show a projectId, a topicId or a destinationId. Name a topic by its title and its scheduled day, and a destination by its name and provider.
2. Find the topic with `list_calendar_topics`. The topic must have an article. When it has none, offer to write it with the content-calendar skill.
3. Read the destinations with `list_publishing_destinations`:
   - No destination: give the user the `connectUrl`, where the owner connects one in Keytomic, then stop.
   - A destination with status `needs_attention`: give the user its `fixUrl`, and do not publish to it.
   - Several destinations: ask the user which one.
4. Pick the mode. `draft` (the default) sends the article as a draft. `publish` makes it live. A destination with draftMode `not_supported` takes only `publish`. For a destination with draftMode `receiver_decides` (a webhook), the receiver decides.
5. Before the first call to `publish_draft`, tell the user the article, the destination and the mode, then get a yes. A first draft to a GoHighLevel destination runs on this call.
6. Call `publish_draft` without `confirm`. When the result has status `plan`, nothing was sent. Show the plan to the user: the destination, the mode, the expected URL, whether it adds a new post or updates an earlier one, and whether it uses a daily publish, with the publishes left today. Call again with `confirm` set to `true` only after the user agrees.
7. When a call that publishes times out or answers `internal_error`, read `get_publication_status` before you call `publish_draft` again. The publish can still finish, and a second call can use a second daily publish or add a second post.
8. When the user asks whether or where an article was published, read `get_publication_status`. It gives the latest publication for each mode of each destination that has one.

## The report

- After a publish: the status (`sent_as_draft`, `published`, or `sent_to_receiver` for a webhook), the public URL (a draft has none), whether it updated an earlier post, and whether it used a daily publish.
- For a status question: each destination with its mode, its status (`pending`, `succeeded` or `failed`) and its URL. For a failed publication, give its `failureMessage` and its `fixUrl`.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- `publishing_destination_unavailable`: the destination was removed, needs a reconnect, or belongs to another site. Read `list_publishing_destinations` again, and give the user the `fixUrl`.
- `destination_setup_required`: the destination needs a setting first. Give the user the `fixUrl`, then stop.
- `draft_mode_not_supported`: the destination cannot keep a draft. Offer mode `publish`, which makes the article live and returns a plan first.
- `publication_in_progress`: a publish of this article to this destination has not ended. Read `get_publication_status` before you try again.
- `publish_failed`: the destination did not confirm the publish, so the article may or may not be there. Give the user the `fixUrl`, and read `get_publication_status` before you try again.
- `article_slug_taken`: another article of the site uses the slug. Ask the user for a new slug, change it with `update_draft` after the user's yes, then call `publish_draft` again.
- `draft_not_found`: Keytomic cannot find the article now. Read the topic again with `get_calendar_topic`.
- `topic_not_in_plan`: the topic is locked for now. Tell the user so.
- `image_studio_setup_required`: the user must save the site's Image Studio settings in Keytomic first.
- `usage_limit_reached`: the site used all its daily publishes. Give `limitName`, what was used of `limit`, and `resetsAt` when the error has it. Then stop the publishes.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
- Any other error with a `fixUrl`: give the user the message and the `fixUrl`, then stop.
