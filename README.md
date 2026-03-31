# claudecode-pack

Reverse-engineered unpacked source tree for `@anthropic-ai/claude-code@2.1.88`, reconstructed from the published npm tarball and its `cli.js.map` source map.

## What this repo contains

- Original npm package tarball: `anthropic-ai-claude-code-2.1.88.tgz`
- Unpacked package files: `unpacked/package/`
- Reconstructed sources from `cli.js.map`: `extracted/`
- Analysis artifacts: `map-analysis/`

## How this was produced

1. Ran:
   ```bash
   npm pack @anthropic-ai/claude-code@2.1.88
   ```
2. Extracted the tarball.
3. Parsed `cli.js.map`.
4. Rebuilt files from the source map's `sources` + `sourcesContent` entries.
5. Wrote structure summaries into `map-analysis/`.

## Reconstruction result

- Total reconstructed files: **4756**
- Top-level distribution:
  - `src`: **1902** files
  - `node_modules`: **2850** files
  - `vendor`: **4** files

## High-level Claude Code structure

The recovered source tree suggests Claude Code is organized roughly into these layers:

### 1. CLI entry layer
Key files:
- `extracted/src/main.tsx`
- `extracted/src/entrypoints/*`
- `extracted/src/cli/*`

This appears to be the command-line bootstrap and terminal app entrypoint.

### 2. Command system
Key directory:
- `extracted/src/commands/`

Recovered command files: **207**

Notable areas include command families related to:
- plugin management
- review
- MCP
- context
- remote setup
- usage / clear / rename / add-dir

### 3. Tool runtime
Key directory:
- `extracted/src/tools/`

Recovered tool files: **178**

Notable built-in tools include:
- `BashTool`
- `FileReadTool`
- `FileWriteTool`
- `FileEditTool`
- `GlobTool`
- `GrepTool`
- `WebFetchTool`
- `WebSearchTool`
- `MCPTool`
- `LSPTool`
- `AgentTool`
- `TodoWriteTool`
- `ConfigTool`
- `SkillTool`
- `ScheduleCronTool`
- `Task*Tool`

This strongly suggests Claude Code uses an internal tool-oriented agent runtime.

### 4. Task / agent orchestration
Key files and directories:
- `extracted/src/Task.ts`
- `extracted/src/tools/AgentTool/*`

The source includes task IDs, task status handling, agent spawning, agent resuming, and built-in agents.

Interesting built-in agent files include:
- `built-in/planAgent.ts`
- `built-in/verificationAgent.ts`
- `built-in/exploreAgent.ts`
- `built-in/claudeCodeGuideAgent.ts`

### 5. Query / conversation engine
Key files:
- `extracted/src/QueryEngine.ts`
- `extracted/src/query.ts`
- `extracted/src/utils/processUserInput/*`
- `extracted/src/services/api/*`

This looks like the main assistant loop that handles prompts, tool usage, model calls, permissions, and state transitions.

### 6. Terminal UI
Key directories:
- `extracted/src/components/`
- `extracted/src/ink/`
- `extracted/src/screens/`

Recovered counts:
- `components`: **389** files
- `ink`: **96** files
- `hooks`: **104** files

This indicates the product is not just a thin CLI wrapper, but a substantial terminal UI application.

### 7. MCP / remote / bridge layers
Key directories:
- `extracted/src/services/mcp/`
- `extracted/src/remote/`
- `extracted/src/bridge/`
- `extracted/src/server/`

This suggests support for MCP integrations, remote sessions, and bridge/server functionality.

### 8. Memory / skills / settings
Key directories:
- `extracted/src/memdir/`
- `extracted/src/skills/`
- `extracted/src/utils/settings/`

This points to built-in support for memory, skills, and managed settings.

## Recovered module counts inside `src/`

- `commands`: **207**
- `tools`: **178**
- `services`: **130**
- `components`: **389**
- `hooks`: **104**
- `ink`: **96**
- `bridge`: **31**
- `cli`: **19**
- `skills`: **20**
- `memdir`: **8**
- `remote`: **4**
- `server`: **3**

## Useful files in this repo

### Main content
- `unpacked/package/cli.js`
- `unpacked/package/cli.js.map`
- `extracted/src/`

### Analysis outputs
- `map-analysis/sources.json`
- `map-analysis/file-list.txt`
- `map-analysis/top-level-summary.txt`
- `map-analysis/src-summary.txt`
- `map-analysis/tools-tree.txt`
- `map-analysis/services-tree.txt`
- `map-analysis/commands-tree.txt`
- `map-analysis/core-snippets.txt`

## Suggested starting points for inspection

- `extracted/src/main.tsx`
- `extracted/src/QueryEngine.ts`
- `extracted/src/Tool.ts`
- `extracted/src/Task.ts`
- `extracted/src/tools/`
- `extracted/src/commands/`
- `extracted/src/services/`

## Notes

This repository is a structural reconstruction based on the published npm artifact and its distributed source map. It is intended for inspection of architecture and file organization.
