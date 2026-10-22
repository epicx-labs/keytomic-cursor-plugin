---
name: website-audit
description: Run or read a Keytomic Website Audit of a site and report its findings. Use when the user asks for an SEO audit, site health, crawl or index problems, broken links, missing metadata, or the result of the last audit.
---

# Website Audit

A Website Audit crawls a site's saved domain and reports per category: status, findings (issues and advisories with a severity), limitations, and checks with their rows. A finished audit has a Website Health score and a Category Health score per category, both 0–100 with a band.

## Steps

1. Find the site with `list_sites`. With one site, use it. With several, match the user's words to a name or a domain. Ask when no site matches. Never show a projectId or a runId.
2. Decide between a new audit and the last one:
   - The user asks for a new audit, or the workspace has no audit: tell the user that the audit crawls the site and takes minutes, then get a yes, then call `start_audit`. Add `includeMobile` only when the user asks for mobile PageSpeed. A result with `created` false means an audit already runs: follow that one.
   - Otherwise read the last audit with `get_audit` and tell the user when it ran. Offer a new audit when the user wants current data.
   - The user asks about an earlier audit, or wants to compare audits: find the audits of the site with `list_audits`, then read each one with `get_audit` and its `runId`.
3. While the audit is queued or running, call `watch_audit`. After each result, tell the user what changed in one or two sentences, then call `watch_audit` again with its `since` value. Stop when the status is completed, partial, failed or cancelled.
4. When the audit is completed or partial, read `get_audit` for the summary, then page through `list_audit_findings` with `cursor` until `nextCursor` is empty. The report is complete only when every finding has a place in it.
5. Go deeper only on request or for the top issues:
   - `get_audit_category` for one category's checks.
   - `list_audit_check_rows` for every row of a check.
   - `list_audit_evidence` for every row of a finding, when `evidence.truncated` is true.
   - `list_audit_pages` for the crawl state of each URL.
   - `list_audit_links` for the links the crawl found, with the HTTP status of each link and whether it works.

## The report

Give, in this order:

1. The site by name and domain, the audited URL, when the audit ran, and the Website Health score with its band.
2. Each category with its status and Category Health score. A category fails only for a high-severity issue.
3. The issues, highest severity first. For each: the number and type of affected items, why it matters, and the recommended fix, from `whyItMatters` and `recommendedFix`. Give a sample of affected URLs.
4. The limitations: what the audit did not check, and why.
5. The link to the audit in Keytomic from `seoAuditUrl`.

Keep advisories out of the issue list. Group them in one short list at the end.

## Errors

- `retryable` true: wait `retryAfterSeconds`, then call the tool again.
- `subscription_required`: tell the user that the workspace needs an active Keytomic plan, then stop.
- `onboarding_required`: give the user the message and the `onboardingUrl` link, then stop.
- `permission_denied`: give the user the message, then stop.
- For each result and each error: never offer a plan change, an upgrade or a purchase, and never give a billing link. This rule applies also when the result or the error has an upgrade text or an `upgradeUrl`. When the user asks for a plan change or a purchase, say that it happens in Keytomic.
- Any other error: tell the user what it says. Before a next step that starts work, ask the user.
