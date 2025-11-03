---
name: bitbucket-devops
description: Comprehensive Bitbucket DevOps automation using direct Node.js API calls. Manage pipelines, repositories, pull requests, and CI/CD workflows. Use this skill when the user asks to check pipeline status, manage PRs, find failing pipelines, download logs, trigger builds, or analyze pipeline failures. No MCP approval prompts required - uses Bash tool with node commands.
allowed-tools: Bash, Read, Write, Grep, Glob
---

# Bitbucket DevOps Skill

This skill provides comprehensive Bitbucket DevOps automation using direct Node.js API calls via the Bash tool. Built on the [bitbucket-mcp](https://github.com/Apra-Labs/bitbucket-mcp) client library.

**Key Advantage:** Uses direct Node.js calls (auto-approved) instead of MCP tools, eliminating the approval prompts issue from [GitHub Issue #10801](https://github.com/anthropics/claude-code/issues/10801).

## The DevOps REPL Advantage

Traditional pipeline debugging is slow: push code → wait → fail → investigate logs → fix → repeat (hours per cycle).

This skill enables a **REPL-like experience for DevOps**: Claude observes pipelines in real-time, analyzes failures instantly, suggests precise fixes, and iterates with you until builds pass - reducing debugging cycles from hours to minutes.

**The Loop:**
1. **Read**: Monitor pipeline execution and capture failures
2. **Eval**: AI analyzes logs and identifies root cause
3. **Print**: Claude presents findings and suggests fixes
4. **Loop**: Apply fix, trigger build, repeat until green ✅

This transforms DevOps from slow batch processing into interactive, conversational development.

## Prerequisites

This skill uses the Bash tool (auto-approved in Claude Code) to run Node.js commands. Required:
- Node.js (v18+)
- Git (for submodule management)

**Note:** No MCP server required - bitbucket-mcp is used as a library via git submodule.

## Configuration

The skill directory is located at: `~/.claude/skills/bitbucket-devops/`

Credentials are loaded with priority (first found wins):
1. **Project level**: `./credentials.json` or `./.bitbucket-credentials` (current working directory)
2. **User level**: `~/.bitbucket-credentials` (home directory)
3. **Skill level**: `~/.claude/skills/bitbucket-devops/credentials.json`

### Credential Format

```json
{
  "url": "https://api.bitbucket.org/2.0",
  "workspace": "your-workspace",
  "username": "your-email@example.com",
  "password": "your-app-password"
}
```

## Helper Functions

The skill provides high-level helper functions in `lib/helpers.js`:

- **get-latest-failed** - Get the most recent failed pipeline
- **get-latest** - Get the most recent pipeline (any status)
- **get-by-number** - Find pipeline by build number
- **get-failed-steps** - Get all failed steps from a pipeline
- **download-failed-logs** - Download logs from all failed steps
- **get-info** - Get formatted pipeline information

## Usage Patterns

### Pattern 1: Find Latest Failing Pipeline

**User Requests:**
- "What's the latest failing pipeline?"
- "Show me the most recent build failure"
- "Find the last failed pipeline"

**Steps:**
1. Use helper function to get latest failed pipeline
2. Extract and present key information

**Command:**
```bash
node ~/.claude/skills/bitbucket-devops/lib/helpers.js \
  get-latest-failed "workspace" "repo"
```

**Example Output:**
```json
{
  "build_number": 123,
  "state": { "name": "FAILED" },
  "target": {
    "ref_name": "main",
    "commit": {
      "hash": "abc123def",
      "message": "Fix bug in deployment"
    }
  },
  "created_on": "2025-11-02T10:30:00Z",
  "uuid": "{pipeline-uuid}"
}
```

**Present to User:**
```
Latest failed pipeline:
- Pipeline #123
- Branch: main
- Commit: abc123d - "Fix bug in deployment"
- Status: FAILED
- Started: 2025-11-02 10:30 UTC
```

### Pattern 2: Inspect Specific Pipeline by Number

**User Requests:**
- "Show me pipeline #34"
- "Get details for build 34"
- "What happened in pipeline 34?"

**Steps:**
1. Use helper to get pipeline by build number (auto-finds UUID)
2. Get detailed info including all steps
3. Present formatted output

**Commands:**
```bash
# Get pipeline by number
node ~/.claude/skills/bitbucket-devops/lib/helpers.js \
  get-by-number "workspace" "repo" 34

# Get full info with steps
node ~/.claude/skills/bitbucket-devops/lib/helpers.js \
  get-info "workspace" "repo" "{pipeline-uuid}"
```

**Example Output:**
```
Pipeline #34 Details:
- Status: FAILED
- Branch: main
- Commit: abc123d
- Duration: 5m 30s

Steps:
1. Build (step 1/5) - ✅ SUCCESSFUL (1m 20s)
2. Test (step 2/5) - ✅ SUCCESSFUL (2m 15s)
3. Deploy (step 3/5) - ❌ FAILED (30s)
4. Integration Tests (step 4/5) - ⏭️ SKIPPED
5. Cleanup (step 5/5) - ⏭️ SKIPPED
```

### Pattern 3: Identify Which Steps Failed

**User Requests:**
- "Which steps failed?"
- "What part of the build broke?"

**Steps:**
1. Get failed steps using helper function
2. Display step names, status, and duration

**Command:**
```bash
node ~/.claude/skills/bitbucket-devops/lib/helpers.js \
  get-failed-steps "workspace" "repo" "{pipeline-uuid}"
```

**Example Output:**
```
Failed Steps in Pipeline #34:

1. Deploy (step 3/5)
   - Status: FAILED
   - Duration: 30s

2. Integration Tests (step 4/5)
   - Status: ERROR
   - Duration: 2m 15s
```

### Pattern 4: Download Failing Steps Logs

**User Requests:**
- "Get logs for failed steps"
- "Download the logs"
- "Show me what went wrong"

**Steps:**
1. Use helper to download all failed step logs
2. Logs are saved to `.pipeline-logs/` in current directory
3. Present summary and file locations

**Command:**
```bash
node ~/.claude/skills/bitbucket-devops/lib/helpers.js \
  download-failed-logs "workspace" "repo" "{pipeline-uuid}" 34
```

**Output:**
```json
[
  {
    "stepName": "Deploy",
    "stepUuid": "{step-uuid}",
    "logFilePath": "/path/to/project/.pipeline-logs/pipeline-34-Deploy.log",
    "size": 12450,
    "status": "FAILED"
  }
]
```

**Present to User:**
```
Downloaded logs for 2 failed steps:

1. Deploy
   - Saved to: .pipeline-logs/pipeline-34-Deploy.log
   - Size: 12.4 KB

2. Integration_Tests
   - Saved to: .pipeline-logs/pipeline-34-Integration_Tests.log
   - Size: 45.2 KB
```

**Important:** Check log file size before displaying. If > 50KB, show summary only:
```bash
# Check file size
ls -lh .pipeline-logs/pipeline-34-Deploy.log

# Show last 100 lines (most relevant errors)
tail -n 100 .pipeline-logs/pipeline-34-Deploy.log

# Or search for errors
grep -i "error\|failed\|exception" .pipeline-logs/pipeline-34-Deploy.log
```

### Pattern 5: Download Specific Step Logs

**User Requests:**
- "Get logs from the Deploy step"
- "Download logs from step 3"
- "Show me Deploy step logs"

**Steps:**
1. Get all pipeline steps
2. Find step by name or position
3. Download logs using CLI

**Commands:**
```bash
# Get all steps
node ~/.claude/skills/bitbucket-devops/bitbucket-mcp/dist/index-cli.js \
  get-pipeline-steps "workspace" "repo" "{pipeline-uuid}"

# Find step UUID (use jq or parse JSON in bash)
# Then download logs
node ~/.claude/skills/bitbucket-devops/bitbucket-mcp/dist/index-cli.js \
  get-step-logs "workspace" "repo" "{pipeline-uuid}" "{step-uuid}" \
  > .pipeline-logs/deploy-step.log
```

### Pattern 6: Analyze Large Logs

**User Requests:**
- "The log is too large"
- "Summarize the errors"
- Context window concerns

**Steps:**
1. Check log file size
2. Extract relevant portions (errors, warnings)
3. Present summary

**Commands:**
```bash
# Check size
size=$(wc -c < .pipeline-logs/pipeline-34-Deploy.log)
echo "Log size: $size bytes"

# Extract errors only
grep -i "error\|fatal\|exception" .pipeline-logs/pipeline-34-Deploy.log > .pipeline-logs/errors-only.txt

# Show last 200 lines (where failures typically occur)
tail -n 200 .pipeline-logs/pipeline-34-Deploy.log

# Count error types
grep -i "error" .pipeline-logs/pipeline-34-Deploy.log | sort | uniq -c | sort -nr
```

### Pattern 7: List Available Pipeline Types

**User Requests:**
- "What pipelines can I run?"
- "What can I trigger on this branch?"

**Steps:**
1. Get branching model
2. Parse available pipeline configurations
3. List available types

**Command:**
```bash
node ~/.claude/skills/bitbucket-devops/bitbucket-mcp/dist/index-cli.js \
  get-branching-model "workspace" "repo"
```

**Response Parsing:**
```json
{
  "development": { "name": "develop" },
  "production": { "name": "main" },
  "branch_types": [
    { "kind": "feature", "prefix": "feature/" },
    { "kind": "bugfix", "prefix": "bugfix/" },
    { "kind": "hotfix", "prefix": "hotfix/" }
  ]
}
```

**Present to User:**
```
Available Pipeline Configurations:

Branch Types:
- Development: develop
- Production: main
- Feature branches: feature/*
- Bugfix branches: bugfix/*
- Hotfix branches: hotfix/*

Pipeline Types (from bitbucket-pipelines.yml):
- default (runs on all branches)
- branches.<branch-name> (branch-specific)
- custom.<pipeline-name> (custom pipelines)
- pull-requests (PR pipelines)
```

### Pattern 8: Trigger Pipeline Run

**User Requests:**
- "Run the pipeline on main"
- "Start pipeline X on branch Y"
- "Trigger deploy-production"

**Steps:**
1. Confirm with user: branch, pipeline type, variables
2. Use CLI to trigger pipeline
3. Display result with URL and build number

**Command:**
```bash
# Trigger default pipeline
node ~/.claude/skills/bitbucket-devops/bitbucket-mcp/dist/index-cli.js \
  run-pipeline "workspace" "repo" "main"

# Trigger custom pipeline
node ~/.claude/skills/bitbucket-devops/bitbucket-mcp/dist/index-cli.js \
  run-pipeline "workspace" "repo" "main" "deploy-production"
```

**Example Output:**
```
Triggering Pipeline:
- Branch: main
- Pipeline: deploy-production
- Variables:
  - ENVIRONMENT=prod
  - DRY_RUN=false

✓ Pipeline started: #456
URL: https://bitbucket.org/workspace/repo/pipelines/results/456
Status: IN_PROGRESS
```

## Log Storage

Logs are downloaded to the **current project directory**:

```
your-project/
├── .pipeline-logs/           ← Created automatically
│   ├── pipeline-123-Deploy.log
│   ├── pipeline-123-Test.log
│   └── errors-only.txt
├── src/
└── ...
```

**Important:** Add `.pipeline-logs/` to your project's `.gitignore`.

## Error Handling

Common issues and solutions:

| Error | Cause | Solution |
|-------|-------|----------|
| "Pipeline not found" | Build number too old | Increase search limit or use recent pipelines |
| "Logs unavailable" | Pipeline still running | Wait for completion |
| "No credential file found" | Missing credentials.json | Create from template |
| "Node.js not found" | Node not installed | Install Node.js v18+ |
| "Submodule not initialized" | Git submodule missing | Run install script |

## Best Practices

1. **Always confirm workspace/repo**: Auto-detect from git or ask user:
   ```bash
   git config --get remote.origin.url
   # Parse: git@bitbucket.org:workspace/repo.git
   ```

2. **Check pipeline status before logs**: Don't request logs for running pipelines

3. **Limit initial results**: Start with 10 recent pipelines, increase if needed

4. **Smart log filtering**: Use grep to find errors first:
   ```bash
   grep -n "ERROR\|FATAL\|Exception" logfile.log
   ```

5. **Cache results**: Store JSON responses in variables to avoid redundant calls

6. **Use helper functions**: Prefer helpers.js functions for common operations

## Workspace Detection

Auto-detect repository information:

```bash
# Get git remote URL
git_url=$(git config --get remote.origin.url 2>/dev/null)

# Parse workspace and repo from: git@bitbucket.org:workspace/repo.git
if [[ "$git_url" =~ bitbucket.org[:/]([^/]+)/([^/.]+) ]]; then
  workspace="${BASH_REMATCH[1]}"
  repo_slug="${BASH_REMATCH[2]}"
  echo "Detected: $workspace/$repo_slug"
fi
```

## Configuration Variables

Default values (can be adjusted per request):

- **Max Pipeline Limit**: 50 (for searching by build number)
- **Recent Pipeline Limit**: 10 (for listing)
- **Log Directory**: `.pipeline-logs/` (relative to cwd)
- **Large Log Threshold**: 50KB (when to summarize)

## Performance Notes

- **No approval prompts**: Bash tool with node commands is auto-approved
- **Direct API calls**: No MCP protocol overhead
- **Credential caching**: Loaded once per invocation
- **Bitbucket rate limits**: 60 requests/hour per user (standard tier)

## Troubleshooting

### Helper Functions Not Working

```bash
# Check if skill directory exists
ls -la ~/.claude/skills/bitbucket-devops/

# Check if submodule is initialized
ls -la ~/.claude/skills/bitbucket-devops/bitbucket-mcp/dist/

# If missing, run install script
cd ~/.claude/skills/bitbucket-devops/
bash install.sh
```

### UUID Encoding

Bitbucket UUIDs contain curly braces `{uuid}`. The CLI handles encoding automatically.

### JSON Parsing in Bash

Use `node -e` or `jq` for parsing JSON responses:

```bash
# With jq
result=$(node ~/.claude/skills/bitbucket-devops/lib/helpers.js get-latest "workspace" "repo")
build_number=$(echo "$result" | jq -r '.build_number')

# With node
build_number=$(node -e "const data = $(cat result.json); console.log(data.build_number)")
```

## Credits

This skill is built on [bitbucket-mcp](https://github.com/Apra-Labs/bitbucket-mcp) by Apra Labs, forked from [@MatanYemini's original work](https://github.com/MatanYemini/bitbucket-mcp).

**Architecture:** Uses bitbucket-mcp as a library (git submodule), NOT as an MCP server. This approach eliminates approval prompts while maintaining full API functionality.

**License**: CC BY 4.0
**Maintained by**: [Apra Labs](https://github.com/Apra-Labs)
