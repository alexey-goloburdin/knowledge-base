Run on Windows in powershell:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
npx @playwright/mcp@latest --port 3000 --host 0.0.0.0 --allowed-hosts '*'
```

check IP:

```powershell
ipconfig
```

Get WSL address, for example, `172.27.112.1`.

.mcp.json for pi:

```json
{
  "mcpServers": {
    "playwright": {
      "url": "http://172.27.112.1:3000/mcp"
    }
  }
}
```

Run pi:

```shell
http_proxy="" https_proxy="" pi
```