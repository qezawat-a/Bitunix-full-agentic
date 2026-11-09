# J+ROCK — Agentic Trading Console

A full agentic AI assistant fused with a Bitunix futures trading engine:

- **Agentic assistant** with real function-calling tools (filesystem, terminal, engine control, GitHub push), a slash-command system (`/help /models /set_model /connect_provider /add_mcp /mcp /add_skill /skill /add_soul /soul /think /memory /code_review /bug_fix /verify /simplify /team_spawn /check_ai /check_system /autopsy /diagnose /push`), soul persona layer, durable memory, skill registry, MCP server registry, and an honesty-first policy (never fabricates prices, results, or permissions).
- **AutoSetModelByKey** — paste any provider key and it discovers which models that key can actually run (OpenAI-compatible, Anthropic, Gemini adapters included), auto-picks the strongest, and live-probes it before saving.
- **Trading engine** with 16 strategies (Vegas tunnels, golden/death cross, VWAP, hammer/morning-star/shooting-star/dark-cloud, Bollinger breakout, mid-band MACD, MA200+MACD, TEMA×VWAP, DEMA55, three white soldiers), multi-timeframe voting, agent-computed TP/SL (liquidation-aware stop, confidence-scaled take-profit, breakeven + trailing), reversal logic, and SIM/LIVE modes on the Bitunix futures API (double-SHA256 signed).

> Truth policy: SIM results are always labeled. Without a Bitunix key the engine paper-trades on real public market data; if the exchange is unreachable it falls back to a clearly-flagged offline-sim feed.

---

## 1. Requirements

- **Node.js ≥ 20**
- **PostgreSQL ≥ 14** (local, Docker, or hosted: Neon / Supabase / Railway / Vercel Postgres)

## 2. Run locally

```bash
git clone <your-repo-url> jrock && cd jrock
npm install

# database
createdb app_db                    # or: docker run -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:16
cp .env.example .env               # then edit DATABASE_URL
npx drizzle-kit push               # create all tables

npm run dev                        # dev mode → http://localhost:3000
# or production:
npm run build && npm run start
```

## 3. Environment variables (`.env`)

| Variable | Purpose | Required |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | ✅ |
| `AGENT_API_KEY` / `AGENT_BASE_URL` / `AGENT_MODEL` | LLM provider (auto-seeded into Settings on first boot) | optional — set later in UI |
| `AGENT_PROVIDER` | `openai` \| `anthropic` \| `gemini` | optional |
| `BITUNIX_API_KEY` / `BITUNIX_SECRET_KEY` | Bitunix futures keys — enables LIVE mode | optional (SIM works without) |
| `GITHUB_TOKEN` / `GITHUB_REPO` | PAT (`repo` scope) + `owner/repo` for `/push` | optional |
| `CRON_SECRET` | protects the cron tick endpoint | recommended in production |

Keys can also be entered at runtime in **Settings** — they are stored server-side in the database and always masked in API responses.

## 4. Deploy

### A. Vercel (easiest)

1. Push this repo to GitHub (use the built-in `/push` or Settings → GitHub, or `git push`).
2. Import in Vercel → add env var `DATABASE_URL` (Neon serverless Postgres is a one-click fit).
3. Run once locally or via CI: `npx drizzle-kit push` against that database.
4. Deploy. **Engine ticks on serverless:** the dashboard drives ticks while open; for 24/7 scanning add Vercel Cron (included `vercel.json` does this every minute):
   ```json
   { "crons": [{ "path": "/api/engine?tick=1", "schedule": "* * * * *" }] }
   ```
   Set `CRON_SECRET` and Vercel sends it as a bearer token automatically. Raise the schedule interval if your plan limits cron frequency.

### B. VPS / Node server (best for full-time trading)

```bash
git clone <repo> && cd jrock
npm install && cp .env.example .env   # fill DATABASE_URL (+ exchange keys for LIVE)
npx drizzle-kit push
npm run build
pm2 start npm --name jrock -- run start   # or: systemd unit running `npm run start`
```

The browser console drives engine ticks; to also tick when nobody is watching, add:

```bash
* * * * * curl -s -H "Authorization: Bearer $CRON_SECRET" https://your-domain/api/engine?tick=1 >/dev/null
```

(or a node cron / systemd timer hitting the same endpoint as fast as every 5–15s).

### C. Docker

```bash
docker build -t jrock . 2>/dev/null || cat > Dockerfile <<'EOF'
FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm","run","start"]
EOF
docker build -t jrock .
docker run -p 3000:3000 --env-file .env jrock
```

### D. Railway / Render / Fly

Standard Next.js: build `npm run build`, start `npm run start`, attach a managed Postgres, set env vars, run `npx drizzle-kit push` once, then point a cron job at `/api/engine?tick=1`.

## 5. First run — 3 quick wins

1. **Connect a brain:** Settings → Provider → paste any API key → **PROBE + CONNECT (AutoSet)**. Or in chat: `/connect_provider https://api.openai.com/v1 sk-...` then `/models`, `/set_model auto`.
2. **Start the trader (paper):** type `engine on`, then `start trading`. Watch signals, positions with agent-set TP/SL, the equity curve, and `pnl`. Tune with the Quick Settings grid or commands (`min confidence 75`, `symbols BTCUSDT SOLUSDT`, `timeframes 5m,15m`, `trailing trigger 45`…).
3. **Go LIVE (optional, real money):** Settings → Exchange → paste Bitunix futures key+secret → `GO LIVE`. The engine refuses live mode without keys; margin per live order is conservatively capped. **Trading futures can lose money — start in SIM.**

## 6. Command cheat sheet

Everything works as plain language in chat (`status`, `pnl`, `scan now`, `close position all`) or as slash commands. Full manual: `/help`. Diagnostics: `/check_system` (DB, exchange feed, engine, permissions), `/autopsy` (losing-trade post-mortem), `/diagnose` (full report).

## 7. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Dashboard says `OFFLINE FEED` | Exchange unreachable from the host ( egress blocked / region ). Engine falls back to a labeled synthetic feed. |
| `/models` shows fallback list | Key lacks `/models` permission; pick from the curated list and let the live probe validate. |
| `/verify` refuses | Terminal access is OFF by design → Settings → Permissions. |
| `/push` 404/401 | PAT needs classic `repo` scope; repo is auto-created if missing. |
| DB errors on boot | Re-run `npx drizzle-kit push` with the correct `DATABASE_URL`. |

## 8. Structure

```
src/
  app/                      Next.js App Router (console UI + /api routes)
  app/api/chat|engine|config|models|github
  components/               Chat (command palette), Dashboard, Settings, Markdown
  db/schema.ts              sessions, messages, config, memories, skills, mcps,
                            trading_state, positions, trades, tpsl_events, signals, …
  lib/agent/                providers (3 adapters + AutoSetModelByKey), commands,
                            tools (function calling), agent loop, github, config
  lib/trading/              indicators, 16 strategies, bitunix client, engine core
```
