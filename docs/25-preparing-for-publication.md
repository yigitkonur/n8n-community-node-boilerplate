# 25 - Preparing for Publication

> Pre-publish checklist and requirements

---

## Pre-Publish Checklist

### Package.json
- [ ] `name` starts with `n8n-nodes-`
- [ ] `version` is set correctly
- [ ] `description` is meaningful
- [ ] `keywords` includes `n8n-community-node-package`
- [ ] `author` information complete
- [ ] `repository` URL correct
- [ ] `license` is MIT (for n8n Cloud)

### Code Quality
- [ ] `npm run lint` passes
- [ ] `npm run build` succeeds
- [ ] All operations tested manually
- [ ] Error handling implemented
- [ ] No console.log statements

### Documentation
- [ ] README.md updated
- [ ] Credentials documented
- [ ] Operations documented
- [ ] LICENSE.md present

### Files
- [ ] Icons included
- [ ] `*.node.json` metadata complete
- [ ] No sensitive data in code

---

## Update package.json

```json
{
  "name": "n8n-nodes-your-service",
  "version": "1.0.0",
  "description": "n8n node for Your Service",
  "keywords": ["n8n-community-node-package"],
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/you/n8n-nodes-your-service.git"
  }
}
```

---

## Build for Production

```bash
npm run build
npm run lint
```

---

## Next Steps

- [26 - Publishing to npm](./26-publishing-to-npm.md)
- [27 - n8n Cloud Verification](./27-n8n-cloud-verification.md)
