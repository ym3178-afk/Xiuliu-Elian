# Xinliu · Real Third-Party OAuth Account Linking

This version replaces the previous mock “connected” state with a real OAuth flow:

1. The front end requests an official authorization URL from the back end.
2. The user signs in on the official Baidu Netdisk / Yuque / Feishu / DingTalk authorization page.
3. The provider redirects back to this project’s server.
4. The server exchanges the authorization code for an access token.
5. Tokens are stored only on the server, never in HTML or localStorage.
6. The front end receives the success message and shows the real account name and connected state.
7. The user can disconnect the account.

## Why GitHub Pages alone is not enough

GitHub Pages can only host static files. It cannot securely store OAuth client secrets or perform the authorization-code-to-access-token exchange. Real account linking therefore requires a server. The repository can still live on GitHub, but the Node.js server should be deployed to a host such as Render, Railway, Fly.io, or a VPS. Another option is to host the front end on GitHub Pages and deploy the back end separately.

## Run locally

```bash
npm install
cp .env.example .env
# Add each provider's Client ID / Client Secret to .env
npm start
```

Open:

```text
http://localhost:8787
```

## OAuth provider configuration

Create an application in each provider’s developer portal and configure these callback URLs:

- Baidu: `https://YOUR-BACKEND-DOMAIN/api/oauth/baidu/callback`
- Yuque: `https://YOUR-BACKEND-DOMAIN/api/oauth/yuque/callback`
- Feishu: `https://YOUR-BACKEND-DOMAIN/api/oauth/feishu/callback`
- DingTalk: `https://YOUR-BACKEND-DOMAIN/api/oauth/dingtalk/callback`

Then add the corresponding Client ID / Client Secret to the server environment variables. **Never put a Client Secret in `index.html` or commit it to GitHub.**

## GitHub Pages + separate back end

If the front end remains on GitHub Pages, set this before the main script runs in `index.html`:

```html
<script>
window.IFLOW_OAUTH_API_BASE = 'https://YOUR-OAUTH-BACKEND';
</script>
```

And set the back-end environment variable:

```text
APP_ORIGIN=https://YOUR-USERNAME.github.io
```

## Current scope

This version implements real account authorization and linking. Listing, selecting, and importing third-party files requires additional provider API calls on the back end; those states should not be faked by changing button text alone.

## Security notes

- OAuth `state` is HMAC-signed and expires after 10 minutes to reduce callback-forgery risk.
- Access tokens are never returned to the browser.
- Client Secrets are read only from server environment variables.
- This is a prototype. Tokens are currently stored in Node.js process memory and are lost after a restart. For production, use encrypted persistence in a database, Redis, or KV store.
