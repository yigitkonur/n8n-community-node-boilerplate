# 17 - Error Handling Patterns

> Robust error handling strategies for n8n nodes

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** to implement proper error handling. Required for production-quality nodes.

| Pattern | Use Case |
|---------|----------|
| [Basic Pattern](#basic-pattern) | Standard try/catch with continueOnFail |
| [NodeOperationError](#nodeoperationerror) | Throw user-friendly errors |
| [Sanitizing Errors](#sanitizing-errors) | **CRITICAL**: Never expose credentials |
| [HTTP Error Handling](#http-error-handling) | Handle API status codes |
| [Logging](#logging) | Debug and audit trail |

**⚠️ Security Critical**: Always sanitize error messages to remove API keys and secrets before exposing to users.

**Related**:
- [13-custom-execute-methods.md](./13-custom-execute-methods.md) - Where errors occur
- [30-troubleshooting-guide.md](./30-troubleshooting-guide.md) - Debug issues

---

## Basic Pattern

```typescript
import { NodeOperationError } from 'n8n-workflow';

async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const items = this.getInputData();
  const returnData: INodeExecutionData[] = [];

  for (let i = 0; i < items.length; i++) {
    try {
      const result = await performOperation();
      returnData.push({ json: result, pairedItem: { item: i } });
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
```

---

## NodeOperationError

```typescript
// Basic error
throw new NodeOperationError(this.getNode(), 'Something went wrong');

// With item index
throw new NodeOperationError(this.getNode(), error, { itemIndex: i });

// With description
throw new NodeOperationError(
  this.getNode(),
  'API rate limit exceeded',
  { description: 'Wait 60 seconds before retrying' }
);
```

---

## Sanitizing Errors

Never expose credentials:

```typescript
catch (error) {
  const message = error instanceof Error ? error.message : 'Unknown error';
  const sanitized = message.replace(
    /api[-_]?key[s]?\s*[:=]\s*[a-z0-9_-]{10,}/gi,
    '[REDACTED]'
  );
  
  this.logger.error('Operation failed', { error: sanitized });
  throw new NodeOperationError(this.getNode(), sanitized, { itemIndex: i });
}
```

---

## HTTP Error Handling

```typescript
try {
  const response = await this.helpers.httpRequestWithAuthentication.call(
    this, 'myApi', options
  );
} catch (error) {
  if (error.response) {
    const status = error.response.status;
    const message = error.response.data?.message || error.message;
    
    if (status === 401) {
      throw new NodeOperationError(this.getNode(), 'Invalid credentials');
    }
    if (status === 404) {
      throw new NodeOperationError(this.getNode(), 'Resource not found');
    }
    if (status === 429) {
      throw new NodeOperationError(this.getNode(), 'Rate limit exceeded');
    }
    throw new NodeOperationError(this.getNode(), `API Error: ${message}`);
  }
  throw error;
}
```

---

## Logging

```typescript
this.logger.info('Starting operation', { itemCount: items.length });
this.logger.debug('Processing item', { itemIndex: i, data: params });
this.logger.warn('Retrying request', { attempt: 2 });
this.logger.error('Operation failed', { error: sanitizedError });
```

---

## Next Steps

- [13 - Custom Execute Methods](./13-custom-execute-methods.md)
- [30 - Troubleshooting Guide](./30-troubleshooting-guide.md)
