# 14 - List Search Methods

> Dynamic dropdown population with search and pagination

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** to create searchable dropdowns that load options from an API.

| Feature | Supported |
|---------|-----------|
| Search/filter | ✅ Yes |
| Pagination | ✅ Yes |
| Dependent parameters | ✅ Yes |
| Show URLs/descriptions | ✅ Yes |

**When to Use**:
- User needs to select from dynamic data (users, projects, issues)
- List is too large to hardcode
- Values depend on other selections

**Prerequisites**:
- [10-creating-your-first-node.md](./10-creating-your-first-node.md) - Basic node structure
- [15-resource-locators.md](./15-resource-locators.md) - Connects to resource locator

**Reference Directory**: `nodes/GithubIssues/listSearch/`

---

## Overview

List Search methods power dynamic dropdowns that:
- Load options from APIs
- Support search/filtering
- Handle pagination
- Show additional context (URLs, descriptions)

---

## Basic Structure

```typescript
// In node file
export class MyNode implements INodeType {
  description: INodeTypeDescription = { /* ... */ };

  methods = {
    listSearch: {
      async searchItems(
        this: ILoadOptionsFunctions,
        filter?: string,
        paginationToken?: string,
      ): Promise<INodeListSearchResult> {
        // Fetch and return items
      },
    },
  };
}
```

---

## Return Type: INodeListSearchResult

```typescript
interface INodeListSearchResult {
  results: INodeListSearchItems[];
  paginationToken?: string | number;  // For next page
}

interface INodeListSearchItems {
  name: string;       // Display text
  value: string;      // Stored value
  url?: string;       // Optional: clickable link
  description?: string; // Optional: secondary text
}
```

---

## Complete Example

```typescript
import type {
  ILoadOptionsFunctions,
  INodeListSearchItems,
  INodeListSearchResult,
} from 'n8n-workflow';
import { apiRequest } from '../shared/transport';

type Repository = {
  name: string;
  html_url: string;
  description: string | null;
};

type SearchResponse = {
  items: Repository[];
  total_count: number;
};

export async function getRepositories(
  this: ILoadOptionsFunctions,
  filter?: string,
  paginationToken?: string,
): Promise<INodeListSearchResult> {
  // Get dependent parameter value
  const owner = this.getCurrentNodeParameter('owner', { extractValue: true });
  
  // Handle pagination
  const page = paginationToken ? +paginationToken : 1;
  const perPage = 100;
  
  // Build search query
  const query = `${filter ?? ''} user:${owner} fork:true`;
  
  let response: SearchResponse = { items: [], total_count: 0 };
  
  try {
    response = await apiRequest.call(this, 'GET', '/search/repositories', {
      q: query,
      page,
      per_page: perPage,
    });
  } catch {
    // Return empty if owner has no repos
  }
  
  // Map to list items
  const results: INodeListSearchItems[] = response.items.map((repo) => ({
    name: repo.name,
    value: repo.name,
    url: repo.html_url,
    description: repo.description || undefined,
  }));
  
  // Calculate next pagination token
  const hasMore = page * perPage < response.total_count;
  const nextToken = hasMore ? String(page + 1) : undefined;
  
  return {
    results,
    paginationToken: nextToken,
  };
}
```

---

## Connecting to Resource Locator

```typescript
{
  displayName: 'Repository',
  name: 'repository',
  type: 'resourceLocator',
  default: { mode: 'list', value: '' },
  required: true,
  modes: [
    {
      displayName: 'From List',
      name: 'list',
      type: 'list',
      placeholder: 'Select a repository...',
      typeOptions: {
        searchListMethod: 'getRepositories',  // Method name
        searchable: true,                      // Enable search
        searchFilterRequired: false,           // Don't require search input
      },
    },
    // ... other modes
  ],
}
```

---

## Dependent Parameters

Access other parameter values using `getCurrentNodeParameter`:

```typescript
export async function getIssues(
  this: ILoadOptionsFunctions,
  filter?: string,
): Promise<INodeListSearchResult> {
  // Get parent parameters
  const owner = this.getCurrentNodeParameter('owner', { extractValue: true });
  const repo = this.getCurrentNodeParameter('repository', { extractValue: true });
  
  // Use in API call
  const issues = await apiRequest.call(
    this,
    'GET',
    `/repos/${owner}/${repo}/issues`,
    { q: filter }
  );
  
  return {
    results: issues.map(issue => ({
      name: `#${issue.number}: ${issue.title}`,
      value: String(issue.number),
      url: issue.html_url,
    })),
  };
}
```

---

## Load Options vs List Search

### loadOptions (Simple Dropdowns)

```typescript
// For type: 'options' with typeOptions.loadOptionsMethod
methods = {
  loadOptions: {
    async getProjects(this: ILoadOptionsFunctions): Promise<INodePropertyOptions[]> {
      const projects = await fetchProjects();
      return projects.map(p => ({
        name: p.name,
        value: p.id,
      }));
    },
  },
};
```

### listSearch (Resource Locators)

```typescript
// For type: 'resourceLocator' with searchListMethod
methods = {
  listSearch: {
    async searchProjects(
      this: ILoadOptionsFunctions,
      filter?: string,
      paginationToken?: string,
    ): Promise<INodeListSearchResult> {
      // Supports search and pagination
    },
  },
};
```

---

## Multiple List Search Methods

```typescript
export class MyNode implements INodeType {
  description: INodeTypeDescription = { /* ... */ };

  methods = {
    listSearch: {
      getUsers,         // Imported from ./listSearch/getUsers
      getProjects,      // Imported from ./listSearch/getProjects  
      getIssues,        // Imported from ./listSearch/getIssues
    },
  };
}
```

---

## File Organization

```
nodes/MyNode/
├── MyNode.node.ts
└── listSearch/
    ├── getUsers.ts
    ├── getProjects.ts
    └── getIssues.ts
```

`listSearch/getUsers.ts`:

```typescript
import type {
  ILoadOptionsFunctions,
  INodeListSearchResult,
} from 'n8n-workflow';
import { apiRequest } from '../shared/transport';

export async function getUsers(
  this: ILoadOptionsFunctions,
  filter?: string,
  paginationToken?: string,
): Promise<INodeListSearchResult> {
  // Implementation
}
```

---

## Using Credentials

```typescript
export async function getProjects(
  this: ILoadOptionsFunctions,
  filter?: string,
): Promise<INodeListSearchResult> {
  const credentials = await this.getCredentials('myServiceApi') as {
    apiKey: string;
    projectId: number;
  };
  
  // Use credentials for API calls
  const client = new MyClient(credentials.apiKey);
  const projects = await client.getProjects({ search: filter });
  
  return {
    results: projects.map(p => ({
      name: p.name,
      value: p.id,
    })),
  };
}
```

---

## Error Handling

```typescript
export async function getItems(
  this: ILoadOptionsFunctions,
  filter?: string,
): Promise<INodeListSearchResult> {
  try {
    const items = await fetchItems(filter);
    return {
      results: items.map(item => ({
        name: item.name,
        value: item.id,
      })),
    };
  } catch (error) {
    // Return empty results on error (don't throw)
    console.error('Failed to load items:', error);
    return { results: [] };
    
    // Or throw with message:
    // throw new Error(`Failed to load items: ${error.message}`);
  }
}
```

---

## Best Practices

1. **Handle empty states** - Return `{ results: [] }` gracefully
2. **Add descriptions** - Help users identify the right item
3. **Include URLs** - Allow users to verify selections
4. **Limit page size** - Don't fetch too many at once (50-100)
5. **Cache when appropriate** - Reduce API calls

---

## Next Steps

- [15 - Resource Locators](./15-resource-locators.md)
- [18 - Helper Functions and Utilities](./18-helper-functions-and-utilities.md)
