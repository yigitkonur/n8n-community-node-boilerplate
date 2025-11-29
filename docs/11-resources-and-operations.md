# 11 - Resources and Operations

> Organizing multi-resource APIs with the resource/operation pattern

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** when your API has multiple resources (users, projects, tasks, etc.) and you want to organize code cleanly.

| You Need This When | You DON'T Need This When |
|-------------------|-------------------------|
| ✅ API has 2+ resources | ❌ Single-resource API |
| ✅ Each resource has CRUD ops | ❌ Simple node with few operations |
| ✅ Want maintainable code | ❌ Quick prototype |

**Directory Structure Created**:
```
nodes/MyNode/
├── MyNode.node.ts
└── resources/
    ├── user/
    │   ├── index.ts
    │   ├── create.ts
    │   └── getAll.ts
    └── project/
        └── ...
```

**Prerequisites**:
- [10-creating-your-first-node.md](./10-creating-your-first-node.md) - Basic node structure
- [12-declarative-routing.md](./12-declarative-routing.md) - Routing patterns

**Reference Directory**: `nodes/GithubIssues/resources/`

---

## The Resource/Operation Pattern

Most APIs have multiple resources (users, projects, tasks) with operations (create, read, update, delete). n8n uses a standard pattern:

```
Resource → Operation → Parameters
   ↓           ↓            ↓
 "User"     "Create"     {name, email}
```

---

## Directory Structure

```
nodes/MyService/
├── MyService.node.ts           # Main node file
├── resources/
│   ├── user/
│   │   ├── index.ts            # Exports userDescription
│   │   ├── create.ts           # Create user fields
│   │   ├── get.ts              # Get user fields
│   │   └── getAll.ts           # Get many users fields
│   ├── project/
│   │   ├── index.ts
│   │   ├── create.ts
│   │   └── getAll.ts
│   └── task/
│       ├── index.ts
│       ├── create.ts
│       ├── get.ts
│       ├── update.ts
│       └── delete.ts
└── shared/
    └── descriptions.ts         # Reusable property definitions
```

---

## Main Node File

`MyService.node.ts`:

```typescript
import { NodeConnectionTypes, INodeType, INodeTypeDescription } from 'n8n-workflow';
import { userDescription } from './resources/user';
import { projectDescription } from './resources/project';
import { taskDescription } from './resources/task';

export class MyService implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Service',
    name: 'myService',
    icon: 'file:myservice.svg',
    group: ['transform'],
    version: 1,
    subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}',
    description: 'Interact with My Service API',
    defaults: { name: 'My Service' },
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    
    credentials: [
      { name: 'myServiceApi', required: true },
    ],
    
    requestDefaults: {
      baseURL: 'https://api.myservice.com/v1',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
    },
    
    properties: [
      // Resource selector
      {
        displayName: 'Resource',
        name: 'resource',
        type: 'options',
        noDataExpression: true,
        options: [
          { name: 'User', value: 'user' },
          { name: 'Project', value: 'project' },
          { name: 'Task', value: 'task' },
        ],
        default: 'user',
      },
      
      // Spread resource descriptions
      ...userDescription,
      ...projectDescription,
      ...taskDescription,
    ],
  };
}
```

---

## Resource Index File

`resources/user/index.ts`:

```typescript
import type { INodeProperties } from 'n8n-workflow';
import { userCreateFields } from './create';
import { userGetFields } from './get';
import { userGetAllFields } from './getAll';

const showOnlyForUser = {
  resource: ['user'],
};

export const userDescription: INodeProperties[] = [
  // Operation selector
  {
    displayName: 'Operation',
    name: 'operation',
    type: 'options',
    noDataExpression: true,
    displayOptions: {
      show: showOnlyForUser,
    },
    options: [
      {
        name: 'Create',
        value: 'create',
        action: 'Create a user',
        description: 'Create a new user',
        routing: {
          request: {
            method: 'POST',
            url: '/users',
          },
        },
      },
      {
        name: 'Get',
        value: 'get',
        action: 'Get a user',
        description: 'Get a user by ID',
        routing: {
          request: {
            method: 'GET',
            url: '=/users/{{$parameter.userId}}',
          },
        },
      },
      {
        name: 'Get Many',
        value: 'getAll',
        action: 'Get many users',
        description: 'Get all users',
        routing: {
          request: {
            method: 'GET',
            url: '/users',
          },
        },
      },
    ],
    default: 'getAll',
  },
  
  // Spread operation-specific fields
  ...userCreateFields,
  ...userGetFields,
  ...userGetAllFields,
];
```

---

## Operation Field Files

### create.ts

```typescript
import type { INodeProperties } from 'n8n-workflow';

const showOnlyForUserCreate = {
  resource: ['user'],
  operation: ['create'],
};

export const userCreateFields: INodeProperties[] = [
  {
    displayName: 'Name',
    name: 'name',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: showOnlyForUserCreate,
    },
    description: 'The name of the user',
    routing: {
      send: {
        type: 'body',
        property: 'name',
      },
    },
  },
  {
    displayName: 'Email',
    name: 'email',
    type: 'string',
    required: true,
    default: '',
    placeholder: 'user@example.com',
    displayOptions: {
      show: showOnlyForUserCreate,
    },
    description: 'The email of the user',
    routing: {
      send: {
        type: 'body',
        property: 'email',
      },
    },
  },
  {
    displayName: 'Additional Fields',
    name: 'additionalFields',
    type: 'collection',
    placeholder: 'Add Field',
    default: {},
    displayOptions: {
      show: showOnlyForUserCreate,
    },
    options: [
      {
        displayName: 'Role',
        name: 'role',
        type: 'options',
        options: [
          { name: 'Admin', value: 'admin' },
          { name: 'Member', value: 'member' },
          { name: 'Viewer', value: 'viewer' },
        ],
        default: 'member',
        routing: {
          send: {
            type: 'body',
            property: 'role',
          },
        },
      },
    ],
  },
];
```

### get.ts

```typescript
import type { INodeProperties } from 'n8n-workflow';

const showOnlyForUserGet = {
  resource: ['user'],
  operation: ['get'],
};

export const userGetFields: INodeProperties[] = [
  {
    displayName: 'User ID',
    name: 'userId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: showOnlyForUserGet,
    },
    description: 'The ID of the user to retrieve',
  },
];
```

### getAll.ts

```typescript
import type { INodeProperties } from 'n8n-workflow';

const showOnlyForUserGetAll = {
  resource: ['user'],
  operation: ['getAll'],
};

export const userGetAllFields: INodeProperties[] = [
  {
    displayName: 'Return All',
    name: 'returnAll',
    type: 'boolean',
    default: false,
    displayOptions: {
      show: showOnlyForUserGetAll,
    },
    description: 'Whether to return all results or only up to a given limit',
  },
  {
    displayName: 'Limit',
    name: 'limit',
    type: 'number',
    default: 50,
    displayOptions: {
      show: {
        ...showOnlyForUserGetAll,
        returnAll: [false],
      },
    },
    typeOptions: {
      minValue: 1,
      maxValue: 100,
    },
    description: 'Max number of results to return',
    routing: {
      send: {
        type: 'query',
        property: 'limit',
      },
      output: {
        maxResults: '={{$value}}',
      },
    },
  },
  {
    displayName: 'Filters',
    name: 'filters',
    type: 'collection',
    placeholder: 'Add Filter',
    default: {},
    displayOptions: {
      show: showOnlyForUserGetAll,
    },
    options: [
      {
        displayName: 'Role',
        name: 'role',
        type: 'options',
        options: [
          { name: 'Admin', value: 'admin' },
          { name: 'Member', value: 'member' },
          { name: 'Viewer', value: 'viewer' },
        ],
        default: '',
        routing: {
          send: {
            type: 'query',
            property: 'role',
          },
        },
      },
      {
        displayName: 'Status',
        name: 'status',
        type: 'options',
        options: [
          { name: 'Active', value: 'active' },
          { name: 'Inactive', value: 'inactive' },
        ],
        default: '',
        routing: {
          send: {
            type: 'query',
            property: 'status',
          },
        },
      },
    ],
  },
];
```

---

## Shared Descriptions

`shared/descriptions.ts`:

```typescript
import type { INodeProperties } from 'n8n-workflow';

// Reusable user ID field with resource locator
export const userIdField: INodeProperties = {
  displayName: 'User',
  name: 'userId',
  type: 'resourceLocator',
  default: { mode: 'list', value: '' },
  required: true,
  modes: [
    {
      displayName: 'From List',
      name: 'list',
      type: 'list',
      placeholder: 'Select a user...',
      typeOptions: {
        searchListMethod: 'getUsers',
        searchable: true,
      },
    },
    {
      displayName: 'By ID',
      name: 'id',
      type: 'string',
      placeholder: 'user_abc123',
    },
  ],
};

// Reusable project ID field
export const projectIdField: INodeProperties = {
  displayName: 'Project',
  name: 'projectId',
  type: 'resourceLocator',
  default: { mode: 'list', value: '' },
  required: true,
  modes: [
    {
      displayName: 'From List',
      name: 'list',
      type: 'list',
      placeholder: 'Select a project...',
      typeOptions: {
        searchListMethod: 'getProjects',
        searchable: true,
      },
    },
    {
      displayName: 'By ID',
      name: 'id',
      type: 'string',
      placeholder: 'proj_abc123',
    },
  ],
};
```

Using shared descriptions:

```typescript
// In resources/task/create.ts
import { userIdField, projectIdField } from '../../shared/descriptions';

export const taskCreateFields: INodeProperties[] = [
  {
    ...userIdField,
    displayName: 'Assignee',
    name: 'assigneeId',
    displayOptions: {
      show: showOnlyForTaskCreate,
    },
  },
  {
    ...projectIdField,
    displayOptions: {
      show: showOnlyForTaskCreate,
    },
  },
  // ... other fields
];
```

---

## Best Practices

### 1. Consistent Naming

| Resource | File | Export |
|----------|------|--------|
| user | `resources/user/index.ts` | `userDescription` |
| project | `resources/project/index.ts` | `projectDescription` |
| task | `resources/task/index.ts` | `taskDescription` |

### 2. Display Options Pattern

```typescript
const showOnlyForUserCreate = {
  resource: ['user'],
  operation: ['create'],
};
```

### 3. Standard Operations

| Operation | Value | HTTP Method |
|-----------|-------|-------------|
| Create | `create` | POST |
| Get | `get` | GET |
| Get Many | `getAll` | GET |
| Update | `update` | PATCH/PUT |
| Delete | `delete` | DELETE |

### 4. Additional Fields Pattern

Use `collection` type for optional fields:

```typescript
{
  displayName: 'Additional Fields',
  name: 'additionalFields',
  type: 'collection',
  placeholder: 'Add Field',
  default: {},
  options: [
    // Optional fields here
  ],
}
```

---

## Next Steps

- [12 - Declarative Routing](./12-declarative-routing.md)
- [15 - Resource Locators](./15-resource-locators.md)
