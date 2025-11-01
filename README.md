# Claude Bitbucket Pipelines Skill

A [Claude Code](https://claude.ai/code) skill for debugging and managing Bitbucket CI/CD pipelines. Built on the [bitbucket-mcp](https://github.com/MatanYemini/bitbucket-mcp) Model Context Protocol server.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

## Features

🔍 **Find Failures Fast**
- Identify the latest failing pipeline instantly
- Locate specific pipeline runs by number

📊 **Deep Analysis**
- List all steps with their status and timing
- Identify exactly which steps failed

📥 **Smart Log Management**
- Download logs from failing steps automatically
- Get logs from specific steps
- Auto-slice large logs into manageable chunks

🚀 **Pipeline Control**
- List available pipeline types
- Trigger new pipeline runs with custom variables

## Prerequisites

### 1. Claude Code

Install the Claude Code extension for VSCode:
- [Visual Studio Code Extension](https://marketplace.visualstudio.com/items?itemName=Anthropic.claude-code)

### 2. Bitbucket MCP Server

This skill requires the [bitbucket-mcp](https://github.com/MatanYemini/bitbucket-mcp) server by [@MatanYemini](https://github.com/MatanYemini).

**Installation:**

```bash
# Clone the bitbucket-mcp repository
git clone https://github.com/MatanYemini/bitbucket-mcp.git
cd bitbucket-mcp

# Install dependencies
npm install

# Build the server
npm run build
```

### 3. Bitbucket App Password

Create a Bitbucket App Password with these scopes:

1. Go to: https://bitbucket.org/account/settings/app-passwords/
2. Click "Create app password"
3. Select these permissions:
   - ✅ **Repository**: Read
   - ✅ **Pipelines**: Read, Write

4. Save the generated password (you'll need it for configuration)

## Installation

### Step 1: Install the Skill

Copy the skill to your Claude Code skills directory:

```bash
# Windows
mkdir %USERPROFILE%\.claude\skills\bitbucket-pipeline-debug
copy SKILL.md %USERPROFILE%\.claude\skills\bitbucket-pipeline-debug\

# macOS/Linux
mkdir -p ~/.claude/skills/bitbucket-pipeline-debug
cp SKILL.md ~/.claude/skills/bitbucket-pipeline-debug/
```

**Or** clone this repository directly into your skills directory:

```bash
# Windows
cd %USERPROFILE%\.claude\skills
git clone https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill.git bitbucket-pipeline-debug

# macOS/Linux
cd ~/.claude/skills
git clone https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill.git bitbucket-pipeline-debug
```

### Step 2: Configure MCP Server

Add the bitbucket-mcp server to your VSCode settings:

**File**: `settings.json` (Open with: `Ctrl+Shift+P` → "Preferences: Open User Settings (JSON)")

```json
{
  "mcp.servers": {
    "bitbucket": {
      "command": "node",
      "args": ["/path/to/bitbucket-mcp/dist/index.js"],
      "env": {
        "BITBUCKET_URL": "https://api.bitbucket.org/2.0",
        "BITBUCKET_WORKSPACE": "your-workspace",
        "BITBUCKET_USERNAME": "your-username",
        "BITBUCKET_PASSWORD": "your-app-password-here"
      }
    }
  }
}
```

**Replace:**
- `/path/to/bitbucket-mcp/dist/index.js` → Actual path to your bitbucket-mcp installation
- `your-workspace` → Your Bitbucket workspace name
- `your-username` → Your Bitbucket username (usually your email)
- `your-app-password-here` → The app password you created

### Step 3: Restart Claude Code

**Important**: Claude Code needs to restart to load the new skill.

- Close and reopen VSCode
- Or use: `Ctrl+Shift+P` → "Developer: Reload Window"

### Step 4: Add .pipeline-logs to .gitignore

In each project where you use this skill, add to `.gitignore`:

```
# Pipeline debug logs
.pipeline-logs/
```

## Usage

Just ask Claude naturally! The skill activates automatically for pipeline-related questions.

### Examples

**Find Latest Failure:**
```
You: What's the latest failed pipeline?
```

**Inspect Specific Pipeline:**
```
You: Show me details for pipeline #34
```

**Analyze Failures:**
```
You: Which steps failed in pipeline #34?
You: Get logs for the failed steps
```

**Trigger New Build:**
```
You: Run the deploy-production pipeline on main
You: Trigger staging deployment with DRY_RUN=true
```

**Work Across Projects:**
```
You: Show latest failure in workspace/other-repo
You: Get pipeline #45 from my-workspace/my-project
```

## How It Works

### Log Storage

Logs are downloaded to your **current project directory**:

```
your-project/
├── .pipeline-logs/           ← Created automatically
│   ├── pipeline-123-deploy.log
│   ├── pipeline-123-test.log
│   └── metadata.json
├── src/
└── ...
```

This means:
- ✅ Logs stay with the relevant project
- ✅ Each project has its own log directory
- ✅ Easy to add to `.gitignore`
- ✅ No global state to manage

### Workspace Detection

The skill automatically works with any workspace/repo you specify:

- **Explicit**: "Show failures in workspace/repo"
- **From MCP config**: Uses your configured workspace by default
- **From git remote**: Detects workspace from current project

## Known Limitations

### MCP Tool Approval Prompts

⚠️ **Current Limitation**: The VSCode extension requires manual approval (click "Yes") for each MCP tool call.

**Why?** This is a Claude Code VSCode extension limitation, not a bug in this skill.

**Tracking**: [GitHub Issue #10801](https://github.com/anthropics/claude-code/issues/10801)

**Impact:**
- You'll need to approve each Bitbucket API call
- Unattended automation is not currently possible
- Typical pipeline debug session: 5-10 approvals

**Future:** We're hoping Anthropic adds an "Always Allow" option or respects the `bypassPermissions` configuration.

## Troubleshooting

### Skill not activating

**Solution**: Restart VSCode to reload skills
- `Ctrl+Shift+P` → "Developer: Reload Window"

### "Pipeline not found"

**Possible causes:**
- Pipeline number is incorrect
- Pipeline is too old (try recent pipelines first)
- Wrong workspace/repo

**Solution:**
```
You: List recent pipelines
You: Show me the last 20 builds
```

### "Permission denied"

**Check:**
- App password has `Repository: Read` and `Pipeline: Read/Write` scopes
- Username and password are correct in `settings.json`
- Workspace name is correct

### Logs unavailable

**Reasons:**
- Pipeline is still running (wait for completion)
- Logs expired (Bitbucket retention policy)
- Network connectivity issues

## Credits

This skill is built on top of:

- **[bitbucket-mcp](https://github.com/MatanYemini/bitbucket-mcp)** by [@MatanYemini](https://github.com/MatanYemini) - The MCP server that powers this skill
- **[Claude Code](https://claude.ai/code)** by [Anthropic](https://www.anthropic.com/) - The AI coding assistant platform

## Contributing

We welcome contributions! See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

**Ideas for contributions:**
- Additional pipeline management features
- Better log analysis and error detection
- Support for Bitbucket Server (self-hosted)
- Integration with other CI/CD platforms

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

**You are free to:**
- ✅ Share — copy and redistribute in any medium or format
- ✅ Adapt — remix, transform, and build upon the material

**Under the following terms:**
- **Attribution** — You must give appropriate credit to Apra Labs and link to this repository

See [LICENSE](./LICENSE) for full details.

## Support

- 🐛 **Bug Reports**: [GitHub Issues](https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill/discussions)
- 📖 **Documentation**: This README and [SKILL.md](./SKILL.md)

## Roadmap

- [ ] Auto-detect workspace from git config
- [ ] Support for Bitbucket Server (self-hosted)
- [ ] Pipeline comparison (compare two runs)
- [ ] Build time trends and analytics
- [ ] Slack/Discord notifications integration
- [ ] Waiting for: Auto-approval support (Issue #10801)

---

**Maintained by [Apra Labs](https://github.com/Apra-Labs)**

Built with ❤️ for the Claude Code community
