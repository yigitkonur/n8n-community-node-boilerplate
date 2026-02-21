# n8n Custom Node Development: AI Agent Rules

> **Version**: 2.0.0  
> **Last Updated**: 2025-11-29  
> **Purpose**: Comprehensive guidance for AI coding assistants building n8n custom nodes.

---

## 1. PROJECT OVERVIEW

This is the official n8n custom node starter repository. It teaches AI assistants how to build custom n8n community nodes using real-world examples, comprehensive documentation, and established patterns. The repository contains working node implementations, credential configurations, and 31 documentation files covering all aspects of n8n node development.

**Key Stats**: This approach powers 1,500+ community nodes with millions of downloads (and growing rapidly).

---

## 2. TECHNOLOGY STACK

### Core Stack

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Runtime** | Node.js | v20.15.0+ (v22 LTS also supported) | JavaScript runtime |
| **Language** | TypeScript | 5.6+ | Static typing for reliability |
| **Framework** | n8n Node SDK | Latest | Node development framework |
| **CLI Tool** | @n8n/node-cli | Latest | Build tooling, dev server |

### Recommended DX Stack (Maximum Developer Experience)

| Tool | Version | Purpose | Why |
|------|---------|---------|-----|
| **Vite** | 5.x | Bundler/Dev Server | Fast HMR; native ESM |
| **Vitest** | 1.x | Test Framework | Fast; native ESM; better DX |
| **Zod** | 3.x | Runtime Validation | Type-safe param validation with UX feedback |
| **ESLint + Prettier** | Latest | Code Quality | Enforces consistency |
| **Husky + lint-staged** | Latest | Pre-commit | Prevents bad commits reaching main |
| **semantic-release** | Latest | Versioning | Auto-semver + changelog + npm publish |
| **pnpm** | 9.x | Package Manager | Fast installs; strict peer deps |
| **TurboRepo** | Latest | Build Caching | Speeds up CI on subsequent runs |

---

## 3. PROJECT STRUCTURE

### Standard Structure (This Repository)

```
n8n-nodes-starter/
├── credentials/                    # Authentication configurations
│   ├── GithubIssuesApi.credentials.ts      # API key auth example
│   └── GithubIssuesOAuth2Api.credentials.ts # OAuth2 auth example
├── nodes/                          # Node implementations
│   ├── Example/                    # Basic programmatic node
│   │   ├── Example.node.ts
│   │   └── Example.node.json
│   └── GithubIssues/              # Advanced declarative node
│       ├── GithubIssues.node.ts   # Main entry point
│       ├── GithubIssues.node.json # Node metadata
│       ├── resources/             # Resource-based organization
│       │   ├── issue/             # Issue CRUD operations
│       │   └── issueComment/      # Comment operations
│       ├── listSearch/            # Dynamic dropdown methods
│       └── shared/                # Reusable utilities
│           ├── transport.ts       # API request wrapper
│           └── descriptions.ts    # UI component definitions
├── docs/                          # 31 documentation files (00-30)
├── icons/                         # Node icons (SVG)
├── package.json                   # Node registration & metadata
└── tsconfig.json                  # TypeScript configuration
```

### Scalable Monorepo Structure (50+ Nodes)

```
my-n8n-nodes/
├── packages/
│   ├── nodes-base/
│   │   ├── MyApiNode/
│   │   │   ├── src/
│   │   │   │   ├── MyApiNode.node.ts       # Main INodeType implementation
│   │   │   │   ├── description.ts          # INodeTypeDescription with properties
│   │   │   │   ├── GenericFunctions.ts     # Shared helpers
│   │   │   │   ├── services/
│   │   │   │   │   ├── TableOperations.ts
│   │   │   │   │   ├── UserOperations.ts
│   │   │   │   │   └── ApiClient.ts
│   │   │   │   └── credentials/
│   │   │   │       └── MyApiCredentials.credentials.ts
│   │   │   ├── __tests__/
│   │   │   │   ├── MyApiNode.test.ts
│   │   │   │   └── services/TableOperations.test.ts
│   │   │   ├── vite.config.ts
│   │   │   └── package.json
│   │   └── MyTrigger/
│   │       └── ...
├── .vscode/settings.json
├── .github/workflows/
│   ├── ci.yml
│   ├── release.yml
│   └── test.yml
├── tsconfig.json                           # Shared TypeScript config
├── package.json                            # Root monorepo config
├── pnpm-workspace.yaml
└── turbo.json                              # Build caching config
```

---

## 4. NODE DEVELOPMENT DECISION TREE

When starting a new n8n node, follow this decision tree:

```
Is it a REST API with standard CRUD operations?
├─ YES → Use DECLARATIVE style
│  └─ Read: docs/12-declarative-routing.md
│  └─ Reference: nodes/GithubIssues/GithubIssues.node.ts
│
├─ Does it have an official SDK/client library?
│  └─ YES → Use PROGRAMMATIC style
│     └─ Read: docs/13-custom-execute-methods.md
│     └─ Reference: nodes/Example/Example.node.ts
│
├─ Is it GraphQL, WebSocket, or non-REST?
│  └─ YES → Use PROGRAMMATIC style
│
└─ Complex authentication (request signing, multi-step)?
   └─ YES → Use PROGRAMMATIC style
```

---

## 5. CODE PATTERNS

### Pattern A: Declarative REST API Node

**When to use**: Standard REST APIs with CRUD operations

**File structure**:
```bash
nodes/MyService/
├── MyService.node.ts              # Main entry point
├── MyService.node.json            # Metadata
├── myservice.svg                  # Icon
├── resources/                     # One folder per resource
│   ├── user/
│   │   ├── index.ts              # Operations + common fields
│   │   ├── create.ts             # Create operation
│   │   ├── get.ts                # Get operation
│   │   ├── getAll.ts             # List with pagination
│   │   ├── update.ts             # Update operation
│   │   └── delete.ts             # Delete operation
│   └── project/
│       └── [same structure]
├── listSearch/                    # Dynamic dropdowns
└── shared/
    ├── descriptions.ts            # Reusable UI components
    ├── transport.ts               # API request wrapper
    └── utils.ts                   # Helper functions
```

**Main node structure**:
```typescript
// nodes/YourService/YourService.node.ts
import type { INodeType, INodeTypeDescription } from 'n8n-workflow';
import { userDescription } from './resources/user';
import { projectDescription } from './resources/project';

export class YourService implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Your Service',
    name: 'yourService',
    icon: 'file:yourservice.svg',
    group: ['transform'],
    version: 1,
    subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}',
    description: 'Interact with Your Service API',
    defaults: { name: 'Your Service' },
    inputs: ['main'],
    outputs: ['main'],
    credentials: [
      { name: 'yourServiceApi', required: true }
    ],
    requestDefaults: {
      baseURL: 'https://api.yourservice.com/v1',
      headers: { 'Content-Type': 'application/json' }
    },
    properties: [
      {
        displayName: 'Resource',
        name: 'resource',
        type: 'options',
        noDataExpression: true,
        options: [
          { name: 'User', value: 'user' },
          { name: 'Project', value: 'project' }
        ],
        default: 'user',
      },
      ...userDescription,
      ...projectDescription,
    ],
  };
}
```

**Routing types**:
- `routing.send.type: 'body'` → Request body
- `routing.send.type: 'query'` → URL parameter
- `routing.send.type: 'header'` → HTTP header
- `routing.output.maxResults: '={{$value}}'` → Limit results

---

### Pattern B: Resource-Operation Dispatch (Programmatic)

**When to use**: Complex logic, SDK integration, non-REST protocols (Used by Supabase, Monday, Notion)

```typescript
// MyApiNode.node.ts
export class MyApiNode implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My API',
    name: 'myApi',
    group: ['transform'],
    version: 1,
    properties: [
      {
        displayName: 'Resource',
        name: 'resource',
        type: 'options',
        options: [
          { name: 'Table', value: 'table' },
          { name: 'User', value: 'user' }
        ],
        default: 'table'
      },
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        displayOptions: { show: { resource: ['table'] } },
        options: [
          { name: 'Get All', value: 'getAll' },
          { name: 'Create', value: 'create' }
        ],
        default: 'getAll'
      }
    ]
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const resource = this.getNodeParameter('resource', 0) as string;
    const operation = this.getNodeParameter('operation', 0) as string;

    // Dispatch to service
    if (resource === 'table') {
      if (operation === 'getAll') {
        return await new TableOperations().getAll.call(this);
      } else if (operation === 'create') {
        return await new TableOperations().create.call(this);
      }
    } else if (resource === 'user') {
      if (operation === 'getAll') {
        return await new UserOperations().getAll.call(this);
      }
    }

    throw new Error(`Unknown resource: ${resource}, operation: ${operation}`);
  }
}
```

**Service File Pattern**:

```typescript
// services/TableOperations.ts
export class TableOperations {
  async getAll(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const credentials = await this.getCredentials('myApi') as ICredentials;
    const tableId = this.getNodeParameter('tableId', 0) as string;

    try {
      const response = await this.helpers.httpRequest({
        method: 'GET',
        url: `https://api.myservice.com/tables/${tableId}/rows`,
        headers: {
          'Authorization': `Bearer ${credentials.apiKey}` 
        },
        qs: {
          limit: 100,
          offset: 0
        }
      });

      return this.helpers.returnJsonArray(response.data);
    } catch (error) {
      if (this.helpers.isNodeApiError(error)) {
        throw error;
      }
      throw new NodeOperationError(this.getNode(), error as Error);
    }
  }

  async create(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const credentials = await this.getCredentials('myApi');
    const tableId = this.getNodeParameter('tableId', 0) as string;
    const items = this.getInputData();

    const created = [];
    for (const item of items) {
      const response = await this.helpers.httpRequest({
        method: 'POST',
        url: `https://api.myservice.com/tables/${tableId}/rows`,
        headers: { 'Authorization': `Bearer ${credentials.apiKey}` },
        body: item.json
      });
      created.push({ json: response.data });
    }

    return [created];
  }
}
```

---

### Pattern C: Resource Locator (Multi-Mode Selection)

```typescript
export const userSelect: INodeProperties = {
  displayName: 'User',
  name: 'userId',
  type: 'resourceLocator',
  default: { mode: 'list', value: '' },
  required: true,
  modes: [
    {
      displayName: 'From List',
      name: 'list',
      type: 'list',
      placeholder: 'Select a user...',
      typeOptions: {
        searchListMethod: 'getUsers',
        searchable: true,
        searchFilterRequired: false
      }
    },
    {
      displayName: 'By URL',
      name: 'url',
      type: 'string',
      placeholder: 'https://example.com/users/123',
      extractValue: {
        type: 'regex',
        regex: '/users/([0-9]+)'
      }
    },
    {
      displayName: 'By ID',
      name: 'id',
      type: 'string',
      placeholder: '123'
    }
  ]
};
```

---

### Pattern D: Cursor-Based Pagination

```typescript
// services/PaginatedOperations.ts
export class PaginatedOperations {
  async getAll(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const credentials = await this.getCredentials('myApi');
    const limit = (this.getNodeParameter('limit', 0) as number) || 100;
    
    let cursor: string | undefined;
    const allItems: IDataObject[] = [];

    while (true) {
      const response = await this.helpers.httpRequest({
        method: 'GET',
        url: 'https://api.myservice.com/items',
        headers: { 'Authorization': `Bearer ${credentials.apiKey}` },
        qs: {
          limit,
          cursor: cursor || undefined
        }
      });

      allItems.push(...response.data.items);

      if (!response.data.nextCursor || allItems.length >= limit) {
        break;
      }

      cursor = response.data.nextCursor;
      // Small delay to respect rate limits
      await new Promise(resolve => setTimeout(resolve, 100));
    }

    return [allItems.map(item => ({ json: item }))];
  }
}
```

---

### Pattern E: Batching for Performance

```typescript
async batchCreate(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const items = this.getInputData();
  const batchSize = 50;
  const results: IDataObject[] = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);

    // Process batch in parallel
    const batchResults = await Promise.allSettled(
      batch.map(item =>
        this.helpers.httpRequest({
          method: 'POST',
          url: 'https://api.myservice.com/items',
          body: item.json
        })
      )
    );

    // Collect results (including errors)
    batchResults.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        results.push(result.value.data);
      } else {
        results.push({ error: result.reason.message, itemIndex: i + index });
      }
    });

    // Small delay between batches to respect rate limits
    if (i + batchSize < items.length) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }

  return [results.map(r => ({ json: r }))];
}
```

---

### Pattern F: Webhook Triggers

```typescript
// MyTrigger.node.ts
export class MyTrigger implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My API Trigger',
    name: 'myApiTrigger',
    group: ['trigger'],
    version: 1,
    webhooks: [
      {
        name: 'default',
        httpMethod: 'POST',
        responseMode: 'onReceived',
        path: 'webhook',
      },
    ],
    properties: []
  };

  webhookMethods = {
    default: {
      async checkExists(this: IHookFunctions): Promise<boolean> {
        // Check if webhook already exists
        return false;
      },
      async create(this: IHookFunctions): Promise<boolean> {
        // Register webhook with external service
        const webhookUrl = this.getNodeWebhookUrl('default');
        // ... register with API
        return true;
      },
      async delete(this: IHookFunctions): Promise<boolean> {
        // Unregister webhook from external service
        return true;
      },
    },
  };
}
```

---

## 6. TYPE SAFETY & VALIDATION

### Zod Runtime Validation Pattern

```typescript
// src/types.ts
import { z } from 'zod';

export const MyApiCredentialsSchema = z.object({
  apiKey: z.string().min(1, 'API Key is required'),
  baseUrl: z.string().url().optional()
});

export type MyApiCredentials = z.infer<typeof MyApiCredentialsSchema>;

export const TableOperationSchema = z.object({
  resource: z.literal('table'),
  operation: z.enum(['getAll', 'create', 'update', 'delete']),
  tableId: z.string().min(1),
  limit: z.number().min(1).max(1000).default(100),
  offset: z.number().min(0).default(0)
});

export type TableOperation = z.infer<typeof TableOperationSchema>;
```

**Using in Node:**

```typescript
// src/services/TableOperations.ts
import { TableOperationSchema, MyApiCredentialsSchema } from '../types';

export class TableOperations {
  async getAll(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    try {
      // Validate parameters
      const params = TableOperationSchema.parse({
        resource: this.getNodeParameter('resource', 0),
        operation: this.getNodeParameter('operation', 0),
        tableId: this.getNodeParameter('tableId', 0),
        limit: this.getNodeParameter('limit', 0),
        offset: this.getNodeParameter('offset', 0)
      });

      // Validate credentials
      const rawCreds = await this.getCredentials('myApi');
      const credentials = MyApiCredentialsSchema.parse(rawCreds);

      // Now parameters and credentials are type-safe
      const response = await this.helpers.httpRequest({
        method: 'GET',
        url: `${credentials.baseUrl || 'https://api.myservice.com'}/tables/${params.tableId}`,
        qs: {
          limit: params.limit,
          offset: params.offset
        }
      });

      return this.helpers.returnJsonArray(response.data);
    } catch (error) {
      if (error instanceof z.ZodError) {
        // User-friendly error message
        const messages = error.errors.map(e => `${e.path.join('.')}: ${e.message}`);
        throw new NodeOperationError(this.getNode(), `Invalid parameters: ${messages.join('; ')}`);
      }
      throw error;
    }
  }
}
```

---

## 7. CREDENTIALS PATTERNS

### API Key Authentication

```typescript
// credentials/YourServiceApi.credentials.ts
import type { ICredentialType, INodeProperties } from 'n8n-workflow';

export class YourServiceApi implements ICredentialType {
  name = 'yourServiceApi';
  displayName = 'Your Service API';
  documentationUrl = 'https://docs.yourservice.com';
  
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
    },
  ];

  authenticate = {
    type: 'generic',
    properties: {
      headers: {
        Authorization: '=Bearer {{$credentials.apiKey}}',
      },
    },
  };
}
```

### OAuth2 Authentication

Reference: `docs/09-oauth2-credentials.md` and `credentials/GithubIssuesOAuth2Api.credentials.ts`

---

## 8. TESTING STRATEGY (90%+ Coverage Target)

### Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import path from 'path';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      include: ['src/**/*.ts'],
      exclude: ['src/**/*.test.ts'],
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    },
    setupFiles: ['./test/setup.ts']
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  }
});
```

### Unit Test Pattern

```typescript
// __tests__/MyApiNode.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import type { IExecuteFunctions } from 'n8n-workflow';
import { TableOperations } from '../src/services/TableOperations';

describe('TableOperations', () => {
  let mockExecuteFunctions: Partial<IExecuteFunctions>;

  beforeEach(() => {
    mockExecuteFunctions = {
      getNodeParameter: vi.fn((param) => {
        const params = {
          resource: 'table',
          operation: 'getAll',
          tableId: 'tbl_123'
        };
        return params[param as keyof typeof params];
      }),
      getCredentials: vi.fn().mockResolvedValue({ apiKey: 'test-key' }),
      helpers: {
        httpRequest: vi.fn(),
        returnJsonArray: vi.fn((data) => [{ json: data }]),
        isNodeApiError: vi.fn(() => false)
      },
      getNode: vi.fn(() => ({ name: 'test' })),
      getInputData: vi.fn(() => [])
    };
  });

  it('fetches all table rows', async () => {
    const mockData = [{ id: 1, name: 'Row 1' }];
    (mockExecuteFunctions.helpers!.httpRequest as any).mockResolvedValue({
      data: mockData
    });

    const operations = new TableOperations();
    const result = await operations.getAll.call(mockExecuteFunctions as IExecuteFunctions);

    expect(mockExecuteFunctions.helpers!.httpRequest).toHaveBeenCalledWith(
      expect.objectContaining({
        method: 'GET',
        url: expect.stringContaining('/tables/tbl_123/rows')
      })
    );
  });
});
```

---

## 9. DEVELOPMENT WORKFLOW

### Local Development (Native)

```bash
# Clone and setup
git clone https://github.com/yigitkonur/n8n-node-boilerplate.git my-nodes
cd my-nodes
pnpm install
pnpm dev  # Starts dev server with hot reload

# Access n8n at http://localhost:5678
```

### Docker Development (Production Parity)

```yaml
# docker-compose.yml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    volumes:
      - ./dist:/home/node/.n8n/custom/MyApiNode
      - n8n_data:/root/.n8n
    environment:
      - N8N_CUSTOM_EXTENSIONS=/home/node/.n8n/custom
      - NODE_ENV=development
    command: n8n start

volumes:
  n8n_data:
```

### VS Code Configuration

**.vscode/settings.json:**
```json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    }
  }
}
```

### Debugging in VS Code

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Vitest Debug",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "pnpm",
      "runtimeArgs": ["test", "--inspect-brk", "--watch"],
      "console": "integratedTerminal"
    }
  ]
}
```

---

## 10. PERFORMANCE CHECKLIST

| Aspect | Best Practice | Implementation |
|--------|---------------|-----------------|
| **API Requests** | Use native n8n `httpRequest` helper | Automatic retry + timeout |
| **Pagination** | Cursor-based, configurable limit | Loop until no cursor, max 1000 per call |
| **Batching** | `Promise.allSettled()` for parallel ops | Process 50-100 items at once |
| **Caching** | Cache credentials/auth tokens | Store in execution scope |
| **Memory** | Stream items instead of buffering | Use `yield` for large datasets |
| **Rate Limiting** | Respect API limits with delays | Add `setTimeout(100)` between calls |
| **Timeouts** | Set explicit timeouts | `httpRequest` options: `timeout: 30000` |
| **Error Handling** | Granular try-catch per operation | Return partial results with errors |

---

## 11. PUBLISHING WORKFLOW

### Semantic Release with GitHub Actions

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'pnpm'
      
      - run: pnpm ci
      - run: pnpm test --coverage
      - run: pnpm build
      
      - uses: cycjimmy/semantic-release-action@v3
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Package.json for Publishing

```json
{
  "name": "@myorg/n8n-nodes-myapi",
  "version": "0.1.0",
  "description": "n8n node for MyAPI",
  "keywords": ["n8n-community-node-package"],
  "repository": "github:myorg/n8n-nodes-myapi",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "n8n": {
    "n8nNodesApiVersion": 1,
    "nodes": ["dist/nodes/MyApi/MyApi.node.js"],
    "credentials": ["dist/credentials/MyApiApi.credentials.js"]
  }
}
```

---

## 12. DOCUMENTATION NAVIGATION

### Complete Documentation Tree (31 Files)

```
docs/
├── docs/00-documentation-index.md          # Master index and navigation
├── docs/01-project-structure-overview.md   # Repository layout explained
├── docs/02-package-json-configuration.md   # npm package setup for n8n
├── docs/03-typescript-configuration.md     # tsconfig.json settings
├── docs/04-node-anatomy-and-architecture.md # Node structure deep dive
├── docs/05-declarative-vs-programmatic-nodes.md # Architecture choice guide
├── docs/06-node-properties-reference.md    # All property types explained
├── docs/07-credentials-system-overview.md  # Auth architecture
├── docs/08-api-key-credentials.md          # API key implementation
├── docs/09-oauth2-credentials.md           # OAuth2 implementation
├── docs/10-creating-your-first-node.md     # Step-by-step tutorial
├── docs/11-resources-and-operations.md     # Multi-resource organization
├── docs/12-declarative-routing.md          # routing.send patterns
├── docs/13-custom-execute-methods.md       # execute() implementation
├── docs/14-list-search-methods.md          # Dynamic dropdown methods
├── docs/15-resource-locators.md            # Multi-mode input fields
├── docs/16-pagination-handling.md          # Cursor/offset pagination
├── docs/17-error-handling-patterns.md      # Error handling best practices
├── docs/18-helper-functions-and-utilities.md # Reusable utilities
├── docs/19-external-sdk-integration.md     # Third-party SDK patterns
├── docs/20-icons-and-branding.md           # SVG icon requirements
├── docs/21-node-json-metadata.md           # node.json configuration
├── docs/22-development-workflow.md         # Dev server and testing
├── docs/23-testing-strategies.md           # Unit and integration tests
├── docs/24-linting-and-code-quality.md     # ESLint configuration
├── docs/25-preparing-for-publication.md    # Pre-publish checklist
├── docs/26-publishing-to-npm.md            # npm publish workflow
├── docs/27-n8n-cloud-verification.md       # Cloud verification process
├── docs/28-complete-code-examples.md       # Full working examples
├── docs/29-common-patterns-and-recipes.md  # Reusable code patterns
└── docs/30-troubleshooting-guide.md        # Common issues and fixes
```

### Quick Reference by Task

| Task | Primary Doc | Secondary Doc |
|------|-------------|---------------|
| **Getting Started** | `docs/10-creating-your-first-node.md` | `docs/00-documentation-index.md` |
| **Choose Architecture** | `docs/05-declarative-vs-programmatic-nodes.md` | `docs/04-node-anatomy-and-architecture.md` |
| **REST API Node** | `docs/12-declarative-routing.md` | `docs/06-node-properties-reference.md` |
| **SDK Integration** | `docs/13-custom-execute-methods.md` | `docs/19-external-sdk-integration.md` |
| **API Key Auth** | `docs/08-api-key-credentials.md` | `docs/07-credentials-system-overview.md` |
| **OAuth2 Auth** | `docs/09-oauth2-credentials.md` | `docs/07-credentials-system-overview.md` |
| **Multi-Resource** | `docs/11-resources-and-operations.md` | `docs/04-node-anatomy-and-architecture.md` |
| **Dynamic Dropdowns** | `docs/14-list-search-methods.md` | `docs/15-resource-locators.md` |
| **Pagination** | `docs/16-pagination-handling.md` | `docs/12-declarative-routing.md` |
| **Error Handling** | `docs/17-error-handling-patterns.md` | `docs/30-troubleshooting-guide.md` |
| **Testing** | `docs/23-testing-strategies.md` | `docs/22-development-workflow.md` |
| **Publishing** | `docs/25-preparing-for-publication.md` | `docs/26-publishing-to-npm.md` |

---

## 13. DEVELOPMENT COMMANDS

```bash
# Install dependencies
npm install

# Development with hot reload
npm run dev

# Build for production
npm run build

# Lint code
npm run lint

# Auto-fix lint issues
npm run lint:fix

# Run tests
npm run test

# Run tests with coverage
npm run test:coverage

# Create release
npm run release
```

---

## 14. COMMON PITFALLS & SOLUTIONS

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| **No error handling** | Node crashes on API error | Use `try-catch` + `NodeOperationError` |
| **Hardcoded credentials** | Security vulnerability | Always use `getCredentials()` |
| **Synchronous operations** | Node hangs n8n | Ensure all operations are `async` |
| **Large arrays in memory** | OOM on big datasets | Use pagination/streaming patterns |
| **No pagination** | API rate limits hit | Implement cursor/offset pagination |
| **Missing TypeScript types** | Runtime errors in production | Use strict TypeScript + Zod validation |
| **Tests in production** | Bloated bundle | Exclude `**/*.test.ts` from build |
| **Timeout issues** | Long-running ops fail | Set explicit timeouts, use queue mode |
| **Initialize SDK per item** | Poor performance | Initialize SDK ONCE before loop |
| **Forget `pairedItem`** | Broken item linking | Always include `pairedItem: { item: i }` |

---

## 15. PACKAGE.JSON REGISTRATION

**CRITICAL**: All nodes and credentials MUST be registered:

```json
{
  "name": "n8n-nodes-yourservice",
  "n8n": {
    "n8nNodesApiVersion": 1,
    "nodes": [
      "dist/nodes/YourService/YourService.node.js"
    ],
    "credentials": [
      "dist/credentials/YourServiceApi.credentials.js"
    ]
  }
}
```

---

## 16. FILE REFERENCES

When building nodes, reference these key files:

- **Declarative node**: `nodes/GithubIssues/GithubIssues.node.ts`
- **Programmatic node**: `nodes/Example/Example.node.ts`
- **API key credential**: `credentials/GithubIssuesApi.credentials.ts`
- **OAuth2 credential**: `credentials/GithubIssuesOAuth2Api.credentials.ts`
- **Resource organization**: `nodes/GithubIssues/resources/issue/`
- **List search methods**: `nodes/GithubIssues/listSearch/`
- **Transport layer**: `nodes/GithubIssues/shared/transport.ts`
- **UI descriptions**: `nodes/GithubIssues/shared/descriptions.ts`

---

## 17. QUICK START CHECKLIST

```bash
# 1. Initialize project (5 min)
git clone https://github.com/yigitkonur/n8n-node-boilerplate.git my-nodes
cd my-nodes && pnpm install

# 2. Create your first node (10 min)
mkdir -p nodes/MyApiNode
# Implement MyApiNode.node.ts + description.ts

# 3. Add credentials (5 min)
# Create credentials/MyApiCredentials.credentials.ts

# 4. Write tests (10 min)
# Create __tests__/MyApiNode.test.ts

# 5. Start development (2 min)
pnpm dev  # Watch mode

# 6. Test locally (5 min)
# Open http://localhost:5678

# 7. Publish to npm (3 min)
# Set up semantic-release + GitHub Actions

# Total: ~45 minutes to production-ready node
```

---

## 18. KEY TAKEAWAYS

✅ **Use Vite + Vitest + TypeScript + Zod** for maximum DX and quality  
✅ **Docker for local dev** matches production environment  
✅ **Resource-operation dispatch** scales to 100+ operations  
✅ **Service-based organization** enables testing and maintenance  
✅ **90%+ test coverage** prevents production bugs  
✅ **Pagination/batching** essential for scalability  
✅ **Semantic versioning** + GitHub Actions automates releases  
✅ **Comprehensive docs** drive adoption and contributions
