# GlockApps MCP

Run real inbox placement tests from your AI assistant. Ask it to start a test, and it hands you
the seed addresses to send to; ask again once you have sent, and it tells you who put you in the
inbox, who put you in spam, and what to fix first.

The seed network is live mailboxes at Gmail, Outlook, Yahoo, AOL, Zoho, corporate Exchange and
30+ other providers.

```
https://api.glockapps.com/mcp
```

This repository is documentation and the registry manifest. The server itself is hosted by
GlockApps — there is nothing here to install or run.

## What you need

- A **GlockApps account** with API access on the plan. Your API key is on
  [Account → API / MCP](https://app.glockapps.com/account/api).
- **Test credits.** Every test costs one, whether it is started here or in the GlockApps web app.

## Quick connect

### Claude.ai, Claude Desktop

Settings → Connectors → **Add custom connector**. Paste the address above and leave Client ID and
Client Secret empty. Select **Connect**: a GlockApps page opens and asks for your API key once.

### Claude Code

```bash
claude mcp add --transport http glockapps https://api.glockapps.com/mcp
```

Then `/mcp` in the session, and authenticate in the browser window it opens.

### ChatGPT

Turn on developer mode (Settings → Security and login), then Plugins → **+** → **Create app** →
**Create MCP App**. Paste the address, leave Authentication on **OAuth**, select **Create**, and
paste your API key on the GlockApps page that opens. Developer mode is required while the
connector is not yet in the ChatGPT directory.

### Cursor, Windsurf, Continue and other clients

Clients that cannot do OAuth can send the API key directly. Copy a config from
[`configs/`](configs) and put your key in it:

```json
{
    "mcpServers": {
        "glockapps": {
            "url": "https://api.glockapps.com/mcp",
            "headers": { "Authorization": "Bearer YOUR_GLOCKAPPS_API_KEY" }
        }
    }
}
```

Full per-client instructions: [`docs/connect.md`](docs/connect.md).

## How a test works

The one thing to understand before you start: **this connector cannot send your email.** It gives
you addresses to send to, and it reads what arrives.

1. `list_projects` — resolves the project everything else needs.
2. `create_test` — **costs 1 credit.** Returns the seed addresses and a marker.
3. **You send.** Send the campaign you want to test, from the platform you are testing, to every
   seed address, with the marker included — either the `X-API-Campaign-id` header or the
   `id: ...` line in the body.
4. `wait_for_test` — results are usually usable about 10 minutes after sending.
5. `get_results`, then `diagnose` for what to fix, worst first.

**A send without the marker never matches the test.** Nothing arrives, the test never finishes,
and the credit is still spent. If that happens, ask your assistant to delete the test: the credit
comes back while no result has arrived.

Some providers accept mail slowly, so a test that has not "finished" after an hour is normal, not
broken.

## Things to ask it

- "Start an inbox placement test on my main project."
- "Has the test finished? What is the inbox rate?"
- "Why did this campaign land in spam?"
- "Compare this test with yesterday's. Did the fix work?"
- "Which providers are filtering us, and what should I change first?"

## Tools

Fifteen tools, thirteen of them read-only. Full reference with parameters:
[`docs/tools.md`](docs/tools.md).

| | |
| --- | --- |
| **Read** | `list_projects` `list_folders` `list_providers` `list_tests` `get_balance` `get_seed_list` `get_test_status` `wait_for_test` `get_results` `diagnose` `get_content_analysis` `compare_tests` `get_shared_link` |
| **Write** | `create_test` — costs 1 credit |
| **Destructive** | `delete_test` — removes the test and its results; refunds the credit only while no result has arrived |

## Security

- **Scopes.** `inbox_placement:read` and `inbox_placement:write`. The connector cannot see or
  change your password, billing details or sending accounts.
- **It never sends email.** Your campaign is sent by you, from your own platform.
- **Your key is not shared with the AI provider.** In the OAuth flow it is exchanged on a
  GlockApps page for a token scoped to this connector; it stays on your GlockApps account.
- **Regenerating your key does not break an OAuth connection.** The connector records which
  account it is linked to, not the key, and picks up the current one on each call. It *does* end
  any connection using the static-key method above.
- **Rate limits.** Test creation is capped per hour and per day on top of your credit balance.
  `get_balance` reports the effective number.

## Troubleshooting

**"No test credits left"** — top up in GlockApps, or run the test from the web app. `get_balance`
tells you what is left before you start.

**The test never finishes and shows nothing** — almost always a send without the marker. Delete
the test to get the credit back, start a fresh one, and follow its sending instructions exactly.

**Some results missing after an hour** — usually fine; slower providers take hours. Nothing at all
within ten minutes of sending means the marker was missing.

**"Shareable report links are unavailable"** — not part of every subscription. Your assistant can
still read per-mailbox results directly.

**Connecting fails with "API key not recognised"** — copy it again from
[Account → API / MCP](https://app.glockapps.com/account/api); API access has to be enabled on your
plan.

## Support

[Open an issue](https://github.com/glocksoftware/glockapps-mcp/issues) or email
support@glock-apps.com.

- Setup guide: https://glockapps.com/mcp-setup/
- About the connector: https://glockapps.com/mcp/
- Privacy policy: https://glockapps.com/privacy-policies/
