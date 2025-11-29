# AI Agent Guide: Understanding n8n Node Architecture Through Code

> **READ-ONLY REFERENCE**: Do not modify source files. Use this guide to understand patterns, then apply them to your own implementations.

---

## Learning Path: Read Files in This Order

### Phase 1: Programmatic Node Pattern

**File**: `nodes/Example/Example.node.ts` (79 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 1-7 | Import types from n8n-workflow | Required interfaces for TypeScript |
| 9-35 | `description` object | Defines UI: displayName, icon, properties users see |
| 23-34 | `properties` array | Each object = one UI field |
| 41-77 | `execute()` method | **Core logic** - runs when node executes |
| 42 | `getInputData()` | Get items from previous node |
| 50-74 | Item loop | Process each input item individually |
| 52 | `getNodeParameter()` | Get user-configured value for this item |
| 55 | `item.json.myString = myString` | Add/modify output data |
| 59-73 | Error handling | `continueOnFail()` or throw `NodeOperationError` |

**Key Insight**: Programmatic nodes use `execute()` for full control. Use when: custom logic, non-REST APIs, SDK integration, triggers.

```
Data Flow:
Input Items → execute() → Loop each item → getNodeParameter() → Process → Return items
```

---

### Phase 2: Declarative Node Pattern

**File**: `nodes/GithubIssues/GithubIssues.node.ts` (97 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 1-6 | Import resource descriptions | Modular organization by resource |
| 23-42 | `credentials` array | Supports multiple auth types (API key + OAuth2) |
| 43-49 | `requestDefaults` | Base URL + headers for ALL API calls |
| 72-81 | Resource picker | Users choose: Issue or Issue Comment |
| 82-83 | `...issueDescription` | Spread operator merges all operation properties |
| 87-91 | `listSearch` methods | Enable dynamic dropdowns |

**Key Insight**: Declarative nodes have NO `execute()` method. Properties with `routing` config automatically make HTTP requests.

```
Pattern:
properties: [
  { /* auth selector */ },
  { /* resource picker */ },
  ...issueDescription,        // ← Imported from resources/issue/index.ts
  ...issueCommentDescription, // ← Imported from resources/issueComment/index.ts
]
```

**Decision Point**: REST API → Declarative | Complex logic/SDK → Programmatic

---

### Phase 3: Resource Organization

**File**: `nodes/GithubIssues/resources/issue/index.ts` (76 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 1-6 | Import operation properties | Each operation in separate file |
| 10-11 | `issueDescription` export | Single array containing all Issue properties |
| 20-29 | Get Many operation | Inline `routing` with method + URL |
| 25 | URL expression | `=/repos/{{$parameter.owner}}/...` injects params |
| 31-40 | Get operation | Different URL pattern for single item |
| 42-52 | Create operation | POST method with body routing |
| 55-70 | Common fields | Owner + Repository selectors (reused) |
| 71-74 | Operation spreads | `...getAllProperties`, `...getProperties`, etc. |

**Key Insight**: Each resource folder contains:
```
issue/
├── index.ts     ← Operation picker + common fields + property composition
├── get.ts       ← "Get" operation-specific properties
├── getAll.ts    ← "Get Many" operation-specific properties  
└── create.ts    ← "Create" operation-specific properties
```

---

### Phase 4: Operation Properties

**File**: `nodes/GithubIssues/resources/issue/create.ts` (75 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 3-6 | `displayOptions.show` | Only show when resource=issue AND operation=create |
| 8-23 | Title field | `routing.send.type: 'body'` → sends to request body |
| 18-23 | `routing.send` | `property: 'title'` = API field name |
| 46-72 | Labels collection | `fixedCollection` with expression transform |
| 62-65 | Value expression | `={{ $value.map(l => l.label) }}` → array format |

**Routing Send Types**:
```typescript
routing: {
  send: {
    type: 'body',    // → request.body.fieldName
    type: 'query',   // → ?fieldName=value
    type: 'header',  // → request.headers.fieldName
    property: 'apiFieldName',
    value: '={{ expression }}'  // Optional transform
  }
}
```

**File**: `nodes/GithubIssues/resources/issue/getAll.ts` (125 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 9-32 | Limit field | `routing.send` to query param `per_page` |
| 28-30 | `output.maxResults` | Limits items returned to workflow |
| 34-62 | Return All toggle | Enables pagination |
| 50-60 | Pagination config | Uses Link header for next page URL |
| 64-120 | Filters collection | Nested options with individual routing |

**Pagination Pattern**:
```typescript
routing: {
  operations: {
    pagination: {
      type: 'generic',
      properties: {
        continue: '={{ $response.headers.link?.includes("next") }}',
        request: { url: '={{ /* next page URL from Link header */ }}' }
      }
    }
  }
}
```

**File**: `nodes/GithubIssues/resources/issue/get.ts` (15 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 9-13 | Reuses `issueSelect` | DRY principle - import from shared |

**Key Insight**: Minimal file because it reuses `issueSelect` from `shared/descriptions.ts`.

---

### Phase 5: Shared UI Components

**File**: `nodes/GithubIssues/shared/descriptions.ts` (152 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 3-55 | `repoOwnerSelect` | Resource Locator with 3 input modes |
| 9-19 | List mode | Searchable dropdown via `searchListMethod: 'getUsers'` |
| 20-38 | URL mode | Paste GitHub URL, extract owner with regex |
| 39-53 | Name mode | Manual entry with validation |
| 57-121 | `repoNameSelect` | Same pattern for repository |
| 123-152 | `issueSelect` | Issue selector using `getIssues` |

**Resource Locator Pattern**:
```typescript
{
  type: 'resourceLocator',
  modes: [
    { 
      name: 'list',
      type: 'list',
      typeOptions: { searchListMethod: 'getUsers' }  // ← Links to listSearch
    },
    { 
      name: 'url',
      type: 'string',
      extractValue: { type: 'regex', regex: '...' }  // ← Parse URL
    },
    { 
      name: 'name',
      type: 'string'  // ← Direct input
    }
  ]
}
```

**Why 3 Modes**: Flexible UX - users can search, paste URL, or type directly.

---

### Phase 6: List Search Methods

**File**: `nodes/GithubIssues/listSearch/getUsers.ts` (50 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 18-22 | Function signature | `ILoadOptionsFunctions` context |
| 23-24 | Pagination token | Page number from previous call |
| 26-29 | Empty response init | `results: [], paginationToken: undefined` |
| 31-38 | Try-catch | Graceful failure returns empty results |
| 32-36 | API call | Uses `githubApiRequest` helper |
| 41-45 | Transform response | `{ name, value, url }` format required |
| 47-48 | Next page token | Increment page if results exist |

**Return Contract**:
```typescript
interface INodeListSearchResult {
  results: Array<{ name: string; value: string; url?: string }>;
  paginationToken?: string;  // undefined = no more pages
}
```

**File**: `nodes/GithubIssues/listSearch/getRepositories.ts` (51 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 22 | `getCurrentNodeParameter('owner')` | Get sibling field value |
| 25-26 | Build search query | Combine filter + owner context |
| 37-39 | Graceful failure | Return empty if owner has no repos |

**Key Difference**: `getCurrentNodeParameter()` for same-level fields vs `getNodeParameter()` for item data.

**File**: `nodes/GithubIssues/listSearch/getIssues.ts` (50 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 30-32 | Get multiple params | Owner + Repository needed |
| 33-35 | Build query | `repo:owner/name` + user filter |

---

### Phase 7: Transport Layer

**File**: `nodes/GithubIssues/shared/transport.ts` (33 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 1-9 | Import types | Multiple context types supported |
| 11-16 | Function signature | Works in execute, loadOptions, hooks |
| 18 | Get auth method | User's authentication selection |
| 20-26 | Build request options | Method, URL, query, body |
| 28-29 | Credential mapping | `accessToken` → `githubIssuesApi` |
| 31 | `httpRequestWithAuthentication` | n8n handles auth injection |

**Abstraction Pattern**:
```typescript
// Instead of duplicating auth logic:
await githubApiRequest.call(this, 'GET', '/users', { q: 'filter' });

// Transport handles:
// 1. Build full URL
// 2. Select credential type
// 3. Inject auth headers
// 4. Make request
```

---

### Phase 8: Utilities

**File**: `nodes/GithubIssues/shared/utils.ts` (15 lines)

| Lines | What It Does | Why It Matters |
|-------|--------------|----------------|
| 1-14 | `parseLinkHeader()` | Parse GitHub pagination header |
| 4-10 | Regex extraction | Get URL and rel type (next, prev, last) |

**GitHub Link Header**:
```
Link: <https://api.github.com?page=2>; rel="next", <https://api.github.com?page=5>; rel="last"
```

**Parsed Result**:
```javascript
{ next: 'https://...?page=2', last: 'https://...?page=5' }
```

**Usage in Pagination**:
```typescript
continue: '={{ !!parseLinkHeader($response.headers?.link).next }}'
```

---

## Architecture Summary

### File Organization Template

```
nodes/YourNode/
├── YourNode.node.ts           ← Entry: credentials, resource picker, requestDefaults
├── YourNode.node.json         ← Metadata: categories, docs URL
├── resources/
│   ├── resourceA/
│   │   ├── index.ts           ← Operations array + common fields
│   │   ├── get.ts             ← Get operation properties
│   │   ├── getAll.ts          ← Get Many + pagination
│   │   └── create.ts          ← Create + body routing
│   └── resourceB/
│       └── index.ts
├── listSearch/
│   ├── getResourceA.ts        ← Dynamic dropdown for A
│   └── getResourceB.ts        ← Dynamic dropdown for B
└── shared/
    ├── descriptions.ts        ← Reusable field definitions
    ├── transport.ts           ← API call wrapper
    └── utils.ts               ← Helper functions
```

### Decision Tree

```
New Integration?
    │
    ├── REST API with standard CRUD? → DECLARATIVE
    │   └── Use: routing in properties, requestDefaults, no execute()
    │
    ├── Official SDK available? → PROGRAMMATIC
    │   └── Use: execute() method, SDK initialization, manual calls
    │
    ├── Complex auth (signing, tokens)? → PROGRAMMATIC
    │   └── Use: execute() with custom auth logic
    │
    └── Non-REST (GraphQL, WebSocket)? → PROGRAMMATIC
        └── Use: execute() with protocol-specific handling
```

### Routing Quick Reference

| Send Type | Result | Example |
|-----------|--------|---------|
| `body` | `request.body.field = value` | POST/PUT data |
| `query` | `?field=value` | GET parameters |
| `header` | `request.headers.field = value` | Custom headers |

### Expression Syntax

| Expression | Meaning |
|------------|---------|
| `{{$value}}` | Current field's value |
| `{{$parameter.fieldName}}` | Another field's value |
| `{{$response.body}}` | API response body |
| `{{$response.headers}}` | API response headers |

---

## Study Checklist

- [ ] Understand `execute()` flow in Example.node.ts
- [ ] Understand declarative structure in GithubIssues.node.ts
- [ ] Understand resource organization in resources/issue/index.ts
- [ ] Understand `routing.send` in create.ts
- [ ] Understand pagination in getAll.ts
- [ ] Understand Resource Locator in shared/descriptions.ts
- [ ] Understand list search return format in listSearch/*.ts
- [ ] Understand transport abstraction in shared/transport.ts
- [ ] Understand property spreading with `...` operator
- [ ] Understand `displayOptions.show` conditional display

---

## Quick Start: Building Your Own Node

1. **Choose pattern**: REST API → copy GithubIssues structure | Other → copy Example structure
2. **Create files**: Follow file organization template above
3. **Define resources**: One folder per API resource
4. **Add operations**: One file per operation (get, getAll, create, update, delete)
5. **Build shared components**: Resource locators, transport, utilities
6. **Register in package.json**: Add to `n8n.nodes` and `n8n.credentials` arrays

**Remember**: These files are reference implementations. Study the patterns, then apply to your API.
