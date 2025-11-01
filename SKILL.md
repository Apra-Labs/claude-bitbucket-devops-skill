---
name: bitbucket-pipeline-debug
description: Debug and manage Bitbucket pipelines using the bitbucket-mcp server. Use this skill when the user asks to check pipeline status, find failing pipelines, download logs, trigger builds, or analyze pipeline failures. Supports finding latest failures, inspecting specific pipeline runs, downloading step logs, and triggering new pipeline runs.
allowed-tools: mcp__bitbucket-mcp__*
---

# Bitbucket Pipeline Debug Skill

This skill helps debug and manage Bitbucket CI/CD pipelines using the [bitbucket-mcp](https://github.com/MatanYemini/bitbucket-mcp) MCP server.

## Configuration

This skill requires the bitbucket-mcp server to be configured in your VSCode settings.

See the [Installation Guide](./README.md#installation) for setup instructions.

## Usage Patterns

### 1. Find Latest Failing Pipeline

**User Request Examples:**
- "What's the latest failing pipeline?"
- "Show me the most recent build failure"
- "Find the last failed pipeline"

**Steps:**
1. Use `mcp__bitbucket-mcp__listPipelineRuns` with `status: "FAILED"` and `limit: 1`
2. Extract pipeline number, branch, commit info
3. Present summary with pipeline URL

**Example Output:**
```
Latest failed pipeline:
- Pipeline #123
- Branch: main
- Commit: abc123 - "Fix bug in deployment"
- Status: FAILED
- Started: 2025-11-01 10:30 UTC
```

### 2. Inspect Specific Pipeline by Number

**User Request Examples:**
- "Show me pipeline #34"
- "Get details for build 34"
- "What happened in pipeline 34?"

**Steps:**
1. Use `mcp__bitbucket-mcp__getPipelineRun` with the pipeline UUID
   - Note: Convert pipeline number to UUID by listing recent pipelines first
2. Display pipeline details: status, branch, trigger, duration
3. List all steps with their status

**Example Output:**
```
Pipeline #34 Details:
- Status: FAILED
- Branch: feature/new-api
- Triggered by: push
- Duration: 45 minutes
- Steps:
  ✓ Build (2m 30s)
  ✓ Test (5m 15s)
  ✗ Deploy (failed after 30s)
```

### 3. Identify Failing Steps

**User Request Examples:**
- "Which steps failed in pipeline #34?"
- "Show me the failed steps"
- "What part of the build broke?"

**Steps:**
1. Use `mcp__bitbucket-mcp__getPipelineSteps` to get all steps
2. Filter steps where `state: "FAILED"` or `state: "ERROR"`
3. Display step names, duration, and exit codes

**Example Output:**
```
Failed Steps in Pipeline #34:
1. Deploy (step 3/5)
   - Status: FAILED
   - Duration: 30s
   - Exit Code: 1

2. Integration Tests (step 4/5)
   - Status: ERROR
   - Duration: 2m 15s
   - Exit Code: 137 (OOM)
```

### 4. Download Failing Steps Logs

**User Request Examples:**
- "Get logs for the failed steps"
- "Download logs from the failures"
- "Show me what went wrong"

**Steps:**
1. Use `mcp__bitbucket-mcp__getPipelineSteps` to identify failed steps
2. For each failed step, use `mcp__bitbucket-mcp__getPipelineStepLogs`
3. Save logs to `.pipeline-logs/` in the current project directory
4. Present log summary and file locations

**Important:** Check log size before displaying. If > 10KB, slice into chunks.

### 5. Download Specific Step Logs

**User Request Examples:**
- "Get logs for the Deploy step"
- "Show me the Test step output"
- "Download logs from step 3"

**Steps:**
1. Use `mcp__bitbucket-mcp__getPipelineSteps` to list all steps
2. Match user's step name/number to step UUID
3. Use `mcp__bitbucket-mcp__getPipelineStepLogs` with step UUID
4. Display or save logs to project's `.pipeline-logs/` directory

### 6. Slice Large Logs

**When to Use:**
- Log file > 10,000 characters
- User asks to "analyze logs in chunks"
- Context window concerns

**Steps:**
1. Determine total log size
2. Calculate optimal chunk size (aim for ~5000 chars per chunk)
3. Split log by:
   - Logical boundaries (error messages, section markers)
   - Line breaks
   - Time intervals if timestamps present
4. Present chunks sequentially with navigation
5. Create index showing:
   - Chunk N of M
   - Line range
   - Key markers (errors, warnings)

**Example Output:**
```
Log File: deploy-step.log (45,000 chars)
Split into 9 chunks:

Chunk 1/9 (lines 1-200): Build preparation
Chunk 2/9 (lines 201-400): Dependency installation
...
Chunk 5/9 (lines 801-1000): ⚠️ Contains 3 errors
...
```

### 7. List Pipeline Types for Branch

**User Request Examples:**
- "What pipelines are available?"
- "Show me pipeline types"
- "What can I trigger on this branch?"

**Steps:**
1. Use `mcp__bitbucket-mcp__getRepository` to get repo details
2. Use `mcp__bitbucket-mcp__getRepositoryBranchingModel` to understand branch structure
3. List available pipeline configurations:
   - Default pipeline
   - Custom pipelines (from selector patterns)
   - Branch-specific pipelines

**Note:** Pipeline types are defined in `bitbucket-pipelines.yml`:
- `pipelines.default`
- `pipelines.branches.<branch-name>`
- `pipelines.custom.<custom-name>`
- `pipelines.pull-requests`

### 8. Trigger Pipeline Run

**User Request Examples:**
- "Run the deploy pipeline on main"
- "Trigger a custom build"
- "Start pipeline X on branch Y"

**Steps:**
1. Confirm with user: branch, pipeline type, and any variables
2. Use `mcp__bitbucket-mcp__runPipeline` with:
   - `target.ref_type`: "branch"
   - `target.ref_name`: branch name
   - `target.selector_type`: "default" or "custom"
   - `target.selector_pattern`: custom pipeline name (if custom)
   - `variables`: array of {key, value} if needed
3. Display pipeline URL and build number
4. Optionally: Monitor pipeline status

**Example Output:**
```
Triggering Pipeline:
- Branch: main
- Type: custom
- Pipeline: deploy-production
- Variables:
  - ENVIRONMENT=prod
  - DRY_RUN=false

Pipeline started: #456
URL: https://bitbucket.org/workspace/repo/pipelines/results/456
```

## Helper Functions

### Get Pipeline UUID from Number

Pipeline numbers (like #34) are display IDs. The API requires UUIDs.

**Solution:**
1. List recent pipelines with `mcp__bitbucket-mcp__listPipelineRuns`
2. Each result includes both build number and UUID
3. Match the build number to extract UUID

### Log Storage

Save downloaded logs to the current project's directory:
```
${projectRoot}/.pipeline-logs/
  ├── pipeline-{number}-{step-name}.log
  └── metadata.json
```

**Note:** The `.pipeline-logs/` directory should be added to `.gitignore`.

### Error Handling

Common issues:
- **Pipeline not found**: May be too old, increase limit in list query
- **Logs unavailable**: Pipeline may still be running or logs expired
- **Permission denied**: Check Bitbucket app password scopes
- **MCP approval required**: User must approve each MCP tool call (VSCode limitation)

## Best Practices

1. **Always confirm repo/workspace**: Ask user if ambiguous
2. **Check pipeline status**: Don't try to get logs for running pipelines
3. **Limit initial results**: Start with recent 10 pipelines, fetch more if needed
4. **Smart log slicing**: Look for error markers to find relevant sections
5. **Cache pipeline data**: Avoid redundant API calls within same session

## Workspace Detection

The skill should automatically detect:
- **Workspace**: From MCP server configuration or user request
- **Repository**: From current project git remote or user request
- **Log Directory**: `${projectRoot}/.pipeline-logs/`

## Configuration Variables

Default values (can be overridden per request):

- **Max Log Display**: 10,000 characters
- **Chunk Size**: 5,000 characters
- **Recent Pipeline Limit**: 10
- **Log Directory**: `.pipeline-logs/` (relative to project root)

## Notes

- This skill requires MCP tool approval for each call (VSCode extension limitation)
- For automated/unattended use, wait for [GitHub Issue #10801](https://github.com/anthropics/claude-code/issues/10801) resolution
- Bitbucket API rate limits apply (60 requests/hour per user)
- App passwords require appropriate scopes: `pipeline`, `repository:read`
