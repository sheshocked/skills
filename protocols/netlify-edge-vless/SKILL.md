---
name: netlify-edge-vless
description: Deploy WebSocket proxy relays on Netlify Edge (Deno runtime) to bypass VPS IP blocks.
category: protocols
tags: [netlify-edge, deno, websocket-proxy, vless-relay, dns-fronting]
---

# Netlify Edge Vless

## When to Use
Use when direct connections to your VPS are blocked by the ISP, but connections to Netlify CDN nodes are open.

## Prerequisites
- Netlify account and CLI interface.
- VLESS+WS inbound on VPS.

## Workflow
1. Write a Deno Edge Function upgrading incoming requests to WebSockets.
2. Establish dynamic bridge loops connecting client streams to the origin server.
3. Deploy function with secret paths mapping connection configurations.

## Key Patterns

### Deno Edge Function Code (netlify/edge-functions/vless.js)
```javascript
export default async (request, context) => {
  const url = new URL(request.url);
  if (url.pathname !== "/my-secret-ws") {
    return new Response("Not Found", { status: 404 });
  }

  if (request.headers.get("upgrade") !== "websocket") {
    return new Response("Unauthorized", { status: 401 });
  }

  try {
    const { socket: clientSocket, response } = Deno.upgradeWebSocket(request);
    const originSocket = new WebSocket("ws://185.71.219.72:80/ws-path");

    let keepalive;
    originSocket.onopen = () => {
      keepalive = setInterval(() => {
        if (originSocket.readyState === WebSocket.OPEN) {
          originSocket.send(new Uint8Array([0x09])); // Raw WebSocket ping frame
        }
      }, 15000);
    };

    clientSocket.onmessage = (e) => {
      if (originSocket.readyState === WebSocket.OPEN) originSocket.send(e.data);
    };
    originSocket.onmessage = (e) => {
      if (clientSocket.readyState === WebSocket.OPEN) clientSocket.send(e.data);
    };

    const closeAll = () => {
      clearInterval(keepalive);
      try { clientSocket.close(); } catch(e) {}
      try { originSocket.close(); } catch(e) {}
    };

    clientSocket.onclose = closeAll;
    originSocket.onclose = closeAll;

    return response;
  } catch (err) {
    return new Response("Websocket connection failed", { status: 500 });
  }
};
```

## Pitfalls
- **Memory leaks in background loops:** Ensure `clearInterval` is called when either socket is closed.
- **Client header filtering:** Strip excess headers that the Netlify proxy blocks before passing them to the origin.

## Verification
- Test using curl upgrading connection, verifying `101 Switching Protocols` returns.

