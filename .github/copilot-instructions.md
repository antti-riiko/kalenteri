# MCP Calendar Lab - AI Coding Assistant Instructions

## Project Overview

This is an Express.js application implementing a voice-controlled calendar assistant using the Model Context Protocol (MCP). The app consists of:

- **MCP Server**: Exposes calendar tools (create, list events) backed by CalDAV
- **MCP Client**: Orchestrates OpenAI API calls with tool calling for natural language processing
- **Voice Interface**: Records audio, transcribes via Whisper, and provides text-to-speech responses
- **CalDAV Integration**: Stores calendar data using the tsdav library

## Architecture Patterns

### MCP Tool Registration

Register tools in `src/mcp-server/index.ts` using Zod schemas for input validation:

```typescript
mcpServer.registerTool(
  "toolName",
  {
    title: "Human-readable title",
    description: "Tool purpose and usage",
    inputSchema: z.object({...}).strict(),
  },
  async (input) => {
    // Tool implementation
    return { content: [{ type: "text", text: "Result" }] };
  },
);
```

### API Route Structure

REST endpoints follow `/api/v1/{resource}` pattern with controllers in `src/api/v1/controllers/` and routes in `src/api/v1/routes/`. Use Express middleware for cross-cutting concerns.

### Date/Time Handling

- Use Luxon `DateTime` for timezone conversions
- Store all dates as UTC in CalDAV/iCal format
- Convert local times to UTC using `toUtc()` utility from `src/utils/ical-lib.ts`
- Input schemas expect local date/time + explicit IANA timezone

### Error Handling

Throw `CustomError` instances with message and status code. The error middleware in `src/middlewares.ts` handles formatting and HTTP responses.

## Development Workflow

### Environment Setup

Create `.env` file with:

- `MCP_SERVER_URL`: Points to MCP server endpoint (often `http://localhost:3000/api/v1/mcp`)
- `OPENAI_PROXY_URL`: Base URL for OpenAI-compatible API proxy
- `CALDAV_SERVER_URL`: CalDAV server URL (defaults to `http://localhost:5232/`)
- `OPENAI_MODEL`: Model name (defaults to `gpt-4o`)

### Commands

- `npm run dev`: Start development server with hot reload
- `npm run build`: Compile TypeScript to `dist/` with path alias resolution
- `npm start`: Run production server from `dist/index.js`
- `npm test`: Run Jest tests

### Testing

- Integration tests in `tests/` directory
- Test CalDAV operations by creating/listing/deleting events
- Mock external dependencies for unit tests

## Key Conventions

### Path Aliases

Use `@/*` imports for `src/` directory. Configured in `tsconfig.json` and resolved by `tsc-alias`.

### Fetch Utility

Use `fetchData` from `src/utils/fetchData.ts` for API calls. It handles JSON parsing and error responses automatically.

### File Uploads

Audio transcription uses Multer middleware. Files are stored in memory and cleaned up after processing.

### iCal Generation

Events are created as iCal strings using `generateICal()` from `src/utils/ical-lib.ts`. All times stored as UTC.

### MCP Client Orchestration

The MCP client in `src/mcp-client/index.ts` implements a tool-calling loop:

1. List available tools from MCP server
2. Send prompt + tools to OpenAI API
3. Execute tool calls and append results
4. Repeat until no more tool calls or max rounds reached

## Integration Points

### CalDAV

- Singleton client instance in `src/calDav/calendarClient.ts`
- Operations: create, list, delete events
- Uses primary calendar from authenticated account

### OpenAI Proxy

- Chat completions: `POST /v1/chat/completions` with tools array
- Audio transcription: `POST /v1/audio/transcriptions` with FormData
- Authentication handled by proxy service

### MCP Transport

- Server: `StreamableHTTPServerTransport` for HTTP-based MCP
- Client: `StreamableHTTPClientTransport` for tool execution

## Common Patterns

### Adding New MCP Tools

1. Define Zod schema in `src/mcp-server/index.ts`
2. Register tool with title, description, and handler
3. Update MCP client's system prompt if needed
4. Add corresponding CalDAV operation if required

### Voice Processing Pipeline

1. Frontend records audio as WebM blob
2. `transcribeAudio` middleware sends to Whisper API
3. Transcribed text becomes `req.body.prompt`
4. MCP client processes prompt with tool calling
5. Response returned as JSON with `answer` and `toolCalls` count

### Timezone Handling

- Always accept local time + IANA timezone in APIs
- Convert to UTC for storage using Luxon
- Display times in user's local timezone in responses</content>
  <parameter name="filePath">c:\Users\riiko\kalenteri\mcp-lab-starter\.github\copilot-instructions.md
