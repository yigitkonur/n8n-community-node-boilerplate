# 08 - API Key Credentials

> Implementing token-based authentication

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** when creating credentials for APIs that use API keys, tokens, or bearer authentication.

| What You're Building | This Document Covers |
|---------------------|---------------------|
| Bearer token auth | ✅ Yes - see "Bearer Token" section |
| Custom header auth | ✅ Yes - see "Custom Header" section |
| API key + Project ID | ✅ Yes - see "Multi-Field Credentials" |
| OAuth2 | ❌ No - read → [09-oauth2-credentials.md](./09-oauth2-credentials.md) |

**After Creating Credentials, Read**:
- [10-creating-your-first-node.md](./10-creating-your-first-node.md) - Connect to your node
- [02-package-json-configuration.md](./02-package-json-configuration.md) - Register credentials

**Reference File**: `credentials/GithubIssuesApi.credentials.ts`

---

## Complete Example

```typescript
import type {
  IAuthenticateGeneric,
  Icon,
  ICredentialTestRequest,
  ICredentialType,
  INodeProperties,
} from 'n8n-workflow';

export class MyServiceApi implements ICredentialType {
  name = 'myServiceApi';
  
  displayName = 'My Service API';
  
  icon: Icon = { 
    light: 'file:../icons/myservice.svg', 
    dark: 'file:../icons/myservice.dark.svg' 
  };
  
  documentationUrl = 'https://docs.myservice.com/api/authentication';
  
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
      required: true,
      description: 'Your API key from the My Service dashboard',
      placeholder: 'ms_live_...',
    },
  ];
  
  authenticate: IAuthenticateGeneric = {
    type: 'generic',
    properties: {
      headers: {
        Authorization: '=Bearer {{$credentials.apiKey}}',
      },
    },
  };
  
  test: ICredentialTestRequest = {
    request: {
      baseURL: 'https://api.myservice.com',
      url: '/v1/me',
      method: 'GET',
    },
  };
}
```

---

## Common Authentication Patterns

### Bearer Token (Most Common)

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

**Example APIs**: OpenAI, Stripe, GitHub

### Token Header (Without Bearer)

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    headers: {
      Authorization: '=token {{$credentials.accessToken}}',
    },
  },
};
```

**Example APIs**: GitHub (classic tokens)

### Custom Header

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    headers: {
      'X-API-Key': '={{$credentials.apiKey}}',
    },
  },
};
```

**Example APIs**: Many internal/enterprise APIs

### Multiple Headers

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

### Query Parameter

```typescript
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    qs: {
      apiKey: '={{$credentials.apiKey}}',
    },
  },
};
```

**Example APIs**: Legacy APIs, some mapping services

### Basic Auth

```typescript
properties: INodeProperties[] = [
  {
    displayName: 'Username',
    name: 'username',
    type: 'string',
    default: '',
    required: true,
  },
  {
    displayName: 'Password',
    name: 'password',
    type: 'string',
    typeOptions: { password: true },
    default: '',
    required: true,
  },
];

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

---

## Multi-Field Credentials

### API Key + Project ID

```typescript
properties: INodeProperties[] = [
  {
    displayName: 'API Key',
    name: 'apiKey',
    type: 'string',
    typeOptions: { password: true },
    default: '',
    required: true,
    description: 'Your API key from the dashboard (Settings > API Keys)',
    placeholder: 'lat_...',
  },
  {
    displayName: 'Project ID',
    name: 'projectId',
    type: 'number',
    default: 0,
    required: true,
    description: 'Your project ID (found in project settings)',
    placeholder: '12345',
  },
];
```

### Environment Selection

```typescript
properties: INodeProperties[] = [
  {
    displayName: 'Environment',
    name: 'environment',
    type: 'options',
    options: [
      { name: 'Production', value: 'production' },
      { name: 'Staging', value: 'staging' },
      { name: 'Development', value: 'development' },
    ],
    default: 'production',
    description: 'Select the API environment',
  },
  {
    displayName: 'API Key',
    name: 'apiKey',
    type: 'string',
    typeOptions: { password: true },
    default: '',
    required: true,
  },
];

// Use in node to select base URL
// const baseUrl = credentials.environment === 'production'
//   ? 'https://api.example.com'
//   : 'https://staging-api.example.com';
```

### Region Selection

```typescript
properties: INodeProperties[] = [
  {
    displayName: 'Region',
    name: 'region',
    type: 'options',
    options: [
      { name: 'US', value: 'us' },
      { name: 'EU', value: 'eu' },
      { name: 'Asia Pacific', value: 'apac' },
    ],
    default: 'us',
  },
  {
    displayName: 'API Key',
    name: 'apiKey',
    type: 'string',
    typeOptions: { password: true },
    default: '',
    required: true,
  },
];
```

---

## Credential Testing

### Simple GET Request

```typescript
test: ICredentialTestRequest = {
  request: {
    baseURL: 'https://api.myservice.com',
    url: '/v1/me',
    method: 'GET',
  },
};
```

### With Custom Headers

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

### POST Request

```typescript
test: ICredentialTestRequest = {
  request: {
    baseURL: 'https://api.myservice.com',
    url: '/v1/auth/verify',
    method: 'POST',
    body: JSON.stringify({}),
    headers: {
      'Content-Type': 'application/json',
    },
  },
};
```

### No Test (SDK-based)

When using SDKs, credential testing happens in the node's loadOptions:

```typescript
// In credential file - no test defined
// test: ICredentialTestRequest = { ... };

// Note: Credential testing happens in the node's loadOptions methods
// when fetching data, which validates both API key and project ID
```

---

## Using Credentials in Nodes

### Declarative (Automatic)

When using declarative routing, n8n automatically applies credentials:

```typescript
// Node definition
credentials: [
  { name: 'myServiceApi', required: true },
],

requestDefaults: {
  baseURL: 'https://api.myservice.com',
},

// Authentication is automatically applied via credential's `authenticate` config
```

### Programmatic (Manual)

```typescript
async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  // Type your credentials
  const credentials = await this.getCredentials('myServiceApi') as {
    apiKey: string;
    projectId: number;
  };
  
  // Use with HTTP helper (applies authentication automatically)
  const response = await this.helpers.httpRequestWithAuthentication.call(
    this,
    'myServiceApi',  // Credential name
    {
      url: 'https://api.myservice.com/data',
      method: 'GET',
    }
  );
  
  // Or use with SDK
  const client = new MyClient(credentials.apiKey, {
    projectId: credentials.projectId,
  });
}
```

---

## Security Best Practices

1. **Always use `password: true`** for sensitive fields
2. **Never log credentials** - sanitize error messages
3. **Use specific types** - type credentials in execute method
4. **Provide documentation URL** - help users find their keys
5. **Add placeholder examples** - show expected format

```typescript
// Sanitizing errors
catch (error) {
  const sanitizedError = error.message.replace(
    /api[-_]?key[s]?\s*[:=]\s*[a-z0-9_-]{10,}/gi,
    '[REDACTED]'
  );
  throw new Error(sanitizedError);
}
```

---

## Next Steps

- [09 - OAuth2 Credentials](./09-oauth2-credentials.md)
- [13 - Custom Execute Methods](./13-custom-execute-methods.md)
