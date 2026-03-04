# pre-push — Claude Code 스킬

> 한국어 | **[English](README.md)**

Claude Code용 필수 프리-푸시 보안 및 품질 파이프라인입니다. 푸시 요청 시 자동으로 실행되어 위험한 코드가 원격 저장소에 올라가는 것을 차단합니다.

## 최적화된 기술 스택

**모던 TypeScript 풀스택 웹 개발**에 최적화되어 있습니다:

| 레이어 | 기술 |
|--------|------|
| **프론트엔드** | React, Next.js (App Router), TypeScript, Tailwind CSS |
| **백엔드** | Next.js API Routes, Node.js, REST / tRPC |
| **데이터베이스** | PostgreSQL, Supabase (RLS, 마이그레이션, Prisma) |
| **인프라** | Docker, Docker Compose, Terraform, nginx |
| **패키지 매니저** | npm, yarn, pnpm |
| **CI/CD** | GitHub Actions, Vercel, Railway |

> 어떤 git 프로젝트에도 동작합니다. Python, Go, Ruby 프로젝트도 시크릿 스캔과 공급망 체크의 혜택을 받을 수 있습니다. 빌드/테스트 및 리뷰 에이전트만 TypeScript 특화입니다.

## 설치

```bash
git clone https://github.com/coinangel-kr/claude-pre-push-skill.git ~/.claude/skills/pre-push
```

끝입니다. Claude Code는 `~/.claude/skills/` 폴더를 자동으로 인식합니다 — 재시작 불필요.

### 설치 후 첫 번째 푸시

첫 푸시 시 스킬이 번들된 리뷰 에이전트 6개를 `~/.claude/agents/`에 자동 설치합니다:

```
ℹ️  Installed 6 agent(s) to ~/.claude/agents/. Restart Claude Code to activate them, then re-run your push command.
```

Claude Code를 한 번 재시작한 뒤 다시 푸시하면 이후부터는 완전 자동으로 동작합니다.

## 동작 방식

`git push` 요청마다 8단계 파이프라인을 실행합니다:

| 단계 | 내용 |
|------|------|
| **Setup** | 첫 사용 시 번들 에이전트 자동 설치 |
| **1. 평가 & 스캔** | 시크릿 스캐너 실행 + 브랜치/diff 메타데이터 수집 (단일 bash 호출) |
| **2. 라우팅** | 시크릿 발견 시 차단; 문서만 변경된 경우 빠른 종료 |
| **3. 공급망 체크** | 신규 의존성 목록화, 오타 스쿼팅 의심 패키지 플래그 (경고만) |
| **4. 빌드 & 테스트** | `npm run build` + `npm test` 실행 — 소스 파일 변경 시에만 |
| **5. quick-validator** | TypeScript 타입 체크 + ESLint — diff 50줄 미만이면 스킵 |
| **6. 리뷰 에이전트** | 병렬 리뷰: code-reviewer 항상 실행, 나머지는 조건부 트리거 |
| **7. 게이트 체크** | Critical/High → 차단; Medium → 경고; Low → 리포트 |
| **8. 리포트 & 푸시** | 단계별 소요 시간 포함 요약 리포트 출력 후 푸시 |

## 번들 에이전트

스킬에 포함된 전문 에이전트 6개가 `~/.claude/agents/`에 자동 설치됩니다:

| 에이전트 | 모델 | 역할 |
|---------|------|------|
| `code-reviewer` | Sonnet | 코드 품질, 보안 패턴, React 베스트 프랙티스 |
| `security-reviewer` | **Opus** | OWASP Top 10, 인증 로직, Injection, SSRF, 레이트 리밋 |
| `database-reviewer` | Sonnet | PostgreSQL, Supabase RLS, 쿼리 최적화, 스키마 설계 |
| `quick-validator` | **Haiku** | TypeScript 타입 체크 + ESLint 전용 — 빠르고 저렴 |
| `refactor-cleaner` | Sonnet | 데드 코드, 미사용 import, 중복 컴포넌트 제거 |
| `build-error-resolver` | Sonnet | 최소 변경으로 TypeScript/빌드 에러 수정 |

> 이 에이전트들은 스탠드얼론으로도 유용합니다. 설치 후 Claude가 모든 프로젝트에서 자동으로 활용합니다.

## 탐지 패턴

Perl 스캐너가 12가지 패턴을 검사합니다 — **추가된 라인(`+`)만** 검사하므로 시크릿 제거 커밋은 차단되지 않습니다:

| 패턴 | 심각도 |
|------|--------|
| AWS Access Key ID (`AKIA...`) | 🚨 Critical |
| 개인 키 (`-----BEGIN ... PRIVATE KEY-----`) | 🚨 Critical |
| 연결 문자열 내 비밀번호 (`://user:pass@host`) | 🚨 Critical |
| 플랫폼 토큰 (GitHub 6종, Slack, Stripe live) | 🚨 Critical |
| Dockerfile `ENV` 시크릿 | 🚨 Critical |
| Google / Gemini API 키 (`AIza...`) | 🚨 Critical |
| LLM API 키 (Anthropic, OpenAI, HuggingFace, Replicate, Groq) | 🚨 Critical |
| Azure Storage / SAS / 연결 문자열 | 🚨 Critical |
| 하드코딩된 자격증명 — 따옴표 값 | ⚠️ High |
| 하드코딩된 자격증명 — 따옴표 없는 YAML/ENV 값 | ⚠️ High |
| npm 레지스트리 인증 토큰 (`.npmrc`) | ⚠️ High |
| 미해결 머지 컨플릭트 마커 | 🚨 Critical |

## security-reviewer 트리거 조건

다음 중 하나라도 해당하면 `security-reviewer` (Opus)가 실행됩니다:

- API 라우트: `src/app/api/**`, `**/routes/**`, `**/controllers/**`
- 인증/미들웨어: `**/auth*`, `**/middleware*`, `**/guard*`, `**/permission*`
- 설정/시크릿: `**/.env*`, `**/config*`, `**/secrets*`
- 인프라: `Dockerfile`, `docker-compose*.yml`, `*.tf`, `nginx.conf`
- diff 내 위험 패턴: `child_process`, `exec(`, `eval(`, `dangerouslySetInnerHTML`
- 민감한 파일명 키워드: `secret`, `token`, `password`, `key`, `credential`
- `package.json`에 신규 패키지 추가

## 파이프라인 우회

"**skip review**" 또는 "**force push**"라고 말하면 파이프라인 전체를 건너뛰고 즉시 푸시합니다. Claude가 경고 메시지를 출력합니다.

## 요구사항

- **Claude Code** (서브에이전트 지원 필요)
- **Perl** — macOS 및 대부분의 Linux에 기본 설치됨
- **npm** — 선택사항, 빌드/테스트 단계에서 사용
- **Git**

## 업데이트

```bash
cd ~/.claude/skills/pre-push && git pull
```

## 라이선스

MIT — [LICENSE.txt](LICENSE.txt) 참고

## 버전

v2.0.0 — by [coinangel](https://github.com/coinangel-kr)
