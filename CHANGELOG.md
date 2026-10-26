# Changelog

Versions here match what the server reports in `initialize` and what is published to the MCP
registry as `com.glockapps/mcp`.

## 1.1.0 — 2026-10-29

### Added
- `get_results`, `get_test_status` and `wait_for_test` return `trackedSeeds` and `seedCountNote`
  when `seedAccountsTotal` is higher than the number of seeds that report a placement. The
  difference is GlockApps monitoring addresses that receive the test but never report where it
  landed. Stats and rates cover the `trackedSeeds`, and nothing is missing.
- `diagnose` returns `suppressed.outdated`, with the `flags` it left out and a `note`, when it
  drops an inbox-rate advisory that no longer applies (see Changed).
- Any tool can return `error: KEY_ROTATED` with `retryable: true` on an OAuth connection, when
  the account's API key was regenerated a moment before the call. Retry the same call: the
  connection picks up the new key by itself.

### Changed
- `get_results`, `diagnose` and `compare_tests` no longer report the `GAPPS_WARM_UP` (inbox rate
  below 50%) and `GAPPS_BOOK_A_CALL` (inbox rate below 70%) advisories once the test's current
  inbox rate is at or above that threshold. GlockApps checks the rate once, 20 minutes into a
  test, and seeds that arrive later can lift it past the threshold.

### Fixed
- `get_shared_link` answered that GlockApps had returned no shareable link, for every test. It
  now returns the test's report link.
- Gmail's Primary tab was counted as `other` rather than inbox in the per-provider and
  per-mailbox placement, so `get_results` could disagree with its own overall rates and with the
  GlockApps web report. Primary now counts as inbox in `get_results`, `diagnose` and
  `compare_tests`, as it does in the web report, and folder names match regardless of case.
- Regenerating the API key in GlockApps could disconnect an OAuth connection that was in use at
  that moment. The connection now stays connected; a call made at that moment returns
  `KEY_ROTATED` (see Added).
- Connecting from a client that opens the GlockApps API key page twice, such as Cursor, failed
  with "Request could not be verified" when the earlier page was submitted. Every open page can
  now be submitted.
- Connecting Cursor in Chrome could fail after the API key was accepted, because the browser
  blocked the redirect back to Cursor. The GlockApps page now hands back to the client in a way
  the browser allows.
- The GlockApps API key page expired after 10 minutes, too soon to fetch the key in another tab
  and come back. It now lasts 30 minutes, and its error pages say to click Connect again in the
  AI app.
- Security fixes.

---

## 1.0.0 — 2026-09-29

First public release.

- Fifteen tools: thirteen read-only, `create_test` (1 credit) and `delete_test`.
- OAuth 2.1 with dynamic client registration and PKCE, for Claude and ChatGPT.
- A GlockApps API key as a bearer token, for clients without OAuth.
- Streamable HTTP at `https://api.glockapps.com/mcp`.
