# 02 - Package.json Configuration

> Essential configuration for n8n community node packages

---

## 🤖 AI Agent Context

**USE THIS DOCUMENT** to configure package.json for n8n node discovery. This is **required** for your node to work.

| Critical Fields | Why |
|-----------------|-----|
| `name: "n8n-nodes-*"` | Must start with `n8n-nodes-` |
| `keywords: ["n8n-community-node-package"]` | Required for discovery |
| `n8n.nodes[]` | Points to compiled `.js` files |
| `n8n.credentials[]` | Points to compiled `.js` files |

**After Creating Your Node Files, Update**:
1. Add node path to `n8n.nodes[]`
2. Add credential path to `n8n.credentials[]`
3. Update `name`, `description`, `author`

**Quick Check**: All paths in `n8n` block point to `dist/` folder (compiled JS), not source TS.

**Related Documents**:
- [01-project-structure-overview.md](./01-project-structure-overview.md) - Directory layout
- [03-typescript-configuration.md](./03-typescript-configuration.md) - Build config

---

## Complete Example

```json
{
  "name": "n8n-nodes-your-service",
  "version": "0.1.0",
  "description": "n8n community node for Your Service - brief description",
  "license": "MIT",
  "homepage": "https://github.com/your-org/n8n-nodes-your-service#readme",
  "keywords": [
    "n8n-community-node-package",
    "n8n",
    "your-service",
    "automation"
  ],
  "author": {
    "name": "Your Name",
    "email": "your@email.com"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/your-org/n8n-nodes-your-service.git"
  },
  "bugs": {
    "url": "https://github.com/your-org/n8n-nodes-your-service/issues"
  },
  "engines": {
    "node": ">=18.0.0"
  },
  "scripts": {
    "build": "n8n-node build",
    "build:watch": "tsc --watch",
    "dev": "n8n-node dev",
    "lint": "n8n-node lint",
    "lint:fix": "n8n-node lint --fix",
    "release": "n8n-node release",
    "prepublishOnly": "n8n-node prerelease"
  },
  "files": [
    "dist"
  ],
  "n8n": {
    "n8nNodesApiVersion": 1,
    "strict": true,
    "credentials": [
      "dist/credentials/YourServiceApi.credentials.js",
      "dist/credentials/YourServiceOAuth2Api.credentials.js"
    ],
    "nodes": [
      "dist/nodes/YourService/YourService.node.js"
    ]
  },
  "devDependencies": {
    "@n8n/node-cli": "*",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0",
    "release-it": "^19.0.0",
    "typescript": "^5.0.0"
  },
  "peerDependencies": {
    "n8n-workflow": "*"
  },
  "dependencies": {
    "@your-service/sdk": "^1.0.0"
  }
}
```

---

## Required Fields

### Package Name

```json
{
  "name": "n8n-nodes-your-service"
}
```

**Requirements**:
- **Must start with** `n8n-nodes-`
- Use lowercase letters, numbers, and hyphens
- Be unique on npm

### Keywords

```json
{
  "keywords": [
    "n8n-community-node-package"
  ]
}
```

**Critical**: The keyword `n8n-community-node-package` is **required** for n8n to discover your package.

---

## The `n8n` Configuration Block

This is the most important section—it tells n8n where to find your nodes and credentials.

```json
{
  "n8n": {
    "n8nNodesApiVersion": 1,
    "strict": true,
    "credentials": [
      "dist/credentials/YourServiceApi.credentials.js"
    ],
    "nodes": [
      "dist/nodes/YourService/YourService.node.js"
    ]
  }
}
```

| Field | Purpose |
|-------|---------|
| `n8nNodesApiVersion` | API version (always `1` currently) |
| `strict` | Enable strict validation (recommended: `true`) |
| `credentials` | Array of paths to credential JS files |
| `nodes` | Array of paths to node JS files |

**Important**: Paths point to **compiled JS files** in `dist/`, not TypeScript sources.

---

## Scripts Reference

### Using @n8n/node-cli (Recommended)

```json
{
  "scripts": {
    "build": "n8n-node build",
    "dev": "n8n-node dev",
    "lint": "n8n-node lint",
    "lint:fix": "n8n-node lint --fix",
    "release": "n8n-node release",
    "prepublishOnly": "n8n-node prerelease"
  }
}
```

| Script | Description |
|--------|-------------|
| `build` | Compile TypeScript and copy assets |
| `dev` | Start n8n with hot reload for development |
| `lint` | Check code with n8n linter rules |
| `lint:fix` | Auto-fix linting issues |
| `release` | Create a new version release |
| `prepublishOnly` | Pre-publish checks (runs before npm publish) |

### Manual Build (Alternative)

```json
{
  "scripts": {
    "build": "tsc && gulp build:icons",
    "dev": "tsc --watch",
    "lint": "eslint 'nodes/**/*.ts' 'credentials/**/*.ts'",
    "prepublishOnly": "npm run build && npm run lint"
  }
}
```

---

## Dependencies

### DevDependencies

```json
{
  "devDependencies": {
    "@n8n/node-cli": "*",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0",
    "typescript": "^5.0.0"
  }
}
```

| Package | Purpose |
|---------|---------|
| `@n8n/node-cli` | Build tooling, dev server, linting |
| `eslint` | Code linting |
| `prettier` | Code formatting |
| `typescript` | TypeScript compiler |

### PeerDependencies

```json
{
  "peerDependencies": {
    "n8n-workflow": "*"
  }
}
```

**Always use `*` for `n8n-workflow`** — n8n provides this at runtime.

### Dependencies (Optional)

```json
{
  "dependencies": {
    "@your-service/sdk": "^1.0.0"
  }
}
```

**For n8n Cloud verification**: External dependencies may disqualify your node from verification. Use n8n's built-in HTTP helpers when possible.

---

## Files Field

```json
{
  "files": [
    "dist"
  ]
}
```

Only the `dist/` folder is published to npm. This keeps your package size minimal.

---

## Engine Requirements

```json
{
  "engines": {
    "node": ">=18.0.0"
  }
}
```

Match n8n's minimum Node.js version (currently 18+).

---

## Complete Minimal Example

```json
{
  "name": "n8n-nodes-simple-service",
  "version": "1.0.0",
  "description": "Simple Service node for n8n",
  "license": "MIT",
  "keywords": ["n8n-community-node-package"],
  "author": "Your Name",
  "scripts": {
    "build": "n8n-node build",
    "dev": "n8n-node dev",
    "lint": "n8n-node lint"
  },
  "files": ["dist"],
  "n8n": {
    "n8nNodesApiVersion": 1,
    "credentials": ["dist/credentials/SimpleApi.credentials.js"],
    "nodes": ["dist/nodes/Simple/Simple.node.js"]
  },
  "devDependencies": {
    "@n8n/node-cli": "*",
    "typescript": "^5.0.0"
  },
  "peerDependencies": {
    "n8n-workflow": "*"
  }
}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing `n8n-community-node-package` keyword | Add to keywords array |
| Paths in `n8n.nodes` point to `.ts` files | Change to `.js` in `dist/` |
| Using pinned version for `n8n-workflow` | Use `"*"` instead |
| Missing `files` field | Add `"files": ["dist"]` |

---

## Next Steps

- [03 - TypeScript Configuration](./03-typescript-configuration.md)
- [04 - Node Anatomy and Architecture](./04-node-anatomy-and-architecture.md)
