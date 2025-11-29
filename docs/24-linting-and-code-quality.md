# 24 - Linting and Code Quality

> ESLint configuration and best practices

---

## Running the Linter

```bash
# Check for issues
npm run lint

# Auto-fix issues
npm run lint:fix
```

---

## ESLint Configuration

`eslint.config.mjs`:

```javascript
import n8nConfig from '@n8n/eslint-config/recommended';

export default [
  ...n8nConfig,
  {
    ignores: ['dist/**', 'node_modules/**'],
  },
];
```

---

## Common Lint Rules

| Rule | Description |
|------|-------------|
| `n8n-nodes-base/node-*` | Node-specific rules |
| `n8n-nodes-base/cred-*` | Credential rules |
| `@typescript-eslint/*` | TypeScript rules |

---

## Common Issues

### Missing Description
```typescript
// ❌ Bad
{ displayName: 'Name', name: 'name', type: 'string', default: '' }

// ✅ Good
{ displayName: 'Name', name: 'name', type: 'string', default: '', description: 'The name of the item' }
```

### Incorrect Naming
```typescript
// ❌ Bad
name: 'My_Node'

// ✅ Good
name: 'myNode'
```

---

## Pre-commit Hook

Add to `package.json`:

```json
{
  "scripts": {
    "precommit": "npm run lint"
  }
}
```

---

## Next Steps

- [22 - Development Workflow](./22-development-workflow.md)
- [25 - Preparing for Publication](./25-preparing-for-publication.md)
