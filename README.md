# github-vibe-flow

GitHub Issue-driven collaboration between Coding Agents and humans.

## Skills

| Skill | Command | Description |
|---|---|---|
| `vf-design` | `/vf-design` | Design documents via brainstorming |
| `vf-issue-create` | `/vf-issue-create` | Generate GitHub Issues from design |
| `vf-execute` | `/vf-execute` | Execute issues with tmux + worktree |
| `vf-monitor` | `/vf-monitor` | Monitor PRs and auto-respond to reviews |
| `vf-merge` | `/vf-merge` | Cleanup after merge |
| `vf-flow` | `/vf-flow` | End-to-end orchestration |

## Dependencies

- [Claude Code](https://claude.com/claude-code) with plugin support
- GitHub MCP Server configured
- tmux
- git

## Installation

```bash
claude plugin add ensekitt/github-vibe-flow
```

## Execution Notation

Define issue execution order with lists `[]` (serial) and sets `()` (parallel):

- `[#1, #2, #3]` — execute sequentially
- `[#1, (#2, #3), #4]` — #1 first, then #2 and #3 in parallel, then #4
