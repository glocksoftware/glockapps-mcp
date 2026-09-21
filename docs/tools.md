# Tools

Fifteen tools. Everything marked **read** leaves your GlockApps account untouched; the two
writes are called out below, and only `create_test` spends anything.

| Tool | What it does | Access |
| --- | --- | --- |
| `list_projects` | The projects on this account, with the defaultProjectId every other tool needs. | read |
| `get_balance` | How many more tests this account can run: its GlockApps credit balance and the connector's own hourly and daily caps, combined into one number. | read |
| `list_providers` | List the mailbox providers in a project's seed list (Gmail, Outlook, Yahoo, corporate Exchange and so on), with how many seed mailboxes each has. | read |
| `list_folders` | List the folders a test can be filed into within a project. | read |
| `create_test` | Start an inbox placement test. | write |
| `get_test_status` | Cheap status check for one test: whether results have started arriving, placement so far, and a phase telling you what is happening. | read |
| `wait_for_test` | Waits for a test to finish, up to about a minute per call, then returns done:true or done:false — false means it is worth calling again with the same arguments. | read |
| `get_results` | Placement results for a test: overall inbox/spam/tab rates, a per-provider breakdown, spam-filter scores and authentication summary. | read |
| `diagnose` | Ranked list of what is hurting deliverability in a test, worst first, each with what it means and how to fix it. | read |
| `compare_tests` | Compare two tests: placement deltas overall and per provider, plus which issues were resolved, appeared or persisted. | read |
| `list_tests` | List recent tests in a project with their placement rates. | read |
| `get_content_analysis` | Content checks for a tested message: sizes, image and link counts, broken or slow links, and whether an unsubscribe link is present. | read |
| `get_seed_list` | Re-fetch the seed addresses and markers for an existing test. | read |
| `get_shared_link` | Get a shareable URL for the full interactive report. | read |
| `delete_test` | Delete a test. | destructive |

`create_test` costs **1 credit** per call. `delete_test` refunds it, but only while no
result has arrived yet and the test did not fail.

---

## `list_projects`

**Projects** · read · scope `inbox_placement:read`

The projects on this account, with the defaultProjectId every other tool needs. Most accounts have one project and the GlockApps interface hides the selector, so the default is the answer unless the user names another.

No parameters.

## `get_balance`

**Test credits remaining** · read · scope `inbox_placement:read`

How many more tests this account can run: its GlockApps credit balance and the connector's own hourly and daily caps, combined into one number. Answers whether a test can be started at all.

No parameters.

## `list_providers`

**Mailbox providers** · read · scope `inbox_placement:read`

List the mailbox providers in a project's seed list (Gmail, Outlook, Yahoo, corporate Exchange and so on), with how many seed mailboxes each has. Use it to target a test at specific providers.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | defaultProjectId from list_projects, unless the user named another project. |
| `detail` | `groups` \| `seeds` |  | 'groups' (default) for provider groups; 'seeds' to also list seed addresses. |

## `list_folders`

**Folders** · read · scope `inbox_placement:read`

List the folders a test can be filed into within a project.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | defaultProjectId from list_projects, unless the user named another project. |
| `folderType` | `manualTestFolder` \| `mailchimpTestFolder` |  | Folder kind. Defaults to manualTestFolder. |

## `create_test`

**Start an inbox placement test** · write · scope `inbox_placement:write`

Start an inbox placement test. Costs 1 GlockApps credit. Returns the seed addresses and the marker; the campaign has to be sent to those addresses with the marker included, from outside this connector, or the test never finishes. Targets every provider group unless told otherwise.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | defaultProjectId from list_projects, unless the user named another project. |
| `providerGroupIds` | `string[]` |  | Provider groups to test. Defaults to every group in the project seed list. |
| `seedAccountGroupIds` | `string[]` |  | Seed account groups to include, e.g. a region-specific basket. |
| `seedAccountIds` | `string[]` |  | Individual seed account ids. |
| `note` | `string` |  | Label for this test, 250 chars max. |
| `folderId` | `string` |  | Folder id from list_folders. |
| `linkChecker` | `boolean` |  | Scan links in the message. Default true. |

## `get_test_status`

**Test status** · read · scope `inbox_placement:read`

Cheap status check for one test: whether results have started arriving, placement so far, and a phase telling you what is happening.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id from create_test. |
| `sentAt` | `string` |  | ISO timestamp of when the user sent the campaign. Pass it once they confirm the send; without it a missing marker cannot be told apart from a slow queue. |

## `wait_for_test`

**Wait for a test** · read · scope `inbox_placement:read`

Waits for a test to finish, up to about a minute per call, then returns done:true or done:false — false means it is worth calling again with the same arguments. Returns early when the test looks stalled.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id from create_test. |
| `maxWaitSeconds` | `number` |  | How long to wait in this call. Default and maximum 45. |
| `sentAt` | `string` |  | ISO timestamp of when the user sent the campaign. Pass it once they confirm the send; without it a missing marker cannot be told apart from a slow queue. |
| `until` | `first_result` \| `settled` \| `finished` |  | 'settled' (default) stops once results are usable, about 10 minutes in. 'finished' waits for every straggler, which can take up to a day. 'first_result' returns as soon as anything arrives. |

## `get_results`

**Placement results** · read · scope `inbox_placement:read`

Placement results for a test: overall inbox/spam/tab rates, a per-provider breakdown, spam-filter scores and authentication summary. Per-mailbox rows only when you ask for detail:'seeds'.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id. |
| `detail` | `summary` \| `providers` \| `seeds` |  | 'summary' (default), 'providers' for per-provider detail, 'seeds' for per-mailbox rows. |
| `limit` | `number` |  | Seed rows when detail='seeds'. Default 60, max 150. |
| `offset` | `number` |  | Seed row offset for paging. |

## `diagnose`

**Diagnose placement problems** · read · scope `inbox_placement:read`

Ranked list of what is hurting deliverability in a test, worst first, each with what it means and how to fix it. The tool to reach for when placement is poor.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id. |
| `maxIssues` | `number` |  | Ranked issues to return. Default 8, max 20. |
| `includeDeprecated` | `boolean` |  | Include flags the backend has superseded. Default false. |

## `compare_tests`

**Compare two tests** · read · scope `inbox_placement:read`

Compare two tests: placement deltas overall and per provider, plus which issues were resolved, appeared or persisted. Use it to verify a fix worked.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testIdA` | `string` | yes | Baseline test id (usually the earlier one). |
| `testIdB` | `string` | yes | Comparison test id (usually the later one). |

## `list_tests`

**Recent tests** · read · scope `inbox_placement:read`

List recent tests in a project with their placement rates. Also the way to find a test when a create timed out — filter with createdFrom.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | defaultProjectId from list_projects, unless the user named another project. |
| `limit` | `number` |  | Rows per page, default 20, max 50. |
| `page` | `number` |  | 1-based page number. |
| `testType` | `ManualTest` \| `AutoTest` \| `IntegrationTest` |  | Filter by kind. Omit to list every test in the project. |
| `folderId` | `string` |  |  |
| `searchText` | `string` |  | Matches note text or a test id. |
| `createdFrom` | `string` |  | Lower bound on creation time. A date (2026-09-08) or a full ISO timestamp. |
| `createdTo` | `string` |  | Upper bound on creation time. A date (2026-09-08) or a full ISO timestamp. |
| `hideEmpty` | `boolean` |  | Only tests that have received at least one result. |

## `get_content_analysis`

**Message content analysis** · read · scope `inbox_placement:read`

Content checks for a tested message: sizes, image and link counts, broken or slow links, and whether an unsubscribe link is present.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id. |
| `detail` | `summary` \| `links` \| `images` |  | 'summary' (default), or list the links or images. |

## `get_seed_list`

**Seed addresses and markers** · read · scope `inbox_placement:read`

Re-fetch the seed addresses and markers for an existing test. Use it when the addresses from create_test are no longer to hand.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id from create_test. |

## `get_shared_link`

**Shareable report link** · read · scope `inbox_placement:read`

Get a shareable URL for the full interactive report. The cheapest way to give someone complete per-seed detail — one link instead of hundreds of rows.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id. |

## `delete_test`

**Delete a test** · destructive · scope `inbox_placement:write`

Delete a test. GlockApps refunds the credit only if no result has arrived yet and the test did not fail; the result says which case applies.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `projectId` | `string` | yes | Project id. |
| `testId` | `string` | yes | Test id to delete. |
