# 15 - Resource Locators

> Multi-mode input fields for flexible resource selection

---

## Overview

Resource Locators provide multiple ways to select a resource:
- **List mode** - Searchable dropdown
- **URL mode** - Paste a URL, extract the ID
- **ID mode** - Enter the ID directly

---

## Basic Structure

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
        regex: 'github.com/[^/]+/([^/]+)',
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
      placeholder: 'my-repo',
    },
  ],
}
```

---

## Mode Types

### List Mode
```typescript
{
  displayName: 'From List',
  name: 'list',
  type: 'list',
  placeholder: 'Select...',
  typeOptions: {
    searchListMethod: 'getItems',
    searchable: true,
    searchFilterRequired: false,
  },
}
```

### URL Mode with Extraction
```typescript
{
  displayName: 'By URL',
  name: 'url',
  type: 'string',
  placeholder: 'https://example.com/items/123',
  extractValue: {
    type: 'regex',
    regex: '/items/([0-9]+)',
  },
  validation: [{
    type: 'regex',
    properties: {
      regex: 'https://example.com/items/[0-9]+',
      errorMessage: 'Invalid URL format',
    },
  }],
}
```

### ID/Name Mode
```typescript
{
  displayName: 'By ID',
  name: 'id',
  type: 'string',
  placeholder: 'item_123',
  validation: [{
    type: 'regex',
    properties: {
      regex: 'item_[a-z0-9]+',
      errorMessage: 'Invalid ID format',
    },
  }],
  url: '=https://example.com/items/{{$value}}',
}
```

---

## Accessing Values

```typescript
// In execute method - get the resolved value
const repoValue = this.getNodeParameter('repository', i, '', {
  extractValue: true,
}) as string;

// In listSearch - get current value for dependencies  
const owner = this.getCurrentNodeParameter('owner', {
  extractValue: true,
});
```

---

## Complete Example

```typescript
export const repoOwnerSelect: INodeProperties = {
  displayName: 'Repository Owner',
  name: 'owner',
  type: 'resourceLocator',
  default: { mode: 'list', value: '' },
  required: true,
  modes: [
    {
      displayName: 'From List',
      name: 'list',
      type: 'list',
      placeholder: 'Select an owner...',
      typeOptions: {
        searchListMethod: 'getUsers',
        searchable: true,
      },
    },
    {
      displayName: 'Link',
      name: 'url',
      type: 'string',
      placeholder: 'https://github.com/n8n-io',
      extractValue: {
        type: 'regex',
        regex: 'github.com/([^/]+)',
      },
      validation: [{
        type: 'regex',
        properties: {
          regex: 'https://github.com/[^/]+',
          errorMessage: 'Not a valid GitHub URL',
        },
      }],
    },
    {
      displayName: 'By Name',
      name: 'name',
      type: 'string',
      placeholder: 'n8n-io',
      url: '=https://github.com/{{$value}}',
    },
  ],
};
```

---

## Next Steps

- [14 - List Search Methods](./14-list-search-methods.md)
- [06 - Node Properties Reference](./06-node-properties-reference.md)
