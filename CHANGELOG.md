# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-11-01

### Added
- Initial release of Claude Bitbucket Pipelines Skill
- 8 core usage patterns:
  1. Find latest failing pipeline
  2. Inspect specific pipeline by number
  3. Identify failing steps in a pipeline
  4. Download logs from failing steps
  5. Download logs from specific steps
  6. Auto-slice large logs into manageable chunks
  7. List available pipeline types for branches
  8. Trigger pipeline runs with custom variables
- Project-relative log storage (`.pipeline-logs/`)
- Cross-project support (specify any workspace/repo)
- Automatic workspace detection from MCP config
- Smart log chunking for files >10KB
- Comprehensive documentation:
  - README.md with quick start guide
  - INSTALL.md with step-by-step instructions
  - QUICK_REFERENCE.md for common commands
  - CONTRIBUTING.md for contributors
  - SKILL.md with detailed skill logic
- CC BY 4.0 license
- Proper attribution to bitbucket-mcp by @MatanYemini

### Known Issues
- MCP tool calls require manual approval in VSCode extension
  - Tracking: [GitHub Issue #10801](https://github.com/anthropics/claude-code/issues/10801)
  - Impact: Users must click "Yes" for each Bitbucket API call
  - Workaround: None currently available in VSCode extension

## [Unreleased]

### Planned Features
- Auto-detect workspace from git config
- Support for Bitbucket Server (self-hosted)
- Pipeline comparison (compare two runs side-by-side)
- Build time trends and analytics
- Integration with notification services (Slack, Discord)
- Advanced log analysis with AI-powered error detection
- Pipeline template suggestions based on failures

### Awaiting External Dependencies
- Auto-approval support (depends on Claude Code issue #10801 resolution)
- Skill hot-reload without VSCode restart (Claude Code feature request)

---

## Version History

- **1.0.0** (2025-11-01) - Initial public release

## Upgrade Guide

### From Pre-release to 1.0.0

If you were using a pre-release version:

1. **Backup your custom configurations** (if any)
2. **Remove old skill:**
   ```bash
   rm -rf ~/.claude/skills/bitbucket-pipeline-debug
   ```
3. **Install 1.0.0** following [INSTALL.md](./INSTALL.md)
4. **Restore custom configurations**
5. **Restart VSCode**

---

**Maintained by [Apra Labs](https://github.com/Apra-Labs)**
