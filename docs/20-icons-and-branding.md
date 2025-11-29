# 20 - Icons and Branding

> SVG icons with dark/light mode support

---

## Icon Requirements

- **Format**: SVG only
- **Size**: 24x24px (or scalable)
- **Variants**: Light and dark mode versions
- **Location**: Node directory or shared `/icons/`

---

## Directory Structure

```
├── icons/                          # Shared icons
│   ├── myservice.svg               # Light mode
│   └── myservice.dark.svg          # Dark mode
└── nodes/MyNode/
    └── mynode.svg                  # Node-specific icon
```

---

## Referencing Icons

### Single Icon

```typescript
icon: 'file:mynode.svg'
```

### Light/Dark Variants

```typescript
icon: { 
  light: 'file:../../icons/github.svg', 
  dark: 'file:../../icons/github.dark.svg' 
}
```

### In Credentials

```typescript
icon: Icon = { 
  light: 'file:../icons/myservice.svg', 
  dark: 'file:../icons/myservice.dark.svg' 
}
```

---

## SVG Example

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
  <circle cx="12" cy="12" r="10"/>
  <path d="M12 6v6l4 2"/>
</svg>
```

---

## Best Practices

1. **Keep icons simple** - Clear at small sizes
2. **Use `currentColor`** - Adapts to context
3. **Test both modes** - Verify visibility
4. **Optimize SVGs** - Remove unnecessary metadata

---

## Next Steps

- [21 - Node JSON Metadata](./21-node-json-metadata.md)
