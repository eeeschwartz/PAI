# CLAUDE_CONFIG_DIR Fix - How to Make PAI_DIRECTORY Work as Your .claude Directory

## The Problem

PAI documentation was inconsistent about how to make Claude Code automatically load your PAI_DIRECTORY configuration (settings, hooks, statusline) when it starts. This caused:

1. **Statusline not loading** - The statusline-command.sh wasn't being executed automatically
2. **Hooks not executing** - Dynamic requirements loading wasn't happening on startup
3. **Settings confusion** - Claude Code was looking in ~/.claude instead of PAI_DIRECTORY

## Root Cause

Claude Code needs the `CLAUDE_CONFIG_DIR` environment variable to know where to look for configuration. Without it:
- Claude Code defaults to `~/.claude/` or `~/.config/claude/`
- It creates project-specific `.claude/` directories
- Your PAI_DIRECTORY settings, hooks, and statusline are ignored

## The Solution

Set `CLAUDE_CONFIG_DIR` to point to your `PAI_DIRECTORY`:

```bash
# Add to ~/.zshrc or ~/.bashrc
export PAI_DIR="/path/to/PAI/PAI_DIRECTORY"
export PAI_HOME="$HOME"
export CLAUDE_CONFIG_DIR="$PAI_DIR"  # This is the critical line!

# Reload shell
source ~/.zshrc
```

## How It Works

When `CLAUDE_CONFIG_DIR` is set:

1. **Startup**: Claude Code looks in `$CLAUDE_CONFIG_DIR` for `settings.json`
2. **Settings**: Reads all your MCP servers, hooks, statusLine config from PAI_DIRECTORY
3. **StatusLine**: Executes `statusline-command.sh` automatically on every prompt
4. **Hooks**: Runs your `UserPromptSubmit`, `SessionStart`, and other hooks from PAI_DIRECTORY/hooks/
5. **Context**: Your load-dynamic-requirements.ts is called via the UserPromptSubmit hook

## Configuration Precedence

Claude Code checks in this order:
1. `$CLAUDE_CONFIG_DIR/settings.json` (if CLAUDE_CONFIG_DIR is set)
2. `$XDG_CONFIG_HOME/claude/settings.json` (Linux/Unix)
3. `~/.config/claude/settings.json` (modern default)
4. `~/.claude/settings.json` (legacy default)
5. `.claude/settings.json` (project-local)

By setting `CLAUDE_CONFIG_DIR=$PAI_DIR`, you ensure PAI_DIRECTORY is always checked first.

## Verification

After setting the environment variable and reloading your shell:

```bash
# Verify it's set
echo $CLAUDE_CONFIG_DIR
# Should output: /path/to/PAI/PAI_DIRECTORY

# Check Claude Code will find your settings
ls -la "$CLAUDE_CONFIG_DIR/settings.json"
# Should show your settings file

# Check statusline command
ls -la "$CLAUDE_CONFIG_DIR/statusline-command.sh"
# Should show your statusline script
```

## Testing

1. **Close all Claude Code sessions**
2. **Open a new terminal** (to get the new environment variables)
3. **Start Claude Code** from that terminal
4. **You should see**:
   - Your custom statusline appears immediately
   - Hooks execute when you submit prompts
   - All your MCP servers load
   - Dynamic requirements load based on context

## Changes Made to PAI Documentation

Updated the following files to include `CLAUDE_CONFIG_DIR`:

1. `README.md` - Main installation guide
2. `PAI_DIRECTORY/documentation/README.md` - System documentation
3. `PAI_DIRECTORY/documentation/quick-start.md` - Quick start guide

All now include this critical environment variable in their setup instructions.

## Why This Matters

**Before**: PAI was just a collection of files that required manual setup and didn't work automatically.

**After**: PAI becomes your true personal AI infrastructure that:
- Loads automatically when Claude Code starts
- Shows your custom statusline with real-time metrics
- Executes hooks for dynamic context loading
- Provides seamless integration between all PAI components

## Common Issues

### "My statusline still doesn't show"
- Make sure you opened Claude Code from a NEW terminal after setting CLAUDE_CONFIG_DIR
- Verify: `echo $CLAUDE_CONFIG_DIR` shows the correct path

### "Hooks aren't running"
- Check your settings.json has the hooks configured
- Verify hook scripts are executable: `chmod +x $CLAUDE_CONFIG_DIR/hooks/*.ts`

### "Settings aren't being read"
- Confirm settings.json exists: `ls -la $CLAUDE_CONFIG_DIR/settings.json`
- Check for JSON syntax errors: `jq . $CLAUDE_CONFIG_DIR/settings.json`

## References

- [Claude Code Settings Documentation](https://docs.claude.com/en/docs/claude-code/settings)
- [GitHub Issue #2277 - Config Directory Behavior](https://github.com/anthropics/claude-code/issues/2277)
- [GitHub Issue #3833 - CLAUDE_CONFIG_DIR Behavior](https://github.com/anthropics/claude-code/issues/3833)

## Summary

**The one-line fix**: Add `export CLAUDE_CONFIG_DIR="$PAI_DIR"` to your shell config.

This single line makes everything work:
- ✅ Statusline loads automatically
- ✅ Hooks execute on startup
- ✅ Settings read from PAI_DIRECTORY
- ✅ Complete PAI integration

---

*Fixed: October 8, 2025*
*Issue: Conflicting documentation about PAI_DIRECTORY setup*
