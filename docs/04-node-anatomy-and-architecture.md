# 04 - Node Anatomy and Architecture

> Understanding INodeType and the complete node structure

---

## 🤖 AI Agent Context

**READ THIS DOCUMENT** to understand the complete node structure. Reference for all node configuration options.

| Section | What It Covers |
|---------|---------------|
| [INodeType Interface](#core-interface-inodetype) | Main class structure |
| [Description Fields](#description-fields-reference) | All metadata options |
| [Credentials Config](#credentials-configuration) | Auth connection |
| [Node Patterns](#node-architecture-patterns) | Different architectures |
| [Execution Context](#execution-context) | Available `this` methods |

**Key Understanding**:
- Every node implements `INodeType` interface
- `description` object defines UI and behavior
- `methods` object adds loadOptions/listSearch
- `execute` method (optional) handles custom logic

**Related**:
- [05-declarative-vs-programmatic-nodes.md](./05-declarative-vs-programmatic-nodes.md) - Choose approach
- [06-node-properties-reference.md](./06-node-properties-reference.md) - Property types

**Reference File**: `nodes/GithubIssues/GithubIssues.node.ts`

---

## Core Interface: INodeType

Every n8n node implements the `INodeType` interface:

```typescript
import { INodeType, INodeTypeDescription } from 'n8n-workflow';

export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    // Required: Node metadata and configuration
  };

  // Optional: Custom methods
  methods?: {
    loadOptions?: { [key: string]: Function };
    listSearch?: { [key: string]: Function };
  };

  // Optional: Custom execution logic
  async execute?(this: IExecuteFunctions): Promise<INodeExecutionData[][]>;
}
```

---

## INodeTypeDescription Structure

### Complete Example

```typescript
description: INodeTypeDescription = {
  // Identity
  displayName: 'GitHub Issues',
  name: 'githubIssues',
  icon: { light: 'file:github.svg', dark: 'file:github.dark.svg' },
  
  // Categorization
  group: ['input'],
  version: 1,
  
  // Display
  subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}',
  description: 'Consume issues from the GitHub API',
  
  // Defaults
  defaults: {
    name: 'GitHub Issues',
  },
  
  // Connections
  inputs: [NodeConnectionTypes.Main],
  outputs: [NodeConnectionTypes.Main],
  
  // AI Tool Support
  usableAsTool: true,
  
  // Authentication
  credentials: [
    {
      name: 'githubIssuesApi',
      required: true,
      displayOptions: {
        show: { authentication: ['accessToken'] },
      },
    },
  ],
  
  // HTTP Defaults (for declarative nodes)
  requestDefaults: {
    baseURL: 'https://api.github.com',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json',
    },
  },
  
  // Node Properties (UI fields)
  properties: [
    // ... property definitions
  ],
};
```

---

## Description Fields Reference

### Identity Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `displayName` | string | Yes | Human-readable name shown in UI |
| `name` | string | Yes | Internal identifier (camelCase, unique) |
| `icon` | string \| object | Yes | Node icon reference |

**Icon Formats**:
```typescript
// Single icon
icon: 'file:myicon.svg'

// Light/dark variants
icon: { light: 'file:icon.svg', dark: 'file:icon.dark.svg' }

// Built-in icon
icon: 'fa:code'
```

### Categorization

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `group` | string[] | Yes | Category: `'input'`, `'output'`, `'transform'` |
| `version` | number \| number[] | Yes | Node version(s) |

### Display

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `subtitle` | string | No | Dynamic subtitle using expressions |
| `description` | string | Yes | Brief description (shown in node panel) |

**Subtitle Expression Example**:
```typescript
subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}'
// Displays: "Get: Issue"
```

### Defaults

```typescript
defaults: {
  name: 'GitHub Issues',  // Default node name in canvas
  color: '#0366d6',       // Optional: node color
}
```

### Connections

```typescript
import { NodeConnectionTypes } from 'n8n-workflow';

inputs: [NodeConnectionTypes.Main],
outputs: [NodeConnectionTypes.Main],
```

**Connection Types**:
| Type | Use Case |
|------|----------|
| `NodeConnectionTypes.Main` | Standard data flow |
| `NodeConnectionTypes.AiTool` | AI agent tool input |
| Multiple outputs | Error handling, branching |

**Multiple Outputs Example**:
```typescript
outputs: [NodeConnectionTypes.Main, NodeConnectionTypes.Main],
outputNames: ['Success', 'Error'],
```

### AI Tool Support

```typescript
usableAsTool: true,  // Allow node as AI agent tool
```

---

## Credentials Configuration

### Single Credential

```typescript
credentials: [
  {
    name: 'myServiceApi',
    required: true,
  },
],
```

### Multiple Auth Methods

```typescript
credentials: [
  {
    name: 'myServiceApi',
    required: true,
    displayOptions: {
      show: { authentication: ['accessToken'] },
    },
  },
  {
    name: 'myServiceOAuth2Api',
    required: true,
    displayOptions: {
      show: { authentication: ['oAuth2'] },
    },
  },
],
```

This requires a property to select authentication:

```typescript
{
  displayName: 'Authentication',
  name: 'authentication',
  type: 'options',
  options: [
    { name: 'Access Token', value: 'accessToken' },
    { name: 'OAuth2', value: 'oAuth2' },
  ],
  default: 'accessToken',
}
```

---

## Request Defaults (Declarative Nodes)

```typescript
requestDefaults: {
  baseURL: 'https://api.github.com',
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json',
  },
},
```

Used with declarative routing—automatically applied to all requests.

---

## Node Architecture Patterns

### Pattern 1: Simple Declarative

```typescript
export class SimpleNode implements INodeType {
  description: INodeTypeDescription = {
    // ... metadata
    properties: [
      {
        displayName: 'Get User',
        name: 'operation',
        type: 'options',
        options: [{
          name: 'Get User',
          value: 'getUser',
          routing: {
            request: {
              method: 'GET',
              url: '/users/{{$parameter.userId}}',
            },
          },
        }],
        default: 'getUser',
      },
    ],
  };
  // No execute method needed!
}
```

### Pattern 2: Resource-Based

```typescript
import { userOperations } from './resources/user';
import { projectOperations } from './resources/project';

export class ComplexNode implements INodeType {
  description: INodeTypeDescription = {
    properties: [
      {
        displayName: 'Resource',
        name: 'resource',
        type: 'options',
        options: [
          { name: 'User', value: 'user' },
          { name: 'Project', value: 'project' },
        ],
        default: 'user',
      },
      ...userOperations,
      ...projectOperations,
    ],
  };
}
```

### Pattern 3: Custom Execute

```typescript
export class CustomNode implements INodeType {
  description: INodeTypeDescription = { /* ... */ };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    for (let i = 0; i < items.length; i++) {
      const param = this.getNodeParameter('myParam', i) as string;
      // Custom logic here
      returnData.push({ json: { result: param } });
    }

    return [returnData];
  }
}
```

### Pattern 4: With Methods

```typescript
export class MethodsNode implements INodeType {
  description: INodeTypeDescription = { /* ... */ };

  methods = {
    loadOptions: {
      async getUsers(this: ILoadOptionsFunctions) {
        // Fetch users for dropdown
        return [{ name: 'User 1', value: 'user1' }];
      },
    },
    listSearch: {
      async searchUsers(this: ILoadOptionsFunctions, filter?: string) {
        // Search users with filtering
        return { results: [{ name: 'User', value: 'user1' }] };
      },
    },
  };
}
```

---

## Execution Context

When `execute` or methods run, `this` provides:

| Method | Description |
|--------|-------------|
| `this.getInputData()` | Get incoming items |
| `this.getNodeParameter(name, index)` | Get parameter value |
| `this.getCredentials(name)` | Get credential values |
| `this.helpers.request(options)` | Make HTTP requests |
| `this.logger.info/error/debug()` | Logging |
| `this.continueOnFail()` | Check if should continue on error |

---

## Best Practices

1. **Use camelCase for `name`** - Must be unique across all nodes
2. **Version your nodes** - Start at `1`, increment for breaking changes
3. **Use expressions in `subtitle`** - Shows current operation/resource
4. **Set `usableAsTool: true`** - If node can work as AI tool
5. **Organize with resources** - For multi-resource APIs

---

## Next Steps

- [05 - Declarative vs Programmatic Nodes](./05-declarative-vs-programmatic-nodes.md)
- [06 - Node Properties Reference](./06-node-properties-reference.md)
