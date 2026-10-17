# GitLab MCP Server

> A production-grade **Model Context Protocol (MCP)** server for GitLab. Engineered for autonomous AI coding assistants, agents, and pair programmers — featuring deep integration with **Google Antigravity**, **Claude Code**, **Cursor**, and **VS Code**.

[![Author](https://img.shields.io/badge/Author-aryannverse-blue.svg)](https://github.com/aryannverse)
[![Tools](https://img.shields.io/badge/Tools-218%20Available-orange.svg)](#complete-tool-inventory-218-tools)
[![Toolsets](https://img.shields.io/badge/Toolsets-20%20Categories-green.svg)](#toolsets-overview)
[![Runtime](https://img.shields.io/badge/Runtime-Node.js%20%3E%3D%2018%20%7C%20TypeScript-informational.svg)](#prerequisites)
[![Deployment](https://img.shields.io/badge/Deployment-Docker%20%7C%20stdio%20%7C%20HTTP-purple.svg)](#docker-deployment)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

---

## Overview

This repository provides a comprehensive, production-ready **GitLab Model Context Protocol (MCP)** server. It exposes a rich set of **218 tools** across **20 toolsets**, granting AI agents full read, write, and orchestration capabilities across GitLab SaaS (`gitlab.com`) or self-hosted GitLab CE/EE instances.

### Key Features & Capabilities

- 🚀 **CI/CD Runner Management**: Built-in 6-tool suite for managing GitLab runners (list, inspect, pause, resume, delete, and job execution logs).
- 🤖 **Native Google Antigravity Integration**: Pre-configured and tested for both **Antigravity IDE** and **Antigravity CLI** via Docker.
- 🛡️ **Fine-Grained Safety Tiers**: Three distinct permission modes (`full`, `modify`, `readonly`) preventing accidental deletions in production environments.
- ⚡ **Dynamic On-Demand Toolsets**: Keep agent context windows lean by starting with 9 default toolsets and loading opt-in toolsets at runtime using `discover_tools`.
- 🔌 **Multi-Transport Architecture**: Native support for `stdio` (local process IPC), `Streamable HTTP` (remote sessions), and `SSE` (Server-Sent Events).

---

## Table of Contents

- [Toolsets Overview](#toolsets-overview)
- [Spotlight: CI/CD Runner Management](#spotlight-cicd-runner-management)
- [Quick Start & Client Configurations](#quick-start--client-configurations)
  - [Google Antigravity IDE & CLI](#google-antigravity-ide--cli)
  - [Docker Setup (Recommended)](#docker-setup-recommended)
  - [Claude Code & Claude Desktop](#claude-code--claude-desktop)
  - [Cursor IDE](#cursor-ide)
  - [VS Code / GitHub Copilot](#vs-code--github-copilot)
- [Authentication Methods](#authentication-methods)
- [Security & Permission Modes](#security--permission-modes)
- [Environment Variables Reference](#environment-variables-reference)
- [Tools by Category](#tools-by-category)
- [Complete Tool Inventory (218 Tools)](#complete-tool-inventory-218-tools)
- [How Custom Tools Are Built](#how-custom-tools-are-built)
- [Development & Verification](#development--verification)

---

## Toolsets Overview

Tools are grouped into **20 domain toolsets**. The server boots with **9 core toolsets active by default (114 tools)** to minimize token overhead. Opt-in toolsets (104 tools) can be activated via environment variables or on the fly by the AI.

| Toolset ID | Tools | Default State | Core Responsibilities |
|---|:---:|:---:|---|
| `merge_requests` | 43 | **Active** | MR creation, updates, diffs, conflicts, approvals, threads, and draft notes |
| `issues` | 24 | **Active** | Issue tracking, description patches, links, discussions, and todo items |
| `branches` | 15 | **Active** | Branch creation/deletion, protected branch policies, commit history, blame, and statuses |
| `projects` | 10 | **Active** | Project details, member listing, group iterations, namespace verification |
| `repositories` | 7 | **Active** | File reading, directory trees, single-file edits, and atomic multi-file commits |
| `users` | 7 | **Active** | User details, profile lookup (`whoami`), activity events, and markdown uploads |
| `labels` | 5 | **Active** | Project and group label inspection and creation |
| `ci` | 4 | **Active** | `.gitlab-ci.yml` linting, project CI validation, CI Catalog components |
| `groups` | 1 | **Active** | Group and subgroup creation with namespace hierarchy |
| `pipelines` | 19 | *Opt-in* | Pipeline triggers, status checks, job logs/traces, manual job plays, cancellations, and artifacts |
| `workitems` | 18 | *Opt-in* | GitLab Work Items, hierarchy, custom field definitions, and timeline events |
| `milestones` | 17 | *Opt-in* | Milestone tracking, burndown events, group and project associations |
| `wiki` | 10 | *Opt-in* | Project and group wiki documentation pages CRUD |
| `variables` | 10 | *Opt-in* | CI/CD project and group variables with environment scope isolation |
| `releases` | 7 | *Opt-in* | Release management, evidence collection, and release asset downloads |
| `runners` | 6 | *Opt-in* | **CI/CD runner management: list, inspect, pause, resume, delete, and job auditing** |
| `tags` | 5 | *Opt-in* | Tag management, release tagging, and X.509 signature verification |
| `dependency_proxy` | 4 | *Opt-in* | Docker Hub proxy cache settings, cached blob listings, and cache purge |
| `webhooks` | 3 | *Opt-in* | Project and group webhook inspection, event history, and delivery payloads |
| `search` | 3 | *Opt-in* | Global, group, and project code search (Zoekt / Advanced Search) |

*(Plus 2 standalone meta tools: `discover_tools` and `execute_graphql`.)*

---

## Spotlight: CI/CD Runner Management

Standard GitLab MCP servers only support pipelines and jobs, leaving the underlying runner infrastructure inaccessible. I designed and implemented the **`runners`** toolset to bridge this gap.

### The 6 Custom Runner Tools

1. **`list_runners`**
   - List GitLab CI/CD runners across instance, group, or project levels.
   - Filters:
     - `type`: `instance_type` (shared runners), `group_type`, or `project_type`
     - `status`: `online`, `offline`, `stale`, `never_contacted`, `active`, `paused`
     - `tag_list`: Filter by tags (e.g. `["docker", "gpu", "linux"]`)
     - `project_id` / `group_id`: Scope results to a specific project or group

2. **`get_runner`**
   - Retrieve deep runner specifications:
     - Platform, architecture, software version, IP address, tag list, active status, maximum timeout, and timestamp of last contact.

3. **`pause_runner`**
   - Temporarily disable a runner from picking up new CI/CD jobs. The runner remains registered but inactive, perfect for maintenance or host patching.

4. **`resume_runner`**
   - Re-enable a paused runner so it immediately resumes job consumption from the queue.

5. **`delete_runner`** *(destructive)*
   - Permanently remove a decommissioned runner from the GitLab instance.

6. **`list_runner_jobs`**
   - Audit all jobs executed by a specific runner.
   - Filter by status (`running`, `success`, `failed`, `canceled`) to quickly diagnose runner-specific failures.

### Practical Scenarios

```markdown
# Scenario 1: Triage offline or stale runners
list_runners(status: "stale")                # Find runners that missed heartbeat
get_runner(runner_id: "53980205")            # Inspect IP, OS, and tags
list_runner_jobs(runner_id: "53980205", status: "failed") # Check failed builds

# Scenario 2: Host maintenance workflow
pause_runner(runner_id: "53980205")          # Safely drain runner
# ... apply updates or reboot host machine ...
resume_runner(runner_id: "53980205")         # Put runner back in rotation
```

---

## Quick Start & Client Configurations

### Google Antigravity IDE & CLI

My primary environment is **Google Antigravity**. To configure this MCP in Antigravity:

Edit your MCP configuration file:
- **Antigravity IDE**: `~/.gemini/antigravity/mcp_config.json`
- **Antigravity CLI**: `~/.gemini/antigravity-cli/mcp_config.json`

```json
{
  "mcpServers": {
    "gitlab-mcp": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e", "GITLAB_PERSONAL_ACCESS_TOKEN=your_personal_access_token",
        "-e", "GITLAB_API_URL=https://gitlab.com/api/v4",
        "-e", "GITLAB_TOOLSETS=all",
        "custom-gitlab-mcp:latest"
      ]
    }
  }
}
```

### Docker Setup (Recommended)

Running inside Docker ensures all Node.js and TypeScript dependencies are isolated.

```bash
# 1. Clone this repository
git clone https://github.com/aryannverse/Custom-GitLab-MCP.git
cd Custom-GitLab-MCP

# 2. Build the Docker image
docker build -t custom-gitlab-mcp:latest .

# 3. Test running the image
echo '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"whoami","arguments":{}}}' | \
  docker run -i --rm \
    -e GITLAB_PERSONAL_ACCESS_TOKEN="your_gitlab_token" \
    -e GITLAB_API_URL="https://gitlab.com/api/v4" \
    -e GITLAB_TOOLSETS="all" \
    custom-gitlab-mcp:latest
```

### Claude Code & Claude Desktop

In `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "gitlab": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e", "GITLAB_PERSONAL_ACCESS_TOKEN=your_token_here",
        "-e", "GITLAB_API_URL=https://gitlab.com/api/v4",
        "-e", "GITLAB_TOOLSETS=all",
        "custom-gitlab-mcp:latest"
      ]
    }
  }
}
```

### Cursor IDE

In Cursor **Settings > Features > MCP**:
- **Name**: `GitLab`
- **Type**: `command`
- **Command**:
  ```bash
  docker run -i --rm -e GITLAB_PERSONAL_ACCESS_TOKEN=your_token -e GITLAB_API_URL=https://gitlab.com/api/v4 -e GITLAB_TOOLSETS=all custom-gitlab-mcp:latest
  ```

### VS Code / GitHub Copilot

In `.vscode/mcp.json`:

```json
{
  "servers": {
    "gitlab": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e", "GITLAB_PERSONAL_ACCESS_TOKEN=your_token",
        "-e", "GITLAB_API_URL=https://gitlab.com/api/v4",
        "-e", "GITLAB_TOOLSETS=all",
        "custom-gitlab-mcp:latest"
      ]
    }
  }
}
```

---

## Authentication Methods

The server supports 4 authentication mechanisms:

1. **Personal Access Token (PAT)** *(Recommended)*
   - Simplest configuration. Create a PAT in GitLab under **User Settings > Access Tokens** with scope `api` (or `read_api` for read-only setups).
   - Set via `GITLAB_PERSONAL_ACCESS_TOKEN=glpat-...`

2. **Local OAuth 2.0 PKCE Browser Flow**
   - Recommended when you do not wish to store long-lived tokens on disk.
   - Set `GITLAB_USE_OAUTH=true`, `GITLAB_OAUTH_CLIENT_ID`, and `GITLAB_OAUTH_REDIRECT_URI`.
   - On first request, automatically launches your default browser for user authentication.

3. **Remote MCP OAuth Proxy**
   - For remote web clients like Claude.ai. Unauthenticated calls receive `401 + WWW-Authenticate`, initiating standard OAuth 2.0 discovery.
   - Set `GITLAB_MCP_OAUTH=true` and `STREAMABLE_HTTP=true`.

4. **Remote Multi-User Authorization**
   - For shared multi-user server instances.
   - Set `REMOTE_AUTHORIZATION=true` and `STREAMABLE_HTTP=true`. Callers pass their token per HTTP session via `Authorization: Bearer <token>` or `Private-Token: <token>`.

---

## Security & Permission Modes

To safeguard repositories against accidental destruction by autonomous agents, configure the permission mode:

```bash
# Full access — allows all operations (default)
GITLAB_PERMISSION_MODE=full

# Modify mode — allows reads, creates, and updates, but strictly blocks all 28 permanent delete tools
GITLAB_PERMISSION_MODE=modify

# Read-only mode — allows only the 117 query tools, rejecting all modifications
GITLAB_PERMISSION_MODE=readonly
```

### Tool Filtering Controls

- **`GITLAB_TOOLSETS`**: Comma-separated list of toolsets to enable (e.g. `runners,pipelines,merge_requests`) or `all`.
- **`GITLAB_TOOLS`**: Comma-separated list of individual tools to allow (e.g. `list_runners,get_runner`).
- **`GITLAB_DENIED_TOOLS_REGEX`**: Regular expression pattern to deny specific tools (e.g. `^delete_.*`).

---

## Environment Variables Reference

| Variable | Default | Description |
|---|:---:|---|
| `GITLAB_PERSONAL_ACCESS_TOKEN` | - | GitLab personal access token |
| `GITLAB_API_URL` | `https://gitlab.com/api/v4` | Base GitLab API URL (supports self-hosted endpoints) |
| `GITLAB_TOOLSETS` | 9 default toolsets | Comma-separated list of toolset IDs or `all` |
| `GITLAB_TOOLS` | - | Comma-separated list of specific tools to whitelist |
| `GITLAB_DENIED_TOOLS_REGEX` | - | Regex pattern of tools to block |
| `GITLAB_PERMISSION_MODE` | `full` | Permission tier: `full`, `modify`, or `readonly` |
| `STREAMABLE_HTTP` | `false` | Set to `true` to run Streamable HTTP transport |
| `HOST` | `127.0.0.1` | Server listening IP address (`0.0.0.0` in Docker) |
| `PORT` | `3002` | Server listening port |
| `GITLAB_DISABLE_VERSION_CHECK` | `false` | Suppress new version notices on stderr |

---

## Tools by Category

### `merge_requests` (43 tools) — **Default: Active**

- `merge_merge_request` *(destructive)* — Merge a merge request
- `approve_merge_request`  — Approve a merge request
- `unapprove_merge_request`  — Unapprove a merge request
- `get_merge_request_approval_state` *(read-only)* — Get merge request approval details including approvers
- `get_branch` *(read-only)* — Get branch details (commit, protection status)
- `list_branches` *(read-only)* — List branches in project with search filter
- `get_merge_request_conflicts` *(read-only)* — Get the conflicts of a merge request
- `list_merge_request_pipelines` *(read-only)* — List pipelines for a merge request with pagination
- `get_merge_request` *(read-only)* — Get details of a merge request (mergeRequestIid or branchName required). Set include_summaries=true for deployment/commit/approval summaries
- `get_merge_request_diffs` *(read-only)* — Get the changes/diffs of a merge request (mergeRequestIid or branchName required)
- `list_merge_request_changed_files` *(read-only)* — List changed file paths in a merge request without diff content (mergeRequestIid or branchName required)
- `list_merge_request_diffs` *(read-only)* — List merge request diffs with pagination (mergeRequestIid or branchName required)
- `get_merge_request_file_diff` *(read-only)* — Get diffs for specific files from a merge request (mergeRequestIid or branchName required)
- `list_merge_request_versions` *(read-only)* — List all versions of a merge request
- `get_merge_request_version` *(read-only)* — Get a specific version of a merge request
- `update_merge_request`  — Update a merge request (mergeRequestIid or branchName required)
- `create_merge_request`  — Create a new merge request
- `list_merge_requests` *(read-only)* — List merge requests (without project_id: user's MRs; with project_id: project MRs)
- `get_branch_diffs` *(read-only)* — Get diffs between two branches or commits
- `mr_discussions` *(read-only)* — List discussion items for a merge request
- `create_merge_request_note`  — Add a new note to a merge request
- `update_merge_request_note`  — Modify an existing merge request note
- `delete_merge_request_note` *(destructive)* — Delete an existing merge request note
- `get_merge_request_note` *(read-only)* — Get a specific note for a merge request
- `get_merge_request_notes` *(read-only)* — List notes for a merge request
- `delete_merge_request_discussion_note` *(destructive)* — Delete a discussion note on a merge request
- `update_merge_request_discussion_note`  — Update a discussion note on a merge request
- `create_merge_request_discussion_note`  — Add a new discussion note to an existing merge request thread
- `get_draft_note` *(read-only)* — Get a single draft note from a merge request
- `list_draft_notes` *(read-only)* — List draft notes for a merge request
- `create_draft_note`  — Create a draft note for a merge request
- `update_draft_note`  — Update an existing draft note
- `delete_draft_note` *(destructive)* — Delete a draft note
- `publish_draft_note`  — Publish a single draft note
- `bulk_publish_draft_notes`  — Publish all draft notes for a merge request
- `create_merge_request_thread`  — Create a new thread on a merge request
- `resolve_merge_request_thread`  — Resolve a thread on a merge request
- `list_merge_request_emoji_reactions` *(read-only)* — List all emoji reactions on a merge request
- `list_merge_request_note_emoji_reactions` *(read-only)* — List all emoji reactions on a merge request note. Pass discussion_id for discussion thread replies.
- `create_merge_request_emoji_reaction`  — Add an emoji reaction to a merge request (e.g. thumbsup, rocket, eyes)
- `delete_merge_request_emoji_reaction` *(destructive)* — Remove an emoji reaction from a merge request
- `create_merge_request_note_emoji_reaction`  — Add an emoji reaction to a merge request note. Pass discussion_id for discussion thread replies.
- `delete_merge_request_note_emoji_reaction` *(destructive)* — Remove an emoji reaction from a merge request note. Pass discussion_id for discussion thread replies.

### `issues` (24 tools) — **Default: Active**

- `create_issue`  — Create a new issue
- `list_issues` *(read-only)* — List issues (default: created by current user; use scope='all' for all)
- `my_issues` *(read-only)* — List issues assigned to the authenticated user
- `get_issue` *(read-only)* — Get details of a specific issue. Returns a slim milestone by default; set full_response=true for the complete milestone object
- `update_issue`  — Update an issue. Returns a slim confirmation by default; set full_response=true for the complete updated issue object
- `update_issue_description_patch`  — Apply a patch (search/replace or unified diff) to an issue description. Reduces token usage by allowing small changes without sending the full description. Supports dry_run to preview changes and create_note to summarize updates.
- `delete_issue` *(destructive)* — Delete an issue
- `list_todos` *(read-only)* — List GitLab to-do items for the current user
- `mark_todo_done`  — Mark a GitLab to-do item as done
- `mark_all_todos_done`  — Mark all pending GitLab to-do items as done for the current user
- `create_issue_note`  — Add a note to an issue, optionally replying to a discussion thread
- `update_issue_note`  — Modify an existing issue thread note
- `list_issue_links` *(read-only)* — List all issue links for a specific issue
- `list_issue_discussions` *(read-only)* — List discussions for an issue
- `get_issue_link` *(read-only)* — Get a specific issue link
- `create_issue_link`  — Create an issue link between two issues
- `delete_issue_link` *(destructive)* — Delete an issue link
- `create_note`  — Create a new note (comment) to an issue or merge request
- `list_issue_emoji_reactions` *(read-only)* — List all emoji reactions on an issue
- `list_issue_note_emoji_reactions` *(read-only)* — List all emoji reactions on an issue note. Pass discussion_id for discussion thread replies.
- `create_issue_emoji_reaction`  — Add an emoji reaction to an issue (e.g. thumbsup, rocket, eyes)
- `delete_issue_emoji_reaction` *(destructive)* — Remove an emoji reaction from an issue
- `create_issue_note_emoji_reaction`  — Add an emoji reaction to an issue note. Pass discussion_id for discussion thread replies.
- `delete_issue_note_emoji_reaction` *(destructive)* — Remove an emoji reaction from an issue note. Pass discussion_id for discussion thread replies.

### `repositories` (7 tools) — **Default: Active**

- `search_repositories` *(read-only)* — Search for GitLab projects
- `create_repository`  — Create a new GitLab project
- `get_file_contents` *(read-only)* — Get contents of a file or directory from a GitLab project
- `push_files` *(destructive)* — Push multiple files in a single commit
- `create_or_update_file`  — Create or update a file in a GitLab project
- `fork_repository`  — Fork a project to your account or specified namespace
- `get_repository_tree` *(read-only)* — List files and directories in a repository

### `branches` (15 tools) — **Default: Active**

- `create_branch`  — Create a new branch
- `get_branch` *(read-only)* — Get branch details (commit, protection status)
- `list_branches` *(read-only)* — List branches in project with search filter
- `delete_branch` *(destructive)* — Delete branch from project
- `list_protected_branches` *(read-only)* — List protected branches in a project, supports search filter
- `get_protected_branch` *(read-only)* — Get details of a single protected branch (access levels, force push settings)
- `protect_branch` *(destructive)* — Protect a repository branch (set push/merge/unprotect access levels)
- `unprotect_branch` *(destructive)* — Remove protection from a previously protected branch
- `update_default_branch` *(destructive)* — Change the default branch of a project
- `list_commits` *(read-only)* — List repository commits with filtering options
- `get_commit` *(read-only)* — Get details of a specific commit
- `get_commit_diff` *(read-only)* — Get changes/diffs of a specific commit
- `get_file_blame` *(read-only)* — Get git blame for a file at a given ref. Each entry maps a contiguous range of source lines to the commit that last changed them (id, author, authored_date, message). Use range_start/range_end to limit blame to specific lines.
- `list_commit_statuses` *(read-only)* — List statuses for a commit
- `create_commit_status`  — Create or update the status of a commit

### `projects` (10 tools) — **Default: Active**

- `get_project` *(read-only)* — Get details of a specific project
- `list_projects` *(read-only)* — List projects accessible by the current user
- `update_project`  — Update project settings such as description, visibility, default branch, and feature access levels
- `list_project_members` *(read-only)* — List members of a GitLab project
- `list_namespaces` *(read-only)* — List all namespaces (users and groups) available to the current user. Filter by kind='group' for groups only.
- `get_namespace` *(read-only)* — Get details of a namespace (user or group) by ID or path. Groups are namespaces with kind='group'.
- `verify_namespace` *(read-only)* — Verify if a namespace path exists. Use parent_id to scope the check to a specific parent namespace — required for nested namespaces where the same path may exist under different parents.
- `list_group_projects` *(read-only)* — List projects in a group
- `list_group_iterations` *(read-only)* — List group iterations with filtering options
- `health_check` *(read-only)* — Verify server status and authentication

### `labels` (5 tools) — **Default: Active**

- `list_labels` *(read-only)* — List labels for a project
- `get_label` *(read-only)* — Get a single label from a project
- `create_label`  — Create a new label in a project
- `update_label`  — Update an existing label in a project
- `delete_label` *(destructive)* — Delete a label from a project

### `ci` (4 tools) — **Default: Active**

- `validate_ci_lint` *(read-only)* — Validate provided GitLab CI/CD YAML content for a project
- `validate_project_ci_lint` *(read-only)* — Validate an existing .gitlab-ci.yml configuration for a project
- `list_ci_catalog_resources` *(read-only)* — List GitLab CI/CD Catalog resources/components visible to the user
- `get_ci_catalog_resource` *(read-only)* — Get details for a GitLab CI/CD Catalog resource, including versions and components

### `groups` (1 tools) — **Default: Active**

- `create_group`  — Create new group or subgroup

### `pipelines` (19 tools) — *Opt-in* (`GITLAB_TOOLSETS=pipelines`)

- `list_pipelines` *(read-only)* — List pipelines with filtering options
- `get_pipeline` *(read-only)* — Get details of a specific pipeline
- `list_deployments` *(read-only)* — List deployments with filtering options
- `get_deployment` *(read-only)* — Get details of a specific deployment
- `list_environments` *(read-only)* — List environments in a project
- `get_environment` *(read-only)* — Get details of a specific environment
- `list_pipeline_jobs` *(read-only)* — List all jobs in a specific pipeline
- `list_pipeline_trigger_jobs` *(read-only)* — List trigger jobs (bridges) in a pipeline
- `get_pipeline_job` *(read-only)* — Get details of a GitLab pipeline job number
- `get_pipeline_job_output` *(read-only)* — Get the output/trace of a pipeline job with optional pagination
- `create_pipeline`  — Create a new pipeline for a branch or tag
- `retry_pipeline`  — Retry a failed or canceled pipeline
- `cancel_pipeline`  — Cancel a running pipeline
- `play_pipeline_job`  — Run a manual pipeline job
- `retry_pipeline_job`  — Retry a failed or canceled pipeline job
- `cancel_pipeline_job`  — Cancel a running pipeline job
- `list_job_artifacts` *(read-only)* — List artifact files in a job's archive
- `download_job_artifacts` *(read-only)* — Download job artifact archive (zip) and save to a local path
- `get_job_artifact_file` *(read-only)* — Get content of a single file from a job's artifacts

### `milestones` (17 tools) — *Opt-in* (`GITLAB_TOOLSETS=milestones`)

- `list_milestones` *(read-only)* — List milestones with filtering options
- `get_milestone` *(read-only)* — Get details of a specific milestone
- `create_milestone`  — Create a new milestone
- `edit_milestone`  — Edit an existing milestone
- `delete_milestone` *(destructive)* — Delete a milestone
- `get_milestone_issue` *(read-only)* — Get issues associated with a specific milestone
- `get_milestone_merge_requests` *(read-only)* — Get merge requests associated with a specific milestone
- `promote_milestone`  — Promote a milestone to the next stage
- `get_milestone_burndown_events` *(read-only)* — Get burndown events for a specific milestone
- `list_group_milestones` *(read-only)* — List group milestones with filtering options
- `get_group_milestone` *(read-only)* — Get details of a specific group milestone
- `create_group_milestone`  — Create a new group milestone
- `edit_group_milestone`  — Edit an existing group milestone
- `delete_group_milestone` *(destructive)* — Delete a group milestone
- `get_group_milestone_issue` *(read-only)* — Get issues associated with a specific group milestone
- `get_group_milestone_merge_requests` *(read-only)* — Get merge requests associated with a specific group milestone
- `get_group_milestone_burndown_events` *(read-only)* — Get burndown events for a specific group milestone

### `wiki` (10 tools) — *Opt-in* (`GITLAB_TOOLSETS=wiki`)

- `list_wiki_pages` *(read-only)* — List wiki pages in a project
- `get_wiki_page` *(read-only)* — Get details of a specific wiki page
- `create_wiki_page`  — Create a wiki page in a project
- `update_wiki_page`  — Update a wiki page in a project
- `delete_wiki_page` *(destructive)* — Delete a wiki page from a project
- `list_group_wiki_pages` *(read-only)* — List wiki pages in a group
- `get_group_wiki_page` *(read-only)* — Get details of a specific group wiki page
- `create_group_wiki_page`  — Create a wiki page in a group
- `update_group_wiki_page`  — Update a wiki page in a group
- `delete_group_wiki_page` *(destructive)* — Delete a wiki page from a group

### `releases` (7 tools) — *Opt-in* (`GITLAB_TOOLSETS=releases`)

- `list_releases` *(read-only)* — List all releases for a project
- `get_release` *(read-only)* — Get a release by tag name
- `create_release`  — Create a new release
- `update_release`  — Update an existing release
- `delete_release` *(destructive)* — Delete a release (does not delete the tag)
- `create_release_evidence`  — Create release evidence (Premium/Ultimate)
- `download_release_asset` *(read-only)* — Download a release asset file by direct asset path

### `tags` (5 tools) — *Opt-in* (`GITLAB_TOOLSETS=tags`)

- `list_tags` *(read-only)* — List repository tags for a project
- `get_tag` *(read-only)* — Get a repository tag by name
- `create_tag`  — Create a new repository tag
- `delete_tag` *(destructive)* — Delete a repository tag
- `get_tag_signature` *(read-only)* — Get the X.509 signature of a signed tag (404 if unsigned)

### `users` (7 tools) — **Default: Active**

- `get_users` *(read-only)* — Get GitLab user details by usernames
- `get_user` *(read-only)* — Get user details by ID
- `whoami` *(read-only)* — Get current authenticated user details
- `list_events` *(read-only)* — List events for the authenticated user (before/after: YYYY-MM-DD)
- `get_project_events` *(read-only)* — List events for a project (before/after: YYYY-MM-DD)
- `upload_markdown`  — Upload a file for use in markdown content
- `download_attachment` *(read-only)* — Download an uploaded file from a project (images returned as base64; use local_path to save to disk)

### `workitems` (18 tools) — *Opt-in* (`GITLAB_TOOLSETS=workitems`)

- `get_work_item` *(read-only)* — Get a work item with full details including status, hierarchy, type, and widgets
- `list_work_items` *(read-only)* — List work items with filters (type, state, search, assignees, labels)
- `create_work_item`  — Create a work item (issue, task, incident, epic, etc.) with full field support
- `update_work_item`  — Update a work item (title, description, labels, assignees, state, parent, custom fields, etc.)
- `convert_work_item_type`  — Convert a work item to a different type
- `list_work_item_statuses` *(read-only)* — List available statuses for a work item type (Premium/Ultimate)
- `list_custom_field_definitions` *(read-only)* — List custom field definitions for a work item type
- `move_work_item`  — Move a work item to a different project
- `list_work_item_notes` *(read-only)* — List notes and discussions on a work item
- `create_work_item_note`  — Add a note to a work item (supports Markdown, internal notes, threads)
- `list_work_item_emoji_reactions` *(read-only)* — List all emoji reactions on a work item
- `list_work_item_note_emoji_reactions` *(read-only)* — List all emoji reactions on a work item note (comment, thread, or thread reply)
- `create_work_item_emoji_reaction`  — Add an emoji reaction to a work item (e.g. thumbsup, rocket, eyes)
- `delete_work_item_emoji_reaction` *(destructive)* — Remove an emoji reaction from a work item
- `create_work_item_note_emoji_reaction`  — Add an emoji reaction to a work item note (comment, thread, or thread reply)
- `delete_work_item_note_emoji_reaction` *(destructive)* — Remove an emoji reaction from a work item note (comment, thread, or thread reply)
- `get_timeline_events` *(read-only)* — List timeline events for an incident
- `create_timeline_event`  — Create a timeline event on an incident

### `webhooks` (3 tools) — *Opt-in* (`GITLAB_TOOLSETS=webhooks`)

- `list_webhooks` *(read-only)* — List webhooks for a project or group
- `list_webhook_events` *(read-only)* — List recent webhook events (past 7 days)
- `get_webhook_event` *(read-only)* — Get full details of a specific webhook event

### `search` (3 tools) — *Opt-in* (`GITLAB_TOOLSETS=search`)

- `search_code` *(read-only)* — Search for code across all projects (requires advanced search or Zoekt)
- `search_project_code` *(read-only)* — Search for code within a specific project (requires advanced search or Zoekt)
- `search_group_code` *(read-only)* — Search for code within a specific group (requires advanced search or Zoekt)

### `variables` (10 tools) — *Opt-in* (`GITLAB_TOOLSETS=variables`)

- `list_project_variables` *(read-only)* — List CI/CD variables for a project
- `get_project_variable` *(read-only)* — Get a single CI/CD variable from a project
- `create_project_variable`  — Create a CI/CD variable for a project
- `update_project_variable`  — Update an existing CI/CD variable in a project
- `delete_project_variable` *(destructive)* — Delete a CI/CD variable from a project
- `list_group_variables` *(read-only)* — List CI/CD variables for a group
- `get_group_variable` *(read-only)* — Get a single CI/CD variable from a group
- `create_group_variable`  — Create a CI/CD variable for a group
- `update_group_variable`  — Update an existing CI/CD variable in a group
- `delete_group_variable` *(destructive)* — Delete a CI/CD variable from a group

### `dependency_proxy` (4 tools) — *Opt-in* (`GITLAB_TOOLSETS=dependency_proxy`)

- `get_dependency_proxy_settings` *(read-only)* — Get dependency proxy settings for a group
- `update_dependency_proxy_settings`  — Update dependency proxy settings for a group (enable/disable, credentials for authenticated Docker Hub pulls)
- `list_dependency_proxy_blobs` *(read-only)* — List cached dependency proxy blobs for a group
- `purge_dependency_proxy_cache` *(destructive)* — Schedule purge of all cached dependency proxy blobs for a group

### `runners` (6 tools) — *Opt-in* (`GITLAB_TOOLSETS=runners`)

- `list_runners` *(read-only)* — List GitLab CI/CD runners. Can be scoped to all runners, a specific project, or a specific group. Supports filtering by type, status, and tags.
- `get_runner` *(read-only)* — Get detailed information about a specific GitLab CI/CD runner including platform, architecture, tags, version, IP address, and last contact time.
- `pause_runner`  — Pause (disable) a GitLab CI/CD runner so it will not pick up new jobs. The runner remains registered but inactive.
- `resume_runner`  — Resume (re-enable) a previously paused GitLab CI/CD runner so it can start picking up jobs again.
- `delete_runner` *(destructive)* — Permanently delete a GitLab CI/CD runner. This action is irreversible — the runner must be re-registered to be used again.
- `list_runner_jobs` *(read-only)* — List jobs that have been executed by a specific GitLab CI/CD runner. Useful for auditing runner activity and debugging job failures.


### `meta` (2 tools) — **Default: Active**

- `discover_tools` *(read-only)* — Discover and activate additional tool categories at runtime for the current session.
- `execute_graphql` *(read-only / mutation)* — Execute custom GraphQL queries and mutations against GitLab.

---

## Complete Tool Inventory (218 Tools)

<details>
<summary><b>Click to expand full numbered list of all 218 tools</b></summary>

<!-- TOOLS-START -->

1. `merge_merge_request` *(destructive)* - Merge a merge request
2. `approve_merge_request`  - Approve a merge request
3. `unapprove_merge_request`  - Unapprove a merge request
4. `get_merge_request_approval_state` *(read-only)* - Get merge request approval details including approvers
5. `get_merge_request_conflicts` *(read-only)* - Get the conflicts of a merge request
6. `list_merge_request_pipelines` *(read-only)* - List pipelines for a merge request with pagination
7. `execute_graphql` *(read-only)* - Execute a GitLab GraphQL query
8. `create_or_update_file`  - Create or update a file in a GitLab project
9. `search_repositories` *(read-only)* - Search for GitLab projects
10. `create_repository`  - Create a new GitLab project
11. `create_group`  - Create new group or subgroup
12. `get_file_contents` *(read-only)* - Get contents of a file or directory from a GitLab project
13. `push_files` *(destructive)* - Push multiple files in a single commit
14. `create_issue`  - Create a new issue
15. `create_merge_request`  - Create a new merge request
16. `fork_repository`  - Fork a project to your account or specified namespace
17. `create_branch`  - Create a new branch
18. `get_branch` *(read-only)* - Get branch details (commit, protection status)
19. `list_branches` *(read-only)* - List branches in project with search filter
20. `delete_branch` *(destructive)* - Delete branch from project
21. `list_protected_branches` *(read-only)* - List protected branches in a project, supports search filter
22. `get_protected_branch` *(read-only)* - Get details of a single protected branch (access levels, force push settings)
23. `protect_branch` *(destructive)* - Protect a repository branch (set push/merge/unprotect access levels)
24. `unprotect_branch` *(destructive)* - Remove protection from a previously protected branch
25. `update_default_branch` *(destructive)* - Change the default branch of a project
26. `get_merge_request` *(read-only)* - Get details of a merge request (mergeRequestIid or branchName required). Set include_summaries=true for deployment/commit/approval summaries
27. `get_merge_request_diffs` *(read-only)* - Get the changes/diffs of a merge request (mergeRequestIid or branchName required)
28. `list_merge_request_changed_files` *(read-only)* - List changed file paths in a merge request without diff content (mergeRequestIid or branchName required)
29. `list_merge_request_diffs` *(read-only)* - List merge request diffs with pagination (mergeRequestIid or branchName required)
30. `get_merge_request_file_diff` *(read-only)* - Get diffs for specific files from a merge request (mergeRequestIid or branchName required)
31. `list_merge_request_versions` *(read-only)* - List all versions of a merge request
32. `get_merge_request_version` *(read-only)* - Get a specific version of a merge request
33. `get_branch_diffs` *(read-only)* - Get diffs between two branches or commits
34. `update_merge_request`  - Update a merge request (mergeRequestIid or branchName required)
35. `create_note`  - Create a new note (comment) to an issue or merge request
36. `create_merge_request_thread`  - Create a new thread on a merge request
37. `resolve_merge_request_thread`  - Resolve a thread on a merge request
38. `mr_discussions` *(read-only)* - List discussion items for a merge request
39. `delete_merge_request_discussion_note` *(destructive)* - Delete a discussion note on a merge request
40. `update_merge_request_discussion_note`  - Update a discussion note on a merge request
41. `create_merge_request_discussion_note`  - Add a new discussion note to an existing merge request thread
42. `create_merge_request_note`  - Add a new note to a merge request
43. `delete_merge_request_note` *(destructive)* - Delete an existing merge request note
44. `get_merge_request_note` *(read-only)* - Get a specific note for a merge request
45. `get_merge_request_notes` *(read-only)* - List notes for a merge request
46. `update_merge_request_note`  - Modify an existing merge request note
47. `get_draft_note` *(read-only)* - Get a single draft note from a merge request
48. `list_draft_notes` *(read-only)* - List draft notes for a merge request
49. `create_draft_note`  - Create a draft note for a merge request
50. `update_draft_note`  - Update an existing draft note
51. `delete_draft_note` *(destructive)* - Delete a draft note
52. `publish_draft_note`  - Publish a single draft note
53. `bulk_publish_draft_notes`  - Publish all draft notes for a merge request
54. `list_merge_request_emoji_reactions` *(read-only)* - List all emoji reactions on a merge request
55. `list_merge_request_note_emoji_reactions` *(read-only)* - List all emoji reactions on a merge request note. Pass discussion_id for discussion thread replies.
56. `create_merge_request_emoji_reaction`  - Add an emoji reaction to a merge request (e.g. thumbsup, rocket, eyes)
57. `delete_merge_request_emoji_reaction` *(destructive)* - Remove an emoji reaction from a merge request
58. `create_merge_request_note_emoji_reaction`  - Add an emoji reaction to a merge request note. Pass discussion_id for discussion thread replies.
59. `delete_merge_request_note_emoji_reaction` *(destructive)* - Remove an emoji reaction from a merge request note. Pass discussion_id for discussion thread replies.
60. `update_issue_note`  - Modify an existing issue thread note
61. `create_issue_note`  - Add a note to an issue, optionally replying to a discussion thread
62. `list_issue_emoji_reactions` *(read-only)* - List all emoji reactions on an issue
63. `list_issue_note_emoji_reactions` *(read-only)* - List all emoji reactions on an issue note. Pass discussion_id for discussion thread replies.
64. `create_issue_emoji_reaction`  - Add an emoji reaction to an issue (e.g. thumbsup, rocket, eyes)
65. `delete_issue_emoji_reaction` *(destructive)* - Remove an emoji reaction from an issue
66. `create_issue_note_emoji_reaction`  - Add an emoji reaction to an issue note. Pass discussion_id for discussion thread replies.
67. `delete_issue_note_emoji_reaction` *(destructive)* - Remove an emoji reaction from an issue note. Pass discussion_id for discussion thread replies.
68. `list_issues` *(read-only)* - List issues (default: created by current user; use scope='all' for all)
69. `my_issues` *(read-only)* - List issues assigned to the authenticated user
70. `get_issue` *(read-only)* - Get details of a specific issue. Returns a slim milestone by default; set full_response=true for the complete milestone object
71. `update_issue`  - Update an issue. Returns a slim confirmation by default; set full_response=true for the complete updated issue object
72. `update_issue_description_patch`  - Apply a patch (search/replace or unified diff) to an issue description. Reduces token usage by allowing small changes without sending the full description. Supports dry_run to preview changes and create_note to summarize updates.
73. `delete_issue` *(destructive)* - Delete an issue
74. `list_todos` *(read-only)* - List GitLab to-do items for the current user
75. `mark_todo_done`  - Mark a GitLab to-do item as done
76. `mark_all_todos_done`  - Mark all pending GitLab to-do items as done for the current user
77. `list_issue_links` *(read-only)* - List all issue links for a specific issue
78. `list_issue_discussions` *(read-only)* - List discussions for an issue
79. `get_issue_link` *(read-only)* - Get a specific issue link
80. `create_issue_link`  - Create an issue link between two issues
81. `delete_issue_link` *(destructive)* - Delete an issue link
82. `list_namespaces` *(read-only)* - List all namespaces (users and groups) available to the current user. Filter by kind='group' for groups only.
83. `get_namespace` *(read-only)* - Get details of a namespace (user or group) by ID or path. Groups are namespaces with kind='group'.
84. `verify_namespace` *(read-only)* - Verify if a namespace path exists. Use parent_id to scope the check to a specific parent namespace — required for nested namespaces where the same path may exist under different parents.
85. `get_project` *(read-only)* - Get details of a specific project
86. `list_projects` *(read-only)* - List projects accessible by the current user
87. `update_project`  - Update project settings such as description, visibility, default branch, and feature access levels
88. `list_project_members` *(read-only)* - List members of a GitLab project
89. `list_labels` *(read-only)* - List labels for a project
90. `get_label` *(read-only)* - Get a single label from a project
91. `create_label`  - Create a new label in a project
92. `update_label`  - Update an existing label in a project
93. `delete_label` *(destructive)* - Delete a label from a project
94. `list_group_projects` *(read-only)* - List projects in a group
95. `list_wiki_pages` *(read-only)* - List wiki pages in a project
96. `get_wiki_page` *(read-only)* - Get details of a specific wiki page
97. `create_wiki_page`  - Create a wiki page in a project
98. `update_wiki_page`  - Update a wiki page in a project
99. `delete_wiki_page` *(destructive)* - Delete a wiki page from a project
100. `list_group_wiki_pages` *(read-only)* - List wiki pages in a group
101. `get_group_wiki_page` *(read-only)* - Get details of a specific group wiki page
102. `create_group_wiki_page`  - Create a wiki page in a group
103. `update_group_wiki_page`  - Update a wiki page in a group
104. `delete_group_wiki_page` *(destructive)* - Delete a wiki page from a group
105. `get_repository_tree` *(read-only)* - List files and directories in a repository
106. `list_pipelines` *(read-only)* - List pipelines with filtering options
107. `get_pipeline` *(read-only)* - Get details of a specific pipeline
108. `list_deployments` *(read-only)* - List deployments with filtering options
109. `get_deployment` *(read-only)* - Get details of a specific deployment
110. `list_environments` *(read-only)* - List environments in a project
111. `get_environment` *(read-only)* - Get details of a specific environment
112. `list_pipeline_jobs` *(read-only)* - List all jobs in a specific pipeline
113. `list_pipeline_trigger_jobs` *(read-only)* - List trigger jobs (bridges) in a pipeline
114. `get_pipeline_job` *(read-only)* - Get details of a GitLab pipeline job number
115. `get_pipeline_job_output` *(read-only)* - Get the output/trace of a pipeline job with optional pagination
116. `validate_ci_lint` *(read-only)* - Validate provided GitLab CI/CD YAML content for a project
117. `validate_project_ci_lint` *(read-only)* - Validate an existing .gitlab-ci.yml configuration for a project
118. `list_ci_catalog_resources` *(read-only)* - List GitLab CI/CD Catalog resources/components visible to the user
119. `get_ci_catalog_resource` *(read-only)* - Get details for a GitLab CI/CD Catalog resource, including versions and components
120. `create_pipeline`  - Create a new pipeline for a branch or tag
121. `retry_pipeline`  - Retry a failed or canceled pipeline
122. `cancel_pipeline`  - Cancel a running pipeline
123. `play_pipeline_job`  - Run a manual pipeline job
124. `retry_pipeline_job`  - Retry a failed or canceled pipeline job
125. `cancel_pipeline_job`  - Cancel a running pipeline job
126. `list_job_artifacts` *(read-only)* - List artifact files in a job's archive
127. `download_job_artifacts` *(read-only)* - Download job artifact archive (zip) and save to a local path
128. `get_job_artifact_file` *(read-only)* - Get content of a single file from a job's artifacts
129. `list_merge_requests` *(read-only)* - List merge requests (without project_id: user's MRs; with project_id: project MRs)
130. `list_milestones` *(read-only)* - List milestones with filtering options
131. `get_milestone` *(read-only)* - Get details of a specific milestone
132. `create_milestone`  - Create a new milestone
133. `edit_milestone`  - Edit an existing milestone
134. `delete_milestone` *(destructive)* - Delete a milestone
135. `get_milestone_issue` *(read-only)* - Get issues associated with a specific milestone
136. `get_milestone_merge_requests` *(read-only)* - Get merge requests associated with a specific milestone
137. `promote_milestone`  - Promote a milestone to the next stage
138. `get_milestone_burndown_events` *(read-only)* - Get burndown events for a specific milestone
139. `list_group_milestones` *(read-only)* - List group milestones with filtering options
140. `get_group_milestone` *(read-only)* - Get details of a specific group milestone
141. `create_group_milestone`  - Create a new group milestone
142. `edit_group_milestone`  - Edit an existing group milestone
143. `delete_group_milestone` *(destructive)* - Delete a group milestone
144. `get_group_milestone_issue` *(read-only)* - Get issues associated with a specific group milestone
145. `get_group_milestone_merge_requests` *(read-only)* - Get merge requests associated with a specific group milestone
146. `get_group_milestone_burndown_events` *(read-only)* - Get burndown events for a specific group milestone
147. `get_users` *(read-only)* - Get GitLab user details by usernames
148. `get_user` *(read-only)* - Get user details by ID
149. `whoami` *(read-only)* - Get current authenticated user details
150. `list_commits` *(read-only)* - List repository commits with filtering options
151. `get_commit` *(read-only)* - Get details of a specific commit
152. `get_commit_diff` *(read-only)* - Get changes/diffs of a specific commit
153. `get_file_blame` *(read-only)* - Get git blame for a file at a given ref. Each entry maps a contiguous range of source lines to the commit that last changed them (id, author, authored_date, message). Use range_start/range_end to limit blame to specific lines.
154. `list_commit_statuses` *(read-only)* - List statuses for a commit
155. `create_commit_status`  - Create or update the status of a commit
156. `list_group_iterations` *(read-only)* - List group iterations with filtering options
157. `upload_markdown`  - Upload a file for use in markdown content
158. `download_attachment` *(read-only)* - Download an uploaded file from a project (images returned as base64; use local_path to save to disk)
159. `health_check` *(read-only)* - Verify server status and authentication
160. `list_events` *(read-only)* - List events for the authenticated user (before/after: YYYY-MM-DD)
161. `get_project_events` *(read-only)* - List events for a project (before/after: YYYY-MM-DD)
162. `list_releases` *(read-only)* - List all releases for a project
163. `get_release` *(read-only)* - Get a release by tag name
164. `create_release`  - Create a new release
165. `update_release`  - Update an existing release
166. `delete_release` *(destructive)* - Delete a release (does not delete the tag)
167. `create_release_evidence`  - Create release evidence (Premium/Ultimate)
168. `download_release_asset` *(read-only)* - Download a release asset file by direct asset path
169. `list_tags` *(read-only)* - List repository tags for a project
170. `get_tag` *(read-only)* - Get a repository tag by name
171. `create_tag`  - Create a new repository tag
172. `delete_tag` *(destructive)* - Delete a repository tag
173. `get_tag_signature` *(read-only)* - Get the X.509 signature of a signed tag (404 if unsigned)
174. `get_work_item` *(read-only)* - Get a work item with full details including status, hierarchy, type, and widgets
175. `list_work_items` *(read-only)* - List work items with filters (type, state, search, assignees, labels)
176. `create_work_item`  - Create a work item (issue, task, incident, epic, etc.) with full field support
177. `update_work_item`  - Update a work item (title, description, labels, assignees, state, parent, custom fields, etc.)
178. `convert_work_item_type`  - Convert a work item to a different type
179. `list_work_item_statuses` *(read-only)* - List available statuses for a work item type (Premium/Ultimate)
180. `list_custom_field_definitions` *(read-only)* - List custom field definitions for a work item type
181. `move_work_item`  - Move a work item to a different project
182. `list_work_item_notes` *(read-only)* - List notes and discussions on a work item
183. `create_work_item_note`  - Add a note to a work item (supports Markdown, internal notes, threads)
184. `list_work_item_emoji_reactions` *(read-only)* - List all emoji reactions on a work item
185. `list_work_item_note_emoji_reactions` *(read-only)* - List all emoji reactions on a work item note (comment, thread, or thread reply)
186. `create_work_item_emoji_reaction`  - Add an emoji reaction to a work item (e.g. thumbsup, rocket, eyes)
187. `delete_work_item_emoji_reaction` *(destructive)* - Remove an emoji reaction from a work item
188. `create_work_item_note_emoji_reaction`  - Add an emoji reaction to a work item note (comment, thread, or thread reply)
189. `delete_work_item_note_emoji_reaction` *(destructive)* - Remove an emoji reaction from a work item note (comment, thread, or thread reply)
190. `get_timeline_events` *(read-only)* - List timeline events for an incident
191. `create_timeline_event`  - Create a timeline event on an incident
192. `list_webhooks` *(read-only)* - List webhooks for a project or group
193. `list_webhook_events` *(read-only)* - List recent webhook events (past 7 days)
194. `get_webhook_event` *(read-only)* - Get full details of a specific webhook event
195. `search_code` *(read-only)* - Search for code across all projects (requires advanced search or Zoekt)
196. `search_project_code` *(read-only)* - Search for code within a specific project (requires advanced search or Zoekt)
197. `search_group_code` *(read-only)* - Search for code within a specific group (requires advanced search or Zoekt)
198. `list_project_variables` *(read-only)* - List CI/CD variables for a project
199. `get_project_variable` *(read-only)* - Get a single CI/CD variable from a project
200. `create_project_variable`  - Create a CI/CD variable for a project
201. `update_project_variable`  - Update an existing CI/CD variable in a project
202. `delete_project_variable` *(destructive)* - Delete a CI/CD variable from a project
203. `list_group_variables` *(read-only)* - List CI/CD variables for a group
204. `get_group_variable` *(read-only)* - Get a single CI/CD variable from a group
205. `create_group_variable`  - Create a CI/CD variable for a group
206. `update_group_variable`  - Update an existing CI/CD variable in a group
207. `delete_group_variable` *(destructive)* - Delete a CI/CD variable from a group
208. `get_dependency_proxy_settings` *(read-only)* - Get dependency proxy settings for a group
209. `update_dependency_proxy_settings`  - Update dependency proxy settings for a group (enable/disable, credentials for authenticated Docker Hub pulls)
210. `list_dependency_proxy_blobs` *(read-only)* - List cached dependency proxy blobs for a group
211. `purge_dependency_proxy_cache` *(destructive)* - Schedule purge of all cached dependency proxy blobs for a group
212. `list_runners` *(read-only)* - List GitLab CI/CD runners. Can be scoped to all runners, a specific project, or a specific group. Supports filtering by type, status, and tags.
213. `get_runner` *(read-only)* - Get detailed information about a specific GitLab CI/CD runner including platform, architecture, tags, version, IP address, and last contact time.
214. `pause_runner`  - Pause (disable) a GitLab CI/CD runner so it will not pick up new jobs. The runner remains registered but inactive.
215. `resume_runner`  - Resume (re-enable) a previously paused GitLab CI/CD runner so it can start picking up jobs again.
216. `delete_runner` *(destructive)* - Permanently delete a GitLab CI/CD runner. This action is irreversible — the runner must be re-registered to be used again.
217. `list_runner_jobs` *(read-only)* - List jobs that have been executed by a specific GitLab CI/CD runner. Useful for auditing runner activity and debugging job failures.
218. `discover_tools` *(read-only)* - Discover and activate additional tool categories for this session. Available categories: merge_requests, issues, repositories, branches, projects, labels, ci, groups, pipelines, milestones, wiki, releases, tags, users, workitems, webhooks, search, variables, dependency_proxy, runners. Already-active categories are listed in the response.

<!-- TOOLS-END -->

</details>

---

## How Custom Tools Are Built

This server is designed to be easily extensible. Every tool follows a strict **4-file pipeline**:

```
1. schemas.ts                 Define input parameters using Zod (descriptions guide the LLM)
       │
       ▼
2. tools/registry.ts          Register tool in allTools, assign toolset, and categorize permissions
       │
       ▼
3. index.ts                   Implement GitLab API fetch logic and route in handleToolCall()
       │
       ▼
4. scripts/generate-tool-docs.ts   Add toolset metadata for documentation generation & type checking
```

### Verification Checklist
When implementing or modifying tools:
1. Run `npm run build` to verify TypeScript compilation.
2. Run `npm run test:schema` to validate input parsing.
3. Run `npm run check:skill-sync` to ensure agent skill definitions match `tools/registry.ts`.

---

## Development & Verification

```bash
# Install local dependencies
npm install

# Compile TypeScript
npm run build

# Run schema and validation tests
npm run test:schema

# Verify skill definitions stay synchronized with the registry
npm run check:skill-sync

# Verify runtime dependencies
npm run check:runtime-deps

# Run full test suite
npm test
```

---

## Author & Acknowledgments

- **Maintained by**: **aryannverse** ([@aryannverse](https://github.com/aryannverse))
- **Based on**: `@zereight/mcp-gitlab`
- **License**: [MIT License](./LICENSE)
