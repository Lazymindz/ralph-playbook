<!-- cSpell:disable -->

# Sandbox Environments for AI Agent Workflows

_Security model:_ The sandbox (Docker/E2B) provides the security boundary. Inside the sandbox, Claude runs with full permissions because the container itself is isolated.

_Security philosophy:_

> "It's not if it gets popped, it's when it gets popped. And what is the blast radius?"

Run on dedicated VMs or local Docker sandboxes. Restrict network connectivity, provide only necessary credentials, and ensure no access to private data beyond what the task requires.

---

## Options

### 1. E2B

- purpose-built for AI agents and LLM workflows
- pre-built template `anthropic-claude-code` ships with Claude Code CLI ready
- single-line SDK calls in Python or JavaScript
- full filesystem + git for progress.txt, prd.json, and repo operations
- 24-hour session limits on Pro plan

_Pricing:_

- Hobby: Free ($0/month + usage costs) + $100 one-time usage-cost credit
- Pro: $150/month + usage costs (24-hour session limits)
- Usage:
  - $0.000028/s for 2 vCPU
  - $0.0000045/GiB/s for memory

_Specs:_

- Firecracker microVM isolation (~150ms cold start)
- Timeout: Up to 24 hours on Pro plan
- Full Linux filesystem with git support
- Node.js, curl, ripgrep pre-installed

_Links:_

- https://e2b.dev/
- https://e2b.dev/docs
- https://e2b.dev/blog/python-guide-run-claude-code-in-an-e2b-sandbox

---

### 2. Modal

_Pricing:_

- Starter: Free ($0/month + usage costs) + $30/mo usage-cost credit
- Usage:
  - $0.00003942/s for 2 vCPU
  - $0.00000672/GiB/s for memory

_Specs:_

- gVisor containers (2-5s cold start when idle)
- timeout: Up to 24 hours configurable
- ephemeral filesystem + optional Volumes for persistence

_Links:_

- https://modal.com/products/sandboxes
- https://modal.com/pricing

---

### 3. Cloudflare Sandboxes

- in beta
- best if already in Cloudflare ecosystem
- edge-native (200+ global locations)
- pay for active CPU only

_Limitations:_

- Ephemeral disks only (reset on sleep/restart)
- No persistent disk support currently
- Newer offering, still maturing

_Links:_

- https://developers.cloudflare.com/sandbox/
- https://developers.cloudflare.com/containers/pricing/

---

## Comparison Table

| Feature          | E2B                 | Modal            | Cloudflare       |
| ---------------- | ------------------- | ---------------- | ---------------- |
| Setup            | Very Easy           | Easy             | Easy             |
| Free Tier        | $100 credit         | $30/month        | Workers Standard |
| Isolation        | Firecracker microVM | gVisor container | Container        |
| Cold Start       | ~150ms              | 2-5 seconds      | Sub-second       |
| Max Timeout      | 24 hours (Pro)      | 24 hours         | Configurable     |
| Claude CLI       | Prebuilt template   | Manual           | Manual           |
| Git Support      | Yes                 | Yes              | Yes              |
| Persistent Files | 24 hours            | Via Volumes      | No               |
| Best For         | AI agent loops      | ML workloads     | Edge apps        |

---

## Other Options

### Daytona

- Sub-90ms sandbox creation
- $200 free compute
- Python/TypeScript SDK
- Good E2B alternative
- https://www.daytona.io/

### Fly.io (Sprites)

- Firecracker VM isolation
- 60-second timeout limit (restrictive for Ralph)
- Better for short-lived tasks

### Google Cloud Run

- Two-layer sandbox isolation
- More complex setup
- https://docs.cloud.google.com/run/docs/code-execution

### Replit

- Full development environment
- Effort-based pricing ($0.25+/checkpoint)
- Better for interactive dev, not autonomous loops

---

## Local Docker Options

### Docker Official Sandboxes

_Quick Start:_

```bash
docker sandbox run claude                  # Basic
docker sandbox run -w ~/my-project claude  # Custom workspace
docker sandbox run claude "your task"      # With prompt
docker sandbox run claude -c               # Continue last session
```

_Key Details:_

- Credentials stored in persistent volume `docker-claude-sandbox-data`
- `--dangerously-skip-permissions` enabled by default
- Base image includes: Node.js, Python 3, Go, Git, Docker CLI, GitHub CLI, ripgrep, jq
- Container persists in background; re-running reuses same container
- Non-root user with sudo access

_Links:_ https://docs.docker.com/ai/sandboxes/claude-code/

---

## Comparison: E2B vs Docker Local

| Aspect            | E2B (Cloud)         | Docker Local           |
| ----------------- | ------------------- | ---------------------- |
| Setup             | SDK call            | `docker sandbox run`   |
| Isolation         | Firecracker microVM | Container              |
| Cost              | ~$0.05/hr           | Free (your hardware)   |
| Max Duration      | 24 hours            | Unlimited              |
| Network           | Full internet       | Full internet          |
| State Persistence | Session-based       | Volume-based           |
| Multi-tenant Safe | Yes                 | No (local only)        |
| Best For          | Production, CI/CD   | Local dev, prototyping |

---

## Recommendation for This Project

### For Production/Multi-tenant: Use E2B

1. Pre-built Claude Code template = zero setup friction
2. 24-hour sessions handle long-running autonomous agents
3. Full filesystem for progress.txt, prd.json, git repos
4. Proven in production (Lovable, Quora use it)
5. True isolation (Firecracker microVM)

### For Local Development: Use Docker Sandboxes

1. _Quick prototyping_: `docker sandbox run claude`
2. _With git automation_: `claude-sandbox` (TextCortex)
3. _Minimal setup_: Uses persistent credentials volume
4. Free - runs on your own hardware
5. Unlimited session duration
