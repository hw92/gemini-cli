# Agent-Native Principles in Gemini CLI

Analysis of the 5 agent-native architecture principles (from "Agent-native Architectures: How to Build Apps After Code Ends" by Dan Shipper & Claude) mapped against the Gemini CLI codebase.

## Principles Found in the Codebase

### 1. Granularity — YES, strongly present

Tools are atomic primitives: `ReadFile`, `WriteFile`, `Edit`, `Grep`, `Glob`, `Shell`, `Ls`. The agent composes them in a loop to achieve outcomes. There's no `analyze_and_organize_codebase` mega-tool — the agent figures it out with primitives. This is textbook agent-native granularity.

### 2. Parity — YES, effectively

The agent has `ShellTool` (bash access) plus file read/write/edit/search tools. Anything a developer can do at a terminal, the agent can achieve through its tool set. The article uses Claude Code as the example of parity done right — this codebase follows the same pattern.

### 3. Composability — YES

New features are added via prompts and skills, not code. The skill/extension system lets you define new capabilities as prompt-described outcomes. The `.gemini/` extension system with agents, skills, and custom context files enables "new features by writing new prompts."

### 4. Emergent Capability — YES

Because tools are atomic and the agent has bash + file system access, users can ask open-ended things the developers didn't anticipate. The MCP server integration further enables this — dynamically discovered tools (`DiscoveredMCPTool`) let the agent access capabilities that weren't built in. This maps directly to the article's "dynamic capability discovery" pattern.

### 5. Improvement Over Time — YES, strongly present

- **GEMINI.md** files = the article's `context.md` pattern almost exactly. Hierarchical discovery (global -> project -> subdirectory) with JIT loading.
- **MemoryTool** lets the agent write/update its own context files.
- **Developer-level refinement**: System prompt snippets ship with updates.
- **User-level customization**: Users edit GEMINI.md and settings.

## Implementation Patterns Found

| Article Pattern | Present? | Gemini CLI Implementation |
|---|---|---|
| Files as universal interface | YES | Filesystem is the primary workspace. Tools operate on files. |
| context.md pattern | YES | `GEMINI.md` — nearly identical concept, hierarchical discovery |
| Agent loop with completion signal | YES | `complete_task` tool explicitly ends the loop — not heuristic-based |
| Model tier selection | YES | `ModelRouterService` with auto-routing, fast/pro models, per-agent overrides |
| Partial completion / task tracking | YES | `WriteTodosTool` with pending/in_progress/completed statuses |
| Context limits handling | YES | Automatic history compression when approaching token limits |
| Shared workspace | YES | Agent and user work in the same project directory |
| Context injection | YES | `PromptProvider` injects environment, git state, memory, MCP instructions |
| Approval / user agency | YES | Policy engine with DEFAULT/AUTO_EDIT/YOLO modes, stakes-based confirmation |
| Progressive disclosure | YES | Simple prompts work immediately; plan mode, extensions, MCP for power users |
| Dynamic capability discovery | YES | MCP servers — agent discovers tools at runtime |
| Checkpoint / resume | YES | Session recording, session resumption |
| Plan mode | YES | `EnterPlanModeTool`/`ExitPlanModeTool` — read-only exploration then execution |

## What's NOT Strongly Present

| Article Pattern | Status |
|---|---|
| Graduating to code (primitives -> domain tools -> optimized code paths) | Not an explicit progression — tools are either primitives or MCP-discovered |
| CRUD completeness audit | No formal audit mechanism — relies on bash/file primitives as catch-all |
| Self-modification (agent editing its own prompts/code) | Limited to GEMINI.md via MemoryTool; agent doesn't modify its own system prompt or tool code |
| Mobile-specific patterns | N/A — this is a CLI tool |

## Summary

This codebase is a strong implementation of the agent-native architecture described in the article. All 5 core principles are present. The most direct parallel: GEMINI.md is essentially the article's `context.md` pattern, and the atomic-tools-in-a-loop architecture is exactly what the article prescribes. The policy engine maps to the article's approval framework. MCP integration maps to dynamic capability discovery.

Both this project and Claude Code (the article's reference implementation) converged on the same architecture independently.

## Key Architecture Files

- Agent loop: `packages/core/src/agents/local-executor.ts`
- Tool definitions: `packages/core/src/tools/tools.ts`
- System prompt: `packages/core/src/prompts/promptProvider.ts`
- Policy engine: `packages/core/src/policy/policy-engine.ts`
- Memory discovery: `packages/core/src/utils/memoryDiscovery.ts`
- Model routing: `packages/core/src/models/` directory
- Extension system: `packages/core/src/config/config.ts`

*Written by Claude (claude-opus-4-6) | 2026-02-05 16:00 PST*
