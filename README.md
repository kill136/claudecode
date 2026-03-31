# Claude Code Source

Deobfuscated source code of `@anthropic-ai/claude-code` v2.1.76.

## Overview

This repository contains the reverse-engineered TypeScript source code from Anthropic's [Claude Code](https://www.npmjs.com/package/@anthropic-ai/claude-code) CLI tool. The official npm package ships as a single minified `cli.js` bundle (~12MB); this repo restores the original module structure with readable names, comments, and type annotations.

**1,884 TypeScript/TSX source files | 35MB | 5,066 bundled modules**

## Project Structure

```
├── package.json          # Dependencies & build scripts
├── tsconfig.json         # TypeScript configuration
├── src/                  # Source code (deobfuscated)
│   ├── entrypoints/      # CLI, SDK, MCP entry points
│   ├── tools/            # 40+ tool implementations
│   ├── AgentTool/        # Sub-agent spawning & orchestration
│   ├── BashTool/         # Shell command execution with sandbox
│   ├── FileEditTool/     # String replacement file editing
│   ├── FileReadTool/     # File reading (text, PDF, images)
│   ├── FileWriteTool/    # File creation/overwrite
│   ├── GlobTool/         # File pattern matching
│   ├── GrepTool/         # Content search (ripgrep-based)
│   ├── WebFetchTool/     # URL fetching & content extraction
│   ├── WebSearchTool/    # Web search integration
│   ├── SkillTool/        # Slash command / skill execution
│   ├── LSPTool/          # Language Server Protocol integration
│   ├── NotebookEditTool/ # Jupyter notebook editing
│   └── ...               # TaskCreate/Get/Update/List/Stop, Cron, MCP, etc.
├── bridge/               # IDE integration & remote control (CCR)
├── query/                # Core query engine & token budget
├── query.ts              # Main conversation loop (~68K lines)
├── QueryEngine.ts        # High-level session management
├── commands/             # 88+ slash commands
├── commands.ts           # Command registry with feature gates
├── components/           # React + Ink TUI components
├── services/             # API, analytics, MCP, telemetry
├── state/                # Zustand-like app state management
├── context/              # System prompt & context assembly
├── hooks/                # Pre/post tool execution hooks
├── skills/               # Built-in skill definitions
├── plugins/              # Plugin system (manifest, marketplace)
├── coordinator/          # Multi-agent coordination mode
├── buddy/                # Virtual companion system (Easter egg)
├── vim/                  # Full Vim keybinding support
├── voice/                # Voice input mode
├── ink/                  # Custom Ink renderer & reconciler
├── native-ts/            # TS ports of native modules (yoga, nucleo, syntect)
├── remote/               # Remote session management
├── server/               # Direct connect server
├── memdir/               # Auto-memory system (MEMORY.md)
├── keybindings/          # Keyboard shortcut system
├── migrations/           # Version upgrade migrations
├── upstreamproxy/        # Container HTTPS proxy (CCR)
├── schemas/              # Zod validation schemas
├── types/                # TypeScript type definitions
├── utils/                # Utilities (permissions, sandbox, config, etc.)
├── constants/            # Prompts, keys, system constants
├── tasks/                # Background task types (Agent, Shell, Workflow)
│   ├── Tool.ts           # Base tool interface & registry
│   ├── tools.ts          # Tool list with feature-gated imports
│   ├── cost-tracker.ts   # API cost tracking
│   ├── history.ts        # Conversation history (JSONL)
│   └── setup.ts          # Initialization & configuration
└── dist/                 # Build output (not committed)
```

## Key Architectural Insights

### Build System
- Built with **Bun bundler** — uses `bun:bundle` `feature()` for compile-time dead code elimination (DCE)
- `MACRO.*` build-time constants (VERSION, BUILD_TIME, etc.) injected via `--define`
- React Compiler (`react/compiler-runtime`) for optimized component rendering
- Single-file output: all 5,066 modules bundled into one `cli.js`

### Feature Gates
Internal feature flags control functionality via `feature()` from `bun:bundle`:
- `PROACTIVE` / `KAIROS` — Proactive agent capabilities
- `AGENT_TRIGGERS` / `AGENT_TRIGGERS_REMOTE` — Scheduled/remote agent execution
- `MONITOR_TOOL` — Live monitoring tool
- `ABLATION_BASELINE` — A/B testing baseline
- `KAIROS_GITHUB_WEBHOOKS` — GitHub webhook subscriptions

### Internal-Only Features
Some tools/packages are restricted to Anthropic employees (`USER_TYPE === 'ant'`):
- `REPLTool` — Interactive REPL execution
- `SuggestBackgroundPRTool` — Background PR suggestions
- `@ant/claude-for-chrome-mcp` — Chrome browser integration
- `@anthropic-ai/sandbox-runtime` — Sandbox isolation runtime

### Tool System
- All tools implement a common interface (`Tool` from `Tool.ts`)
- Input validation via **Zod** with `lazySchema()` for deferred initialization
- Permission system: `canUseTool()` hooks with caching
- Concurrent file edit protection via timestamp + content change detection
- `ToolSearchTool` enables lazy tool discovery (deferred tool loading)

### State Management
- **AppStateStore** — Zustand-like reactive state for the TUI
- **QueryEngine** — Manages conversation sessions, message history, compaction
- **Session persistence** — JSONL-based history with paste storage

## Building from Source

### Prerequisites
- [Bun](https://bun.sh/) >= 1.0
- Node.js >= 18

### Steps

```bash
# Install dependencies
bun install

# Build (you'll need to create stubs for internal @ant/* packages)
bun build src/entrypoints/cli.tsx --outdir dist --target node \
  --external "react/compiler-runtime" \
  --define "MACRO.VERSION='\"2.1.76\"'" \
  --define "MACRO.BUILD_TIME='\"$(date -I)\"'" \
  --define "MACRO.FEEDBACK_CHANNEL='\"stable\"'" \
  --define "MACRO.ISSUES_EXPLAINER='\"report issues\"'" \
  --define "MACRO.PACKAGE_URL='\"\"'" \
  --define "MACRO.NATIVE_PACKAGE_URL='\"\"'" \
  --define "MACRO.VERSION_CHANGELOG='\"\"'"

# Patch react/compiler-runtime import
sed -i 's|from "react/compiler-runtime"|from "./react-compiler-runtime.js"|g' dist/cli.js

# Run
node dist/cli.js --version
```

> **Note**: ~10 source files are missing from this deobfuscation (internal tools, some type definitions). Stubs are needed for: `@ant/*` packages, `@anthropic-ai/sandbox-runtime`, `REPLTool`, `TungstenTool`, `color-diff-napi`, `modifiers-napi`.

## Known Missing Files

| File | Description |
|------|-------------|
| `tools/REPLTool/` | Interactive REPL (ant-only) |
| `tools/TungstenTool/` | Internal monitoring tool |
| `tools/SuggestBackgroundPRTool/` | Background PR tool (ant-only) |
| `tools/VerifyPlanExecutionTool/` | Plan verification tool |
| `keybindings/types.ts` | Keybinding type definitions |
| `utils/filePersistence/types.ts` | File persistence types |
| `services/compact/cachedMicrocompact.ts` | Cached micro-compaction |
| `services/contextCollapse/` | Context collapse service |
| `types/connectorText.ts` | Connector text block types |

## Disclaimer

This repository is for **educational and research purposes only**. All rights belong to Anthropic PBC. The original source is subject to [Anthropic's legal agreements](https://code.claude.com/docs/en/legal-and-compliance).

## Source Version

- **Package**: `@anthropic-ai/claude-code`
- **Version**: 2.1.76
- **Date**: 2026-03-26
