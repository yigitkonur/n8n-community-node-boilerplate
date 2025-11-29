# 29 - Common Patterns and Recipes

> Solutions to common implementation challenges

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** for ready-to-use UI patterns. Copy and adapt to your node.

| Pattern | When to Use |
|---------|-------------|
| [Return All + Limit](#return-all--limit-pattern) | Paginated list endpoints |
| [Additional Fields](#additional-fields-pattern) | Optional parameters |
| [Filters](#filters-pattern) | Query filtering |
| [Dynamic Options](#dynamic-options-loading) | API-loaded dropdowns |
| [Dependent Dropdowns](#dependent-dropdowns) | Cascading selections |
| [Simplify Output](#simplify-output-toggle) | Clean response data |

**These patterns are commonly combined**. For example, a "Get Many" operation typically uses:
- Return All + Limit
- Filters
- (Optional) Simplify Output

**Related**:
- [06-node-properties-reference.md](./06-node-properties-reference.md) - Property types
- [28-complete-code-examples.md](./28-complete-code-examples.md) - Full examples

---

## Return All + Limit Pattern

```typescript
{
  displayName: 'Return All',
  name: 'returnAll',
  type: 'boolean',
  default: false,
},
{
  displayName: 'Limit',
  name: 'limit',
  type: 'number',
  default: 50,
  displayOptions: { show: { returnAll: [false] } },
  typeOptions: { minValue: 1, maxValue: 100 },
}
```

---

## Additional Fields Pattern

```typescript
{
  displayName: 'Additional Fields',
  name: 'additionalFields',
  type: 'collection',
  placeholder: 'Add Field',
  default: {},
  options: [
    { displayName: 'Field 1', name: 'field1', type: 'string', default: '' },
    { displayName: 'Field 2', name: 'field2', type: 'number', default: 0 },
  ],
}
```

---

## Filters Pattern

```typescript
{
  displayName: 'Filters',
  name: 'filters',
  type: 'collection',
  placeholder: 'Add Filter',
  default: {},
  options: [
    {
      displayName: 'Status',
      name: 'status',
      type: 'options',
      options: [
        { name: 'Active', value: 'active' },
        { name: 'Inactive', value: 'inactive' },
      ],
      default: '',
      routing: { send: { type: 'query', property: 'status' } },
    },
  ],
}
```

---

## Dynamic Options Loading

```typescript
{
  displayName: 'Project',
  name: 'project',
  type: 'options',
  typeOptions: { loadOptionsMethod: 'getProjects' },
  default: '',
}

// In node methods:
methods = {
  loadOptions: {
    async getProjects(this: ILoadOptionsFunctions) {
      const data = await fetchProjects();
      return data.map(p => ({ name: p.name, value: p.id }));
    },
  },
};
```

---

## Dependent Dropdowns

```typescript
{
  displayName: 'Task',
  name: 'task',
  type: 'options',
  typeOptions: {
    loadOptionsMethod: 'getTasks',
    loadOptionsDependsOn: ['project'],  // Reload when project changes
  },
  default: '',
}
```

---

## Simplify Output Toggle

```typescript
{
  displayName: 'Simplify Output',
  name: 'simplify',
  type: 'boolean',
  default: true,
  description: 'Return simplified output',
}

// In execute:
const simplify = this.getNodeParameter('simplify', i) as boolean;
const output = simplify ? { id: result.id, name: result.name } : result;
```

---

## Next Steps

- [28 - Complete Code Examples](./28-complete-code-examples.md)
- [30 - Troubleshooting Guide](./30-troubleshooting-guide.md)
