# 26 - Publishing to npm

> Step-by-step publishing guide

---

## Prerequisites

1. npm account: https://www.npmjs.com/signup
2. Logged in: `npm login`
3. Package built: `npm run build`

---

## Publishing Steps

### 1. Verify Package

```bash
# Check what will be published
npm pack --dry-run
```

### 2. Publish

```bash
# Publish to npm
npm publish
```

### 3. Verify

Visit: `https://www.npmjs.com/package/n8n-nodes-your-service`

---

## Version Updates

```bash
# Patch: 1.0.0 → 1.0.1
npm version patch

# Minor: 1.0.0 → 1.1.0  
npm version minor

# Major: 1.0.0 → 2.0.0
npm version major

# Then publish
npm publish
```

---

## Using release-it

```bash
npm run release
```

This handles versioning, changelog, and publishing.

---

## Common Issues

| Issue | Solution |
|-------|----------|
| "Package name already exists" | Choose unique name |
| "Must be logged in" | Run `npm login` |
| "Missing files" | Check `files` in package.json |

---

## Next Steps

- [25 - Preparing for Publication](./25-preparing-for-publication.md)
- [27 - n8n Cloud Verification](./27-n8n-cloud-verification.md)
