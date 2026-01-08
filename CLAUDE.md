# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Initial setup (install dependencies, generate Prisma client, run migrations)
npm run setup

# Development server (uses Turbopack)
npm run dev

# Build for production
npm run build

# Run tests
npm test

# Run a single test file
npx vitest run src/lib/__tests__/file-system.test.ts

# Run tests in watch mode
npx vitest

# Lint
npm run lint

# Reset database
npm run db:reset
```

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and the AI generates code that renders in real-time.

### Core Data Flow

1. **Chat API** (`src/app/api/chat/route.ts`) - Receives messages and virtual file system state, streams AI responses using Vercel AI SDK with Claude
2. **AI Tools** - The AI uses two tools to modify the virtual file system:
   - `str_replace_editor` (`src/lib/tools/str-replace.ts`) - Create files, replace strings, insert text
   - `file_manager` (`src/lib/tools/file-manager.ts`) - Rename and delete files
3. **Tool calls** are processed both server-side (for persistence) and client-side (for UI updates via `handleToolCall` in `file-system-context.tsx`)

### Key Systems

**Virtual File System** (`src/lib/file-system.ts`)
- In-memory file system using `VirtualFileSystem` class
- No files written to disk - everything exists in browser memory
- Serializes to JSON for persistence in the database

**Live Preview** (`src/components/preview/PreviewFrame.tsx` + `src/lib/transform/jsx-transformer.ts`)
- Transforms JSX/TSX files using Babel in the browser
- Creates blob URLs and import maps for ES modules
- Renders in sandboxed iframe with React 19 from esm.sh
- Supports Tailwind CSS via CDN

**Context Providers** (`src/lib/contexts/`)
- `FileSystemProvider` - Manages virtual file system state and tool call handling
- `ChatProvider` - Wraps Vercel AI SDK's `useChat` hook, connects chat to file system

### Database

SQLite via Prisma. Schema in `prisma/schema.prisma`:
- `User` - Email/password auth
- `Project` - Stores messages (JSON) and file system data (JSON)

Prisma client is generated to `src/generated/prisma/`.

### Path Aliases

- `@/*` maps to `src/*`
- Components use shadcn/ui (new-york style) in `src/components/ui/`
