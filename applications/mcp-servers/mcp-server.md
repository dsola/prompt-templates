# MCP (Model Context Protocol) Server Template

## Purpose
This template provides comprehensive guidance for building MCP servers that expose tools and resources to AI assistants and other clients.

## Use Case
Use when creating servers that need to provide tools, prompts, or resources to language models through the Model Context Protocol standard.

## Template

```
Create an MCP server with the following specifications:

## Server Overview
[Describe the server's purpose, capabilities it exposes, and use cases]

## Technical Requirements
- Language/SDK: [TypeScript/Node.js / Python]
- MCP SDK: [@modelcontextprotocol/sdk]
- Transport: [stdio / SSE (Server-Sent Events)]
- Runtime: [Node.js 18+ / Python 3.10+]
- Development Tools: [TypeScript / Python type hints]

## Architecture Principles
1. Follow MCP protocol specifications strictly
2. Implement clear tool schemas with proper types
3. Provide comprehensive descriptions for tools
4. Handle errors gracefully with clear messages
5. Implement proper validation for all inputs
6. Support proper capability negotiation
7. Log operations for debugging

## Project Structure

### TypeScript/Node.js
```
mcp-server-name/
├── src/
│   ├── index.ts           # Server entry point
│   ├── tools/             # Tool implementations
│   │   ├── tool1.ts
│   │   └── tool2.ts
│   ├── resources/         # Resource providers
│   │   └── resources.ts
│   ├── prompts/           # Prompt templates
│   │   └── prompts.ts
│   ├── types/             # TypeScript types
│   └── utils/             # Utility functions
├── package.json
├── tsconfig.json
└── README.md
```

### Python
```
mcp-server-name/
├── src/
│   ├── __init__.py
│   ├── server.py          # Server entry point
│   ├── tools/             # Tool implementations
│   │   ├── __init__.py
│   │   ├── tool1.py
│   │   └── tool2.py
│   ├── resources/         # Resource providers
│   └── prompts/           # Prompt templates
├── pyproject.toml
└── README.md
```

## Server Implementation

### TypeScript Example
```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
  Tool,
} from '@modelcontextprotocol/sdk/types.js';

// Define your tools
const tools: Tool[] = [
  {
    name: 'example_tool',
    description: 'A clear description of what this tool does',
    inputSchema: {
      type: 'object',
      properties: {
        parameter1: {
          type: 'string',
          description: 'Description of parameter1',
        },
        parameter2: {
          type: 'number',
          description: 'Description of parameter2',
          optional: true,
        },
      },
      required: ['parameter1'],
    },
  },
];

// Create server
const server = new Server(
  {
    name: 'example-mcp-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
      resources: {},
      prompts: {},
    },
  }
);

// Handle tool listing
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return { tools };
});

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case 'example_tool': {
      const result = await executeExampleTool(args);
      return {
        content: [
          {
            type: 'text',
            text: JSON.stringify(result, null, 2),
          },
        ],
      };
    }
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error('MCP Server running on stdio');
}

main().catch(console.error);
```

### Python Example
```python
import asyncio
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

# Define your tools
TOOLS = [
    Tool(
        name="example_tool",
        description="A clear description of what this tool does",
        inputSchema={
            "type": "object",
            "properties": {
                "parameter1": {
                    "type": "string",
                    "description": "Description of parameter1",
                },
                "parameter2": {
                    "type": "number",
                    "description": "Description of parameter2",
                },
            },
            "required": ["parameter1"],
        },
    )
]

# Create server
app = Server("example-mcp-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return TOOLS

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "example_tool":
        result = await execute_example_tool(arguments)
        return [TextContent(type="text", text=str(result))]
    else:
        raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(
            read_stream,
            write_stream,
            app.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

## Key Features to Implement

### 1. Tools (Primary Feature)
Tools are functions that the AI can call to perform actions:

```typescript
// Example: File system tool
{
  name: 'read_file',
  description: 'Read contents of a file from the filesystem',
  inputSchema: {
    type: 'object',
    properties: {
      path: {
        type: 'string',
        description: 'Absolute or relative path to the file'
      }
    },
    required: ['path']
  }
}

// Implementation
async function readFile(args: { path: string }) {
  const content = await fs.readFile(args.path, 'utf-8');
  return {
    content: [{
      type: 'text',
      text: content
    }]
  };
}
```

### 2. Resources (Optional)
Resources expose data that can be read by clients:

```typescript
// List available resources
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: 'file://project/README.md',
        name: 'Project README',
        mimeType: 'text/markdown',
        description: 'Main project documentation'
      }
    ]
  };
});

// Read a resource
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const uri = request.params.uri;
  // Fetch and return resource content
});
```

### 3. Prompts (Optional)
Prompts provide reusable prompt templates:

```typescript
server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: [
      {
        name: 'code_review',
        description: 'Generate a code review for a file',
        arguments: [
          {
            name: 'file_path',
            description: 'Path to the file to review',
            required: true
          }
        ]
      }
    ]
  };
});
```

### 4. Error Handling
```typescript
try {
  // Tool execution
  const result = await executeTool(args);
  return { content: [{ type: 'text', text: result }] };
} catch (error) {
  if (error instanceof ValidationError) {
    throw new Error(`Validation failed: ${error.message}`);
  }
  if (error instanceof NotFoundError) {
    throw new Error(`Resource not found: ${error.message}`);
  }
  throw new Error(`Internal error: ${error.message}`);
}
```

### 5. Logging & Debugging
```typescript
import { LogLevel } from '@modelcontextprotocol/sdk/types.js';

// Send log messages to client
server.notification({
  method: 'notifications/message',
  params: {
    level: LogLevel.Info,
    data: 'Processing request...'
  }
});
```

## Best Practices

### Tool Design
1. **Clear naming**: Use descriptive, action-oriented names (e.g., `search_files`, `execute_command`)
2. **Comprehensive descriptions**: Explain what the tool does, when to use it, and any limitations
3. **Strong typing**: Define complete JSON schemas for all parameters
4. **Validation**: Validate all inputs before execution
5. **Error messages**: Provide clear, actionable error messages

### Schema Definition
```typescript
// Good schema
{
  name: 'search_code',
  description: 'Search for code patterns across repository files using regex',
  inputSchema: {
    type: 'object',
    properties: {
      pattern: {
        type: 'string',
        description: 'Regular expression pattern to search for'
      },
      file_pattern: {
        type: 'string',
        description: 'Glob pattern to filter files (e.g., "**/*.ts")',
        optional: true
      },
      case_sensitive: {
        type: 'boolean',
        description: 'Whether search should be case-sensitive',
        default: false
      }
    },
    required: ['pattern']
  }
}
```

### Security Considerations
1. **Input validation**: Sanitize and validate all inputs
2. **Path traversal protection**: Prevent access outside allowed directories
3. **Rate limiting**: Implement limits for expensive operations
4. **Sandboxing**: Run in restricted environment when possible
5. **Authentication**: Implement auth if handling sensitive data

## Testing Strategy
```typescript
import { describe, it, expect } from 'vitest';

describe('MCP Server Tools', () => {
  it('should list all available tools', async () => {
    const response = await server.request({
      method: 'tools/list'
    });
    expect(response.tools).toHaveLength(expectedCount);
  });

  it('should execute tool with valid input', async () => {
    const response = await server.request({
      method: 'tools/call',
      params: {
        name: 'example_tool',
        arguments: { parameter1: 'value' }
      }
    });
    expect(response.content[0].text).toBeDefined();
  });

  it('should reject invalid tool input', async () => {
    await expect(server.request({
      method: 'tools/call',
      params: {
        name: 'example_tool',
        arguments: {} // Missing required parameter
      }
    })).rejects.toThrow();
  });
});
```

## Configuration & Distribution

### package.json (TypeScript)
```json
{
  "name": "mcp-server-example",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "mcp-server-example": "./dist/index.js"
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "prepare": "npm run build"
  }
}
```

### Client Configuration
```json
{
  "mcpServers": {
    "example": {
      "command": "node",
      "args": ["/path/to/mcp-server-example/dist/index.js"]
    }
  }
}
```

## Documentation Requirements
1. README with clear installation instructions
2. List of all tools with examples
3. Configuration options
4. Usage examples
5. Troubleshooting guide
```

## Example Prompt

```
Create an MCP server for interacting with a Git repository:

Tools to implement:
1. git_status - Get current repository status
2. git_diff - Get diff for staged/unstaged changes
3. git_log - Get commit history with filtering
4. git_show - Show details of a specific commit
5. git_branch - List/create/delete branches

Technical requirements:
- TypeScript with Node.js
- Use @modelcontextprotocol/sdk
- Stdio transport
- Proper error handling
- Input validation
- Comprehensive tool descriptions
- Support for both simple and advanced git operations

Include unit tests and comprehensive README documentation.
```

## References
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP Examples](https://github.com/modelcontextprotocol/servers)
- [JSON Schema](https://json-schema.org/)
