# AI SEO with Keytomic for Cursor

This plugin brings Keytomic, the end-to-end AI SEO agent, into Cursor. It connects the Cursor agent to one Keytomic workspace through the Keytomic MCP server. It adds four skills that run the common SEO workflows with its tools: Website Audits, Search Console reviews, brand profile and settings, and the content calendar with keyword research, articles and hero images.

[![Add Keytomic MCP to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=keytomic&config=eyJ1cmwiOiJodHRwczovL2FwcC5rZXl0b21pYy5jb20vbWNwIn0=)

## Keytomic: the end-to-end AI SEO agent

Keytomic is the AI SEO agent that runs the whole SEO cycle for your site, for Google and AI search:

1. **Research**: finds the keywords and topics that your customers search for on Google and in AI search.
2. **Write**: plans your content calendar and writes on-brand articles with images.
3. **Publish**: publishes to your site and gets your pages indexed.
4. **Monitor**: tracks your rankings, your Search Console data, and how often AI search mentions your brand.
5. **Improve**: refreshes content that loses traffic, finds technical issues, and finds backlink opportunities.

One platform does the work of a keyword tool, an AI writer, a publishing workflow, and a rank tracker: AI SEO automation from the first keyword to the next ranking gain. Plans start at $99/month, or $79/month when you pay yearly, with a free trial. [See pricing](https://keytomic.com/pricing).

## What the plugin does

- **Website Audit**: start an audit of a site, follow it to the end, and read the findings by category, with the affected pages and the recommended fix.
- **Search Console review**: find the top, new, improving and declining queries and pages, the low-hanging fruits, keyword cannibalization, zombie pages, question queries, and the raw Search Analytics data.
- **Workspace setup**: read a site's brand profile, settings, author profiles, Usage Limits and integrations. Edit the brand profile, the Article Settings, the indexing settings and the author profiles after your yes.
- **Content calendar**: research keywords, read and edit the planned topics, add topics, start a calendar run and follow it, write, read and edit articles, and generate their hero images. Each action that uses credits happens only after your yes.

## Requirements

- A Keytomic workspace on a paid plan or in a trial. A workspace with no active plan, or one still in Onboarding, gets an error with a link to Keytomic instead of data.
- A Google Search Console property connected to the site, for the Search Console tools. The other tools do not need it.
- Cursor 2.5 or later, for plugins.

## Install

**The full plugin, from a clone of this repository.** Copy the repository to your local Cursor plugins folder, then run **Developer: Reload Window**:

```bash
git clone https://github.com/epicx-labs/keytomic-cursor-plugin ~/.cursor/plugins/local/keytomic
```

**The MCP server only, without the skills.** Click **Add Keytomic MCP to Cursor** above, or add this to your `mcp.json`:

```json
{
  "mcpServers": {
    "keytomic": {
      "url": "https://app.keytomic.com/mcp"
    }
  }
}
```

**For a team.** On Cursor Teams and Enterprise, an admin can import this repository as a team marketplace. The repository has a `.cursor-plugin/marketplace.json` for this.

## Sign in

The first tool call asks you to sign in to Keytomic in your browser and to choose a workspace. You do not need an API key. The connection is a Connected App. Keytomic asks for these MCP Scopes: workspace read and write, audit read and run, Search Console read and write, and content read and write. On the consent page, you can clear a scope that you do not want to give. You can disconnect the Connected App at any time on the Keytomic MCP page in Keytomic, under Connected apps.

If the Connected App does not have a scope, the server refuses the tools of that scope with an HTTP 403 `insufficient_scope` response. This occurs when you cleared the scope on the consent page, or when you connected Keytomic before the content scopes were available. Cursor shows an authorization or insufficient scope error for them. This error is not the `permission_denied` tool error, and a role change does not correct it. The other tools continue to operate. First, open **Customize**, find **keytomic** under the MCP servers, log out, and sign in again with all the scopes. If the error continues, open the Keytomic MCP page in Keytomic and click **Disconnect** on the app's row under Connected apps. Then connect Keytomic again and approve all the scopes.

## What the plugin sends and reads

The plugin has no code of its own. It declares one remote MCP server, `https://app.keytomic.com/mcp`, in `mcp.json`, and its skills are text files. Cursor sends each tool call to that server with your sign-in token. The server reads and writes only the data of the workspace you chose. The plugin sends data to no other destination.

Tool calls that spend credits or start work, such as a new audit, a keyword search, a calendar run, an article or a hero image, happen only after you say yes. Edits to the brand profile and the settings spend no credits.

## Skills

| Skill | Use it for |
| --- | --- |
| `website-audit` | Run or read a Website Audit and get a report of the findings |
| `search-console-review` | Answer questions about Google Search performance with the insight tools |
| `workspace-setup` | Read and edit the brand profile, the settings and the author profiles, and read the integrations |
| `content-calendar` | Research keywords, plan and edit the calendar topics, start a calendar run, write and edit articles, and generate hero images |

## Related

- [Keytomic MCP](https://github.com/epicx-labs/keytomic-mcp): install steps for every MCP client, the tool list, the limits and the errors.
- [AI SEO with Keytomic for Claude](https://github.com/epicx-labs/keytomic-claude-plugin): the same skills as a Claude plugin.

## Support

Write to hi@keytomic.com. The privacy policy is at [keytomic.com/privacy-policy](https://keytomic.com/privacy-policy).

## License

MIT. See `LICENSE`.
