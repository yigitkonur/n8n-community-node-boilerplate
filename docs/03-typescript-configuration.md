# 03 - TypeScript Configuration

> Compiler settings and build configuration for n8n nodes

---

## Recommended tsconfig.json

```json
{
  "compilerOptions": {
    "strict": true,
    "module": "commonjs",
    "moduleResolution": "node",
    "target": "es2019",
    "lib": ["es2019", "es2020", "es2022.error"],
    "removeComments": true,
    "useUnknownInCatchVariables": false,
    "forceConsistentCasingInFileNames": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "strictNullChecks": true,
    "preserveConstEnums": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "incremental": true,
    "declaration": true,
    "sourceMap": true,
    "skipLibCheck": true,
    "outDir": "./dist/"
  },
  "include": [
    "credentials/**/*",
    "nodes/**/*",
    "nodes/**/*.json",
    "package.json"
  ]
}
```

---

## Configuration Breakdown

### Strict Mode Settings

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "strictNullChecks": true
  }
}
```

| Option | Purpose |
|--------|---------|
| `strict` | Enable all strict type-checking options |
| `noImplicitAny` | Error on implicit `any` types |
| `noImplicitReturns` | Error when not all code paths return |
| `noUnusedLocals` | Error on unused local variables |
| `strictNullChecks` | Enable strict null/undefined checks |

### Module Settings

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node",
    "esModuleInterop": true
  }
}
```

| Option | Purpose |
|--------|---------|
| `module` | Output CommonJS modules (required for n8n) |
| `moduleResolution` | Use Node.js module resolution |
| `esModuleInterop` | Enable ES module interop with CommonJS |

### Output Settings

```json
{
  "compilerOptions": {
    "target": "es2019",
    "outDir": "./dist/",
    "declaration": true,
    "sourceMap": true,
    "removeComments": true
  }
}
```

| Option | Purpose |
|--------|---------|
| `target` | ECMAScript target version |
| `outDir` | Output directory for compiled files |
| `declaration` | Generate `.d.ts` type declaration files |
| `sourceMap` | Generate source maps for debugging |
| `removeComments` | Strip comments from output |

### Library and Interop

```json
{
  "compilerOptions": {
    "lib": ["es2019", "es2020", "es2022.error"],
    "resolveJsonModule": true,
    "skipLibCheck": true
  }
}
```

| Option | Purpose |
|--------|---------|
| `lib` | Include type definitions for ES features |
| `resolveJsonModule` | Allow importing JSON files |
| `skipLibCheck` | Skip type checking of declaration files (faster builds) |

---

## Include Patterns

```json
{
  "include": [
    "credentials/**/*",
    "nodes/**/*",
    "nodes/**/*.json",
    "package.json"
  ]
}
```

**What's included**:
- All files in `credentials/` directory
- All files in `nodes/` directory
- JSON metadata files (`.node.json`)
- Package.json for type references

---

## Alternative: Minimal Configuration

For simpler projects:

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2019",
    "outDir": "./dist/",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true
  },
  "include": ["credentials/**/*", "nodes/**/*"]
}
```

---

## Build Commands

### Standard Build

```bash
# Full build
npm run build

# Watch mode (rebuild on changes)
npm run build:watch
```

### What Happens During Build

1. **TypeScript Compilation**
   - `.ts` files → `.js` files
   - Type declarations → `.d.ts` files
   - Source maps → `.js.map` files

2. **Asset Copying** (via @n8n/node-cli)
   - SVG icons → copied to dist
   - JSON metadata → copied to dist

### Build Output Structure

```
dist/
├── credentials/
│   ├── MyApi.credentials.js
│   ├── MyApi.credentials.d.ts
│   └── MyApi.credentials.js.map
└── nodes/
    └── MyNode/
        ├── MyNode.node.js
        ├── MyNode.node.d.ts
        ├── MyNode.node.js.map
        ├── MyNode.node.json
        └── my-icon.svg
```

---

## Common TypeScript Patterns in n8n Nodes

### Type Imports

```typescript
import type {
  INodeType,
  INodeTypeDescription,
  IExecuteFunctions,
  INodeExecutionData,
  INodeProperties,
  ILoadOptionsFunctions,
  INodePropertyOptions,
} from 'n8n-workflow';
```

### Class-Based Node

```typescript
export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    // ...
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    // ...
  }
}
```

### Credential Types

```typescript
export class MyApi implements ICredentialType {
  name = 'myApi';
  displayName = 'My API';
  properties: INodeProperties[] = [];
}
```

---

## IDE Configuration

### VS Code Settings

Create `.vscode/settings.json`:

```json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "typescript.preferences.importModuleSpecifier": "relative"
}
```

### Recommended Extensions

- **TypeScript**: Built-in
- **ESLint**: `dbaeumer.vscode-eslint`
- **Prettier**: `esbenp.prettier-vscode`

---

## Troubleshooting

### "Cannot find module 'n8n-workflow'"

Ensure `n8n-workflow` is in `peerDependencies` and run `npm install`:

```json
{
  "peerDependencies": {
    "n8n-workflow": "*"
  }
}
```

### Type Errors with Catch Variables

Use this setting to avoid issues with catch clause variables:

```json
{
  "compilerOptions": {
    "useUnknownInCatchVariables": false
  }
}
```

### Slow Builds

Enable incremental compilation:

```json
{
  "compilerOptions": {
    "incremental": true,
    "skipLibCheck": true
  }
}
```

---

## Next Steps

- [04 - Node Anatomy and Architecture](./04-node-anatomy-and-architecture.md)
- [05 - Declarative vs Programmatic Nodes](./05-declarative-vs-programmatic-nodes.md)
