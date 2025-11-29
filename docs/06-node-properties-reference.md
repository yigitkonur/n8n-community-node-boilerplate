# 06 - Node Properties Reference

> Complete guide to property types and configuration options

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** as a reference when building node UI fields. Contains all property types and patterns.

| Property Type | Use Case | Jump To |
|--------------|----------|---------|
| `string` | Text input, passwords | [string section](#string) |
| `number` | Numeric values | [number section](#number) |
| `boolean` | Toggle switches | [boolean section](#boolean) |
| `options` | Dropdown select | [options section](#options) |
| `multiOptions` | Multi-select | [multiOptions section](#multioptions) |
| `collection` | Optional grouped fields | [collection section](#collection) |
| `fixedCollection` | Repeatable groups | [fixedCollection section](#fixedcollection) |
| `resourceLocator` | Multi-mode selection | [resourceLocator section](#resourcelocator) |

**Common Patterns** (at bottom of doc):
- Resource + Operation pattern
- Return All + Limit pattern
- Additional Fields pattern

**Related Documents**:
- [12-declarative-routing.md](./12-declarative-routing.md) - Add routing to properties
- [14-list-search-methods.md](./14-list-search-methods.md) - Dynamic dropdowns
- [15-resource-locators.md](./15-resource-locators.md) - Deep dive on resourceLocator

---

## Property Structure

Every property follows this base structure:

```typescript
{
  displayName: string;        // Label shown in UI
  name: string;               // Internal identifier
  type: PropertyType;         // Input type
  default: any;               // Default value
  required?: boolean;         // Is required?
  description?: string;       // Help text
  placeholder?: string;       // Placeholder text
  displayOptions?: object;    // Show/hide conditions
  typeOptions?: object;       // Type-specific options
  routing?: object;           // Declarative routing config
}
```

---

## Property Types

### `string`

Basic text input.

```typescript
{
  displayName: 'Name',
  name: 'name',
  type: 'string',
  default: '',
  placeholder: 'Enter name...',
  description: 'The name to use',
}
```

**With password masking**:
```typescript
{
  displayName: 'API Key',
  name: 'apiKey',
  type: 'string',
  typeOptions: { password: true },
  default: '',
}
```

**Multi-line**:
```typescript
{
  displayName: 'Description',
  name: 'description',
  type: 'string',
  typeOptions: { rows: 5 },
  default: '',
}
```

---

### `number`

Numeric input.

```typescript
{
  displayName: 'Limit',
  name: 'limit',
  type: 'number',
  default: 50,
  typeOptions: {
    minValue: 1,
    maxValue: 100,
  },
  description: 'Max results to return',
}
```

---

### `boolean`

Toggle switch.

```typescript
{
  displayName: 'Return All',
  name: 'returnAll',
  type: 'boolean',
  default: false,
  description: 'Whether to return all results',
}
```

---

### `options`

Dropdown selection.

```typescript
{
  displayName: 'Operation',
  name: 'operation',
  type: 'options',
  noDataExpression: true,  // Disable expression input
  options: [
    {
      name: 'Get',
      value: 'get',
      description: 'Get a single item',
      action: 'Get an item',  // For action menu
    },
    {
      name: 'Get Many',
      value: 'getAll',
      description: 'Get many items',
      action: 'Get many items',
    },
    {
      name: 'Create',
      value: 'create',
      description: 'Create a new item',
      action: 'Create an item',
    },
  ],
  default: 'get',
}
```

**With dynamic loading**:
```typescript
{
  displayName: 'Project',
  name: 'project',
  type: 'options',
  typeOptions: {
    loadOptionsMethod: 'getProjects',  // Method in node.methods.loadOptions
  },
  default: '',
}
```

---

### `multiOptions`

Multiple selection.

```typescript
{
  displayName: 'Labels',
  name: 'labels',
  type: 'multiOptions',
  options: [
    { name: 'Bug', value: 'bug' },
    { name: 'Feature', value: 'feature' },
    { name: 'Enhancement', value: 'enhancement' },
  ],
  default: [],
}
```

---

### `collection`

Grouped optional fields.

```typescript
{
  displayName: 'Additional Fields',
  name: 'additionalFields',
  type: 'collection',
  placeholder: 'Add Field',
  default: {},
  options: [
    {
      displayName: 'Description',
      name: 'description',
      type: 'string',
      default: '',
    },
    {
      displayName: 'Due Date',
      name: 'dueDate',
      type: 'dateTime',
      default: '',
    },
  ],
}
```

---

### `fixedCollection`

Structured repeatable groups.

```typescript
{
  displayName: 'Parameters',
  name: 'parameters',
  type: 'fixedCollection',
  typeOptions: {
    multipleValues: true,  // Allow multiple entries
  },
  placeholder: 'Add Parameter',
  default: {},
  options: [
    {
      name: 'parameter',
      displayName: 'Parameter',
      values: [
        {
          displayName: 'Name',
          name: 'name',
          type: 'string',
          default: '',
        },
        {
          displayName: 'Value',
          name: 'value',
          type: 'string',
          default: '',
        },
      ],
    },
  ],
}
```

---

### `resourceLocator`

Multi-mode resource selection.

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
      placeholder: 'Select...',
      typeOptions: {
        searchListMethod: 'getRepositories',
        searchable: true,
      },
    },
    {
      displayName: 'By URL',
      name: 'url',
      type: 'string',
      placeholder: 'https://github.com/owner/repo',
      extractValue: {
        type: 'regex',
        regex: 'github.com/([^/]+/[^/]+)',
      },
      validation: [
        {
          type: 'regex',
          properties: {
            regex: 'https://github.com/.+/.+',
            errorMessage: 'Not a valid GitHub URL',
          },
        },
      ],
    },
    {
      displayName: 'By Name',
      name: 'name',
      type: 'string',
      placeholder: 'owner/repo',
    },
  ],
}
```

---

### `dateTime`

Date and time picker.

```typescript
{
  displayName: 'Start Date',
  name: 'startDate',
  type: 'dateTime',
  default: '',
  description: 'The start date for the query',
}
```

---

### `json`

JSON editor.

```typescript
{
  displayName: 'Request Body',
  name: 'body',
  type: 'json',
  default: '{}',
  description: 'JSON body for the request',
}
```

---

### `hidden`

Hidden field (for internal values).

```typescript
{
  displayName: 'API Version',
  name: 'apiVersion',
  type: 'hidden',
  default: 'v2',
}
```

---

## Display Options

Control when properties are shown/hidden.

### Show Based on Value

```typescript
{
  displayName: 'Issue Number',
  name: 'issueNumber',
  type: 'number',
  default: 0,
  displayOptions: {
    show: {
      resource: ['issue'],
      operation: ['get', 'update', 'delete'],
    },
  },
}
```

### Hide Based on Value

```typescript
{
  displayName: 'Limit',
  name: 'limit',
  type: 'number',
  default: 50,
  displayOptions: {
    hide: {
      returnAll: [true],
    },
  },
}
```

### Complex Conditions

```typescript
displayOptions: {
  show: {
    resource: ['issue'],
    operation: ['create'],
    '/authentication': ['oauth2'],  // Check root-level property
  },
}
```

---

## Routing Configuration

For declarative nodes, define how property values are sent.

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
      property: 'state',
    },
  },
}
```

### Body Parameters

```typescript
{
  displayName: 'Title',
  name: 'title',
  type: 'string',
  default: '',
  routing: {
    send: {
      type: 'body',
      property: 'title',
    },
  },
}
```

### With Value Transformation

```typescript
{
  displayName: 'Labels',
  name: 'labels',
  type: 'collection',
  default: {},
  routing: {
    send: {
      type: 'body',
      property: 'labels',
      value: '={{$value.map((item) => item.label)}}',
    },
  },
}
```

### Output Limiting

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
      maxResults: '={{$value}}',
    },
  },
}
```

---

## Type Options Reference

### String Options

```typescript
typeOptions: {
  password: true,      // Mask input
  rows: 5,             // Multi-line rows
  editor: 'code',      // Code editor
  editorLanguage: 'json',
}
```

### Number Options

```typescript
typeOptions: {
  minValue: 1,
  maxValue: 100,
  numberStepSize: 1,
  numberPrecision: 2,
}
```

### Options (Dynamic Loading)

```typescript
typeOptions: {
  loadOptionsMethod: 'getUsers',
  loadOptionsDependsOn: ['projectId'],  // Reload when this changes
}
```

### Resource Locator (List Mode)

```typescript
typeOptions: {
  searchListMethod: 'searchUsers',
  searchable: true,
  searchFilterRequired: false,
}
```

---

## Common Patterns

### Resource + Operation Pattern

```typescript
// Resource selector
{
  displayName: 'Resource',
  name: 'resource',
  type: 'options',
  noDataExpression: true,
  options: [
    { name: 'User', value: 'user' },
    { name: 'Project', value: 'project' },
  ],
  default: 'user',
},

// Operations shown per resource
{
  displayName: 'Operation',
  name: 'operation',
  type: 'options',
  noDataExpression: true,
  displayOptions: {
    show: { resource: ['user'] },
  },
  options: [
    { name: 'Create', value: 'create', action: 'Create a user' },
    { name: 'Get', value: 'get', action: 'Get a user' },
  ],
  default: 'get',
}
```

### Return All + Limit Pattern

```typescript
{
  displayName: 'Return All',
  name: 'returnAll',
  type: 'boolean',
  default: false,
  description: 'Whether to return all results',
},
{
  displayName: 'Limit',
  name: 'limit',
  type: 'number',
  default: 50,
  displayOptions: {
    show: { returnAll: [false] },
  },
  typeOptions: { minValue: 1, maxValue: 100 },
}
```

---

## Next Steps

- [11 - Resources and Operations](./11-resources-and-operations.md)
- [12 - Declarative Routing](./12-declarative-routing.md)
- [15 - Resource Locators](./15-resource-locators.md)
