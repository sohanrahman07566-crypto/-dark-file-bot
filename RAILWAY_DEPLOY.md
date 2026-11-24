# Railway Deployment

This project is a single Node.js web service: it serves the React admin panel and API and starts the Telegram bot polling/scheduled jobs in the same process. Deploy it as a Railway **Service** (not a separate worker).

## Railway settings

The repository includes `railway.json`, which configures Nixpacks, build, start, health check, and restart policy. If setting commands manually, use:

- Builder: Nixpacks
- Build command: `npm install --legacy-peer-deps --no-audit --no-fund && npm run build`
- Start command: `npm start`
- Health check path: `/api/health`
- Node: `22.12.0` or newer (the package declares `engines.node >=22.12.0`)

Do not use `npm run dev` in production. Railway supplies `PORT`; the server binds to `0.0.0.0` and reads that variable.

## Required environment variables

Set these in Railway -> Service -> Variables:

- `NODE_ENV=production`
- `BOT_TOKEN` = Telegram bot token from BotFather
- `ADMIN_ID` = Telegram numeric user ID of the owner/admin
- `ADMIN_NAME` = admin login name (if used by the login configuration)
- `ADMIN_PASSWORD` = a strong admin-panel password
- `ADMIN_EMAIL` = admin email (if used by the project)
- `SESSION_SECRET` = a long random secret (at least 32 characters)

The project has a default Firebase Realtime Database URL in its storage code. If using a different Firebase database, set `FIREBASE_DATABASE_URL`. If the database rules require privileged access, configure the appropriate `FIREBASE_DATABASE_SECRET` or `FIREBASE_AUTH_TOKEN` securely in Railway Variables; never commit these secrets.

Optional:

- `GEMINI_API_KEY` = only if using Gemini-powered features
- `DATA_FILE_PATH=/app/.data/store.json` = local fallback data path. For local-file persistence, add a Railway Volume mounted at `/app/.data`. Without a volume, Railway's container filesystem is ephemeral across redeploys/restarts. Prefer the configured Firebase cloud persistence for durable app data.

## Deploy

1. Push this project to a GitHub repository.
2. In Railway, create a project -> Deploy from GitHub Repo and select the repository.
3. Add the environment variables above.
4. Deploy. Railway will read `railway.json` automatically.
5. In Service -> Settings -> Networking, generate a public domain. Open that URL to access the admin panel.
6. Check `https://YOUR-DOMAIN/api/health`. It should return JSON with `status: "ok"`.

## Notes

- Keep exactly one live instance/replica unless you have verified the project's bot singleton/locking behavior with a shared lock backend. Multiple replicas can cause Telegram polling conflicts.
- Do not set a fixed `PORT`; Railway injects it.
- Do not add Render-only environment variables or use Render commands.
- A successful image build does not guarantee Telegram connectivity: verify the bot token, admin credentials, Firebase rules/credentials, and Railway runtime logs after deployment.
