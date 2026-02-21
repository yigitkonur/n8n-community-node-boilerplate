# 22 - Development Workflow

> Local development, Docker setup, and debugging

---

## 🤖 AI Agent Context

**READ THIS DOCUMENT** for setting up your development environment.

| Setup Option | Best For | Iteration Time |
|-------------|----------|----------------|
| Native (npm run dev) | Fast iteration | ~2-5 seconds |
| Docker Compose | Production parity | ~5-10 seconds |

**Related**:
- [23-testing-strategies.md](./23-testing-strategies.md) - Testing with Vitest
- [24-linting-and-code-quality.md](./24-linting-and-code-quality.md) - Code quality

---

## Option 1: Native Setup (Fast Iteration)

```bash
# Clone or create project
git clone https://github.com/yigitkonur/n8n-node-boilerplate.git my-nodes
cd my-nodes

# Install dependencies (pnpm recommended)
pnpm install

# Start with hot reload
npm run dev
```

This runs `n8n-node dev` which:
- Builds your node with watch mode
- Starts n8n at http://localhost:5678
- Auto-rebuilds on file changes (~2-5 seconds)

---

## Option 2: Docker Setup (Production Parity)

```yaml
# docker-compose.yml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    volumes:
      - ./dist:/home/node/.n8n/custom/MyApiNode
      - n8n_data:/root/.n8n
    environment:
      - N8N_CUSTOM_EXTENSIONS=/home/node/.n8n/custom
      - NODE_ENV=development
    command: n8n start

volumes:
  n8n_data:
```

**Setup Hot-Reload with Docker:**
```bash
# Terminal 1: Build watch
pnpm build --watch

# Terminal 2: Start Docker
docker-compose up

# Docker volume auto-mounts dist/ changes
```

---

## VS Code Configuration

**.vscode/settings.json:**
```json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "typescript.preferences.includePackageJsonAutoImports": "on",
  "typescript.suggest.autoImports": true,
  "eslint.validate": ["typescript", "typescriptreact"],
  "eslint.format.enable": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    }
  },
  "files.exclude": {
    "**/dist": true,
    "**/node_modules": true,
    "**/.turbo": true
  }
}
```

**Recommended Extensions:**
- `ms-vscode.vscode-typescript-next` - Latest TypeScript features
- `dbaeumer.vscode-eslint` - ESLint integration
- `esbenp.prettier-vscode` - Code formatting
- `vitest.explorer` - Inline test runner

---

## Debugging in VS Code

**.vscode/launch.json:**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Attach to n8n",
      "type": "node",
      "request": "attach",
      "port": 9229,
      "localRoot": "${workspaceFolder}",
      "remoteRoot": "/home/node/.n8n/custom"
    },
    {
      "name": "Vitest Debug",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "pnpm",
      "runtimeArgs": ["test", "--inspect-brk", "--watch"],
      "console": "integratedTerminal",
      "cwd": "${workspaceFolder}"
    }
  ]
}
```

**To Debug Node Execution:**
1. Start n8n: `docker run ... node --inspect=0.0.0.0:9229`
2. Click "Attach to n8n" in VS Code
3. Set breakpoints in `execute()` method
4. Trigger workflow; execution pauses at breakpoints
5. Inspect `$json` values in Debug Console

---

## Build Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Development with hot reload |
| `npm run build` | Production build |
| `npm run build:watch` | Build with file watching |
| `npm run lint` | Check for errors |
| `npm run lint:fix` | Auto-fix lint issues |
| `npm run test` | Run tests |
| `npm run test:coverage` | Run tests with coverage |

---

## Development Workflow

1. **Start dev server**: `npm run dev`
2. **Edit code**: Make changes to `.ts` files
3. **Test in n8n**: Refresh browser to see changes
4. **Run tests**: `npm run test`
5. **Lint**: Run `npm run lint` before committing
6. **Build**: Run `npm run build` for production

---

## Project Files to Edit

| File | When to Edit |
|------|--------------|
| `nodes/*.ts` | Node logic and UI |
| `nodes/*/services/*.ts` | Service operations |
| `credentials/*.ts` | Authentication |
| `package.json` | Metadata and dependencies |
| `tsconfig.json` | TypeScript settings |
| `vitest.config.ts` | Test configuration |

---

## Next Steps

- [23 - Testing Strategies](./23-testing-strategies.md)
- [24 - Linting and Code Quality](./24-linting-and-code-quality.md)
