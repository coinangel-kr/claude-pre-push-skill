# pre-push — Claude Code Skill

A mandatory pre-push security and quality pipeline for Claude Code. Runs automatically whenever you ask Claude to `git push` and blocks dangerous code from reaching remote.

## Tech Stack

This skill is **optimized for modern TypeScript full-stack web development**:

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React, Next.js (App Router), TypeScript, Tailwind CSS |
| **Backend** | Next.js API Routes, Node.js, REST / tRPC |
| **Database** | PostgreSQL, Supabase (RLS, migrations, Prisma) |
| **Infrastructure** | Docker, Docker Compose, Terraform, nginx |
| **Package manager** | npm, yarn, pnpm |
| **CI/CD** | GitHub Actions, Vercel, Railway |

> Works for any git project, but the review agents have deep expertise in the above stack. Python, Go, or Ruby projects will still benefit from the secrets scan and supply chain checks.

## What it does

Every push goes through an 8-step pipeline:

1. **Agent setup** — auto-installs 6 bundled review agents to `~/.claude/agents/` on first use
2. **Secrets scan** (deterministic, blocking) — 12 patterns covering AWS/GCP/Azure/LLM keys, private keys, connection strings, platform tokens, and merge conflict markers
3. **Routing** — blocks on secrets, fast-exits on docs-only changes
4. **Supply chain check** — lists new dependencies, flags potential typosquats
5. **Build & test** — runs `npm run build` and `npm test` only when source files changed
6. **quick-validator** — TypeScript type check + ESLint (skipped for tiny diffs < 50 lines)
7. **Parallel review agents** — code-reviewer always; security-reviewer, database-reviewer, refactor-cleaner triggered conditionally
8. **Gate check** — Critical/High block push; Medium/Low warn

## Bundled Agents

The skill ships with 6 specialized review agents that get installed automatically:

| Agent | Model | Specialization |
|-------|-------|---------------|
| `code-reviewer` | Sonnet | Code quality, security, performance, React patterns |
| `security-reviewer` | **Opus** | OWASP Top 10, auth, injection, SSRF, rate limiting |
| `database-reviewer` | Sonnet | PostgreSQL, Supabase RLS, query optimization, schema design |
| `quick-validator` | **Haiku** | TypeScript type check + ESLint only (fast & cheap) |
| `refactor-cleaner` | Sonnet | Dead code, unused imports, duplicate components |
| `build-error-resolver` | Sonnet | TypeScript/build errors with minimal diff fixes |

> These agents are also useful standalone — once installed, Claude will use them proactively across all your projects.

## Security patterns detected

| Pattern | Severity |
|---------|---------|
| AWS Access Key ID (`AKIA...`) | Critical |
| Private key (`-----BEGIN ... PRIVATE KEY-----`) | Critical |
| Password in connection string | Critical |
| Platform token (GitHub 6 types, Slack, Stripe live) | Critical |
| Dockerfile ENV secret | Critical |
| Google/Gemini API key (`AIza...`) | Critical |
| LLM API key (Anthropic, OpenAI, HuggingFace, Replicate, Groq) | Critical |
| Azure Storage/SAS/connection string | Critical |
| Hardcoded credential assignment — quoted value | High |
| Hardcoded credential assignment — unquoted YAML/ENV | High |
| npm registry auth token | High |
| Unresolved merge conflict markers | Critical |

**Key design decision**: scanner only inspects `+` (added) lines — removing a secret is never blocked.

## Installation

```bash
# Via Claude Code skill install (recommended)
claude skill install pre-push.skill

# Or manual
cp -r pre-push/ ~/.claude/skills/pre-push/
```

Then on your first push, Claude will auto-install the bundled agents and prompt you to restart Claude Code once.

## Requirements

- **Claude Code** with subagent support
- **Perl** — pre-installed on macOS and most Linux distributions
- **npm** — for build/test steps (optional, skipped if no `package.json`)
- **Git** — obviously

## Override

Say **"skip review"** or **"force push"** to bypass the entire pipeline (prints a warning and pushes immediately).

## License

MIT — see [LICENSE.txt](LICENSE.txt)

## Version

v2.0.0 — by [coinangel](https://github.com/coinangel)
