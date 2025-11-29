# 10 - Creating Your First Node

> Step-by-step guide to building a basic n8n node

---

## 🤖 AI Agent Context

**THIS IS THE STARTING POINT** for building any n8n node. Read this document first.

| After Reading This | Read Next Based on Your Approach |
|-------------------|----------------------------------|
| ✅ Basic node structure | **Declarative (REST API)**: → [12-declarative-routing.md](./12-declarative-routing.md) |
| ✅ File creation steps | **Programmatic (SDK)**: → [13-custom-execute-methods.md](./13-custom-execute-methods.md) |
| ✅ Credential setup | **Both need**: → [08-api-key-credentials.md](./08-api-key-credentials.md) |

**Related Documents**:
- [05-declarative-vs-programmatic-nodes.md](./05-declarative-vs-programmatic-nodes.md) - Choose your approach
- [06-node-properties-reference.md](./06-node-properties-reference.md) - All property types
- [02-package-json-configuration.md](./02-package-json-configuration.md) - Register your node

---

## Overview

In this guide, you'll create a simple node that interacts with a REST API. We'll build a node for a fictional "Task Manager" API.

---

## Step 1: Set Up Project Structure

```bash
# Create node directory
mkdir -p nodes/TaskManager

# Create files
touch nodes/TaskManager/TaskManager.node.ts
touch nodes/TaskManager/TaskManager.node.json
touch nodes/TaskManager/taskmanager.svg
touch credentials/TaskManagerApi.credentials.ts
```

Directory structure:
```
├── credentials/
│   └── TaskManagerApi.credentials.ts
├── nodes/
│   └── TaskManager/
│       ├── TaskManager.node.ts
│       ├── TaskManager.node.json
│       └── taskmanager.svg
```

---

## Step 2: Create the Credential

`credentials/TaskManagerApi.credentials.ts`:

```typescript
import type {
  IAuthenticateGeneric,
  ICredentialTestRequest,
  ICredentialType,
  INodeProperties,
} from 'n8n-workflow';

export class TaskManagerApi implements ICredentialType {
  name = 'taskManagerApi';
  displayName = 'Task Manager API';
  documentationUrl = 'https://api.taskmanager.example/docs';
  
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
      required: true,
      description: 'Your Task Manager API key',
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
      baseURL: 'https://api.taskmanager.example',
      url: '/v1/me',
      method: 'GET',
    },
  };
}
```

---

## Step 3: Create the Node (Declarative Style)

`nodes/TaskManager/TaskManager.node.ts`:

```typescript
import { 
  NodeConnectionTypes, 
  type INodeType, 
  type INodeTypeDescription 
} from 'n8n-workflow';

export class TaskManager implements INodeType {
  description: INodeTypeDescription = {
    // Basic info
    displayName: 'Task Manager',
    name: 'taskManager',
    icon: 'file:taskmanager.svg',
    group: ['transform'],
    version: 1,
    subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}',
    description: 'Manage tasks using the Task Manager API',
    
    // Defaults
    defaults: {
      name: 'Task Manager',
    },
    
    // Connections
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    
    // Credentials
    credentials: [
      {
        name: 'taskManagerApi',
        required: true,
      },
    ],
    
    // Request defaults
    requestDefaults: {
      baseURL: 'https://api.taskmanager.example/v1',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
    },
    
    // Properties
    properties: [
      // Resource
      {
        displayName: 'Resource',
        name: 'resource',
        type: 'options',
        noDataExpression: true,
        options: [
          {
            name: 'Task',
            value: 'task',
          },
        ],
        default: 'task',
      },
      
      // Operations
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        noDataExpression: true,
        displayOptions: {
          show: {
            resource: ['task'],
          },
        },
        options: [
          {
            name: 'Create',
            value: 'create',
            action: 'Create a task',
            description: 'Create a new task',
            routing: {
              request: {
                method: 'POST',
                url: '/tasks',
              },
            },
          },
          {
            name: 'Get',
            value: 'get',
            action: 'Get a task',
            description: 'Get a task by ID',
            routing: {
              request: {
                method: 'GET',
                url: '=/tasks/{{$parameter.taskId}}',
              },
            },
          },
          {
            name: 'Get Many',
            value: 'getAll',
            action: 'Get many tasks',
            description: 'Get all tasks',
            routing: {
              request: {
                method: 'GET',
                url: '/tasks',
              },
            },
          },
          {
            name: 'Delete',
            value: 'delete',
            action: 'Delete a task',
            description: 'Delete a task by ID',
            routing: {
              request: {
                method: 'DELETE',
                url: '=/tasks/{{$parameter.taskId}}',
              },
            },
          },
        ],
        default: 'getAll',
      },
      
      // Task ID (for get, delete)
      {
        displayName: 'Task ID',
        name: 'taskId',
        type: 'string',
        required: true,
        default: '',
        displayOptions: {
          show: {
            resource: ['task'],
            operation: ['get', 'delete'],
          },
        },
        description: 'The ID of the task',
      },
      
      // Title (for create)
      {
        displayName: 'Title',
        name: 'title',
        type: 'string',
        required: true,
        default: '',
        displayOptions: {
          show: {
            resource: ['task'],
            operation: ['create'],
          },
        },
        description: 'The title of the task',
        routing: {
          send: {
            type: 'body',
            property: 'title',
          },
        },
      },
      
      // Additional fields (for create)
      {
        displayName: 'Additional Fields',
        name: 'additionalFields',
        type: 'collection',
        placeholder: 'Add Field',
        default: {},
        displayOptions: {
          show: {
            resource: ['task'],
            operation: ['create'],
          },
        },
        options: [
          {
            displayName: 'Description',
            name: 'description',
            type: 'string',
            default: '',
            description: 'Task description',
            routing: {
              send: {
                type: 'body',
                property: 'description',
              },
            },
          },
          {
            displayName: 'Due Date',
            name: 'dueDate',
            type: 'dateTime',
            default: '',
            description: 'When the task is due',
            routing: {
              send: {
                type: 'body',
                property: 'due_date',
              },
            },
          },
          {
            displayName: 'Priority',
            name: 'priority',
            type: 'options',
            options: [
              { name: 'Low', value: 'low' },
              { name: 'Medium', value: 'medium' },
              { name: 'High', value: 'high' },
            ],
            default: 'medium',
            routing: {
              send: {
                type: 'body',
                property: 'priority',
              },
            },
          },
        ],
      },
      
      // Limit (for getAll)
      {
        displayName: 'Limit',
        name: 'limit',
        type: 'number',
        default: 50,
        displayOptions: {
          show: {
            resource: ['task'],
            operation: ['getAll'],
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
    ],
  };
}
```

---

## Step 4: Create Node Metadata

`nodes/TaskManager/TaskManager.node.json`:

```json
{
  "node": "n8n-nodes-taskmanager",
  "nodeVersion": "1.0",
  "codexVersion": "1.0",
  "categories": ["Productivity"],
  "resources": {
    "credentialDocumentation": [
      {
        "url": "https://github.com/your-org/n8n-nodes-taskmanager#credentials"
      }
    ],
    "primaryDocumentation": [
      {
        "url": "https://github.com/your-org/n8n-nodes-taskmanager"
      }
    ]
  }
}
```

---

## Step 5: Add Icon

Create an SVG icon at `nodes/TaskManager/taskmanager.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
  <path d="M9 11l3 3L22 4"/>
  <path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/>
</svg>
```

---

## Step 6: Register in package.json

Update your `package.json`:

```json
{
  "n8n": {
    "n8nNodesApiVersion": 1,
    "credentials": [
      "dist/credentials/TaskManagerApi.credentials.js"
    ],
    "nodes": [
      "dist/nodes/TaskManager/TaskManager.node.js"
    ]
  }
}
```

---

## Step 7: Build and Test

```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

Open n8n at `http://localhost:5678` and search for "Task Manager".

---

## Step 8: Test Your Node

1. **Add credentials**: Go to Credentials > New > Task Manager API
2. **Create workflow**: Add your Task Manager node
3. **Test operations**: Try each operation:
   - Get Many: Should list tasks
   - Create: Should create a new task
   - Get: Should retrieve a specific task
   - Delete: Should remove a task

---

## Next Steps

Now that you have a basic node working:

1. **Add more resources** - See [11 - Resources and Operations](./11-resources-and-operations.md)
2. **Add pagination** - See [16 - Pagination Handling](./16-pagination-handling.md)
3. **Add error handling** - See [17 - Error Handling Patterns](./17-error-handling-patterns.md)
4. **Add dynamic dropdowns** - See [14 - List Search Methods](./14-list-search-methods.md)

---

## Common Issues

### Node doesn't appear

1. Check `package.json` has correct paths
2. Ensure file names match class names
3. Restart dev server

### Credential test fails

1. Verify API URL is correct
2. Check authorization header format
3. Ensure test endpoint exists

### Requests fail

1. Check `requestDefaults.baseURL`
2. Verify routing URLs are correct
3. Check browser network tab for errors
