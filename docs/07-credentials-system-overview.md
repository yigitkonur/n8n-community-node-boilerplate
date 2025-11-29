# 07 - Credentials System Overview

> How n8n handles authentication in community nodes

---

## What Are Credentials?

Credentials in n8n:
- Store sensitive authentication data securely
- Apply authentication to HTTP requests automatically
- Can be tested to verify validity
- Are reusable across multiple workflows

---

## Credential Types

| Type | Use Case | Complexity |
|------|----------|------------|
| **API Key/Token** | Simple token-based auth | Low |
| **HTTP Header Auth** | Custom header authentication | Low |
| **OAuth2** | OAuth 2.0 flows | Medium |
| **OAuth1** | Legacy OAuth 1.0 | Medium |
| **Custom** | Non-standard authentication | High |

---

## Basic Credential Structure

```typescript
import type {
  IAuthenticateGeneric,
  ICredentialTestRequest,
  ICredentialType,
  INodeProperties,
  Icon,
} from 'n8n-workflow';

export class MyServiceApi implements ICredentialType {
  // Unique identifier (matches node credential reference)
  name = 'myServiceApi';
  
  // Display name in UI
  displayName = 'My Service API';
  
  // Optional: Icon for credential
  icon: Icon = 'file:../icons/myservice.svg';
  
  // Link to documentation
  documentationUrl = 'https://docs.myservice.com/api';
  
  // Fields to collect from user
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
      required: true,
    },
  ];
  
  // How to apply credentials to requests
  authenticate: IAuthenticateGeneric = {
    type: 'generic',
    properties: {
      headers: {
        Authorization: '=Bearer {{$credentials.apiKey}}',
      },
    },
  };
  
  // Test the credentials
  test: ICredentialTestRequest = {
    request: {
      baseURL: 'https://api.myservice.com',
      url: '/me',
      method: 'GET',
    },
  };
}
```

---

## Credential Properties

### Standard Fields

```typescript
properties: INodeProperties[] = [
  {
    displayName: 'API Key',
    name: 'apiKey',
    type: 'string',
    typeOptions: { password: true },
    default: '',
    required: true,
    description: 'Your API key from the dashboard',
    placeholder: 'sk_...',
  },
  {
    displayName: 'Environment',
    name: 'environment',
    type: 'options',
    options: [
      { name: 'Production', value: 'production' },
      { name: 'Sandbox', value: 'sandbox' },
    ],
    default: 'production',
  },
]
```

### Secure Fields

Always use `typeOptions: { password: true }` for sensitive data:

```typescript
{
  displayName: 'API Secret',
  name: 'apiSecret',
  type: 'string',
  typeOptions: { password: true },  // Masks input
  default: '',
}
```

---

## Authentication Methods

### 1. Header Authentication

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    headers: {
      Authorization: '=Bearer {{$credentials.apiKey}}',
    },
  },
};
```

### 2. Query Parameter Authentication

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    qs: {
      api_key: '={{$credentials.apiKey}}',
    },
  },
};
```

### 3. Basic Authentication

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    auth: {
      type: 'basic',
      username: '={{$credentials.username}}',
      password: '={{$credentials.password}}',
    },
  },
};
```

### 4. Custom Header Format

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    headers: {
      'X-API-Key': '={{$credentials.apiKey}}',
      'X-API-Secret': '={{$credentials.apiSecret}}',
    },
  },
};
```

### 5. Multiple Methods Combined

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    headers: {
      'X-API-Key': '={{$credentials.apiKey}}',
    },
    qs: {
      client_id: '={{$credentials.clientId}}',
    },
  },
};
```

---

## Credential Testing

### Basic Test

```typescript
test: ICredentialTestRequest = {
  request: {
    baseURL: 'https://api.myservice.com',
    url: '/v1/me',
    method: 'GET',
  },
};
```

### Test with Headers

```typescript
test: ICredentialTestRequest = {
  request: {
    baseURL: 'https://api.myservice.com',
    url: '/v1/verify',
    method: 'GET',
    headers: {
      Accept: 'application/json',
    },
  },
};
```

### No Test Available

If the API doesn't have a verification endpoint:

```typescript
// Don't define test property
// User will see "This credential type does not have a test configured"
```

---

## Connecting Credentials to Nodes

In your node's description:

```typescript
credentials: [
  {
    name: 'myServiceApi',  // Matches credential's `name` property
    required: true,
  },
],
```

### Multiple Credential Options

```typescript
credentials: [
  {
    name: 'myServiceApi',
    required: true,
    displayOptions: {
      show: {
        authentication: ['apiKey'],
      },
    },
  },
  {
    name: 'myServiceOAuth2Api',
    required: true,
    displayOptions: {
      show: {
        authentication: ['oauth2'],
      },
    },
  },
],
```

---

## Using Credentials in Code

### In Execute Method

```typescript
async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const credentials = await this.getCredentials('myServiceApi') as {
    apiKey: string;
    environment: string;
  };
  
  const baseUrl = credentials.environment === 'production'
    ? 'https://api.myservice.com'
    : 'https://sandbox.myservice.com';
  
  // Use with SDK
  const client = new MyServiceClient(credentials.apiKey, { baseUrl });
  
  // Or with HTTP request
  const response = await this.helpers.httpRequestWithAuthentication.call(
    this,
    'myServiceApi',
    {
      url: `${baseUrl}/endpoint`,
      method: 'GET',
    }
  );
}
```

### In Load Options

```typescript
methods = {
  loadOptions: {
    async getProjects(this: ILoadOptionsFunctions) {
      const credentials = await this.getCredentials('myServiceApi') as {
        apiKey: string;
      };
      
      // Fetch and return options
      return [];
    },
  },
};
```

---

## Icon Configuration

```typescript
// Single icon
icon: Icon = 'file:../icons/myservice.svg';

// Light/dark variants
icon: Icon = {
  light: 'file:../icons/myservice.svg',
  dark: 'file:../icons/myservice.dark.svg',
};
```

---

## File Organization

```
credentials/
├── MyServiceApi.credentials.ts        # API key auth
├── MyServiceOAuth2Api.credentials.ts  # OAuth2 auth
```

Register in package.json:

```json
{
  "n8n": {
    "credentials": [
      "dist/credentials/MyServiceApi.credentials.js",
      "dist/credentials/MyServiceOAuth2Api.credentials.js"
    ]
  }
}
```

---

## Next Steps

- [08 - API Key Credentials](./08-api-key-credentials.md)
- [09 - OAuth2 Credentials](./09-oauth2-credentials.md)
