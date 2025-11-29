# 09 - OAuth2 Credentials

> Setting up OAuth2 authentication flows

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** when the target service uses OAuth2 authentication.

| OAuth2 Is For | NOT For |
|--------------|---------|
| ✅ Services with "Connect with X" button | ❌ Simple API keys |
| ✅ User authorization flows | ❌ Server-to-server with static tokens |
| ✅ GitHub, Google, Slack, etc. | ❌ Bearer tokens from dashboard |

**Key Difference from API Keys**:
- OAuth2 requires user interaction (clicking "Authorize")
- Credentials are obtained through a redirect flow
- Access tokens can be refreshed

**Prerequisites**:
- [07-credentials-system-overview.md](./07-credentials-system-overview.md) - How credentials work
- You'll need Client ID and Client Secret from the service's developer console

**Reference File**: `credentials/GithubIssuesOAuth2Api.credentials.ts`

---

## OAuth2 Credential Structure

```typescript
import type { Icon, ICredentialType, INodeProperties } from 'n8n-workflow';

export class MyServiceOAuth2Api implements ICredentialType {
  name = 'myServiceOAuth2Api';
  
  extends = ['oAuth2Api'];  // Inherit OAuth2 base
  
  displayName = 'My Service OAuth2 API';
  
  icon: Icon = { 
    light: 'file:../icons/myservice.svg', 
    dark: 'file:../icons/myservice.dark.svg' 
  };
  
  documentationUrl = 'https://docs.myservice.com/oauth';
  
  properties: INodeProperties[] = [
    {
      displayName: 'Grant Type',
      name: 'grantType',
      type: 'hidden',
      default: 'authorizationCode',
    },
    {
      displayName: 'Authorization URL',
      name: 'authUrl',
      type: 'hidden',
      default: 'https://myservice.com/oauth/authorize',
      required: true,
    },
    {
      displayName: 'Access Token URL',
      name: 'accessTokenUrl',
      type: 'hidden',
      default: 'https://myservice.com/oauth/token',
      required: true,
    },
    {
      displayName: 'Scope',
      name: 'scope',
      type: 'hidden',
      default: 'read write',
    },
    {
      displayName: 'Auth URI Query Parameters',
      name: 'authQueryParameters',
      type: 'hidden',
      default: '',
    },
    {
      displayName: 'Authentication',
      name: 'authentication',
      type: 'hidden',
      default: 'header',
    },
  ];
}
```

---

## Complete GitHub OAuth2 Example

```typescript
import type { Icon, ICredentialType, INodeProperties } from 'n8n-workflow';

export class GithubIssuesOAuth2Api implements ICredentialType {
  name = 'githubIssuesOAuth2Api';
  
  extends = ['oAuth2Api'];
  
  displayName = 'GitHub Issues OAuth2 API';
  
  icon: Icon = { 
    light: 'file:../icons/github.svg', 
    dark: 'file:../icons/github.dark.svg' 
  };
  
  documentationUrl = 'https://docs.github.com/en/apps/oauth-apps';
  
  properties: INodeProperties[] = [
    {
      displayName: 'Grant Type',
      name: 'grantType',
      type: 'hidden',
      default: 'authorizationCode',
    },
    {
      displayName: 'Authorization URL',
      name: 'authUrl',
      type: 'hidden',
      default: 'https://github.com/login/oauth/authorize',
      required: true,
    },
    {
      displayName: 'Access Token URL',
      name: 'accessTokenUrl',
      type: 'hidden',
      default: 'https://github.com/login/oauth/access_token',
      required: true,
    },
    {
      displayName: 'Scope',
      name: 'scope',
      type: 'hidden',
      default: 'repo',
    },
    {
      displayName: 'Auth URI Query Parameters',
      name: 'authQueryParameters',
      type: 'hidden',
      default: '',
    },
    {
      displayName: 'Authentication',
      name: 'authentication',
      type: 'hidden',
      default: 'header',
    },
  ];
}
```

---

## OAuth2 Configuration Fields

### Required Fields

| Field | Purpose | Example |
|-------|---------|---------|
| `authUrl` | Authorization endpoint | `https://service.com/oauth/authorize` |
| `accessTokenUrl` | Token exchange endpoint | `https://service.com/oauth/token` |
| `grantType` | OAuth2 grant type | `authorizationCode` |

### Optional Fields

| Field | Purpose | Example |
|-------|---------|---------|
| `scope` | Requested permissions | `read write delete` |
| `authQueryParameters` | Extra auth params | `access_type=offline` |
| `authentication` | Token placement | `header` or `body` |

---

## Grant Types

### Authorization Code (Most Common)

```typescript
{
  displayName: 'Grant Type',
  name: 'grantType',
  type: 'hidden',
  default: 'authorizationCode',
}
```

Used for: Web applications with user interaction

### Client Credentials

```typescript
{
  displayName: 'Grant Type',
  name: 'grantType',
  type: 'hidden',
  default: 'clientCredentials',
}
```

Used for: Server-to-server without user interaction

### PKCE (Proof Key for Code Exchange)

```typescript
{
  displayName: 'Grant Type',
  name: 'grantType',
  type: 'hidden',
  default: 'pkce',
}
```

Used for: Mobile/SPA applications

---

## Scope Configuration

### Hidden Scope (Fixed)

```typescript
{
  displayName: 'Scope',
  name: 'scope',
  type: 'hidden',
  default: 'repo user:email',
}
```

### User-Selectable Scope

```typescript
{
  displayName: 'Scope',
  name: 'scope',
  type: 'multiOptions',
  options: [
    { name: 'Read Repositories', value: 'repo:read' },
    { name: 'Write Repositories', value: 'repo:write' },
    { name: 'User Email', value: 'user:email' },
  ],
  default: ['repo:read'],
}
```

---

## Authentication Method

### Header (Most Common)

```typescript
{
  displayName: 'Authentication',
  name: 'authentication',
  type: 'hidden',
  default: 'header',
}
```

Sends: `Authorization: Bearer {access_token}`

### Body

```typescript
{
  displayName: 'Authentication',
  name: 'authentication',
  type: 'hidden',
  default: 'body',
}
```

Sends token in request body

---

## Using Multiple Auth Methods in Node

### Node Credential Configuration

```typescript
// In node description
properties: [
  {
    displayName: 'Authentication',
    name: 'authentication',
    type: 'options',
    options: [
      { name: 'Access Token', value: 'accessToken' },
      { name: 'OAuth2', value: 'oAuth2' },
    ],
    default: 'accessToken',
  },
  // ... other properties
],

credentials: [
  {
    name: 'myServiceApi',
    required: true,
    displayOptions: {
      show: { authentication: ['accessToken'] },
    },
  },
  {
    name: 'myServiceOAuth2Api',
    required: true,
    displayOptions: {
      show: { authentication: ['oAuth2'] },
    },
  },
],
```

### Selecting Credential in Code

```typescript
// In transport.ts
export async function myApiRequest(
  this: IExecuteFunctions | ILoadOptionsFunctions,
  method: IHttpRequestMethods,
  resource: string,
) {
  const authenticationMethod = this.getNodeParameter('authentication', 0);
  
  const credentialType = authenticationMethod === 'accessToken'
    ? 'myServiceApi'
    : 'myServiceOAuth2Api';
  
  return this.helpers.httpRequestWithAuthentication.call(
    this,
    credentialType,
    {
      url: `https://api.myservice.com${resource}`,
      method,
      json: true,
    }
  );
}
```

---

## OAuth2 Flow Visualization

```
┌─────────────────┐     1. Click "Connect"      ┌─────────────────┐
│   n8n Editor    │ ────────────────────────────▶│  Service Auth   │
└─────────────────┘                              │     Page        │
                                                 └────────┬────────┘
                                                          │
                                                 2. User approves
                                                          │
                                                          ▼
┌─────────────────┐     3. Authorization code    ┌─────────────────┐
│   n8n Backend   │ ◀────────────────────────────│  Redirect URI   │
└────────┬────────┘                              └─────────────────┘
         │
         │ 4. Exchange code for tokens
         ▼
┌─────────────────┐     5. Access + Refresh     ┌─────────────────┐
│  Token Endpoint │ ────────────────────────────▶│   n8n Stores    │
└─────────────────┘                              │    Tokens       │
                                                 └─────────────────┘
```

---

## Redirect URI

n8n provides the redirect URI automatically. Users need to configure it in the OAuth app:

```
https://your-n8n-instance.com/rest/oauth2-credential/callback
```

For local development:
```
http://localhost:5678/rest/oauth2-credential/callback
```

---

## Testing OAuth2 Credentials

OAuth2 credentials don't use the `test` property. Instead, the OAuth flow validates the credentials during the authorization process.

---

## Common OAuth2 Providers

### Google

```typescript
{
  authUrl: 'https://accounts.google.com/o/oauth2/v2/auth',
  accessTokenUrl: 'https://oauth2.googleapis.com/token',
  scope: 'https://www.googleapis.com/auth/drive.readonly',
  authQueryParameters: 'access_type=offline&prompt=consent',
}
```

### Microsoft

```typescript
{
  authUrl: 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
  accessTokenUrl: 'https://login.microsoftonline.com/common/oauth2/v2.0/token',
  scope: 'openid profile offline_access',
}
```

### Slack

```typescript
{
  authUrl: 'https://slack.com/oauth/v2/authorize',
  accessTokenUrl: 'https://slack.com/api/oauth.v2.access',
  scope: 'channels:read users:read',
}
```

---

## Next Steps

- [07 - Credentials System Overview](./07-credentials-system-overview.md)
- [18 - Helper Functions and Utilities](./18-helper-functions-and-utilities.md)
