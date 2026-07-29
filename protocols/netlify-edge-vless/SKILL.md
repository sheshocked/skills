---
name: netlify-edge-vless
description: Deploy high-performance WebSocket proxy relays on Netlify Edge Functions (Deno runtime) and Cloudflare Workers to bypass IP blocks.
category: protocols
tags: [netlify, edge-functions, deno, cloudflare-workers, websocket-tunnel, obfuscation, vless]
---

# Netlify Edge Functions VLESS/VMess Relay Masterclass

## When to Use
Use when direct connections to your VPS are blocked by the ISP (e.g. MCI, Irancell) but connections to Netlify CDN nodes are open. This proxies client traffic through Netlify Edge Functions to a hidden origin server.

## Prerequisites
- Netlify account.
- Netlify CLI installed (`npm install -g netlify-cli`).
- Target VLESS+WS inbound running on origin server port 80 (or 443 with valid SSL).

## Workflow
1. Write a Deno Edge Function mapping the WebSocket protocol.
2. Upgrade incoming HTTP requests to WebSockets.
3. Establish a backend socket to the target origin server.
4. Bridge the readable and writable streams between client and origin with custom ping keepalives to prevent timeouts.

## Key Patterns

### Deno Edge Function Code (netlify/edge-functions/vless.js)
```javascript
// Deno script running on Netlify Edge nodes
export default async (request, context) => {
  const url = new URL(request.url);
  
  // Obfuscate pathway - restrict to secret path
  if (url.pathname !== "/my-obfuscated-path") {
    return new Response("Not Found", { status: 404 });
  }

  if (request.headers.get("upgrade") !== "websocket") {
    return new Response("Normal Webpage Decoy", { status: 200 });
  }

  try {
    const { socket: clientSocket, response } = Deno.upgradeWebSocket(request);
    
    // Connect to origin VPS WebSocket server
    const targetWsUrl = "ws://185.71.219.72:80/ws-path";
    const originSocket = new WebSocket(targetWsUrl);

    // Keepalive intervals (Netlify kills connections idle for >30s)
    let pingInterval;

    originSocket.onopen = () => {
      pingInterval = setInterval(() => {
        if (originSocket.readyState === WebSocket.OPEN) {
          originSocket.send(new Uint8Array([0x09])); // Raw WebSocket ping frame
        }
      }, 15000);
    };

    clientSocket.onmessage = (event) => {
      if (originSocket.readyState === WebSocket.OPEN) {
        originSocket.send(event.data);
      }
    };

    originSocket.onmessage = (event) => {
      if (clientSocket.readyState === WebSocket.OPEN) {
        clientSocket.send(event.data);
      }
    };

    const cleanup = () => {
      clearInterval(pingInterval);
      try { clientSocket.close(); } catch(e) {}
      try { originSocket.close(); } catch(e) {}
    };

    clientSocket.onclose = cleanup;
    originSocket.onclose = cleanup;
    clientSocket.onerror = cleanup;
    originSocket.onerror = cleanup;

    return response;
  } catch (err) {
    return new Response(`WebSocket upgrade failed: ${err.message}`, { status: 500 });
  }
};
```

### Netlify Config (netlify.toml)
```toml
[[edge_functions]]
path = "/my-obfuscated-path"
function = "vless"
```

## Pitfalls
- **High concurrency connection drops:** Serverless runtimes have strict memory limits. Ensure you clear memory buffers and intervals on socket close events.
- **Client header filtering:** Some clients (like v2rayN) send additional headers that Netlify's reverse proxy blocks. Scrub headers on the proxy wrapper before passing them upstream.

## Verification
- Deploy using `netlify deploy --prod`.
- Verify the connection by hitting the endpoint using curl: `curl -i -H "Upgrade: websocket" -H "Connection: Upgrade" https://your-site.netlify.app/my-obfuscated-path` should return `101 Switching Protocols`.
