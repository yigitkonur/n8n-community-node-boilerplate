# 21 - Node JSON Metadata

> Codex and documentation metadata for n8n catalog

---

## File Location

```
nodes/MyNode/MyNode.node.json
```

---

## Complete Structure

```json
{
  "node": "n8n-nodes-myservice",
  "nodeVersion": "1.0",
  "codexVersion": "1.0",
  "categories": ["Productivity", "Developer Tools"],
  "resources": {
    "credentialDocumentation": [
      {
        "url": "https://github.com/org/repo#credentials"
      }
    ],
    "primaryDocumentation": [
      {
        "url": "https://github.com/org/repo"
      }
    ]
  }
}
```

---

## Fields

| Field | Description |
|-------|-------------|
| `node` | Package name |
| `nodeVersion` | Node version |
| `codexVersion` | Always "1.0" |
| `categories` | n8n panel categories |
| `resources` | Documentation links |

---

## Categories

Common categories:
- `Productivity`
- `Developer Tools`
- `Data & Storage`
- `Marketing`
- `Communication`
- `Sales`
- `Finance`

---

## Next Steps

- [20 - Icons and Branding](./20-icons-and-branding.md)
- [25 - Preparing for Publication](./25-preparing-for-publication.md)
