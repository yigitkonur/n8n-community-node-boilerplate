# 16 - Pagination Handling

> Implementing paginated API responses

---

## Declarative Pagination

### Link Header Pagination (GitHub style)

```typescript
// shared/utils.ts
export function parseLinkHeader(header: string | undefined): Record<string, string> {
  if (!header) return {};
  const links: Record<string, string> = {};
  const parts = header.split(',');
  for (const part of parts) {
    const match = part.match(/<([^>]+)>;\s*rel="([^"]+)"/);
    if (match) links[match[2]] = match[1];
  }
  return links;
}

// In property definition
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

### Offset Pagination

```typescript
routing: {
  send: { paginate: '={{ $value }}' },
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

### Cursor Pagination

```typescript
routing: {
  send: { paginate: '={{ $value }}' },
  operations: {
    pagination: {
      type: 'generic',
      properties: {
        continue: '={{ $response.body.has_more }}',
        request: {
          qs: { cursor: '={{ $response.body.next_cursor }}' },
        },
      },
    },
  },
}
```

---

## Programmatic Pagination

```typescript
async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const returnAll = this.getNodeParameter('returnAll', 0) as boolean;
  const limit = this.getNodeParameter('limit', 0, 50) as number;
  
  let allResults: any[] = [];
  let cursor: string | undefined;
  
  do {
    const response = await this.helpers.httpRequestWithAuthentication.call(
      this, 'myApi',
      {
        url: 'https://api.example.com/items',
        qs: { cursor, limit: 100 },
      }
    );
    
    allResults.push(...response.items);
    cursor = response.next_cursor;
    
    if (!returnAll && allResults.length >= limit) {
      allResults = allResults.slice(0, limit);
      break;
    }
  } while (cursor);
  
  return [allResults.map(item => ({ json: item }))];
}
```

---

## Next Steps

- [12 - Declarative Routing](./12-declarative-routing.md)
- [13 - Custom Execute Methods](./13-custom-execute-methods.md)
