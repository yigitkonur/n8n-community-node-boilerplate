# 05 - Declarative vs Programmatic Nodes

> When to use each approach with comprehensive examples

---

## 🤖 AI Agent Context

**READ THIS DOCUMENT** to decide which approach to use. This is a critical decision point.

| If Your Target Is... | Use This Approach | Then Read |
|---------------------|-------------------|-----------|
| REST API (most common) | **Declarative** | → [12-declarative-routing.md](./12-declarative-routing.md) |
| Official SDK/library | **Programmatic** | → [13-custom-execute-methods.md](./13-custom-execute-methods.md) |
| GraphQL API | **Programmatic** | → [13-custom-execute-methods.md](./13-custom-execute-methods.md) |
| WebSocket/streaming | **Programmatic** | → [13-custom-execute-methods.md](./13-custom-execute-methods.md) |
| Complex auth (signing) | **Programmatic** | → [13-custom-execute-methods.md](./13-custom-execute-methods.md) |

**Decision Summary**:
- **Declarative** = Less code, automatic HTTP handling, best for REST
- **Programmatic** = Full control, SDK support, complex logic

**Reference Files**:
- Declarative: `nodes/GithubIssues/GithubIssues.node.ts`
- Programmatic: `nodes/Example/Example.node.ts`

---

## Overview

n8n supports two primary approaches for building nodes:

| Approach | Best For | Complexity | Code Volume |
|----------|----------|------------|-------------|
| **Declarative** | REST APIs | Low | Minimal |
| **Programmatic** | SDKs, complex logic | Medium-High | More code |

---

## Declarative Style (Recommended)

### When to Use

- Standard REST/HTTP APIs
- CRUD operations
- APIs with consistent patterns
- When you want minimal boilerplate

### Key Features

- **No execute method** - n8n handles requests automatically
- **Routing in properties** - Define HTTP calls inline
- **Built-in pagination** - Declarative pagination support
- **Automatic error handling** - Standard HTTP error handling

### Complete Example

```typescript
import { NodeConnectionTypes, INodeType, INodeTypeDescription } from 'n8n-workflow';

export class GitHubIssues implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'GitHub Issues',
    name: 'githubIssues',
    icon: 'file:github.svg',
    group: ['input'],
    version: 1,
    subtitle: '={{$parameter["operation"]}}',
    description: 'Work with GitHub Issues',
    defaults: { name: 'GitHub Issues' },
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    
    credentials: [
      { name: 'githubApi', required: true },
    ],
    
    // Base configuration for all requests
    requestDefaults: {
      baseURL: 'https://api.github.com',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
    },
    
    properties: [
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        noDataExpression: true,
        options: [
          {
            name: 'Get Issue',
            value: 'get',
            action: 'Get an issue',
            description: 'Retrieve a single issue',
            // Routing defined directly in the option
            routing: {
              request: {
                method: 'GET',
                url: '=/repos/{{$parameter.owner}}/{{$parameter.repo}}/issues/{{$parameter.issueNumber}}',
              },
            },
          },
          {
            name: 'Create Issue',
            value: 'create',
            action: 'Create an issue',
            routing: {
              request: {
                method: 'POST',
                url: '=/repos/{{$parameter.owner}}/{{$parameter.repo}}/issues',
              },
            },
          },
        ],
        default: 'get',
      },
      {
        displayName: 'Owner',
        name: 'owner',
        type: 'string',
        default: '',
        required: true,
        description: 'Repository owner',
      },
      {
        displayName: 'Repository',
        name: 'repo',
        type: 'string',
        default: '',
        required: true,
        description: 'Repository name',
      },
      {
        displayName: 'Issue Number',
        name: 'issueNumber',
        type: 'number',
        default: 0,
        required: true,
        displayOptions: {
          show: { operation: ['get'] },
        },
      },
      {
        displayName: 'Title',
        name: 'title',
        type: 'string',
        default: '',
        required: true,
        displayOptions: {
          show: { operation: ['create'] },
        },
        // Send as request body
        routing: {
          send: {
            type: 'body',
            property: 'title',
          },
        },
      },
    ],
  };
  
  // No execute method needed!
}
```

---

## Programmatic Style

### When to Use

- Third-party SDKs (official client libraries)
- Complex business logic
- Non-REST protocols (WebSocket, GraphQL, etc.)
- Custom authentication flows
- Stream processing

### Key Features

- **Full control** - Complete flexibility over execution
- **SDK integration** - Use official client libraries
- **Custom logic** - Implement any business logic
- **Error handling** - Custom error handling patterns

### Complete Example

```typescript
import type {
  IExecuteFunctions,
  ILoadOptionsFunctions,
  INodeExecutionData,
  INodePropertyOptions,
  INodeType,
  INodeTypeDescription,
} from 'n8n-workflow';
import { NodeOperationError } from 'n8n-workflow';

// External SDK
import { LatitudeClient } from '@latitude/sdk';

export class Latitude implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Latitude',
    name: 'latitude',
    icon: 'file:latitude.svg',
    group: ['transform'],
    version: 1,
    subtitle: '={{$parameter["promptPath"]}}',
    description: 'Execute AI prompts from Latitude',
    defaults: { name: 'Latitude' },
    inputs: ['main'],
    outputs: ['main'],
    
    credentials: [
      { name: 'latitudeApi', required: true },
    ],
    
    properties: [
      {
        displayName: 'Prompt Path',
        name: 'promptPath',
        type: 'options',
        required: true,
        typeOptions: {
          loadOptionsMethod: 'getPrompts',  // Dynamic loading
        },
        default: '',
      },
      {
        displayName: 'Parameters',
        name: 'parametersUi',
        type: 'fixedCollection',
        typeOptions: { multipleValues: true },
        default: {},
        options: [
          {
            name: 'parameter',
            displayName: 'Parameter',
            values: [
              {
                displayName: 'Name',
                name: 'name',
                type: 'options',
                typeOptions: {
                  loadOptionsMethod: 'getPromptParameters',
                  loadOptionsDependsOn: ['promptPath'],
                },
                default: '',
              },
              {
                displayName: 'Value',
                name: 'value',
                type: 'string',
                default: '',
              },
            ],
          },
        ],
      },
    ],
  };

  // Dynamic option loading
  methods = {
    loadOptions: {
      async getPrompts(this: ILoadOptionsFunctions): Promise<INodePropertyOptions[]> {
        const credentials = await this.getCredentials('latitudeApi') as {
          apiKey: string;
          projectId: number;
        };
        
        const client = new LatitudeClient(credentials.apiKey, {
          projectId: credentials.projectId,
        });
        
        const prompts = await client.prompts.getAll();
        
        return prompts.map(p => ({
          name: p.path,
          value: p.path,
        }));
      },
      
      async getPromptParameters(this: ILoadOptionsFunctions): Promise<INodePropertyOptions[]> {
        const promptPath = this.getCurrentNodeParameter('promptPath') as string;
        if (!promptPath) return [];
        
        // ... fetch and return parameters
        return [];
      },
    },
  };

  // Custom execution logic
  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];
    
    // Initialize SDK client
    const credentials = await this.getCredentials('latitudeApi') as {
      apiKey: string;
      projectId: number;
    };
    
    const client = new LatitudeClient(credentials.apiKey, {
      projectId: credentials.projectId,
    });
    
    for (let i = 0; i < items.length; i++) {
      try {
        const promptPath = this.getNodeParameter('promptPath', i) as string;
        const parametersUi = this.getNodeParameter('parametersUi', i) as {
          parameter?: Array<{ name: string; value: string }>;
        };
        
        // Build parameters object
        const parameters: Record<string, any> = {};
        if (parametersUi.parameter) {
          for (const param of parametersUi.parameter) {
            parameters[param.name] = param.value;
          }
        }
        
        // Execute with SDK
        const result = await client.prompts.run(promptPath, {
          parameters,
          stream: false,
        });
        
        returnData.push({
          json: result,
          pairedItem: { item: i },
        });
      } catch (error) {
        if (this.continueOnFail()) {
          returnData.push({
            json: { error: error.message },
            pairedItem: { item: i },
          });
          continue;
        }
        
        throw new NodeOperationError(this.getNode(), error, { itemIndex: i });
      }
    }
    
    return [returnData];
  }
}
```

---

## Comparison Table

| Feature | Declarative | Programmatic |
|---------|-------------|--------------|
| **Code volume** | Minimal | More |
| **HTTP handling** | Automatic | Manual |
| **Error handling** | Built-in | Custom |
| **SDK support** | Limited | Full |
| **Complex logic** | Difficult | Easy |
| **Pagination** | Built-in | Manual |
| **Learning curve** | Lower | Higher |
| **Flexibility** | Limited | Unlimited |

---

## Hybrid Approach

You can combine both approaches:

```typescript
export class HybridNode implements INodeType {
  description: INodeTypeDescription = {
    // ... properties with declarative routing for simple ops
    properties: [
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        options: [
          {
            name: 'Get User',
            value: 'getUser',
            // Simple GET - use declarative
            routing: {
              request: { method: 'GET', url: '/users/{{$parameter.userId}}' },
            },
          },
          {
            name: 'Complex Action',
            value: 'complexAction',
            // No routing - handled in execute
          },
        ],
        default: 'getUser',
      },
    ],
  };

  // Only handles 'complexAction', others are declarative
  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const operation = this.getNodeParameter('operation', 0) as string;
    
    if (operation === 'complexAction') {
      // Custom logic here
      return [[{ json: { result: 'custom' } }]];
    }
    
    // Let n8n handle declarative operations
    return [];
  }
}
```

---

## Decision Flowchart

```
Start
  │
  ▼
Is it a REST API?
  │
  ├── Yes ──► Use Declarative
  │
  └── No
       │
       ▼
    Do you need an SDK?
       │
       ├── Yes ──► Use Programmatic
       │
       └── No
            │
            ▼
         Complex business logic?
            │
            ├── Yes ──► Use Programmatic
            │
            └── No ──► Use Declarative
```

---

## Best Practices

### Declarative

1. **Always set `requestDefaults`** - DRY principle
2. **Use expressions in URLs** - `=/path/{{$parameter.id}}`
3. **Leverage routing.send** - For body/query parameters
4. **Use routing.output** - For response processing

### Programmatic

1. **Initialize SDK once per execution** - Not per item
2. **Use `continueOnFail()`** - Handle errors gracefully
3. **Pair items correctly** - Use `pairedItem: { item: i }`
4. **Log appropriately** - Use `this.logger`
5. **Sanitize errors** - Never leak credentials

---

## Next Steps

- [06 - Node Properties Reference](./06-node-properties-reference.md)
- [12 - Declarative Routing](./12-declarative-routing.md)
- [13 - Custom Execute Methods](./13-custom-execute-methods.md)
