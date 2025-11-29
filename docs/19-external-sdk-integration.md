# 19 - External SDK Integration

> Using third-party SDKs in n8n nodes

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** when integrating an official SDK or client library.

| SDK Integration Covers | Example |
|----------------------|---------|
| Installing SDK | `npm install @service/sdk` |
| Initializing client | `new Client(apiKey)` |
| Using in execute method | `await client.operation()` |
| Dynamic option loading | `loadOptions` with SDK |

**⚠️ Important for n8n Cloud**: External dependencies may prevent verification. Consider using n8n's HTTP helpers for simple APIs.

**Prerequisites**:
- [13-custom-execute-methods.md](./13-custom-execute-methods.md) - Execute method basics
- [08-api-key-credentials.md](./08-api-key-credentials.md) - Credential setup

**Related**:
- [17-error-handling-patterns.md](./17-error-handling-patterns.md) - Handle SDK errors
- [18-helper-functions-and-utilities.md](./18-helper-functions-and-utilities.md) - Create SDK helper

---

## When to Use SDKs

- Official client libraries available
- Complex authentication (signing, tokens)
- Non-REST protocols
- SDK handles edge cases

---

## Setup

### Add Dependency

```json
{
  "dependencies": {
    "@latitude-data/sdk": "^5.2.2"
  }
}
```

**Note**: For n8n Cloud verification, external dependencies may disqualify your node.

---

## Basic Pattern

```typescript
import { Latitude } from '@latitude-data/sdk';

export class LatitudeNode implements INodeType {
  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const credentials = await this.getCredentials('latitudeApi') as {
      apiKey: string;
      projectId: number;
    };
    
    // Initialize SDK once
    const client = new Latitude(credentials.apiKey, {
      projectId: credentials.projectId,
    });
    
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];
    
    for (let i = 0; i < items.length; i++) {
      const promptPath = this.getNodeParameter('promptPath', i) as string;
      
      const result = await client.prompts.run(promptPath, {
        parameters: {},
        stream: false,
      });
      
      returnData.push({ json: result, pairedItem: { item: i } });
    }
    
    return [returnData];
  }
}
```

---

## Dynamic Loading with loadOptions

```typescript
methods = {
  loadOptions: {
    async getPrompts(this: ILoadOptionsFunctions) {
      const credentials = await this.getCredentials('latitudeApi') as {
        apiKey: string;
        projectId: number;
      };
      
      const { Latitude } = await import('@latitude-data/sdk');
      const client = new Latitude(credentials.apiKey, {
        projectId: credentials.projectId,
      });
      
      const prompts = await client.prompts.getAll();
      
      return prompts.map(p => ({
        name: p.path,
        value: p.path,
      }));
    },
  },
};
```

---

## Client Helper Function

```typescript
// GenericFunctions.ts
import type { IExecuteFunctions, ILoadOptionsFunctions } from 'n8n-workflow';
import { Latitude } from '@latitude-data/sdk';

export async function getLatitudeClient(
  this: IExecuteFunctions | ILoadOptionsFunctions,
): Promise<Latitude> {
  const credentials = await this.getCredentials('latitudeApi') as {
    apiKey: string;
    projectId: number;
  };
  
  return new Latitude(credentials.apiKey, {
    projectId: credentials.projectId,
  });
}
```

---

## Best Practices

1. **Initialize once** - Create client before the loop
2. **Type credentials** - Cast to expected shape
3. **Handle errors** - Sanitize before exposing
4. **Use `stream: false`** - For n8n compatibility
5. **Log appropriately** - Debug SDK calls

---

## Next Steps

- [13 - Custom Execute Methods](./13-custom-execute-methods.md)
- [17 - Error Handling Patterns](./17-error-handling-patterns.md)
