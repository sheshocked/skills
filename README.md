# 🧠 Dynamic Developer Curation & Masterpiece Tap

This repository serves as a dynamic tap for **Hermes**, containing production-grade, highly-actionable developer skills. 

Rather than maintaining thousands of generic, auto-generated stubs that bloat context windows, this repo is strictly curated for depth, code quality, and fast debugging.

## Key Focus Areas
- **Android VPN Core & NDK:** Kotlin implementations, WireGuard integrations, NDK Rust builds, and LLDB crash tracing.
- **Network Bypasses & Protocols:** Cohosted multi-core VLESS Reality configurations, Netlify Deno relays, and AmneziaWG obfuscations.
- **Resilience Engineering:** Circuit breaker loops, clean database schema migrations, and secure secrets handling.
- **Design Systems & Animations:** Dynamic OLED black themes, mirrored RTL layouts, and physical spring animations.

---

## Skills Catalog

### 📂 ANDROID
- **[`android-ndk-rust`](./android/android-ndk-rust/SKILL.md)**: Compile low-level Rust dynamic libraries for Android architectures and bind using JNI.
- **[`android-vpn-service`](./android/android-vpn-service/SKILL.md)**: Establish a production-grade Kotlin VpnService, managing local TUN interfaces and JNI loop mappings.
- **[`compose-horizontal-scroll`](./android/compose-horizontal-scroll/SKILL.md)**: Build responsive horizontal LazyRow layouts in Compose, preventing vertical text compression on compact screens.
- **[`ndk-lldb-debugging`](./android/ndk-lldb-debugging/SKILL.md)**: Debug native C++/Rust crashes, parse tombstones using ndk-stack, and bind LLDB via adb.
- **[`wireguard-android`](./android/wireguard-android/SKILL.md)**: Integrate the official WireGuard Go-backend library and manage VPN tunnels in Android applications.

### 📂 PROTOCOLS
- **[`amneziawg-obfuscation`](./protocols/amneziawg-obfuscation/SKILL.md)**: Deploy AmneziaWG (obfuscated WireGuard) to defeat Deep Packet Inspection on restrictive networks.
- **[`cdn-websocket-tunnel`](./protocols/cdn-websocket-tunnel/SKILL.md)**: Establish VLESS + WebSocket + TLS tunnels behind CDN edge proxies (Cloudflare/ArvanCloud).
- **[`netlify-edge-vless`](./protocols/netlify-edge-vless/SKILL.md)**: Deploy WebSocket proxy relays on Netlify Edge (Deno runtime) to bypass VPS IP blocks.
- **[`sing-box-client`](./protocols/sing-box-client/SKILL.md)**: Construct advanced Sing-box client JSON configs with TUN interfaces and DNS routing rules.
- **[`vless-reality-nginx`](./protocols/vless-reality-nginx/SKILL.md)**: Setup VLESS + Reality cohosting with Nginx Stream multiplexing and ssl_preread on port 443.
- **[`xui-panel-tuning`](./protocols/xui-panel-tuning/SKILL.md)**: Optimize x-ui/3x-ui panels: secure listening ports, automate cert renewals, and clean SQLite storage.

### 📂 DEVOPS
- **[`docker-production-deployment`](./devops/docker-production-deployment/SKILL.md)**: Establish multi-stage builds, non-root runtimes, health checks, and Docker Compose configurations.
- **[`linux-server-hardening`](./devops/linux-server-hardening/SKILL.md)**: Hardening sysctl.conf, limits.conf, kernel security, and resource limits on production servers.
- **[`nginx-reverse-proxy`](./devops/nginx-reverse-proxy/SKILL.md)**: Establish HTTP/2, rate limiting, secure headers, and WebSocket upgrades on Nginx.
- **[`ssh-security-hardening`](./devops/ssh-security-hardening/SKILL.md)**: Disable password authentication, enforce key pairs, secure ports, and configure fail2ban.

### 📂 SECURITY
- **[`network-segmentation-iptables`](./security/network-segmentation-iptables/SKILL.md)**: Establish UFW/nftables firewalls, limit open ports, block scanner active probing, and restrict subnets.
- **[`secrets-management-vault`](./security/secrets-management-vault/SKILL.md)**: Deploy HashiCorp Vault, configure transit engine, manage policy access, and read secrets in production.
- **[`web-app-pentesting`](./security/web-app-pentesting/SKILL.md)**: Audit web app endpoints for SQL injection, XSS, CSRF, and SSRF vulnerabilities using curl and sqlmap.

### 📂 ENGINEERING
- **[`database-schema-design`](./engineering/database-schema-design/SKILL.md)**: Build safe migrations, partial indexes, multi-tenant row level security (RLS), and GIN/JSONB indexes.
- **[`system-design-patterns`](./engineering/system-design-patterns/SKILL.md)**: Implement robust system patterns (Circuit Breaker, Singleflight, Rate Limiting) in production APIs.
- **[`test-driven-refactoring`](./engineering/test-driven-refactoring/SKILL.md)**: Refactor legacy codebase architectures safely using mocking, unit tests, and CI boundaries checks.

### 📂 DESIGN
- **[`dark-mode-dynamic`](./design/dark-mode-dynamic/SKILL.md)**: Establish dynamic theme switching, OLED black styles, and state synchronization across layouts.
- **[`figma-to-code`](./design/figma-to-code/SKILL.md)**: Translate complex layout layers, auto-layout frames, grid variables, and styling tokens from Figma to CSS.
- **[`premium-animations-motion`](./design/premium-animations-motion/SKILL.md)**: Structure physics-backed spring animation easing curves and smooth micro-interactions in Framer Motion.
- **[`rtl-arabic-persian-ui`](./design/rtl-arabic-persian-ui/SKILL.md)**: Configure RTL layout mirroring, Vazirmatn font-family, and Persian numbers formatting.

### 📂 AI-TOOLS
- **[`mcp-server-development`](./ai-tools/mcp-server-development/SKILL.md)**: Build Model Context Protocol (MCP) servers in Node/TypeScript, implementing custom tools and resources.
- **[`prompt-cache-tuning`](./ai-tools/prompt-cache-tuning/SKILL.md)**: Structure prompts for LLMs (such as Claude 3.5/3.7) to maximize prompt caching, reducing cost and latency.
- **[`rag-pipeline-building`](./ai-tools/rag-pipeline-building/SKILL.md)**: Build RAG pipelines in Python, chunking text, indexing embeddings in vector databases, and using cross-encoder rerankers.

### 📂 GENERAL
- **[`bash-script-automation`](./general/bash-script-automation/SKILL.md)**: Establish strict mode scripts, process traps, cleanup functions, and log styling.
- **[`fast-debugging-traces`](./general/fast-debugging-traces/SKILL.md)**: Establish high-speed debugging workflows using visual traces, runtime logs analysis, and diagnostic parameters.
- **[`github-api-automation`](./general/github-api-automation/SKILL.md)**: Automate GitHub releases, assets uploading, tags parsing, and runner tracking using octokit and raw curl endpoints.

---
*Generated dynamically with developer-first intent. Free of robotic slop, boilerplate placeholders, and AI structural tells.*
