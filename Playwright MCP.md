Run on Windows in power:

```powershell
`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`
npx @playwright/mcp@latest --port 3000 --host 0.0.0.0 --allowed-hosts '*'
```

check IP:

```powershell
ipconfig
```

.mcp.json for pi:

```json
{
  "mcpServers": {
    "playwright": {
      "url": "http://192.168.0.36:3000/mcp"
    }
  }
}
```