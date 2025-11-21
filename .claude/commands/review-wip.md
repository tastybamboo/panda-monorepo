---
description: Review work in progress across all projects and present actionable options
tags: [project, gitignored]
---

# Work in Progress Review

Analyze all work in progress across the panda monorepo and present options for what to work on next.

## Tasks to Complete

1. **Check Git Status Across All Projects**
   - For each subdirectory (gems/*, automation/*, sites/*):
     - List current branch
     - Show uncommitted changes (git status --short)
     - List unmerged feature branches (excluding main/master)
     - Show any stashed changes

2. **Review Open Pull Requests**
   - Use `gh pr list` in each project to show open PRs
   - Note any failing checks or pending reviews

3. **Search for Work-in-Progress Markers**
   - Search all Markdown files (*.md) for:
     - TODO items
     - WIP markers
     - FIXME comments
     - "In Progress" sections
   - Look in:
     - Project README files
     - Documentation in docs/ folders
     - CHANGELOG files
     - Any notes/ or planning/ directories

4. **Check for Uncommitted Work**
   - Look for modified files
   - Look for untracked files that might be drafts
   - Check for any .draft or .wip files

5. **Review Recent Commits**
   - Show last 3 commits on current branch in each project
   - Highlight any commits that mention "WIP", "TODO", or "draft"

## Output Format

Present findings in a clear, organized format:

1. **Active Development** (current branch != main, with uncommitted changes)
2. **Feature Branches** (unmerged branches ready for work)
3. **Open Pull Requests** (PRs needing attention)
4. **Documented TODOs** (from markdown files)
5. **Recommendations** (prioritized list of what to work on next)

For recommendations, consider:
- Blocked work (failing tests, conflicts)
- Nearly complete work (small changes needed)
- High-impact features
- Technical debt that's been noted

End with a numbered list of 3-5 actionable options, each with:
- What: Brief description
- Where: Which project(s)
- Why: Priority/impact
- Next step: Specific action to take

Be thorough but concise. Focus on actionable insights.
