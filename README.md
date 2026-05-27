# Bright Security workflows

Four manual (`workflow_dispatch`) workflows that drive Bright DAST scans against this project.

| File | Purpose |
|------|---------|
| `bright-discover-crawler.yml` | Discover entrypoints by crawling a URL |
| `bright-discover-har.yml` | Discover entrypoints from a pre-uploaded HAR file |
| `bright-discover-schema.yml` | Discover entrypoints from a pre-uploaded OpenAPI or Postman spec |
| `bright-scan-ple.yml` | Run a security scan against project-level entrypoints (PLE) that are `new`, `changed`, or `vulnerable`; 49 tests pinned |

## Typical flow.

1. Run one of the three `bright-discover-*` workflows to populate/refresh the project's entrypoints.
2. Run `bright-scan-ple.yml` to execute the security scan against the matching entrypoints.

## Required secret

| Name | Where | Purpose |
|------|-------|---------|
| `BRIGHT_TOKEN` | `Settings > Secrets and variables > Actions > Secrets` | Bright API token (Organization > API Keys) |

## Repo variables (defaults picked up by every workflow)

| Name | Current value | Meaning |
|------|---------------|---------|
| `BRIGHT_PROJECT_ID` | Bright project ID | Used when the `project_id` dispatch input is blank |
| `BRIGHT_AUTH_OBJECT_ID` | Bright auth object ID | Used when the `auth_object_id` dispatch input is blank |
| `BRIGHT_HAR_FILE_ID` | Pre-uploaded HAR file ID | Used by `bright-discover-har.yml` when `file_id` input is blank |
| `BRIGHT_SCHEMA_FILE_ID` | Pre-uploaded schema file ID | Used by `bright-discover-schema.yml` when `file_id` input is blank |

Any variable can be overridden per run by filling the matching dispatch input.

## Units of time

**All `*_timeout_seconds` and `timeout_seconds` inputs are in seconds.** Defaults:

| Workflow | Input | Default | Meaning |
|----------|-------|---------|---------|
| All discovery workflows | `wait_timeout_seconds` | `1800` (30 min) | Max time to wait for discovery to finish before the job exits |
| PLE scan | `timeout_seconds` | `3600` (1 hour) | Max time to wait for the first issue at or above the severity gate before the job exits |

The job exits early if the condition is met sooner (discovery completes or a matching issue is reported).

## PLE scan - test coverage

`bright-scan-ple.yml` pins 49 tests (from the full catalog in `nexploit/src/exploiter/tests` `self.id` constants). Excluded on purpose: `password_reset_poisoning`, `webdav`, `lrrl`, `broken_access_control`.

## Where findings appear

- Bright dashboard URL printed in the `Show scan URL` / `Show discovery URL` step

## Schema FAQ

"Let's say we want to run a PLE scan using a different schema every time, should we grab the new uploaded file ID manually from Bright's dashboard every time we want to run a scan?"
The answer is no - you only need to upload it once, if you keep the same file name when you upload the schema file it should keep its ID, and only update its contents, therefore it is suitable for automation
