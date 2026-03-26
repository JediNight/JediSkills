---
description: Search claude-mem memories for context on the current task or a specific topic
arguments:
  - name: topic
    description: What to search for (e.g. "PLATENG-959", "karpenter", "deploy pipeline")
    required: false
---

Search claude-mem persistent memory for relevant context. Use the 3-layer workflow:

1. **Search** claude-mem with `mcp__plugin_claude-mem_mcp-search__search` using the topic "$ARGUMENTS" (or infer from current conversation if no topic provided)
2. **Review** the index results and identify the most relevant observation IDs
3. **Fetch** full details with `mcp__plugin_claude-mem_mcp-search__get_observations` for the top 5-8 most relevant results

Also check the local auto-memory files at `~/.claude/projects/-Users-toksfawibe-Documents/memory/` via MEMORY.md index for any relevant project/feedback memories.

Present a concise summary of what was found, organized by relevance to the current task.
