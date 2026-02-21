# n8n Community Node Development Guide

> **🤖 AI-OPTIMIZED DOCUMENTATION**: This guide is structured for AI code generation agents. Follow the learning paths below based on your task.
> 
> **Stats**: This approach powers 1,500+ community nodes with millions of downloads (and growing rapidly).

---

## 🎯 AI Agent: Start Here

**BEFORE GENERATING ANY CODE**, read the relevant documents in order. Each document contains complete patterns you can copy and adapt.

### Learning Path A: Build a REST API Node (Declarative)

> **Use this path when**: The target service has a REST API and you want minimal boilerplate code.

```
STEP 1: Read → ./10-creating-your-first-node.md      (Basic structure)
STEP 2: Read → ./05-declarative-vs-programmatic.md   (Understand declarative style)
STEP 3: Read → ./08-api-key-credentials.md           (Create credentials)
STEP 4: Read → ./12-declarative-routing.md           (Define API routes)
STEP 5: Read → ./06-node-properties-reference.md     (UI field types)
STEP 6: Read → ./02-package-json-configuration.md    (Register node)
```

**Reference implementation**: `nodes/GithubIssues/` (full declarative node)

### Learning Path B: Build an SDK-Based Node (Programmatic)

> **Use this path when**: You're integrating an official SDK/client library.

```
STEP 1: Read → ./10-creating-your-first-node.md      (Basic structure)
STEP 2: Read → ./05-declarative-vs-programmatic.md   (Understand programmatic style)
STEP 3: Read → ./08-api-key-credentials.md           (Create credentials)
STEP 4: Read → ./13-custom-execute-methods.md        (Write execute logic)
STEP 5: Read → ./19-external-sdk-integration.md      (SDK patterns)
STEP 6: Read → ./17-error-handling-patterns.md       (Handle errors)
STEP 7: Read → ./23-testing-strategies.md            (Write tests - 90%+ coverage)
```

**Reference implementation**: `nodes/Example/Example.node.ts` (execute method pattern)

### Learning Path C: Add Authentication Only

> **Use this path when**: You only need to create credentials for an existing node.

```
API Key/Token:  Read → ./08-api-key-credentials.md
OAuth2:         Read → ./09-oauth2-credentials.md
Both methods:   Read → ./07-credentials-system-overview.md first
```

**Reference implementations**: `credentials/GithubIssuesApi.credentials.ts`, `credentials/GithubIssuesOAuth2Api.credentials.ts`

### Learning Path D: Add Advanced Features

> **Use this path when**: Adding specific features to an existing node.

| Feature | Read This Document |
|---------|-------------------|
| Dynamic dropdowns | → [./14-list-search-methods.md](./14-list-search-methods.md) |
| Multi-mode inputs (list/URL/ID) | → [./15-resource-locators.md](./15-resource-locators.md) |
| Pagination (cursor/offset) | → [./16-pagination-handling.md](./16-pagination-handling.md) |
| Multiple resources | → [./11-resources-and-operations.md](./11-resources-and-operations.md) |
| Error handling | → [./17-error-handling-patterns.md](./17-error-handling-patterns.md) |
| Helper functions | → [./18-helper-functions-and-utilities.md](./18-helper-functions-and-utilities.md) |
| Batching/Performance | → [./29-common-patterns-and-recipes.md](./29-common-patterns-and-recipes.md) |

---

## 📂 Quick Reference: File Locations

When generating code, create files in these locations:

| What You're Creating | File Path | Naming Pattern |
|---------------------|-----------|----------------|
| Node class | `nodes/{NodeName}/{NodeName}.node.ts` | PascalCase |
| Node metadata | `nodes/{NodeName}/{NodeName}.node.json` | PascalCase |
| Node icon | `nodes/{NodeName}/{nodename}.svg` | lowercase |
| API credential | `credentials/{ServiceName}Api.credentials.ts` | PascalCase |
| OAuth2 credential | `credentials/{ServiceName}OAuth2Api.credentials.ts` | PascalCase |
| Resource operations | `nodes/{NodeName}/resources/{resource}/` | lowercase |
| Service operations | `nodes/{NodeName}/services/{Resource}Operations.ts` | PascalCase |
| List search methods | `nodes/{NodeName}/listSearch/` | camelCase files |
| Shared utilities | `nodes/{NodeName}/shared/` | lowercase files |
| Unit tests | `nodes/{NodeName}/__tests__/` | `.test.ts` suffix |

---

## 🚀 Quick Start Checklist (~45 minutes to production)

```bash
# 1. Initialize project (5 min)
git clone https://github.com/yigitkonur/n8n-node-boilerplate.git my-nodes
cd my-nodes && npm install

# 2. Create your first node (10 min)
mkdir -p nodes/MyApiNode
# Implement MyApiNode.node.ts + description.ts

# 3. Add credentials (5 min)
# Create credentials/MyApiCredentials.credentials.ts

# 4. Write tests (10 min)
# Create __tests__/MyApiNode.test.ts

# 5. Start development (2 min)
npm run dev  # Watch mode at http://localhost:5678

# 6. Test locally (5 min)
# Open n8n and test your node

# 7. Publish to npm (3 min)
npm run build && npm publish
```

---

## 📋 Complete Document Index

### Foundation (Read First for New Projects)
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 01 | [Project Structure](./01-project-structure-overview.md) | Directory layout, monorepo patterns | Setting up new project |
| 02 | [package.json Config](./02-package-json-configuration.md) | n8n registration, scripts | Configuring project |
| 03 | [TypeScript Config](./03-typescript-configuration.md) | Compiler settings, path aliases | Build issues |

### Core Concepts (Required Knowledge)
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 04 | [Node Anatomy](./04-node-anatomy-and-architecture.md) | INodeType interface, description | Understanding node structure |
| 05 | [Declarative vs Programmatic](./05-declarative-vs-programmatic-nodes.md) | Two approaches with examples | Choosing approach |
| 06 | [Properties Reference](./06-node-properties-reference.md) | All property types | Building UI fields |

### Authentication
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 07 | [Credentials Overview](./07-credentials-system-overview.md) | How auth works | Understanding system |
| 08 | [API Key Auth](./08-api-key-credentials.md) | Token/key authentication | Most common auth |
| 09 | [OAuth2 Auth](./09-oauth2-credentials.md) | OAuth2 flows | OAuth integrations |

### Building Nodes
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 10 | [First Node Tutorial](./10-creating-your-first-node.md) | Complete walkthrough | **START HERE** |
| 11 | [Resources & Operations](./11-resources-and-operations.md) | Resource-operation dispatch | Complex APIs |
| 12 | [Declarative Routing](./12-declarative-routing.md) | HTTP routing patterns | REST API nodes |
| 13 | [Execute Methods](./13-custom-execute-methods.md) | Programmatic execution | SDK-based nodes |
| 14 | [List Search](./14-list-search-methods.md) | Dynamic dropdowns | Searchable selects |

### Advanced Features
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 15 | [Resource Locators](./15-resource-locators.md) | Multi-mode inputs | Flexible selection |
| 16 | [Pagination](./16-pagination-handling.md) | Cursor/offset pagination | "Return All" feature |
| 17 | [Error Handling](./17-error-handling-patterns.md) | Robust error patterns | Error management |
| 18 | [Helper Functions](./18-helper-functions-and-utilities.md) | Reusable utilities | DRY code |
| 19 | [SDK Integration](./19-external-sdk-integration.md) | Third-party SDKs | Non-REST APIs |

### Metadata & Assets
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 20 | [Icons & Branding](./20-icons-and-branding.md) | SVG icons, dark mode | Visual assets |
| 21 | [Node Metadata](./21-node-json-metadata.md) | Codex JSON file | Catalog listing |

### Development & Testing
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 22 | [Dev Workflow](./22-development-workflow.md) | Local/Docker development, VS Code | Daily development |
| 23 | [Testing](./23-testing-strategies.md) | Vitest, 90%+ coverage | Quality assurance |
| 24 | [Linting](./24-linting-and-code-quality.md) | ESLint + Prettier | Before commit |

### Publishing
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 25 | [Pre-Publish Checklist](./25-preparing-for-publication.md) | Requirements | Before publishing |
| 26 | [npm Publishing](./26-publishing-to-npm.md) | Semantic-release, CI/CD | Release time |
| 27 | [n8n Cloud Verification](./27-n8n-cloud-verification.md) | Get verified | After publishing |

### Reference & Troubleshooting
| # | Document | What You'll Learn | When to Read |
|---|----------|-------------------|--------------|
| 28 | [Code Examples](./28-complete-code-examples.md) | Copy-paste patterns | Quick reference |
| 29 | [Common Patterns](./29-common-patterns-and-recipes.md) | Batching, streaming, performance | Specific solutions |
| 30 | [Troubleshooting](./30-troubleshooting-guide.md) | Common pitfalls & fixes | When stuck |

---

## 🔗 External Resources

- **n8n Official Docs**: https://docs.n8n.io/integrations/creating-nodes/
- **n8n Community Forum**: https://community.n8n.io/
- **@n8n/node-cli**: https://www.npmjs.com/package/@n8n/node-cli
- **n8n Verification Guidelines**: https://docs.n8n.io/integrations/creating-nodes/build/reference/verification-guidelines/
- **n8n-node-boilerplate**: https://github.com/yigitkonur/n8n-node-boilerplate

---

## ⚡ Quick Start Commands

```bash
# Create new node package (interactive)
npm create @n8n/node

# Install dependencies (pnpm recommended)
pnpm install

# Start development (hot reload)
npm run dev

# Build for production
npm run build

# Run tests with coverage
npm run test:coverage

# Check code quality
npm run lint
npm run lint:fix
```

---

## 🎯 Key Takeaways

✅ **Use Vite + Vitest + TypeScript + Zod** for maximum DX and quality  
✅ **Docker for local dev** matches production environment  
✅ **Resource-operation dispatch** scales to 100+ operations  
✅ **Service-based organization** enables testing and maintenance  
✅ **90%+ test coverage** prevents production bugs  
✅ **Pagination/batching** essential for scalability  
✅ **Semantic versioning** + GitHub Actions automates releases

---

*Documentation optimized for AI code generation agents.*  
*Version: 2.1.0 | Last Updated: November 2025 | Fact-checked against n8n v1.121+*
