# Quick Reference

## Common Commands

### Finding Pipelines

```
"What's the latest failed pipeline?"
"Show me pipeline #34"
"List recent builds"
"Find the last 10 pipelines"
```

### Analyzing Failures

```
"Which steps failed in pipeline #34?"
"Get logs for the failed steps"
"Download logs from the Deploy step"
"Show me the error logs"
```

### Triggering Builds

```
"What pipelines can I run?"
"List available pipeline types"
"Trigger the deploy-production pipeline"
"Run staging deployment with DEBUG=true"
```

### Cross-Project Usage

```
"Show latest failure in workspace/other-repo"
"Get pipeline #45 from my-workspace/my-project"
"List pipelines in customer-x/api-service"
```

## MCP Tools Reference

| Tool | Purpose | Usage |
|------|---------|-------|
| `listPipelineRuns` | Find pipelines | List by status, branch, limit |
| `getPipelineRun` | Get details | Requires pipeline UUID |
| `getPipelineSteps` | List steps | Shows all steps in a pipeline |
| `getPipelineStepLogs` | Get logs | Downloads step output |
| `runPipeline` | Start build | Trigger with branch/vars |
| `getRepository` | Repo info | Get configuration |
| `getRepositoryBranchingModel` | Branch model | Available pipelines |

## Workflow Patterns

### Pattern 1: Quick Health Check
```
1. "What's the latest failure?"
   → Returns: Pipeline #123 failed
2. "What failed?"
   → Returns: Deploy step
3. "Get the logs"
   → Downloads and analyzes
```

### Pattern 2: Deep Investigation
```
1. "Show pipeline #34 details"
   → Full pipeline overview
2. "List all steps"
   → Step-by-step breakdown
3. "Get logs for each failed step"
   → Downloads all failure logs
4. "Analyze the errors"
   → AI analysis of root cause
```

### Pattern 3: Build Trigger
```
1. "What pipelines are available?"
   → Lists custom pipelines
2. "Run deploy-production on main"
   → Confirms parameters
3. → Triggers and returns URL
```

## Log Management

### Automatic Storage

Logs are saved to your project:
```
your-project/
└── .pipeline-logs/
    ├── pipeline-123-deploy.log
    ├── pipeline-123-test.log
    └── metadata.json
```

### Large Log Handling

Files >10KB are automatically chunked:
```
"Get logs from pipeline #45"

→ Result:
  Log split into 5 chunks
  Chunk 1/5: Setup (5KB)
  Chunk 2/5: Build (5KB)
  Chunk 3/5: Test (5KB) ⚠️ Contains errors
  Chunk 4/5: Deploy (5KB)
  Chunk 5/5: Cleanup (2.3KB)
```

## Configuration

### MCP Server Setup

**File**: VSCode `settings.json`

```json
{
  "mcp.servers": {
    "bitbucket": {
      "command": "node",
      "args": ["/path/to/bitbucket-mcp/dist/index.js"],
      "env": {
        "BITBUCKET_URL": "https://api.bitbucket.org/2.0",
        "BITBUCKET_WORKSPACE": "your-workspace",
        "BITBUCKET_USERNAME": "your-email@domain.com",
        "BITBUCKET_PASSWORD": "your-app-password"
      }
    }
  }
}
```

### Per-Project .gitignore

Add to each project:
```gitignore
# Pipeline debug logs
.pipeline-logs/
```

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Skill not activating | Restart VSCode |
| "Pipeline not found" | Try listing recent pipelines first |
| "Permission denied" | Check app password scopes |
| "Logs unavailable" | Pipeline may still be running |
| MCP approval prompts | Click "Yes" (VSCode limitation) |

### Best Practices

✅ **Do:**
- Add `.pipeline-logs/` to `.gitignore`
- Use specific pipeline numbers when known
- Let Claude chunk large logs automatically
- Specify workspace/repo when working across projects

❌ **Don't:**
- Commit logs to git
- Try to get logs from running pipelines
- Share app passwords in code/docs

## Tips

### Efficiency

- **Be specific**: "Pipeline #34" is faster than searching
- **Use latest**: "Latest failure" is optimized
- **Batch requests**: "Get all failed step logs" in one go

### Customization

- Logs stay in current project (`.pipeline-logs/`)
- Works across any workspace/repo you specify
- Auto-detects from git config when possible

### Approval Prompts

Each MCP call requires approval (VSCode limitation):
- Typical debug session: 5-10 approvals
- Tracking: [Issue #10801](https://github.com/anthropics/claude-code/issues/10801)
- Future: Hoping for "Always Allow" option

---

**Need more help?** See [README.md](./README.md) or [SKILL.md](./SKILL.md) for detailed docs.
