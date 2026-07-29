---
name: netlify-edge-vless
description: Deploy a serverless VLESS/VMess WebSocket relay on Netlify Edge Functions (Deno runtime) to bypass IP blocks.
category: protocols
tags: [netlify-edge, vless-relay, websocket-proxy, deno, dns-fronting]
---

# Netlify Edge Vless

## When to Use
Use when direct IPs are blocked in Iran, and you want to route client traffic through Netlify CDN nodes to a hidden origin server.

## Prerequisites
- Netlify account and Netlify CLI installed.

## Workflow
1. Write a Deno WebSocket upgrade script.
2. Proxy client socket connection streams to origin Xray WebSocket port.
3. Configure path obfuscation.

## Key Patterns
```javascript
// netlify/edge-functions/vless.js
export default async (request, context) => {
  if (request.headers.get("upgrade") === "websocket") {
    // Connect to origin websocket server
    const targetUrl = "ws://185.71.219.72:8080/mysecretpath";
    const ws = new WebSocket(targetUrl);
    
    const { socket, response } = Deno.upgradeWebSocket(request);
    
    socket.onmessage = (event) => ws.send(event.data);
    ws.onmessage = (event) => socket.send(event.data);
    
    socket.onclose = () => ws.close();
    ws.onclose = () => socket.close();
    
    return response;
  }
  return new Response("Unauthorized", { status: 401 });
};
```

## Pitfalls
- **High Concurrency Timeouts:** Netlify Edge functions kill long-running idle connections. Implement client ping/pong keepalives every 10 seconds.
- **WebSocket path detection:** Obfuscate paths so active scanners get 401/404 pages.

## Verification
- Test function locally with Netlify Dev CLI: `netlify dev`.
- Query websocket path with curl and check `101 Switching Protocols` response.
