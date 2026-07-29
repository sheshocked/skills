# 🧠 Custom Developer Skills Tap

Curated list of exactly **100** high-quality, production-ready, and highly-actionable developer skills. 

No placeholders, no 2,000 blank files. Only real recipes.

## Categories

### 📂 `android`

- **[`android-localization`](./android/android-localization/SKILL.md)** — Configure RTL mirroring layouts, typography, and resources for Arabic and Persian languages.
- **[`android-ndk-rust`](./android/android-ndk-rust/SKILL.md)** — Compile native Rust code for Android targets (aarch64, x86_64) and link via JNI.
- **[`android-security-keystore`](./android/android-security-keystore/SKILL.md)** — Encrypt user credentials and private configs using Android Hardware-Backed Keystore API.
- **[`android-vpn-service`](./android/android-vpn-service/SKILL.md)** — Establish a local TUN interface and route packet streams programmatically on Android using JNI/VPNService.
- **[`compose-horizontal-scroll`](./android/compose-horizontal-scroll/SKILL.md)** — Build premium horizontal scrolling wrappers in Jetpack Compose to prevent vertical layouts compression.
- **[`compose-navigation`](./android/compose-navigation/SKILL.md)** — Establish type-safe navigation graphs and customized screen transition animations in Compose.
- **[`gradle-build-optimization`](./android/gradle-build-optimization/SKILL.md)** — Speed up Android compilation using build caching, configuration caching, and dynamic module structures.
- **[`hilt-dependency-injection`](./android/hilt-dependency-injection/SKILL.md)** — Configure Hilt DI graph, ViewModels injection, and modular dependencies scopes in Android apps.
- **[`jetpack-compose-performance`](./android/jetpack-compose-performance/SKILL.md)** — Diagnose and optimize Jetpack Compose UI recomposition bottlenecks and memory leaks.
- **[`room-database`](./android/room-database/SKILL.md)** — Build Room SQLite local schemas, migrations, relations, and Flow database query models.
- **[`wireguard-android`](./android/wireguard-android/SKILL.md)** — Integrate the official WireGuard Android Go/Rust library for modern VPN tunnel implementation.
- **[`workmanager-background`](./android/workmanager-background/SKILL.md)** — Schedule persistent background tasks with network and device constraints using Android WorkManager.

### 📂 `protocols`

- **[`amneziawg-obfuscation`](./protocols/amneziawg-obfuscation/SKILL.md)** — Deploy AmneziaWG (obfuscated WireGuard) to defeat Deep Packet Inspection on restrictive networks.
- **[`cdn-websocket-tunnel`](./protocols/cdn-websocket-tunnel/SKILL.md)** — Establish VLESS + WebSocket + TLS tunnels behind CDN edge proxies (Cloudflare/ArvanCloud).
- **[`hysteria2-quic`](./protocols/hysteria2-quic/SKILL.md)** — Deploy Hysteria2 QUIC-based VPN protocol, tuning congestion control for high packet loss networks.
- **[`marzban-panel`](./protocols/marzban-panel/SKILL.md)** — Deploy Marzban multi-node management panel, configure subscription endpoints and node TLS.
- **[`netlify-edge-vless`](./protocols/netlify-edge-vless/SKILL.md)** — Deploy a serverless VLESS/VMess WebSocket relay on Netlify Edge Functions (Deno runtime) to bypass IP blocks.
- **[`proxy-chain-routing`](./protocols/proxy-chain-routing/SKILL.md)** — Chain outbound proxy protocols (VLESS -> SOCKS5 -> Residential IP) to hide server IPs.
- **[`sing-box-client`](./protocols/sing-box-client/SKILL.md)** — Construct advanced Sing-box client JSON configurations with TUN interfaces and DNS routing rules.
- **[`tuic-protocol`](./protocols/tuic-protocol/SKILL.md)** — Deploy TUIC proxy protocol over HTTP/3 QUIC transport, optimizing socket latency.
- **[`vless-reality-nginx`](./protocols/vless-reality-nginx/SKILL.md)** — Configure VLESS + REALITY cohosted behind Nginx stream multiplexing using ssl_preread on port 443.
- **[`xui-panel-tuning`](./protocols/xui-panel-tuning/SKILL.md)** — Optimize x-ui/3x-ui panels: secure listening ports, automate cert renewals, and clean SQLite storage.

### 📂 `devops`

- **[`automated-backups`](./devops/automated-backups/SKILL.md)** — Establish automated cron jobs to backup SQLite / Postgres databases, encrypting and uploading to S3.
- **[`ci-cd-github-actions`](./devops/ci-cd-github-actions/SKILL.md)** — Establish build pipelines, caching dependencies, and automatic ssh deployment in GitHub actions.
- **[`dns-operations-cloudflare`](./devops/dns-operations-cloudflare/SKILL.md)** — Automate dynamic DNS updates, Cloudflare records management, and origin certificate renewals.
- **[`docker-production-deployment`](./devops/docker-production-deployment/SKILL.md)** — Establish multi-stage builds, non-root runtimes, health checks, and Docker Compose configurations.
- **[`email-deliverability`](./devops/email-deliverability/SKILL.md)** — Configure SPF, DKIM, DMARC records and secure Postfix servers for high deliverability.
- **[`linux-server-hardening`](./devops/linux-server-hardening/SKILL.md)** — Hardening sysctl.conf, limits.conf, kernel security, and resource limits on production servers.
- **[`nginx-reverse-proxy`](./devops/nginx-reverse-proxy/SKILL.md)** — Establish HTTP/2, rate limiting, secure headers, and WebSocket upgrades on Nginx.
- **[`observability-loki-promtail`](./devops/observability-loki-promtail/SKILL.md)** — Establish structured JSON logging, collection pipeline with Grafana Loki and Promtail.
- **[`prometheus-grafana-monitoring`](./devops/prometheus-grafana-monitoring/SKILL.md)** — Establish metrics collection, node exporter, alerts configuration, and Grafana monitoring panels.
- **[`ssh-security-hardening`](./devops/ssh-security-hardening/SKILL.md)** — Disable password authentication, enforce key pairs, secure ports, and configure fail2ban.

### 📂 `security`

- **[`api-security-hardening`](./security/api-security-hardening/SKILL.md)** — Enforce strict JWT claims validation, rate limiting, secure headers, CORS, and request validations.
- **[`cloud-infrastructure-audit`](./security/cloud-infrastructure-audit/SKILL.md)** — Audit cloud resources configurations, examine IAM role policies, and detect drift.
- **[`container-security`](./security/container-security/SKILL.md)** — Scan Docker containers with Trivy, audit runtime capabilities, and configure read-only root filesystems.
- **[`log-monitoring-siem`](./security/log-monitoring-siem/SKILL.md)** — Deploy Wazuh security agents, analyze syslog alerts, and trigger incident response playbooks.
- **[`malware-analysis-static`](./security/malware-analysis-static/SKILL.md)** — Write YARA rules, inspect ELF/PE file headers, and isolate binary behaviors.
- **[`network-segmentation-iptables`](./security/network-segmentation-iptables/SKILL.md)** — Establish UFW/nftables firewalls, limit open ports, block scanner active probing, and restrict subnets.
- **[`phishing-defense-simulation`](./security/phishing-defense-simulation/SKILL.md)** — Set up GoPhish campaign servers, analyze SMTP message headers, and train filters.
- **[`reverse-engineering-basics`](./security/reverse-engineering-basics/SKILL.md)** — Decompile APKs using jadx, inspect assembly in ghidra, and patch binary resources.
- **[`secrets-management-vault`](./security/secrets-management-vault/SKILL.md)** — Deploy HashiCorp Vault, inject runtime secrets, and audit access policies.
- **[`ssh-security-hardening-advanced`](./security/ssh-security-hardening-advanced/SKILL.md)** — Configure SSH hardware security keys (FIDO2), SSH certificates, and session auditing.
- **[`web-app-pentesting`](./security/web-app-pentesting/SKILL.md)** — Identify OWASP Top 10 vulnerabilities (SQLi, XSS, CSRF) using Burp Suite and sqlmap.

### 📂 `engineering`

- **[`api-design-best-practices`](./engineering/api-design-best-practices/SKILL.md)** — Establish RESTful standard routing, cursor pagination, OpenAPI documentation, and idempotency structures.
- **[`auth-systems-oauth2`](./engineering/auth-systems-oauth2/SKILL.md)** — Establish OAuth2/OIDC flows, handle authorization codes with PKCE parameters, and decode cryptographically signed JWTs.
- **[`caching-strategies-redis`](./engineering/caching-strategies-redis/SKILL.md)** — Establish cache-aside pattern, cache invalidation, singleflight execution, and redis storage.
- **[`database-schema-design`](./engineering/database-schema-design/SKILL.md)** — Design relational databases indexes, manage schema migrations, and apply Row Level Security.
- **[`distributed-concurrency`](./engineering/distributed-concurrency/SKILL.md)** — Manage distributed locks with Redis (Redlock), prevent race conditions, and secure idempotent actions.
- **[`error-handling-resilience`](./engineering/error-handling-resilience/SKILL.md)** — Build structured exception hierarchies, exponential backoff retries, and fallback degradation models.
- **[`graphql-production-architecture`](./engineering/graphql-production-architecture/SKILL.md)** — Prevent GraphQL N+1 queries using DataLoader patterns and secure unified graph federation interfaces.
- **[`message-queues-kafka`](./engineering/message-queues-kafka/SKILL.md)** — Establish Kafka event pipelines, manage partition keys, and configure consumers offsets.
- **[`performance-profiling-python`](./engineering/performance-profiling-python/SKILL.md)** — Audit python performance bottlenecks using cProfile, memory_profiler, and py-spy runtime traces.
- **[`system-design-patterns`](./engineering/system-design-patterns/SKILL.md)** — Implement event-driven microservices architecture, circuit breakers, and rate limiters.

### 📂 `design`

- **[`color-theory-wcag`](./design/color-theory-wcag/SKILL.md)** — Validate WCAG contrast thresholds, apply semantic palettes, and scale dark colors range.
- **[`dark-mode-dynamic`](./design/dark-mode-dynamic/SKILL.md)** — Establish dynamic theme switching, OLED black styles, and state synchronization across layouts.
- **[`dashboard-data-visualization`](./design/dashboard-data-visualization/SKILL.md)** — Establish responsive KPI dashboards layouts, chart contrast guidelines, and grids structures.
- **[`design-system-architecture`](./design/design-system-architecture/SKILL.md)** — Build Figma-aligned component styles mapping, CSS utility classes, and web design tokens.
- **[`figma-to-code`](./design/figma-to-code/SKILL.md)** — Export Figma frame parameters to clean CSS utility frameworks without code pollution.
- **[`onboarding-flow-design`](./design/onboarding-flow-design/SKILL.md)** — Establish progressive onboarding paths, interactive interface overlays, and onboarding triggers.
- **[`premium-animations-motion`](./design/premium-animations-motion/SKILL.md)** — Design physics-backed spring animation systems and micro-interactions in Framer Motion.
- **[`rtl-arabic-persian-ui`](./design/rtl-arabic-persian-ui/SKILL.md)** — Configure RTL layout mirroring, Vazirmatn font-family, and Persian numbers formatting.
- **[`typography-scales`](./design/typography-scales/SKILL.md)** — Format modular font scaling grids, fluid typography calculations, and vertical layout rhythms.
- **[`ui-ux-design-principles`](./design/ui-ux-design-principles/SKILL.md)** — Apply visual contrast hierarchies, reduce users cognitive load, and construct smooth layouts flow.

### 📂 `threed`

- **[`3d-asset-optimization`](./threed/3d-asset-optimization/SKILL.md)** — Reduce polycount via decimation, bake high-poly normals, and optimize GLTF asset packages.
- **[`blender-geometry-nodes`](./threed/blender-geometry-nodes/SKILL.md)** — Construct procedural meshes generators using geometry nodes and custom vector inputs.
- **[`blender-procedural-modeling`](./threed/blender-procedural-modeling/SKILL.md)** — Generate meshes dynamically inside Blender using Python BPY commands scripts.
- **[`glsl-custom-shaders`](./threed/glsl-custom-shaders/SKILL.md)** — Write custom vertex and fragment GLSL shaders inside WebGL rendering loops.
- **[`pbr-texturing-workflow`](./threed/pbr-texturing-workflow/SKILL.md)** — Map metallic, roughness, and height parameters correctly inside PBR workflows.
- **[`procedural-generation-noise`](./threed/procedural-generation-noise/SKILL.md)** — Apply Perlin/Simplex noise generators to create terrains and procedural textures.
- **[`react-three-fiber-scenes`](./threed/react-three-fiber-scenes/SKILL.md)** — Structure 3D canvas environments, light sources, and shadow maps in React.
- **[`threejs-webgl-performance`](./threed/threejs-webgl-performance/SKILL.md)** — Reduce draw calls, compress textures sizes to KTX2, and implement instanced rendering in WebGL.
- **[`unity-mobile-optimization`](./threed/unity-mobile-optimization/SKILL.md)** — Audit Unity CPU cycles, draw calls overhead, and textures resolution profiles.
- **[`unreal-blueprints-basics`](./threed/unreal-blueprints-basics/SKILL.md)** — Build event graphs logic, configure actor communications, and pass variables in Unreal Engine.

### 📂 `ai-tools`

- **[`fine-tuning-local-llms`](./ai-tools/fine-tuning-local-llms/SKILL.md)** — Configure QLoRA hyperparameters, structure tokenized datasets, and run training pipelines.
- **[`llm-agent-architecture`](./ai-tools/llm-agent-architecture/SKILL.md)** — Establish ReAct workflows, tool schemas routing, and session history management.
- **[`llm-evaluation-monitoring`](./ai-tools/llm-evaluation-monitoring/SKILL.md)** — Audit LLM prompt latency, tokens utilization, and factual alignment scores (Ragas).
- **[`local-llm-deployment`](./ai-tools/local-llm-deployment/SKILL.md)** — Deploy Ollama / vLLM inference runners on custom GPU servers, configuring context limits.
- **[`mcp-server-development`](./ai-tools/mcp-server-development/SKILL.md)** — Deploy custom Model Context Protocol servers in Node or Python to extend LLM capabilities.
- **[`ocr-document-extraction`](./ai-tools/ocr-document-extraction/SKILL.md)** — Parse raw layout sections, extract structured tables, and format parsed PDF data.
- **[`prompt-engineering-patterns`](./ai-tools/prompt-engineering-patterns/SKILL.md)** — Build dynamic system instructions templates, few-shot examples configurations, and chain-of-thought routing.
- **[`rag-pipeline-building`](./ai-tools/rag-pipeline-building/SKILL.md)** — Design vector database chunking structures, semantic search systems, and cross-encoder rerankers.
- **[`semantic-search-vector`](./ai-tools/semantic-search-vector/SKILL.md)** — Index embeddings in Milvus/Qdrant databases, optimizing index parameters (HNSW/IVF-FLAT).
- **[`speech-stt-tts-pipelines`](./ai-tools/speech-stt-tts-pipelines/SKILL.md)** — Configure local Whisper transcription loops and generate voice streams using Edge-TTS.

### 📂 `creative`

- **[`anti-slop-prose-writing`](./creative/anti-slop-prose-writing/SKILL.md)** — Eliminate AI tells, em-dashes, sycophancy, listicle patterns, and delve/tapestry diction.
- **[`brand-storytelling-copy`](./creative/brand-storytelling-copy/SKILL.md)** — Formulate targeted brand value propositions using PAS and AIDA frameworks.
- **[`community-building-engagement`](./creative/community-building-engagement/SKILL.md)** — Set up Telegram rules hierarchies, organize online event schedules, and audit engagement metrics.
- **[`content-strategy-growth`](./creative/content-strategy-growth/SKILL.md)** — Analyze target audiences, plan editorial calendars, and map conversion journeys.
- **[`launch-strategy-campaign`](./creative/launch-strategy-campaign/SKILL.md)** — Coordinate multi-channel product launches (Reddit, ProductHunt) maximizing organic reach.
- **[`newsletter-engine-setup`](./creative/newsletter-engine-setup/SKILL.md)** — Design newsletter distribution campaigns, managing automated onboarding subscriber sequences.
- **[`pitch-deck-storytelling`](./creative/pitch-deck-storytelling/SKILL.md)** — Structure investor pitch decks slide layouts, presenting clear metrics and financials.
- **[`podcast-production-audio`](./creative/podcast-production-audio/SKILL.md)** — Apply noise reduction gates, configure dynamic EQ curves, and master audio tracks.
- **[`ux-writing-error-states`](./creative/ux-writing-error-states/SKILL.md)** — Structure helpful microcopy for empty states, payment failures, and validation blocks.
- **[`video-scriptwriting-pacing`](./creative/video-scriptwriting-pacing/SKILL.md)** — Write structured video storyboards utilizing Hook-Retention structures to optimize pacing.

### 📂 `general`

- **[`api-integration-resilience`](./general/api-integration-resilience/SKILL.md)** — Inject exponential backoff retries and circuit breaker decorators inside Python API client requests.
- **[`bash-script-automation`](./general/bash-script-automation/SKILL.md)** — Establish strict mode scripts, process traps, cleanup functions, and log styling.
- **[`cli-tool-ux-design`](./general/cli-tool-ux-design/SKILL.md)** — Integrate rich CLI rendering parameters (colored texts, dynamic progress bars, and inputs).
- **[`csv-excel-data-pipelines`](./general/csv-excel-data-pipelines/SKILL.md)** — Load and clean large CSV datasets in Python Pandas using chunked reading pipelines.
- **[`git-internals-recovery`](./general/git-internals-recovery/SKILL.md)** — Audit git reflog parameters, restore detached commits, and fix complex rebase merge blocks.
- **[`json-yaml-data-parsing`](./general/json-yaml-data-parsing/SKILL.md)** — Parse nested JSON arrays using jq, filter variables, and validate schema outputs.
- **[`markdown-documentation`](./general/markdown-documentation/SKILL.md)** — Configure static MKDocs sites containing interactive mermaid graphs mapping architectures.
- **[`python-automation-scripts`](./general/python-automation-scripts/SKILL.md)** — Build high-speed asynchronous utility scripts, configuring CLI outputs using the Click framework.
- **[`regex-mastery-parsing`](./general/regex-mastery-parsing/SKILL.md)** — Write assertions (lookahead/lookbehind), handle recursive groups, and prevent CPU regex backtracking blocks.
- **[`web-scraping-crawler`](./general/web-scraping-crawler/SKILL.md)** — Build distributed Scrapy crawlers, configure rotating user-agents, and bypass Cloudflare JS challenges.

