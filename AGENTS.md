# Agent Guide for Wekan MCP Server

## Project Overview

This is a **Model Context Protocol (MCP) server** that exposes Wekan board management functionality to AI agents. It uses TypeScript with strict settings and ES modules.

**Key Technologies:**
- TypeScript 5.x with strict type checking
- MCP SDK (`@modelcontextprotocol/sdk`)
- `undici` for HTTP requests
- `zod` for runtime validation
- `dotenv` for environment configuration

---

## Essential Commands

### Build & Run
```bash
# Compile TypeScript to dist/
npm run build

# Run the compiled server
npm run start

# Run in development mode (using ts-node)
npm run dev
```

### Running with npx
```bash
# Run directly from GitHub (builds automatically)
npx github:jonathan-dd/wekan-mcp

# Or install globally and run
npm install -g github:jonathan-dd/wekan-mcp
wekan-mcp
```

### MCP Inspector (Development/Debugging)
```bash
# Launch MCP Inspector with config
npm run inspect

# Build then launch inspector
npm run inspect:watch
```

### Testing
```bash
# Test authentication methods
node test-auth.js

# Test all configuration methods
node test-all-methods.js

# Basic server test
node test-server.js
```

---

## Project Structure

```
├── src/                      # Source files
│   ├── server.ts             # MCP server setup and tool registration
│   └── wekan.ts              # Wekan API client class
├── dist/                     # Compiled JavaScript (after build)
├── test-*.js                 # Test scripts
├── .env.example              # Environment template
├── mcp-inspector-config.json # MCP Inspector configuration
├── get-wekan-token.sh        # Linux/macOS token helper
├── get-wekan-token.ps1       # Windows token helper
└── package.json
```

---

## Configuration

The server requires environment variables to connect to a Wekan instance:

**Required:**
- `WEKAN_BASE_URL` - URL of the Wekan instance (e.g., `https://wekan.example.com`)

**Authentication (choose one):**
- `WEKAN_API_TOKEN` - Pre-generated API token (recommended)
- OR `WEKAN_USERNAME` + `WEKAN_PASSWORD` - Auto-generates token

**Setup:**
```bash
# Copy template
cp .env.example .env

# Or use helper script
./get-wekan-token.sh        # Linux/macOS
./get-wekan-token.ps1       # Windows
```

---

## Code Patterns & Conventions

### Module System
- **ES Modules** with `"type": "module"` in package.json
- Import TypeScript files with `.js` extension: `import { X } from "./file.js"`
- Output goes to `dist/` folder

### TypeScript Configuration
- **Strict mode enabled** - all strict options turned on
- Target: `esnext`, Module: `nodenext`
- Generates source maps and declaration files
- Enforces: `noImplicitReturns`, `noUnusedLocals`, `exactOptionalPropertyTypes`

### HTTP Client Pattern (`src/wekan.ts`)
The `Wekan` class wraps API calls using `undici`:

```typescript
// Request helpers handle auth automatically
async get(path: string): Promise<any>
async post(path: string, body: unknown): Promise<any>
async put(path: string, body: unknown): Promise<any>
```

### Tool Registration Pattern (`src/server.ts`)
Tools are registered with the MCP server using:

```typescript
server.tool("toolName", "Description", {
  paramName: z.string()  // zod validation
}, async (args) => {
  // Implementation
  return { content: [{ type: "text", text: JSON.stringify(result) }] };
});
```

### Naming Conventions
- Files: lowercase with dashes (`test-auth.js`)
- Classes: PascalCase (`Wekan`, `WekanBoard`)
- Interfaces: PascalCase with prefix (`WekanLoginResponse`)
- Private methods: camelCase with `private` modifier
- Environment variables: UPPER_SNAKE_CASE

---

## Authentication Flow

The `Wekan` class supports two authentication methods:

1. **API Token** (preferred): Pass `WEKAN_API_TOKEN` directly
2. **Username/Password**: Set `WEKAN_USERNAME` and `WEKAN_PASSWORD`, server logs in to get token

The client lazy-authenticates on first request - no explicit login call needed.

---

## MCP Tools Available

| Tool | Purpose |
|------|---------|
| `listBoards` | List all accessible Wekan boards |
| `listLists` | Get lists (columns) in a board |
| `listSwimlanes` | Get swimlanes in a board |
| `listCards` | Get cards in a specific list |
| `createCard` | Create a new card with optional metadata |
| `moveCard` | Move card to different list/swimlane |
| `commentCard` | Add comment to existing card |

All tools return JSON text content formatted as: `{ content: [{ type: "text", text: JSON.stringify(data) }] }`

---

## Important Gotchas

1. **Import Extensions**: Always use `.js` extension in TypeScript imports even though files are `.ts`
2. **Fail Fast**: Server throws on startup if config is missing - check stderr for auth errors
3. **Build First**: Must run `npm run build` before testing with `node test-*.js` (tests run against `dist/`)
4. **Environment Loading**: `import 'dotenv/config'` at top of `server.ts` loads `.env` automatically
5. **Trailing Slash**: Server strips trailing slash from `WEKAN_BASE_URL` automatically
6. **No Linting**: Project has no ESLint/Prettier configured - rely on TypeScript strict mode
7. **Binary Shebang**: Entry point (`src/server.ts`) must start with `#!/usr/bin/env node` for npx compatibility

---

## Adding New Tools

To add a new MCP tool:

1. Add API method to `Wekan` class in `src/wekan.ts`:
```typescript
async newMethod(): Promise<NewType> { 
  return this.get(`/api/endpoint`); 
}
```

2. Register tool in `src/server.ts`:
```typescript
server.tool("newTool", "Description", {
  param: z.string()
}, async (args) => {
  const result = await wekan.newMethod();
  return { content: [{ type: "text", text: JSON.stringify(result) }] };
});
```

3. Build and test:
```bash
npm run build
npm run inspect
```

---

## Testing Approach

- **Unit-style tests**: `test-auth.js` - tests auth configurations
- **Integration test**: `test-server.js` - spawns server and monitors output
- **Manual testing**: Use `npm run inspect` for interactive MCP tool testing

Tests spawn the server as a child process and monitor stdout/stderr - they don't import modules directly.
