# Amazon Hotbot

Amazon Hotbot is a small FastAPI service that collects public Amazon seller,
ads, policy, and SP-API updates, deduplicates them into event clusters, scores
their heat, and publishes a daily Top 10 digest to a Feishu custom bot.

## Quick Start

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
python -m amazon_hotbot.cli run-once --dry-run
python -m amazon_hotbot.cli serve
```

Open http://127.0.0.1:8000 after starting the server.

## Configuration

Set secrets in `.env`; do not commit them.

- `FEISHU_WEBHOOK_URL`: Feishu custom bot webhook.
- `OPENAI_API_KEY`: optional OpenAI-compatible API key for better summaries.
- `OPENAI_BASE_URL`: optional, defaults to `https://api.openai.com/v1`.
- `OPENAI_MODEL`: optional, defaults to `gpt-4.1-mini`.
- `DATABASE_URL`: optional SQLite path, defaults to `data/hotbot.sqlite3`.
- `DAILY_SEND_TIME`: defaults to `09:00`.
- `TIMEZONE`: defaults to `Asia/Shanghai`.
- `DRY_RUN`: defaults to `true`.

## Commands

```powershell
python -m amazon_hotbot.cli fetch
python -m amazon_hotbot.cli process
python -m amazon_hotbot.cli notify --dry-run
python -m amazon_hotbot.cli run-once --dry-run
python -m amazon_hotbot.cli serve
```

## Deploy to Vercel

This repository is prepared for Vercel Python Serverless deployment with:

- `api/index.py`
- `vercel.json`
- `.vercelignore`

Recommended deployment flow:

1. Push this folder to GitHub, GitLab, or Bitbucket.
2. Import the repository in Vercel.
3. Set environment variables in the Vercel project:
   - `FEISHU_WEBHOOK_URL`
   - `OPENAI_API_KEY` if LLM summaries are needed
   - `OPENAI_BASE_URL=https://api.openai.com/v1`
   - `OPENAI_MODEL=gpt-4.1-mini`
   - `DATABASE_URL=/tmp/hotbot.sqlite3`
   - `DAILY_SEND_TIME=09:00`
   - `TIMEZONE=Asia/Shanghai`
   - `DRY_RUN=false`
4. Deploy.

Vercel notes:

- Vercel Serverless cannot keep the local `APScheduler` process running. The included Vercel Cron route calls `/api/run/once?dry_run=false` at `0 1 * * *`, which is 09:00 in Asia/Shanghai.
- `/tmp/hotbot.sqlite3` is suitable for a demo but is ephemeral. Use a managed database such as Turso, Neon, Supabase, or another hosted database for durable production history.
- `.env`, `.env.local`, and local data folders are excluded from deployment by `.vercelignore`.

## API

- `GET /api/health`
- `GET /api/items`
- `GET /api/clusters`
- `GET /api/clusters/{id}`
- `GET /api/daily/latest`

The first implementation uses public sources only. Seller Central login sources
are intentionally left as future adapters so no Amazon account credentials are
stored.
