# 28 - Complete Code Examples

> Full working examples for reference

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** for copy-paste code templates. All examples are complete and runnable.

| Example | Use When |
|---------|----------|
| [Minimal Declarative Node](#minimal-declarative-node) | Quick REST API integration |
| [Minimal Credential](#minimal-credential) | Simple API key auth |
| [SDK-Based Node](#sdk-based-node-pattern) | Using external SDK |

**Copy-Paste Workflow**:
1. Copy the template
2. Replace placeholder names (SimpleApi, simpleApi, etc.)
3. Update URLs and authentication
4. Register in package.json

**For More Detailed Patterns**:
- [29-common-patterns-and-recipes.md](./29-common-patterns-and-recipes.md) - UI patterns
- [12-declarative-routing.md](./12-declarative-routing.md) - Routing details
- [13-custom-execute-methods.md](./13-custom-execute-methods.md) - Execute details

---

## Minimal Declarative Node

```typescript
import { NodeConnectionTypes, INodeType, INodeTypeDescription } from 'n8n-workflow';

export class SimpleApi implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Simple API',
    name: 'simpleApi',
    icon: 'file:icon.svg',
    group: ['transform'],
    version: 1,
    description: 'Simple API node',
    defaults: { name: 'Simple API' },
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    credentials: [{ name: 'simpleApi', required: true }],
    requestDefaults: {
      baseURL: 'https://api.example.com',
    },
    properties: [
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        options: [
          {
            name: 'Get Items',
            value: 'getItems',
            routing: {
              request: { method: 'GET', url: '/items' },
            },
          },
        ],
        default: 'getItems',
      },
    ],
  };
}
```

---

## Minimal Credential

```typescript
import type { IAuthenticateGeneric, ICredentialType, INodeProperties } from 'n8n-workflow';

export class SimpleApi implements ICredentialType {
  name = 'simpleApi';
  displayName = 'Simple API';
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
    },
  ];
  authenticate: IAuthenticateGeneric = {
    type: 'generic',
    properties: {
      headers: { Authorization: '=Bearer {{$credentials.apiKey}}' },
    },
  };
}
```

---

## SDK-Based Node Pattern

```typescript
import type { IExecuteFunctions, INodeExecutionData, INodeType } from 'n8n-workflow';
import { NodeOperationError } from 'n8n-workflow';

export class SdkNode implements INodeType {
  description = { /* ... */ };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const credentials = await this.getCredentials('myApi') as { apiKey: string };
    const { MyClient } = await import('@my/sdk');
    const client = new MyClient(credentials.apiKey);
    
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    for (let i = 0; i < items.length; i++) {
      try {
        const result = await client.doSomething();
        returnData.push({ json: result, pairedItem: { item: i } });
      } catch (error) {
        if (this.continueOnFail()) {
          returnData.push({ json: { error: error.message }, pairedItem: { item: i } });
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

## Next Steps

- [29 - Common Patterns and Recipes](./29-common-patterns-and-recipes.md)
- [10 - Creating Your First Node](./10-creating-your-first-node.md)
