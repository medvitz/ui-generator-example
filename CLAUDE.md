# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and Claude generates working React code that renders in real-time.

## Commands

```bash
# Initial setup (install deps, generate Prisma client, run migrations)
npm run setup

# Development server (uses Turbopack)
npm run dev

# Run tests
npm test

# Run single test file
npx vitest path/to/test.ts

# Lint
npm run lint

# Reset database
npm run db:reset
```

## Architecture

### Core Data Flow

1. **Chat Interface** (`src/components/chat/`) sends user messages to `/api/chat`
2. **Chat API** (`src/app/api/chat/route.ts`) processes messages with Claude (or mock provider if no API key)
3. Claude uses **tools** (`src/lib/tools/`) to manipulate the **VirtualFileSystem**
4. **FileSystemContext** (`src/lib/contexts/file-system-context.tsx`) propagates file changes to UI
5. **PreviewFrame** (`src/components/preview/PreviewFrame.tsx`) transforms files and renders in iframe

### Virtual File System

The app operates entirely on a `VirtualFileSystem` class (`src/lib/file-system.ts`) - no files are written to disk. Files are serialized to JSON for persistence in the database.

### JSX Transformation Pipeline

`src/lib/transform/jsx-transformer.ts` handles client-side code transformation:
- Uses `@babel/standalone` to compile JSX/TSX to JS
- Creates blob URLs for each transformed file
- Generates an import map for browser ESM resolution
- Third-party packages are resolved via `esm.sh`
- CSS imports are extracted and injected as style tags

### AI Tools

Claude interacts with the virtual file system through two tools:
- `str_replace_editor` - View, create, replace, and insert text in files
- `file_manager` - Rename and delete files

### Provider System

`src/lib/provider.ts` contains a `MockLanguageModel` that returns static responses when no Anthropic API key is configured. This enables local development without API access.

### Context Providers

- `FileSystemContext` - Manages virtual file state and tool call handling
- `ChatContext` - Wraps Vercel AI SDK's `useChat` with file system integration

## Tech Stack

- Next.js 15 (App Router, Turbopack)
- React 19
- Tailwind CSS v4
- Prisma with SQLite
- shadcn/ui components (new-york style)
- Vercel AI SDK with Anthropic provider

## Path Aliases

`@/` maps to `src/` (configured in tsconfig.json and components.json)

## Development Best Practices

- Don't use any comments in code
