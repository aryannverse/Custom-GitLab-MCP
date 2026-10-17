# Runner Operations

> **Opt-in toolset**: Enable with `GITLAB_TOOLSETS=runners`

## List & Inspect

```
list_runners             -> list runners with filters (type, status, tags, project_id, group_id)
get_runner               -> detailed runner info (architecture, platform, tags, version, contact)
list_runner_jobs         -> jobs executed by a specific runner
```

## Manage & Control

```
pause_runner             -> pause (disable) a runner from picking up new jobs
resume_runner            -> resume (re-enable) a paused runner
delete_runner            -> permanently delete a decommissioned runner (destructive)
```

## Common Patterns

### Audit Runner Health
1. `list_runners` with `status: "offline"` or `status: "stale"` to identify unhealthy runners.
2. `get_runner` with runner_id to inspect details (e.g. IP address, last contact, tag list).
3. `list_runner_jobs` to check if recent builds failed on that specific runner.

### Maintenance Workflow
1. `pause_runner` before host maintenance to drain active work without taking unexpected jobs.
2. Perform updates or maintenance.
3. `resume_runner` to return the runner to service.
