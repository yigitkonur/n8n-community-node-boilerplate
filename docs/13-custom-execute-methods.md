# 13 - Custom Execute Methods

> Programmatic node execution for complex scenarios

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** when building nodes with SDKs, complex logic, or non-REST protocols.

| Custom Execute Is For | NOT For |
|----------------------|---------|
| ✅ SDK integrations | ❌ Simple REST APIs (use declarative) |
| ✅ Complex business logic | ❌ Basic CRUD |
| ✅ Non-REST protocols | ❌ Minimal code preference |
| ✅ Custom auth/signing | |

**Prerequisites** (read first if not already):
- [10-creating-your-first-node.md](./10-creating-your-first-node.md) - Basic node structure
- [08-api-key-credentials.md](./08-api-key-credentials.md) - Set up authentication

**After This Document, Read**:
- [19-external-sdk-integration.md](./19-external-sdk-integration.md) - SDK patterns
- [17-error-handling-patterns.md](./17-error-handling-patterns.md) - Error handling
- [18-helper-functions-and-utilities.md](./18-helper-functions-and-utilities.md) - Helper functions

**Reference File**: `nodes/Example/Example.node.ts`

---

## When to Use Custom Execute

- External SDK integration
- Complex business logic
- Non-standard authentication
- Multi-step operations
- Stream processing
- WebSocket/GraphQL

---

## Basic Structure

```typescript
import type {
  IExecuteFunctions,
  INodeExecutionData,
  INodeType,
  INodeTypeDescription,
} from 'n8n-workflow';
import { NodeOperationError } from 'n8n-workflow';

export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    // ... node description
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    for (let i = 0; i < items.length; i++) {
      try {
        // Get parameters
        const myParam = this.getNodeParameter('myParam', i) as string;
        
        // Do work
        const result = await someAsyncOperation(myParam);
        
        // Add to results
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

## Execution Context (this)

The `this` context provides essential methods:

### Input/Output

| Method | Description |
|--------|-------------|
| `this.getInputData()` | Get all input items |
| `this.getNodeParameter(name, index)` | Get parameter value for item |
| `this.getNode()` | Get node information |
| `this.continueOnFail()` | Check if "Continue On Fail" is enabled |

### Credentials

| Method | Description |
|--------|-------------|
| `this.getCredentials(name)` | Get decrypted credentials |

### HTTP Requests

| Method | Description |
|--------|-------------|
| `this.helpers.request(options)` | Make HTTP request |
| `this.helpers.httpRequestWithAuthentication(credType, options)` | HTTP with auth |
| `this.helpers.requestWithAuthentication(credType, options)` | Legacy request with auth |

### Logging

| Method | Description |
|--------|-------------|
| `this.logger.info(message, meta)` | Info log |
| `this.logger.debug(message, meta)` | Debug log |
| `this.logger.error(message, meta)` | Error log |
| `this.logger.warn(message, meta)` | Warning log |

---

## SDK Integration Pattern

```typescript
import { MyServiceClient } from '@myservice/sdk';

export class MyService implements INodeType {
  description: INodeTypeDescription = { /* ... */ };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    // Get credentials once (not per item)
    const credentials = await this.getCredentials('myServiceApi') as {
      apiKey: string;
      region: string;
    };

    // Initialize client once
    const client = new MyServiceClient({
      apiKey: credentials.apiKey,
      region: credentials.region,
    });

    this.logger.info('MyService client initialized', {
      region: credentials.region,
    });

    // Process each item
    for (let i = 0; i < items.length; i++) {
      try {
        const operation = this.getNodeParameter('operation', i) as string;
        
        let result: any;
        
        switch (operation) {
          case 'create':
            const createData = this.getNodeParameter('createData', i) as object;
            result = await client.create(createData);
            break;
            
          case 'get':
            const itemId = this.getNodeParameter('itemId', i) as string;
            result = await client.get(itemId);
            break;
            
          case 'list':
            const options = this.getNodeParameter('options', i) as object;
            result = await client.list(options);
            break;
            
          default:
            throw new Error(`Unknown operation: ${operation}`);
        }

        returnData.push({
          json: result,
          pairedItem: { item: i },
        });

      } catch (error) {
        this.logger.error('Operation failed', {
          itemIndex: i,
          error: error.message,
        });

        if (this.continueOnFail()) {
          returnData.push({
            json: { error: error.message },
            pairedItem: { item: i },
          });
          continue;
        }
        
        throw new NodeOperationError(
          this.getNode(),
          error,
          { itemIndex: i }
        );
      }
    }

    return [returnData];
  }
}
```

---

## HTTP Request Pattern

```typescript
async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const items = this.getInputData();
  const returnData: INodeExecutionData[] = [];

  for (let i = 0; i < items.length; i++) {
    try {
      const userId = this.getNodeParameter('userId', i) as string;
      
      // Using httpRequestWithAuthentication
      const response = await this.helpers.httpRequestWithAuthentication.call(
        this,
        'myServiceApi',  // Credential type name
        {
          url: `https://api.myservice.com/users/${userId}`,
          method: 'GET',
          json: true,
        }
      );

      returnData.push({
        json: response,
        pairedItem: { item: i },
      });
      
    } catch (error) {
      // ... error handling
    }
  }

  return [returnData];
}
```

---

## Working with Input Data

### Access Previous Node Data

```typescript
const items = this.getInputData();

for (let i = 0; i < items.length; i++) {
  // Access input item's JSON data
  const inputData = items[i].json;
  
  // Use input data in operations
  const userId = inputData.user_id as string;
  const email = inputData.email as string;
}
```

### Expression-Based Parameters

```typescript
// Parameter with expression like: {{$json.userId}}
// getNodeParameter automatically resolves it
const userId = this.getNodeParameter('userId', i) as string;
```

---

## Multiple Outputs

```typescript
description: INodeTypeDescription = {
  // ...
  outputs: [NodeConnectionTypes.Main, NodeConnectionTypes.Main],
  outputNames: ['Success', 'Error'],
};

async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const items = this.getInputData();
  const successData: INodeExecutionData[] = [];
  const errorData: INodeExecutionData[] = [];

  for (let i = 0; i < items.length; i++) {
    try {
      const result = await doOperation();
      successData.push({
        json: result,
        pairedItem: { item: i },
      });
    } catch (error) {
      errorData.push({
        json: { error: error.message, input: items[i].json },
        pairedItem: { item: i },
      });
    }
  }

  // Return array for each output
  return [successData, errorData];
}
```

---

## Binary Data Handling

### Returning Binary Data

```typescript
import { BINARY_ENCODING } from 'n8n-workflow';

async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const items = this.getInputData();
  const returnData: INodeExecutionData[] = [];

  for (let i = 0; i < items.length; i++) {
    const fileUrl = this.getNodeParameter('fileUrl', i) as string;
    
    // Download file
    const response = await this.helpers.httpRequestWithAuthentication.call(
      this,
      'myServiceApi',
      {
        url: fileUrl,
        method: 'GET',
        encoding: 'arraybuffer',
        returnFullResponse: true,
      }
    );

    // Create binary data
    const binaryData = await this.helpers.prepareBinaryData(
      Buffer.from(response.body as ArrayBuffer),
      'download.pdf',
      'application/pdf'
    );

    returnData.push({
      json: {},
      binary: {
        data: binaryData,
      },
      pairedItem: { item: i },
    });
  }

  return [returnData];
}
```

### Reading Binary Data

```typescript
const binaryPropertyName = this.getNodeParameter('binaryPropertyName', i) as string;

const binaryData = items[i].binary?.[binaryPropertyName];
if (!binaryData) {
  throw new NodeOperationError(this.getNode(), 'No binary data found', { itemIndex: i });
}

const buffer = await this.helpers.getBinaryDataBuffer(i, binaryPropertyName);
```

---

## Simplify Output Pattern

```typescript
{
  displayName: 'Simplify Output',
  name: 'simplify',
  type: 'boolean',
  default: true,
  description: 'Whether to return simplified output',
}

// In execute:
const simplify = this.getNodeParameter('simplify', i) as boolean;

let outputData: any;
if (simplify) {
  outputData = {
    id: result.id,
    text: result.response?.text,
    status: result.status,
  };
} else {
  outputData = result;  // Full response
}

returnData.push({
  json: outputData,
  pairedItem: { item: i },
});
```

---

## Security: Sanitizing Errors

Never expose credentials in error messages:

```typescript
catch (error) {
  const errorMessage = error instanceof Error ? error.message : 'Unknown error';
  
  // Remove any API keys from error message
  const sanitizedError = errorMessage.replace(
    /api[-_]?key[s]?\s*[:=]\s*[a-z0-9_-]{10,}/gi,
    '[REDACTED]'
  );

  this.logger.error('Operation failed', {
    itemIndex: i,
    error: sanitizedError,  // Sanitized for logs
  });

  if (this.continueOnFail()) {
    returnData.push({
      json: { error: sanitizedError },
      pairedItem: { item: i },
    });
    continue;
  }

  throw new NodeOperationError(
    this.getNode(),
    new Error(sanitizedError),  // Sanitized for user
    { itemIndex: i }
  );
}
```

---

## Next Steps

- [14 - List Search Methods](./14-list-search-methods.md)
- [17 - Error Handling Patterns](./17-error-handling-patterns.md)
- [19 - External SDK Integration](./19-external-sdk-integration.md)
