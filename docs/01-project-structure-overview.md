# 01 - Project Structure Overview

> Understanding the complete directory layout for n8n community node packages

---

## 🤖 AI Agent Context

**READ THIS DOCUMENT** to understand where to create files. Essential for correct file placement.

| Creating This | Put It Here |
|--------------|-------------|
| Node class | `nodes/{NodeName}/{NodeName}.node.ts` |
| Credential | `credentials/{Name}Api.credentials.ts` |
| Icon | `nodes/{NodeName}/{nodename}.svg` or `icons/` |
| List search | `nodes/{NodeName}/listSearch/` |
| Resources | `nodes/{NodeName}/resources/{resource}/` |
| Shared utils | `nodes/{NodeName}/shared/` |

**File Naming Rules**:
- Node/Credential classes: **PascalCase** (e.g., `GithubIssues.node.ts`)
- Icons: **lowercase** (e.g., `github.svg`)
- Internal names: **camelCase** (e.g., `name: 'githubIssues'`)

**Related**:
- [02-package-json-configuration.md](./02-package-json-configuration.md) - Register files
- [10-creating-your-first-node.md](./10-creating-your-first-node.md) - Create node files

---

## Standard Directory Structure

```
n8n-nodes-your-package/
├── credentials/                    # Authentication definitions
│   ├── YourApi.credentials.ts      # API key/token credentials
│   └── YourOAuth2Api.credentials.ts # OAuth2 credentials
├── icons/                          # Shared icons (optional)
│   ├── your-icon.svg               # Light mode icon
│   └── your-icon.dark.svg          # Dark mode icon
├── nodes/                          # Node implementations
│   └── YourNode/
│       ├── YourNode.node.ts        # Main node class
│       ├── YourNode.node.json      # Codex metadata
│       ├── your-icon.svg           # Node-specific icon
│       ├── GenericFunctions.ts     # Helper functions (optional)
│       ├── listSearch/             # Dynamic dropdown methods
│       │   └── getItems.ts
│       ├── resources/              # Resource-based organization
│       │   └── resource/
│       │       ├── index.ts
│       │       ├── create.ts
│       │       ├── get.ts
│       │       └── getAll.ts
│       └── shared/                 # Shared utilities
│           ├── descriptions.ts
│           ├── transport.ts
│           └── utils.ts
├── dist/                           # Compiled output (generated)
├── node_modules/                   # Dependencies (generated)
├── package.json                    # Package configuration
├── package-lock.json               # Dependency lock file
├── tsconfig.json                   # TypeScript configuration
├── eslint.config.mjs               # ESLint configuration
├── README.md                       # Package documentation
├── LICENSE.md                      # License (MIT recommended)
└── CHANGELOG.md                    # Version history
```

---

## Directory Purposes

### `/credentials/`

Contains credential type definitions that tell n8n how to:
- Collect authentication information from users
- Apply credentials to API requests
- Test credential validity

**Naming Convention**: `{ServiceName}Api.credentials.ts`

```typescript
// Example: credentials/MyServiceApi.credentials.ts
export class MyServiceApi implements ICredentialType {
  name = 'myServiceApi';
  displayName = 'My Service API';
  // ...
}
```

### `/icons/`

Shared icon files for credentials and nodes. Icons should be:
- **SVG format** - Scalable vector graphics
- **24x24px** - Base size
- **Two variants** - Light (`.svg`) and dark (`.dark.svg`) modes

```
icons/
├── github.svg           # Used on light backgrounds
└── github.dark.svg      # Used on dark backgrounds
```

### `/nodes/`

Each node lives in its own subdirectory with all related files:

```
nodes/YourNode/
├── YourNode.node.ts        # Main entry point (REQUIRED)
├── YourNode.node.json      # Metadata for n8n catalog
├── your-icon.svg           # Node icon
└── ...                     # Supporting files
```

### `/nodes/YourNode/resources/`

For nodes with multiple resources, organize by resource type:

```
resources/
├── user/
│   ├── index.ts            # Exports all user operations
│   ├── create.ts           # Create user operation
│   ├── get.ts              # Get single user
│   └── getAll.ts           # List users
└── project/
    ├── index.ts
    └── ...
```

### `/nodes/YourNode/shared/`

Reusable code across operations:

| File | Purpose |
|------|---------|
| `descriptions.ts` | Reusable property definitions (resource locators, common fields) |
| `transport.ts` | HTTP request helper functions |
| `utils.ts` | General utility functions (parsing, formatting) |

### `/nodes/YourNode/listSearch/`

Methods for dynamic dropdown population:

```typescript
// listSearch/getUsers.ts
export async function getUsers(
  this: ILoadOptionsFunctions,
  filter?: string,
): Promise<INodeListSearchResult> {
  // Fetch and return users for dropdown
}
```

---

## File Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Node Class | `{Name}.node.ts` | `GithubIssues.node.ts` |
| Node Metadata | `{Name}.node.json` | `GithubIssues.node.json` |
| Credentials | `{Name}Api.credentials.ts` | `GithubIssuesApi.credentials.ts` |
| OAuth2 Credentials | `{Name}OAuth2Api.credentials.ts` | `GithubIssuesOAuth2Api.credentials.ts` |
| Helper Functions | `GenericFunctions.ts` | `GenericFunctions.ts` |
| Transport | `transport.ts` | `transport.ts` |

---

## Minimal vs Full Structure

### Minimal (Simple Node)

```
n8n-nodes-simple/
├── credentials/
│   └── SimpleApi.credentials.ts
├── nodes/
│   └── Simple/
│       ├── Simple.node.ts
│       └── simple.svg
├── package.json
└── tsconfig.json
```

### Full (Complex Node)

```
n8n-nodes-complex/
├── credentials/
│   ├── ComplexApi.credentials.ts
│   └── ComplexOAuth2Api.credentials.ts
├── icons/
│   ├── complex.svg
│   └── complex.dark.svg
├── nodes/
│   └── Complex/
│       ├── Complex.node.ts
│       ├── Complex.node.json
│       ├── GenericFunctions.ts
│       ├── listSearch/
│       │   ├── getProjects.ts
│       │   └── getUsers.ts
│       ├── resources/
│       │   ├── project/
│       │   │   ├── index.ts
│       │   │   ├── create.ts
│       │   │   └── getAll.ts
│       │   └── task/
│       │       ├── index.ts
│       │       ├── create.ts
│       │       ├── get.ts
│       │       └── update.ts
│       └── shared/
│           ├── descriptions.ts
│           ├── transport.ts
│           └── utils.ts
├── package.json
├── tsconfig.json
├── eslint.config.mjs
├── README.md
├── LICENSE.md
└── CHANGELOG.md
```

---

## Build Output (`/dist/`)

After running `npm run build`, the `dist/` folder mirrors your source structure:

```
dist/
├── credentials/
│   └── MyApi.credentials.js
│   └── MyApi.credentials.d.ts
└── nodes/
    └── MyNode/
        ├── MyNode.node.js
        └── my-icon.svg
```

**Important**: Icons are copied as-is; TypeScript is compiled to JavaScript.

---

## Next Steps

- [02 - Package.json Configuration](./02-package-json-configuration.md)
- [03 - TypeScript Configuration](./03-typescript-configuration.md)
