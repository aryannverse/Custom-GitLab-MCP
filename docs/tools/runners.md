# Runners

CI/CD runner management — list, inspect, pause, resume, delete runners, and view jobs executed by a specific runner.

!!! note "Feature toggle"
    Opt-in. Enable via `GITLAB_TOOLSETS=runners` (or `GITLAB_TOOLSETS=all`), list individual tools in `GITLAB_TOOLS=`, or activate at runtime with the `discover_tools` MCP tool.

## Tools in this group

- [`list_runners`](#list_runners) — ✏️ Writes
- [`get_runner`](#get_runner) — ✏️ Writes
- [`pause_runner`](#pause_runner) — ✏️ Writes
- [`resume_runner`](#resume_runner) — ✏️ Writes
- [`delete_runner`](#delete_runner) — ✏️ Writes
- [`list_runner_jobs`](#list_runner_jobs) — ✏️ Writes

---

### `list_runners`

*✏️ Writes*

List GitLab CI/CD runners. Can be scoped to all runners, a specific project, or a specific group. Supports filtering by type, status, and tags.

**Parameters**

| Parameter | Type | Required | Description |
|---|---|:-:|---|
| `type` | enum (`instance_type` \| `group_type` \| `project_type`) |  | Filter runners by type: instance_type, group_type, or project_type |
| `status` | enum (`online` \| `offline` \| `stale` \| `never_contacted` \| `active` \| `paused`) |  | Filter runners by status |
| `paused` | boolean |  | Whether to include only paused (true) or only active (false) runners |
| `tag_list` | array<string> |  | Filter runners by tags (comma-separated list of tag names) |
| `project_id` | string |  | Project ID or URL-encoded path — lists only runners available to this project |
| `group_id` | string |  | Group ID or URL-encoded path — lists only runners available to this group |
| `page` | integer |  | Page number (default: 1) |
| `per_page` | integer |  | Number of results per page (default: 20, max: 100) |

### `get_runner`

*✏️ Writes*

Get detailed information about a specific GitLab CI/CD runner including platform, architecture, tags, version, IP address, and last contact time.

**Parameters**

| Parameter | Type | Required | Description |
|---|---|:-:|---|
| `runner_id` | string | ✓ | The ID of the runner |

### `pause_runner`

*✏️ Writes*

Pause (disable) a GitLab CI/CD runner so it will not pick up new jobs. The runner remains registered but inactive.

**Parameters**

| Parameter | Type | Required | Description |
|---|---|:-:|---|
| `runner_id` | string | ✓ | The ID of the runner to pause |

### `resume_runner`

*✏️ Writes*

Resume (re-enable) a previously paused GitLab CI/CD runner so it can start picking up jobs again.

**Parameters**

| Parameter | Type | Required | Description |
|---|---|:-:|---|
| `runner_id` | string | ✓ | The ID of the runner to resume / re-activate |

### `delete_runner`

*✏️ Writes*

Permanently delete a GitLab CI/CD runner. This action is irreversible — the runner must be re-registered to be used again.

**Parameters**

| Parameter | Type | Required | Description |
|---|---|:-:|---|
| `runner_id` | string | ✓ | The ID of the runner to permanently delete |

### `list_runner_jobs`

*✏️ Writes*

List jobs that have been executed by a specific GitLab CI/CD runner. Useful for auditing runner activity and debugging job failures.

**Parameters**

| Parameter | Type | Required | Description |
|---|---|:-:|---|
| `runner_id` | string | ✓ | The ID of the runner whose jobs to list |
| `status` | enum (`running` \| `success` \| `failed` \| `canceled`) |  | Filter jobs by status: running, success, failed, or canceled |
| `order_by` | enum (`id`) |  | Order jobs by field (currently only 'id' is supported) |
| `sort` | enum (`asc` \| `desc`) |  | Sort direction: asc or desc (default: desc) |
| `page` | integer |  | Page number (default: 1) |
| `per_page` | integer |  | Number of results per page (default: 20, max: 100) |
