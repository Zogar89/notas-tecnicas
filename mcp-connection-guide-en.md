# How to Connect MercadoLibre MCP in Antigravity/Cursor: The Missing Guide

If you are using **Antigravity** (or Cursor) as your AI-native IDE, you might have noticed that the official MercadoLibre developers documentation haven't caught up with these tools yet. Setting up the **Model Context Protocol (MCP)** for MeLi can be tricky due to specific port requirements and OAuth flows.

After a long troubleshooting session, I've cracked the code on how to get it working perfectly. Here is the definitive guide for Antigravity users.

---

## 1. The "Application Not Ready" Error
The most common issue: you authorize in the browser, but the page says:
> *"La aplicación no está preparada para conectarse a Mercado Libre"*

### The Problem
The official MeLi MCP proxy app strictly expects a redirect to `http://localhost:18999`. However, the `mcp-remote` client used by IDEs like Antigravity often picks a random port (like 17135) if not told otherwise.

### The Fix: Force Port 18999
In your `mcp_config.json` (usually in `%APPDATA%\.gemini\antigravity\`), you must add **18999** as a positional argument.

```json
"mercadolibre": {
  "command": "npx",
  "args": [
    "-y",
    "mcp-remote",
    "https://mcp.mercadolibre.com/mcp",
    "18999"  // <-- REQUIRED for MeLi compatibility
  ],
  "disabled": false
}
```

---

## 2. Eliminate Conflict: Remove Manual Headers
If you previously tried to add a token manually, **delete it**. You cannot use a hardcoded Bearer token while using the OAuth browser flow.

> [!WARNING]
> **Remove the following lines** from your `mcp_config.json`:
> `"--header",`
> `"Authorization: Bearer APP_USR-..."`

Keeping these while trying to log in via browser will cause the authentication exchange to fail with "Existing OAuth client information is required".

---

## 3. Resolving Port Conflicts (EADDRINUSE)
If you get an error saying `address already in use 127.0.0.1:18999`, it means a previous (failed) attempt left a "zombie" process running.

### Windows Recovery (PowerShell):
1. **Kill all Node.js zombies:**
   ```powershell
   taskkill /F /IM node.exe
   ```
2. **Clear the corrupt auth cache:**
   ```powershell
   rmdir /S /Q $HOME\.mcp-auth
   ```

---

## 4. Final Tips for Antigravity/Cursor Users

- **Browser Specifics**: If **Firefox** is your default browser and the "Authorization Successful" page doesn't seem to talk back to your IDE, **copy and paste the link into Chrome**. It often handles the local callback more reliably for dev tools.
- **Beat the Timeout**: Antigravity has a built-in timeout. Once the browser opens, complete the authorization quickly. If it fails, you'll see a `context deadline exceeded` error.
- **Ngrok?**: You **don't** need Ngrok for this setup. The MeLi MCP Proxy is an official app that already has permission to talk to your `localhost:18999`.

### Summary
1. Force **18999** in your config.
2. Remove any **manual headers**.
3. Clear **zombie node processes**.
4. Use **Chrome** if the callback fails.

Happy AI-Coding! 🚀
