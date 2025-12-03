<!--
This file contains the comment text that was posted to Pull Request #3591 (which fixes Issue #3589)
on the pre-commit/pre-commit repository.

PR: #3591
URL: https://github.com/pre-commit/pre-commit/pull/3591

Issue: #3589
URL: https://github.com/pre-commit/pre-commit/issues/3589
-->

# Fix for Blocked `os.sysconf` in Sandboxed Environments

## Is This For You?

Are you using AI agents in a sandboxed environment (e.g., Cursor, GitHub Codespaces, GitPod, Replit, or restricted Docker containers)?

Do your `git commit` commands abort because `pre-commit` raises an exception at import time?

Do `git commit` and `pre-commit` work outside the sandboxed environment?

If so, read on. This fix will allow `pre-commit` to work inside the sandbox.

## Background

AI agents and development tools (like Cursor IDE) run in restricted sandboxed environments that block certain system calls, including `os.sysconf('SC_ARG_MAX')`. This causes `pre-commit` to fail with `PermissionError` or `OSError`, which **prevents `git commit` from working** in AI-assisted environments. When AI tools attempt to run `git commit`, pre-commit hooks fail immediately, blocking the entire commit workflow.

## The Solution

This PR adds a minimal fix:
- Wraps `os.sysconf('SC_ARG_MAX')` in `try/except OSError` to handle sandbox restrictions
- Falls back to POSIX minimum when `os.sysconf` is blocked
- Moves `_max_length` evaluation from import-time to runtime
- Refactors variable names to avoid reusing the same variable for different meanings

The fix is tested, works in Cursor IDE's sandbox, and maintains backward compatibility.

## How to Apply This Fix

For detailed instructions, see [`HOWTO_sysconf_sandbox.md`](https://github.com/MichaelRWolf/pre-commit/blob/issue-3589-sysconf-sandbox/HOWTO_sysconf_sandbox.md).

**Quick overview:**
1. Clone the original pre-commit repository: `git clone https://github.com/pre-commit/pre-commit.git`
2. Add the fork as a remote: `git remote add michaelrwolf https://github.com/MichaelRWolf/pre-commit.git`
3. Fetch the fix:
   - **Static version**: `git fetch michaelrwolf tag v4.5.0-sysconf-fix`
   - **Latest version**: `git fetch michaelrwolf issue-3589-sysconf-sandbox`
4. Merge the changes:
   - **Static version**: `git merge v4.5.0-sysconf-fix`
   - **Latest version**: `git merge michaelrwolf/issue-3589-sysconf-sandbox`
5. Reinstall if using as a package: `pip install -e .`

This enables `pre-commit` (and thus `git commit`) to work in sandboxed AI-assisted environments.
