# pre-push — Claude Code Skill

> **[한국어](README.ko.md)** | English

A mandatory pre-push security and quality pipeline for Claude Code. Runs automatically whenever you ask Claude to push, and blocks dangerous code from reaching remote.

## Tech Stack

Optimized for **modern TypeScript full-stack web development**:

| Layer | Technologies |
|-------|-------------|
| **Frontend** | React, Next.js (App Router), TypeScript, Tailwind CSS |
| **Backend** | Next.js API Routes, Node.js, REST / tRPC |
| **Database** | PostgreSQL, Supabase (RLS, migrations, Prisma) |
| **Infrastructure** | Docker, Docker Compose, Terraform, nginx |
| **Package manager** | npm, yarn, pnpm |
| **CI/CD** | GitHub Actions, Vercel, Railway |

> Works for any git project. Python, Go, or Ruby projects still benefit from the secrets scan and supply chain checks — only the build/test and review agents are TypeScript-specific.

## Installation

```bash
git clone https://github.com/coinangel-kr/claude-pre-push-skill.git ~/.claude/skills/pre-push && cp ~/.claude/skills/pre-push/agents/*.md ~/.claude/agents/
```

Then **restart Claude Code once** to load the agents. That's it — fully functional from the next session.

## Update

```bash
cd ~/.claude/skills/pre-push && git pull && cp agents/*.md ~/.claude/agents/
```

## How it works

Every `git push` request goes through an 8-step pipeline:

| Step | What happens |
|------|-------------|
| **Setup** | Auto-installs bundled agents on first use |
| **1. Assess & Scan** | Runs secrets scanner + collects branch/diff metadata in a single bash call |
| **2. Routing** | Blocks on secrets; fast-exits for docs-only changes |
| **3. Supply chain** | Lists new dependencies, flags potential typosquats (warn only) |
| **4. Build & test** | Runs `npm run build` + `npm test` — only when source files changed |
| **5. quick-validator** | TypeScript type check + ESLint — skipped for diffs < 50 lines |
| **6. Review agents** | Parallel review: code-reviewer always; others triggered conditionally |
| **7. Gate check** | Critical/High → block; Medium → warn; Low → report |
| **8. Report & push** | Timestamped summary with elapsed time per step |

## Bundled Agents

6 specialized agents ship with the skill and install automatically to `~/.claude/agents/`:

| Agent | Model | Role |
|-------|-------|------|
| `code-reviewer` | Sonnet | Code quality, security patterns, React best practices |
| `security-reviewer` | **Opus** | OWASP Top 10, auth logic, injection, SSRF, rate limiting |
| `database-reviewer` | Sonnet | PostgreSQL, Supabase RLS, query optimization, schema design |
| `quick-validator` | **Haiku** | TypeScript type check + ESLint only — fast and cheap |
| `refactor-cleaner` | Sonnet | Dead code, unused imports, duplicate components |
| `build-error-resolver` | Sonnet | TypeScript/build errors with minimal diff fixes |

> These agents are also useful standalone. Once installed, Claude uses them proactively across all your projects.

## Security patterns detected

The Perl scanner checks 12 patterns — only on **added lines** (removing a secret never triggers a block):

| Pattern | Severity |
|---------|----------|
| AWS Access Key ID (`AKIA...`) | 🚨 Critical |
| Private key (`-----BEGIN ... PRIVATE KEY-----`) | 🚨 Critical |
| Password in connection string (`://user:pass@host`) | 🚨 Critical |
| Platform token (GitHub 6 types, Slack, Stripe live) | 🚨 Critical |
| Dockerfile `ENV` secret | 🚨 Critical |
| Google / Gemini API key (`AIza...`) | 🚨 Critical |
| LLM API key (Anthropic, OpenAI, HuggingFace, Replicate, Groq) | 🚨 Critical |
| Azure Storage / SAS / connection string | 🚨 Critical |
| Hardcoded credential assignment — quoted value | ⚠️ High |
| Hardcoded credential assignment — unquoted YAML/ENV | ⚠️ High |
| npm registry auth token (`.npmrc`) | ⚠️ High |
| Unresolved merge conflict markers | 🚨 Critical |

## Security reviewer trigger conditions

`security-reviewer` (Opus) runs when any of the following match:

- API route files: `src/app/api/**`, `**/routes/**`, `**/controllers/**`
- Auth/middleware: `**/auth*`, `**/middleware*`, `**/guard*`, `**/permission*`
- Config/secrets: `**/.env*`, `**/config*`, `**/secrets*`
- Infrastructure: `Dockerfile`, `docker-compose*.yml`, `*.tf`, `nginx.conf`
- Dangerous patterns in diff: `child_process`, `exec(`, `eval(`, `dangerouslySetInnerHTML`
- Sensitive filename keywords: `secret`, `token`, `password`, `key`, `credential`
- New packages added in `package.json`

## Override

Say **"skip review"** or **"force push"** to bypass the entire pipeline. Claude prints a warning and pushes immediately.

## Requirements

- **Claude Code** with subagent support
- **Perl** — pre-installed on macOS and most Linux distributions
- **npm** — optional, for build/test steps
- **Git**

## License

MIT — see [LICENSE.txt](LICENSE.txt)

## Version

v2.0.0 — by [coinangel](https://github.com/coinangel-kr)
