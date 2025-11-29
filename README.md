# n8n Custom Node Starter - Essential Context

## Purpose
Official starter repository for building custom n8n community nodes with example implementations, documentation, and development tooling.

## Project Structure

### Core Files
- **nodes/** - Node implementations
  - `Example/` - Basic programmatic node template
  - `GithubIssues/` - Advanced declarative REST API node with resources/operations
- **credentials/** - Authentication configurations
- **docs/** - 30+ comprehensive guides (00-30)

### Key Examples
```
nodes/GithubIssues/
├── GithubIssues.node.ts          # Main node file
├── resources/                     # Resource-based organization
│   ├── issue/                     # CRUD operations per resource
│   └── issueComment/
├── listSearch/                    # Dynamic dropdowns
└── shared/                        # Reusable utilities
    ├── transport.ts               # API request wrapper
    └── descriptions.ts            # UI configurations
```

## Setup & Initialization

### 1. Create Repository with GitHub CLI

```bash
# Create from this template
gh repo create my-n8n-nodes --template n8n-io/n8n-nodes-starter --public --clone

# Or create manually and initialize
gh repo create my-n8n-nodes --public
git clone https://github.com/yourusername/my-n8n-nodes
cd my-n8n-nodes
```

**GitHub CLI Quick Reference:**
- `gh repo create` - Create new repository
- `gh repo clone` - Clone repository
- `gh auth login` - Authenticate GitHub
- `gh --help` - Full command reference

### 2. Configure .gitignore

**Essential exclusions:**
```gitignore
# Build output
dist/
*.tsbuildinfo

# Dependencies
node_modules/

# Environment & IDE
.env
.DS_Store
.vscode/
.idea/

# Keep out of Git for publication
docs/
README_TEMPLATE.md
```

### 3. Git Initialization & Cleanup

```bash
# Initialize repository
git init

# Clean cache if needed (remove tracked files now in .gitignore)
git rm -r --cached dist/
git rm -r --cached node_modules/

# Initial commit
git add .
git commit -m "Initial commit: n8n custom node starter"

# Connect to remote
git remote add origin https://github.com/yourusername/my-n8n-nodes.git
git branch -M main
git push -u origin main
```

**Git Workflow Commands:**
- `git status` - Check working tree status
- `git add <files>` - Stage changes
- `git commit -m "message"` - Record changes
- `git push` - Update remote repository
- `git pull` - Fetch and merge remote changes
- `git log` - Show commit history

## Development Workflow

### Quick Start Commands

```bash
# Install dependencies (first time)
npm install

# Development (watch + hot reload)
npm run dev
# Opens n8n at http://localhost:5678 with your node loaded

# Build for production
npm run build

# Lint code
npm run lint

# Auto-fix lint issues
npm run lint:fix

# Create release
npm run release
```

### Development Cycle
1. Run `npm run dev` - Starts n8n with watch mode
2. Make changes to node files
3. Changes auto-rebuild and reload in n8n
4. Test in n8n workflow editor
5. Check console for errors

## Configuration Checklist

### 1. Update package.json

**Required fields:**
```json
{
  "name": "n8n-nodes-<your-service>",
  "version": "1.0.0",
  "description": "n8n node for <service>",
  "author": "Your Name <email@example.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourusername/my-n8n-nodes.git"
  },
  "keywords": ["n8n-community-node-package"],
  "n8n": {
    "nodes": [
      "dist/nodes/YourNode/YourNode.node.js"
    ],
    "credentials": [
      "dist/credentials/YourNodeApi.credentials.js"
    ]
  }
}
```

**Key points:**
- Name **must** start with `n8n-nodes-`
- Include `n8n-community-node-package` keyword
- Register all nodes/credentials in `n8n` section

### 2. Essential File Structure

```
my-n8n-nodes/
├── nodes/
│   └── YourNode/
│       ├── YourNode.node.ts      # Main implementation
│       ├── YourNode.node.json    # Metadata
│       └── yournode.svg          # Icon (optional)
├── credentials/
│   └── YourNodeApi.credentials.ts
├── package.json
├── tsconfig.json
├── .gitignore
├── LICENSE.md
└── README.md
```

## Documentation Navigation

### AI Agent Quick Router
| Task | Start Here | Then Read |
|------|-----------|-----------|
| Build new node | `docs/10-creating-your-first-node.md` | → `docs/05-declarative-vs-programmatic-nodes.md` |
| API key auth | `docs/08-api-key-credentials.md` | → `docs/07-credentials-system-overview.md` |
| OAuth2 auth | `docs/09-oauth2-credentials.md` | → `docs/07-credentials-system-overview.md` |
| REST API node | `docs/12-declarative-routing.md` | → `docs/06-node-properties-reference.md` |
| SDK integration | `docs/13-custom-execute-methods.md` | → `docs/19-external-sdk-integration.md` |
| Dynamic dropdowns | `docs/14-list-search-methods.md` | → `docs/15-resource-locators.md` |
| Multi-resource API | `docs/11-resources-and-operations.md` | → `docs/04-node-anatomy-and-architecture.md` |
| Pagination | `docs/16-pagination-handling.md` | → `docs/12-declarative-routing.md` |
| Debugging | `docs/30-troubleshooting-guide.md` | → `docs/17-error-handling-patterns.md` |
| Publishing | `docs/25-preparing-for-publication.md` | → `docs/26-publishing-to-npm.md` |

### Complete Documentation Index (30 files)
- **00-04**: Project setup, structure, TypeScript config, architecture
- **05-09**: Node types (declarative/programmatic), properties, credentials
- **10-15**: Creating nodes, resources, routing, dynamic UI
- **16-21**: Pagination, errors, helpers, SDKs, icons, metadata
- **22-27**: Development, testing, linting, publishing, verification
- **28-30**: Examples, patterns, troubleshooting

**Full guide:** `docs/00-documentation-index.md`

## Prerequisites

**Required:**
- **Node.js v22+** with npm
  - Install via [nvm](https://github.com/nvm-sh/nvm) (Linux/Mac/WSL)
  - Install via [official installer](https://nodejs.org/) (Windows)
- **git**
- **GitHub CLI** (`gh`) - optional but recommended

**Recommended:**
- Follow n8n's [development environment setup guide](https://docs.n8n.io/integrations/creating-nodes/build/node-development-environment/)

**Note:** `@n8n/node-cli` is included as dev dependency - no global n8n install needed

## Publishing Workflow

### Pre-Publication Checklist

1. **Clean documentation:**
   ```bash
   # Remove template files
   rm README_TEMPLATE.md
   
   # Exclude docs from Git (add to .gitignore)
   echo "docs/" >> .gitignore
   git rm -r --cached docs/
   ```

2. **Update README.md:**
   - Use `README_TEMPLATE.md` as starting point
   - Document installation, usage, credentials setup
   - Include example workflows

3. **Update LICENSE.md:**
   - Add your name and year
   - Keep MIT license for n8n Cloud verification

4. **Test thoroughly:**
   - Run `npm run dev` and test all operations
   - Verify error handling
   - Test with real API credentials

5. **Lint and build:**
   ```bash
   npm run lint:fix
   npm run build
   ```

### Publishing to npm

```bash
# Ensure npm is configured
npm login

# Publish package
npm publish

# For scoped packages
npm publish --access public
```

**npm Configuration:**
```bash
npm whoami              # Check logged-in user
npm config get registry # Verify registry (https://registry.npmjs.org/)
```

### Submit for n8n Cloud Verification (Optional)

**Requirements:**
- ✅ MIT license (included in starter)
- ✅ No external package dependencies
- ✅ Follows n8n design guidelines
- ✅ Passes quality and security review

**Submit:** [n8n Creator Portal](https://creators.n8n.io/nodes)

**Benefits:**
- Available in n8n Cloud
- Discoverable in nodes panel
- Verified badge
- Increased community visibility

## Reference Patterns

| Component | File | Use Case |
|-----------|------|----------|
| Declarative node | `nodes/GithubIssues/GithubIssues.node.ts` | REST API integration |
| Programmatic node | `nodes/Example/Example.node.ts` | Custom execution logic |
| API key auth | `credentials/GithubIssuesApi.credentials.ts` | Bearer token |
| OAuth2 auth | `credentials/GithubIssuesOAuth2Api.credentials.ts` | OAuth flow |
| Resource organization | `nodes/GithubIssues/resources/issue/` | Multi-operation structure |
| Dynamic options | `nodes/GithubIssues/listSearch/` | List/search methods |
| Transport layer | `nodes/GithubIssues/shared/transport.ts` | API request wrapper |

## Key Architecture Principles

1. **Declarative for REST APIs** - Use routing for HTTP-based integrations
2. **Programmatic for complex logic** - Custom execute() for non-standard flows
3. **Resource → Operations pattern** - Group API endpoints logically (Issue → Create/Get/Update/Delete)
4. **Separate credentials** - Never hardcode secrets, use credential files
5. **Modular file structure** - Keep operations in separate files for maintainability
6. **Reusable transport layer** - Abstract API client logic in shared utilities
7. **Follow examples** - Use GithubIssues node as reference for best practices

## Troubleshooting

### Node doesn't appear in n8n
1. Verify `npm install` completed successfully
2. Check `package.json` → `n8n.nodes` array includes your node
3. Ensure node file path matches registration (dist/nodes/...)
4. Restart dev server: `npm run dev`
5. Check console for build/registration errors

### TypeScript compilation errors
- Ensure Node.js v22 or higher: `node --version`
- Run `npm install` to get all type definitions
- Check `tsconfig.json` is present and valid

### Lint errors
- Auto-fix: `npm run lint:fix`
- Review [n8n coding guidelines](https://docs.n8n.io/integrations/creating-nodes/)
- Check specific error messages for guidance

### Node registered but not working
- Check browser console for errors
- Verify credentials are properly configured
- Test API endpoints independently (Postman/curl)
- Review `docs/30-troubleshooting-guide.md`

### Build issues
```bash
# Clean build artifacts
rm -rf dist/
npm run build

# Or clean install
rm -rf node_modules/ dist/
npm install
npm run build
```

## Available Scripts

| Script | Description | CLI Equivalent |
|--------|-------------|----------------|
| `npm run dev` | Start n8n with watch mode | `npx n8n-node dev` |
| `npm run build` | Compile TypeScript to dist/ | `npx n8n-node build` |
| `npm run build:watch` | Build with auto-rebuild | - |
| `npm run lint` | Check code quality | `npx n8n-node lint` |
| `npm run lint:fix` | Auto-fix lint issues | `npx n8n-node lint --fix` |
| `npm run release` | Create new release | `npx n8n-node release` |

**CLI Tool:** All scripts use [@n8n/node-cli](https://www.npmjs.com/package/@n8n/node-cli) under the hood

---

**Quick Links:**
- 📖 Full documentation: `docs/00-documentation-index.md`
- 🌐 n8n docs: https://docs.n8n.io/integrations/creating-nodes/
- 💬 Community forum: https://community.n8n.io/
- 🎯 Creator portal: https://creators.n8n.io/nodes