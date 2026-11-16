# monitor-km

Self-hosted [Umami](https://umami.is) analytics for the Kebun Makna game
(https://km.generasimerdeka.com). Dashboard: https://monitor-km.generasimerdeka.com

This repo is a snapshot of Umami's MIT-licensed source (not a fork) plus the files in `.github/`.
It holds no secrets. Hosting: Vercel Hobby (region sin1) + a free Supabase Postgres (Singapore).

## Files that are ours
- `.github/UPSTREAM_VERSION`: the Umami release currently in this repo.
- `.github/workflows/sync-upstream.yml`: daily; copies the latest stable release of the **same major version** into the repo (everything except `.git/` and `.github/`) and pushes to `main`, which triggers a Vercel production deploy. Migrations run in the build.
- `.github/workflows/keepalive.yml`: every 2 days; updates one row in the database so free Supabase does not pause the project.

## Vercel environment variables (Production only; values live in Vercel)
`DATABASE_URL` (transaction pooler, port 6543, `?pgbouncer=true`), `DIRECT_DATABASE_URL` (session pooler, port 5432, used for migrations),
`APP_SECRET`, `DISABLE_TELEMETRY=1`, `TRACKER_SCRIPT_NAME=kebun.js`, `COLLECT_API_ENDPOINT=/api/kebun`.
`TRACKER_SCRIPT_NAME` and `COLLECT_API_ENDPOINT` are read at build time; changing them needs a redeploy.

GitHub secret: `KEEPALIVE_DB_URL` (role `keepalive`, which can only touch `keepalive.beat`).

## Upgrading to a new major version (e.g. 4.x)
The sync workflow fails with a message when the major version changes. Then:
1. Read the Umami release notes (breaking changes, env vars, migrations).
2. Back up the database (Supabase dashboard) and test the new version elsewhere first if the notes mention migrations.
3. Edit `.github/UPSTREAM_VERSION` to the new tag in a manual sync (same clone + `rsync -a --delete --exclude .git/ --exclude .github/`), commit and push.

## Rotating secrets
- Database password: set a new password for the `umami` role in Supabase (SQL editor, `alter role umami password '...'`), update `DATABASE_URL` and `DIRECT_DATABASE_URL` in Vercel, redeploy.
- `keepalive` role: same, then update the `KEEPALIVE_DB_URL` GitHub secret.
- `APP_SECRET`: replace in Vercel and redeploy (this logs everyone out).
- Umami admin password: change it in the dashboard (Settings > Profile).
