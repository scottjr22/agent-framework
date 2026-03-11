# agent-framework
Generic framework to use with ai coding agents
1. Install favorite agent
1. Setup perms
1. Before starting a new feature, MAKE A NEW BRANCH.
   - features should be independent of each other.
1. Define a Feature Requirements Document in `docs/{name_of_feature}/feature.md`, use `docs/` for reference:
   - What the feature is
   - Why we need it
   - Acceptance Criteria (testable)
   - System Constraints
   - Non-goals
   - Mockups/Samples (if applicable)
1. NOTE ALL WORK FROM HERE OUT SHOULD BE ON NEW GIT WORKTREE SO THAT EASY TO ISOLATE & REVERT CHANGES
   - To restart, delete the worktree, make changes to feature.md, and follow instructions starting here.
   
   to see all worktrees
   ```
   git worktree list
   ```
   forcibly remove worktree
   ```
   git worktree remove --force [name-of-worktree]
   ```
   - test that AGENTS.md is being read



## Copilot
- Add AGENTS.md to root 

## Codex
- Add AGENTS.md to root 
- Make sure worktrees setup (npm ci command present, perms, etc.)
config.toml
```
sandbox_mode = "workspace-write"
approval_policy = "on-failure"

[sandbox_workspace_write]
network_access = true%  
```
- test that AGENTS.md is being read

## Claude



## Skills

### Copilot
- ```
.github/skills
``
### Claude
- ```
.claude/skills
``
