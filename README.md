# Attemory Claude Code Marketplace

This directory is a marketplace root for publishing the Attemory Code plugin to
Claude Code.

Local test:

```bash
claude plugin validate .
claude plugin marketplace add .
claude plugin install attemory-code@attemory
```

Users can install with:

```bash
claude plugin marketplace add AttemorySystem/attemory-claude-code
claude plugin install attemory-code@attemory
```

Users still need `attemory-mcp` on `PATH`, a running `attemory-server`, and a
target repository initialized with `atcode init` and `atcode index`.
