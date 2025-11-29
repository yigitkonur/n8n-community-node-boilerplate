# 12 - Declarative Routing

> HTTP routing without custom execute code

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** when building nodes for REST APIs. This is the **recommended approach** for HTTP-based integrations.

| Declarative Routing Is For | NOT For |
|---------------------------|---------|
| ✅ REST APIs | ❌ SDK integrations |
| ✅ CRUD operations | ❌ Complex business logic |
| ✅ Standard HTTP methods | ❌ WebSocket/GraphQL |
| ✅ Minimal boilerplate | ❌ Custom auth flows |

**Prerequisites** (read first if not already):
- [10-creating-your-first-node.md](./10-creating-your-first-node.md) - Basic node structure
- [08-api-key-credentials.md](./08-api-key-credentials.md) - Set up authentication

**After This Document, Read**:
- [06-node-properties-reference.md](./06-node-properties-reference.md) - All property types
- [16-pagination-handling.md](./16-pagination-handling.md) - Handle "Return All"
- [11-resources-and-operations.md](./11-resources-and-operations.md) - Multi-resource APIs

**Reference File**: `nodes/GithubIssues/GithubIssues.node.ts`

---

## Overview

Declarative routing lets you define HTTP requests directly in property definitions. n8n handles the actual HTTP calls automatically.

---

## Request Routing

### Basic Request

```typescript
{
  name: 'Get User',
  value: 'getUser',
  routing: {
    request: {
      method: 'GET',
      url: '/users/123',
    },
  },
}
```

### Dynamic URL Parameters

```typescript
{
  name: 'Get User',
  value: 'getUser',
  routing: {
    request: {
      method: 'GET',
      url: '=/users/{{$parameter.userId}}',  // Expression prefix: =
    },
  },
}
```

### Multiple Path Parameters

```typescript
routing: {
  request: {
    method: 'GET',
    url: '=/repos/{{$parameter.owner}}/{{$parameter.repo}}/issues/{{$parameter.issueNumber}}',
  },
}
```

---

## Send Routing

Control how property values are sent in requests.

### Query Parameters

```typescript
{
  displayName: 'State',
  name: 'state',
  type: 'options',
  options: [
    { name: 'Open', value: 'open' },
    { name: 'Closed', value: 'closed' },
  ],
  default: 'open',
  routing: {
    send: {
      type: 'query',
      property: 'state',  // Sent as ?state=open
    },
  },
}
```

### Request Body

```typescript
{
  displayName: 'Title',
  name: 'title',
  type: 'string',
  default: '',
  routing: {
    send: {
      type: 'body',
      property: 'title',  // Sent as {"title": "value"}
    },
  },
}
```

### Nested Body Properties

```typescript
{
  displayName: 'City',
  name: 'city',
  type: 'string',
  default: '',
  routing: {
    send: {
      type: 'body',
      property: 'address.city',  // Sent as {"address": {"city": "value"}}
    },
  },
}
```

### Request Headers

```typescript
{
  displayName: 'Custom Header',
  name: 'customHeader',
  type: 'string',
  default: '',
  routing: {
    send: {
      type: 'header',
      property: 'X-Custom-Header',
    },
  },
}
```

---

## Value Transformations

Transform values before sending.

### Basic Transformation

```typescript
{
  displayName: 'Labels',
  name: 'labels',
  type: 'collection',
  typeOptions: { multipleValues: true },
  default: { label: '' },
  options: [
    {
      displayName: 'Label',
      name: 'label',
      type: 'string',
      default: '',
    },
  ],
  routing: {
    send: {
      type: 'body',
      property: 'labels',
      value: '={{$value.map((data) => data.label)}}',
    },
  },
}
```

### Conditional Value

```typescript
routing: {
  send: {
    type: 'body',
    property: 'status',
    value: '={{$value === "active" ? 1 : 0}}',
  },
}
```

### Skip Empty Values

```typescript
routing: {
  send: {
    type: 'body',
    property: 'description',
    value: '={{$value || undefined}}',  // Don't send if empty
  },
}
```

---

## Output Routing

Control how responses are processed.

### Limit Results

```typescript
{
  displayName: 'Limit',
  name: 'limit',
  type: 'number',
  default: 50,
  routing: {
    send: {
      type: 'query',
      property: 'per_page',
    },
    output: {
      maxResults: '={{$value}}',  // Limits returned items
    },
  },
}
```

### Post-Processing

```typescript
routing: {
  output: {
    postReceive: [
      {
        type: 'rootProperty',
        properties: {
          property: 'data',  // Extract data from {data: [...]}
        },
      },
    ],
  },
}
```

---

## Pagination

### Link Header Pagination

```typescript
import { parseLinkHeader } from '../../shared/utils';

{
  displayName: 'Return All',
  name: 'returnAll',
  type: 'boolean',
  default: false,
  routing: {
    send: {
      paginate: '={{ $value }}',
      type: 'query',
      property: 'per_page',
      value: '100',
    },
    operations: {
      pagination: {
        type: 'generic',
        properties: {
          continue: `={{ !!(${parseLinkHeader.toString()})($response.headers?.link).next }}`,
          request: {
            url: `={{ (${parseLinkHeader.toString()})($response.headers?.link)?.next ?? $request.url }}`,
          },
        },
      },
    },
  },
}
```

### Offset-Based Pagination

```typescript
routing: {
  send: {
    paginate: '={{ $value }}',
  },
  operations: {
    pagination: {
      type: 'offset',
      properties: {
        limitParameter: 'limit',
        offsetParameter: 'offset',
        pageSize: 100,
        type: 'query',
      },
    },
  },
}
```

### Cursor-Based Pagination

```typescript
routing: {
  send: {
    paginate: '={{ $value }}',
  },
  operations: {
    pagination: {
      type: 'generic',
      properties: {
        continue: '={{ $response.body.has_more }}',
        request: {
          qs: {
            cursor: '={{ $response.body.next_cursor }}',
          },
        },
      },
    },
  },
}
```

---

## Request Configuration

### Additional Query Parameters

```typescript
routing: {
  request: {
    method: 'GET',
    url: '/issues',
    qs: {
      state: 'open',
      sort: 'created',
    },
  },
}
```

### Additional Headers

```typescript
routing: {
  request: {
    method: 'POST',
    url: '/data',
    headers: {
      'X-Custom-Header': 'value',
    },
  },
}
```

### Request Body

```typescript
routing: {
  request: {
    method: 'POST',
    url: '/webhook',
    body: {
      event: 'trigger',
      timestamp: '={{Date.now()}}',
    },
  },
}
```

---

## Complete Declarative Node Example

```typescript
export class DeclarativeNode implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Declarative Example',
    name: 'declarativeExample',
    icon: 'file:icon.svg',
    group: ['transform'],
    version: 1,
    subtitle: '={{$parameter["operation"]}}',
    description: 'Fully declarative node example',
    defaults: { name: 'Declarative Example' },
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    
    credentials: [
      { name: 'exampleApi', required: true },
    ],
    
    requestDefaults: {
      baseURL: 'https://api.example.com',
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
            name: 'Get Items',
            value: 'getItems',
            action: 'Get items',
            routing: {
              request: {
                method: 'GET',
                url: '/items',
              },
            },
          },
          {
            name: 'Create Item',
            value: 'createItem',
            action: 'Create an item',
            routing: {
              request: {
                method: 'POST',
                url: '/items',
              },
            },
          },
        ],
        default: 'getItems',
      },
      
      // Create item fields
      {
        displayName: 'Name',
        name: 'name',
        type: 'string',
        required: true,
        default: '',
        displayOptions: {
          show: { operation: ['createItem'] },
        },
        routing: {
          send: { type: 'body', property: 'name' },
        },
      },
      
      // Get items filter
      {
        displayName: 'Status Filter',
        name: 'status',
        type: 'options',
        options: [
          { name: 'All', value: '' },
          { name: 'Active', value: 'active' },
          { name: 'Archived', value: 'archived' },
        ],
        default: '',
        displayOptions: {
          show: { operation: ['getItems'] },
        },
        routing: {
          send: {
            type: 'query',
            property: 'status',
            value: '={{$value || undefined}}',  // Skip if empty
          },
        },
      },
    ],
  };
  
  // No execute method needed!
}
```

---

## When NOT to Use Declarative Routing

- **SDK-based integrations** - Use execute method
- **Complex request building** - Multi-step processes
- **Non-HTTP protocols** - WebSocket, GraphQL subscriptions
- **Binary data handling** - File uploads/downloads
- **Complex authentication** - Token refresh, signing

---

## Next Steps

- [13 - Custom Execute Methods](./13-custom-execute-methods.md)
- [16 - Pagination Handling](./16-pagination-handling.md)
