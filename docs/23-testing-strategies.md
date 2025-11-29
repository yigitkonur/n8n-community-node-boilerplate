# 23 - Testing Strategies

> Manual and automated testing approaches

---

## Manual Testing

### 1. Start Dev Server
```bash
npm run dev
```

### 2. Create Test Workflow
- Add your node to canvas
- Configure credentials
- Connect to trigger (Manual Trigger)
- Execute and verify output

### 3. Test Each Operation
- Create: Verify data is created
- Get: Verify correct data returned
- Update: Verify changes applied
- Delete: Verify removal

### 4. Test Edge Cases
- Empty inputs
- Invalid IDs
- Missing required fields
- API errors (rate limits, auth failures)

---

## Credential Testing

1. Go to **Credentials** > **New**
2. Select your credential type
3. Enter test credentials
4. Click **Test** button
5. Verify success message

---

## Testing Checklist

- [ ] All operations work correctly
- [ ] Error messages are helpful
- [ ] Credentials validate properly
- [ ] Pagination works (if applicable)
- [ ] Dynamic dropdowns populate
- [ ] Continue On Fail works
- [ ] Output data structure is correct

---

## Debugging Tips

1. **Console logs**: Check browser dev tools
2. **n8n logs**: Check terminal running dev server
3. **Network tab**: Inspect API requests
4. **Expression editor**: Test expressions in n8n

---

## Next Steps

- [22 - Development Workflow](./22-development-workflow.md)
- [30 - Troubleshooting Guide](./30-troubleshooting-guide.md)
