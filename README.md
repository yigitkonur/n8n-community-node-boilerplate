starter kit for building n8n community nodes. two working reference implementations (declarative + programmatic), two credential types (API key + OAuth2), pre-configured AI assistant rules for 10+ editors, and 31 docs covering everything from scratch to npm publish.

```bash
npx @anthropic/create-n8n-node  # or just clone this repo
npm run dev                      # n8n starts at localhost:5678 with hot reload
```

[![n8n](https://img.shields.io/badge/n8n-community_node-93450a.svg?style=flat-square)](https://docs.n8n.io/integrations/creating-nodes/)
[![typescript](https://img.shields.io/badge/typescript-5.9-93450a.svg?style=flat-square)](https://www.typescriptlang.org/)
[![license](https://img.shields.io/badge/license-MIT-grey.svg?style=flat-square)](https://opensource.org/licenses/MIT)

---

## what's inside

two complete node implementations you can copy, rename, and rewire to any API.

### declarative node (GithubIssues)

a fully functional GitHub Issues node that demonstrates the routing-based pattern — no `execute()` method. HTTP calls defined entirely through `routing` config on each property. n8n handles the rest.

**resources and operations:**

| resource | operation | endpoint |
|:---|:---|:---|
| issue | get many | `GET /repos/{owner}/{repo}/issues` |
| issue | get | `GET /repos/{owner}/{repo}/issues/{number}` |
| issue | create | `POST /repos/{owner}/{repo}/issues` |
| issue comment | get many | `GET /repos/{owner}/{repo}/issues/comments` |

**also demonstrates:**
- `resourceLocator` fields with three input modes (searchable dropdown, URL paste, free text)
- `listSearch` methods for live-filtered dropdowns (users, repos, issues)
- Link header pagination with inlined `parseLinkHeader` utility
- conditional UI via `displayOptions`
- `usableAsTool: true` for n8n AI agents

### programmatic node (Example)

minimal stub showing the `execute()` pattern. single string input, item iteration, `continueOnFail()` error handling, `NodeOperationError` with `itemIndex`. copy this when your node needs logic that can't be expressed declaratively.

### credentials

| type | auth method | details |
|:---|:---|:---|
| `GithubIssuesApi` | personal access token | header-injected `Authorization: token {pat}`, built-in test via `GET /user` |
| `GithubIssuesOAuth2Api` | OAuth2 authorization code | extends n8n's `oAuth2Api`, pre-configured GitHub endpoints, `repo` scope |

## AI editor rules

pre-loaded rules so any AI assistant working in this repo already knows n8n node patterns. ships configs for:

Cursor, Windsurf, GitHub Copilot, Continue, Cline, Aider, Gemini CLI, JetBrains AI Assistant, Zed, Roo/RooCode, Claude Code

all point to the same underlying knowledge — n8n node conventions, TypeScript patterns, credential structure.

## getting started

```bash
git clone https://github.com/yigitkonur/n8n-community-node-boilerplate.git
cd n8n-community-node-boilerplate
npm install
```

### dev

```bash
npm run dev          # starts n8n at localhost:5678 with your node loaded
npm run build:watch  # TypeScript watch mode in a separate terminal
```

### build and lint

```bash
npm run build        # compile to dist/
npm run lint         # eslint with n8n's preset
npm run lint:fix     # auto-fix
```

### publish

```bash
npm run release      # version bump + npm publish via release-it
```

only the `dist/` directory ships to npm. sources stay out.

## project structure

```
credentials/
  GithubIssuesApi.credentials.ts          — PAT auth
  GithubIssuesOAuth2Api.credentials.ts    — OAuth2 auth

nodes/
  Example/
    Example.node.ts                       — programmatic pattern stub
  GithubIssues/
    GithubIssues.node.ts                  — main node (declarative routing)
    resources/
      issue/
        index.ts                          — operation dropdown + field composition
        create.ts                         — create operation fields
        get.ts                            — get operation fields
        getAll.ts                         — get many with pagination + filters
      issueComment/
        index.ts                          — comment operations
        getAll.ts                         — get many comments
    listSearch/
      getUsers.ts                         — live user search dropdown
      getRepositories.ts                  — live repo search dropdown
      getIssues.ts                        — live issue search dropdown
    shared/
      transport.ts                        — githubApiRequest() helper
      descriptions.ts                     — reusable resourceLocator definitions
      utils.ts                            — parseLinkHeader() for pagination

docs/                                     — 31 numbered guides (00-30)
```

## key patterns

**spread-based composition** — each resource exports `INodeProperties[]`. the main node spreads them: `...issueDescription, ...issueCommentDescription`. keeps files small.

**conditional UI** — every field uses `displayOptions: { show: { resource: [...], operation: [...] } }` to appear only when relevant.

**three-mode resource locators** — owner/repo/issue fields support searchable dropdown (`list`), URL paste with regex extraction (`url`), and free text with validation (`name`).

**no runtime dependencies** — `n8n-workflow` is a peer dep (provided by n8n at runtime). HTTP calls use n8n's built-in `helpers.httpRequestWithAuthentication`. nothing extra ships.

## tooling

| tool | config |
|:---|:---|
| TypeScript | strict mode, ES2019 target, commonjs modules |
| ESLint | n8n's bundled preset (flat config, ESLint 9) |
| Prettier | tabs, single quotes, trailing commas, 100 char width |
| CI | GitHub Actions — lint + build on push/PR, Node.js 22 |
| release | release-it via `npm run release` |

## docs

the `docs/` directory has 31 numbered guides covering:

- n8n architecture and node lifecycle
- declarative vs programmatic patterns
- credential types and auth flows
- resource locators and dynamic dropdowns
- pagination strategies
- error handling and validation
- testing, debugging, publishing to npm
- AI assistant setup

## license

MIT
