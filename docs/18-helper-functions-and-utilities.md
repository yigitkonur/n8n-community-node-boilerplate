# 18 - Helper Functions and Utilities

> Reusable transport and utility functions

---

## Transport Function (API Requests)

`shared/transport.ts`:

```typescript
import type {
  IExecuteFunctions,
  ILoadOptionsFunctions,
  IHttpRequestMethods,
  IDataObject,
  IHttpRequestOptions,
} from 'n8n-workflow';

export async function apiRequest(
  this: IExecuteFunctions | ILoadOptionsFunctions,
  method: IHttpRequestMethods,
  resource: string,
  qs: IDataObject = {},
  body: IDataObject | undefined = undefined,
) {
  const authMethod = this.getNodeParameter('authentication', 0) as string;
  
  const options: IHttpRequestOptions = {
    method,
    qs,
    body,
    url: `https://api.example.com${resource}`,
    json: true,
  };
  
  const credentialType = authMethod === 'accessToken' 
    ? 'myServiceApi' 
    : 'myServiceOAuth2Api';
  
  return this.helpers.httpRequestWithAuthentication.call(
    this, 
    credentialType, 
    options
  );
}
```

---

## Utility Functions

`shared/utils.ts`:

```typescript
// Parse Link header for pagination
export function parseLinkHeader(header: string | undefined): Record<string, string> {
  if (!header) return {};
  const links: Record<string, string> = {};
  for (const part of header.split(',')) {
    const match = part.match(/<([^>]+)>;\s*rel="([^"]+)"/);
    if (match) links[match[2]] = match[1];
  }
  return links;
}

// Convert camelCase to snake_case
export function toSnakeCase(str: string): string {
  return str.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`);
}

// Clean undefined values from object
export function cleanObject(obj: Record<string, any>): Record<string, any> {
  return Object.fromEntries(
    Object.entries(obj).filter(([_, v]) => v !== undefined && v !== '')
  );
}
```

---

## SDK Client Helper

`GenericFunctions.ts`:

```typescript
import type { IExecuteFunctions, ILoadOptionsFunctions } from 'n8n-workflow';
import { MySDK } from '@myservice/sdk';

export async function getClient(
  this: IExecuteFunctions | ILoadOptionsFunctions,
): Promise<MySDK> {
  const credentials = await this.getCredentials('myServiceApi') as {
    apiKey: string;
    projectId: number;
  };
  
  return new MySDK(credentials.apiKey, {
    projectId: credentials.projectId,
  });
}
```

---

## Next Steps

- [13 - Custom Execute Methods](./13-custom-execute-methods.md)
- [19 - External SDK Integration](./19-external-sdk-integration.md)
