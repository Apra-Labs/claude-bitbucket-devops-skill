# Installation Guide

Step-by-step guide to install and configure the Claude Bitbucket Pipelines Skill.

## Prerequisites Checklist

Before you begin, ensure you have:

- [ ] Claude Code VSCode extension installed
- [ ] Node.js 16 or later installed
- [ ] Bitbucket account with access to repositories
- [ ] Git installed (optional, for cloning)

## Step 1: Install bitbucket-mcp Server

The skill requires the [bitbucket-mcp](https://github.com/MatanYemini/bitbucket-mcp) server.

### Option A: Clone and Build

```bash
# Choose a location for the MCP server
cd ~/projects  # or C:\projects on Windows

# Clone the repository
git clone https://github.com/MatanYemini/bitbucket-mcp.git
cd bitbucket-mcp

# Install dependencies
npm install

# Build the server
npm run build
```

**Result**: MCP server built at `./dist/index.js`

**Note the full path** - you'll need it later. Example:
- macOS/Linux: `/Users/yourname/projects/bitbucket-mcp/dist/index.js`
- Windows: `C:\projects\bitbucket-mcp\dist\index.js`

### Option B: npm Global Install (if available)

```bash
npm install -g bitbucket-mcp
```

Check installation:
```bash
which bitbucket-mcp  # macOS/Linux
where bitbucket-mcp  # Windows
```

## Step 2: Create Bitbucket App Password

1. **Go to Bitbucket Settings**
   - Visit: https://bitbucket.org/account/settings/app-passwords/
   - Or: Your profile → Settings → App passwords

2. **Click "Create app password"**

3. **Configure the password:**
   - **Label**: `Claude Code Pipeline Debug`
   - **Permissions**:
     - ✅ Repository: Read
     - ✅ Pipelines: Read
     - ✅ Pipelines: Write (optional, for triggering builds)

4. **Click "Create"**

5. **Copy the password immediately!**
   - ⚠️ You cannot view it again
   - Format: `ATxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
   - Save it somewhere secure

## Step 3: Configure VSCode Settings

1. **Open VSCode Settings**
   - Press: `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac)
   - Type: `Preferences: Open User Settings (JSON)`
   - Press Enter

2. **Add MCP Server Configuration**

   Add this to your `settings.json`:

   ```json
   {
     "mcp.servers": {
       "bitbucket": {
         "command": "node",
         "args": ["/FULL/PATH/TO/bitbucket-mcp/dist/index.js"],
         "env": {
           "BITBUCKET_URL": "https://api.bitbucket.org/2.0",
           "BITBUCKET_WORKSPACE": "your-workspace-name",
           "BITBUCKET_USERNAME": "your.email@example.com",
           "BITBUCKET_PASSWORD": "your-app-password-here"
         }
       }
     }
   }
   ```

3. **Replace the placeholders:**

   | Placeholder | What to use | Example |
   |-------------|-------------|---------|
   | `/FULL/PATH/TO/bitbucket-mcp/dist/index.js` | Full path from Step 1 | `C:\\projects\\bitbucket-mcp\\dist\\index.js` |
   | `your-workspace-name` | Your Bitbucket workspace | `my-company` |
   | `your.email@example.com` | Your Bitbucket username/email | `john@example.com` |
   | `your-app-password-here` | App password from Step 2 | `ATBBxxxxxxxx...` |

   **Windows users**: Use double backslashes `\\` in paths:
   ```json
   "args": ["C:\\projects\\bitbucket-mcp\\dist\\index.js"]
   ```

4. **Save the file** (`Ctrl+S` / `Cmd+S`)

## Step 4: Install the Skill

### Option A: Manual Copy

1. **Download the skill:**
   - Download `SKILL.md` from this repository
   - Or clone the entire repo: `git clone https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill.git`

2. **Create skills directory:**

   **Windows:**
   ```cmd
   mkdir %USERPROFILE%\.claude\skills\bitbucket-pipeline-debug
   ```

   **macOS/Linux:**
   ```bash
   mkdir -p ~/.claude/skills/bitbucket-pipeline-debug
   ```

3. **Copy SKILL.md:**

   **Windows:**
   ```cmd
   copy SKILL.md %USERPROFILE%\.claude\skills\bitbucket-pipeline-debug\
   ```

   **macOS/Linux:**
   ```bash
   cp SKILL.md ~/.claude/skills/bitbucket-pipeline-debug/
   ```

### Option B: Git Clone

Clone directly into skills directory:

**Windows:**
```cmd
cd %USERPROFILE%\.claude\skills
git clone https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill.git bitbucket-pipeline-debug
```

**macOS/Linux:**
```bash
cd ~/.claude/skills
git clone https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill.git bitbucket-pipeline-debug
```

## Step 5: Restart Claude Code

**Restart VSCode** to load the new skill:

**Method 1: Reload Window**
- Press: `Ctrl+Shift+P` / `Cmd+Shift+P`
- Type: `Developer: Reload Window`
- Press Enter

**Method 2: Close and Reopen**
- Close VSCode completely
- Reopen VSCode

⏱️ **Wait 5-10 seconds** after reopening for skills to load.

## Step 6: Verify Installation

1. **Open Claude Code** in VSCode

2. **Ask a test question:**
   ```
   List the recent pipelines
   ```
   or
   ```
   What's the latest pipeline?
   ```

3. **Expected behavior:**
   - Claude should recognize the question
   - You'll see approval prompts for MCP tools (click "Yes")
   - Claude should return pipeline information

## Step 7: Configure .gitignore (Per Project)

In each project where you'll use this skill:

1. **Open `.gitignore`** in your project

2. **Add this line:**
   ```gitignore
   # Pipeline debug logs
   .pipeline-logs/
   ```

3. **Save the file**

This prevents pipeline logs from being committed to git.

## Troubleshooting Installation

### "Skill not found" or not activating

**Cause**: Skill not loaded or wrong location

**Solutions:**
1. Verify file exists: `~/.claude/skills/bitbucket-pipeline-debug/SKILL.md`
2. Check file has correct frontmatter (starts with `---`)
3. Restart VSCode again
4. Wait 10 seconds after restart

### "MCP server not found" or "Connection failed"

**Cause**: MCP server path or configuration incorrect

**Solutions:**
1. Verify `bitbucket-mcp` is built: check `dist/index.js` exists
2. Check path in `settings.json` is absolute and correct
3. Windows: Ensure backslashes are doubled: `C:\\path\\to\\file`
4. Try running manually: `node /path/to/bitbucket-mcp/dist/index.js`

### "Authentication failed" or "Permission denied"

**Cause**: App password or credentials incorrect

**Solutions:**
1. Verify app password has correct scopes
2. Check username is your Bitbucket email
3. Ensure no extra spaces in `settings.json`
4. Try creating a new app password

### "Pipeline not found"

**Cause**: Workspace or repository name incorrect

**Solutions:**
1. Verify workspace name in Bitbucket URL
2. Check you have access to the repository
3. Try: "List repositories" to see what's accessible

## Next Steps

✅ **Installation complete!**

Now you can:

1. **Read the [Quick Reference](./QUICK_REFERENCE.md)** for common commands
2. **Try the examples** in [README.md](./README.md)
3. **Explore** with your own questions

## Getting Help

Still having issues?

- 📖 [README.md](./README.md) - Full documentation
- 🐛 [GitHub Issues](https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill/issues) - Report problems
- 💬 [Discussions](https://github.com/Apra-Labs/claude-bitbucket-pipelines-skill/discussions) - Ask questions

---

**Installation time:** ~10-15 minutes

**Difficulty:** Intermediate (requires editing JSON and running terminal commands)
