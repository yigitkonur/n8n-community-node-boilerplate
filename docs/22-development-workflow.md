# 22 - Development Workflow

> Local development and hot reload

---

## Setup

```bash
# Clone or create project
git clone https://github.com/n8n-io/n8n-nodes-starter.git
cd n8n-nodes-starter

# Install dependencies
npm install
```

---

## Development Server

```bash
# Start with hot reload
npm run dev
```

This runs `n8n-node dev` which:
- Builds your node with watch mode
- Starts n8n at http://localhost:5678
- Auto-rebuilds on file changes

---

## Build Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Development with hot reload |
| `npm run build` | Production build |
| `npm run build:watch` | Build with file watching |
| `npm run lint` | Check for errors |
| `npm run lint:fix` | Auto-fix lint issues |

---

## Workflow

1. **Start dev server**: `npm run dev`
2. **Edit code**: Make changes to `.ts` files
3. **Test in n8n**: Refresh browser to see changes
4. **Lint**: Run `npm run lint` before committing
5. **Build**: Run `npm run build` for production

---

## Project Files to Edit

| File | When to Edit |
|------|--------------|
| `nodes/*.ts` | Node logic and UI |
| `credentials/*.ts` | Authentication |
| `package.json` | Metadata and dependencies |
| `tsconfig.json` | TypeScript settings |

---

## Next Steps

- [23 - Testing Strategies](./23-testing-strategies.md)
- [24 - Linting and Code Quality](./24-linting-and-code-quality.md)
