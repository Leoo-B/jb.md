# Home Workspace — Termux (Android)

Bukan satu repo: `$HOME` adalah workspace multi-proyek + tempat Hermes Agent
(`~/.hermes`) berjalan. Mayoritas aksi = administrasi server & gateway, bukan
build satu kode.

## Environment

- Termux on Android. Node `v26.4.0` via `node -v`, Python `3.14.6`.
- `~/.bashrc` memuat Railway env, bun PATH, alias `hermes`
  (`hermes -m gendas-combos --provider Leoo-B`), dan memanggil
  `~/start-services.sh` di login shell.
- Tidak ada `pnpm` di PATH — monorepo butuh `corepack enable` dulu.
- Bahasa kerja: Indonesia. Komit & dokumen memakai konvensi
  `feat:`/`fix:`/`chore:`/`test:` (lihat `git log` tiap repo).

## Proyek

| Path | Stack | Perintah |
|---|---|---|
| `~/neostudio` | Next.js 14 App Router (JS, bukan TS), React 18, Tailwind 3.4 | `npm run dev` (port 3000), `npm run build`, `npm run lint` |
| `~/monorepo` | pnpm 9 workspace: `apps/web` (Vite+React+TS), `apps/api` (Hono+Wrangler), `packages/shared` (Zod) | `pnpm install`; `pnpm --filter @neostudio/api dev` (8787); `pnpm --filter @neostudio/web dev` (5173); `pnpm typecheck`; `pnpm --filter @neostudio/web lint`; `pnpm --filter @neostudio/shared test`; `pnpm --filter @neostudio/api test` |
| `~/Leoo-Tools` | HTML/CSS/JS statis + `api/*.js` | buka `index.html` langsung; `requirements.txt` |
| `~/bansos-watch` | Python (cron watcher Discord) | jalan via Hermes cronjob, bukan manual |
| `~/pitcoin` | Python bot + watchdog | `bash ~/pitcoin/run.sh`; `autokeep.sh` dipanggil watchdog |
| `~/camofox-browser`, `~/impeccable`, `~/ui-ux-pro-max-skill` | third-party/skill source | jangan edit sembarangan |

`neostudio` lama (JS) dan `monorepo` (TS rewrite) sama-sama repo
`Leoo-B/neostudio` di branch berbeda — cek `git branch` sebelum push.

## Layanan & infrastruktur

`~/start-services.sh` (dijalankan otomatis `.bashrc`):
`termux-wake-lock` → `9router --tray --skip-update -p 20128` →
tmux `cftunnel` (`~/cloudflared-watchdog.sh`) → tmux `hermes` (gateway restart
loop). DNS wait 30s sebelum gateway start.

- 9Router: `localhost:20128`. State di `~/.9router/db/data.sqlite`
  (tabel `combos`, `models` JSON array). `models.json` hanya mirror — edit DB,
  bukan file. Backup DB sebelum utak-atik.
- Cloudflared: tunnel `neostudio`, subdomain
  `routerin.neostudio.web.id` → 9Router. Watchdog restart tiap 15s + jalankan
  `pitcoin/autokeep.sh`.
- Telegram bot: token baca dari `.env` var `TELEGRAM_BOT_TOKEN`,
  **bukan** `telegram.bot_token` di `config.yaml`. Set `.env` lalu
  `hermes gateway restart`.
- SearXNG di Railway untuk `web_search`.

## Pitfalls

- **Gateway restart/kill diblokir dari dalam proses gateway** (SIGTERM
  propagate). Jalankan dari shell Termux terpisah, bukan dari sesi ini.
- **9Router routing index di RAM**: gateway tulis ulang
  `gateway_routing` + `sessions.json` tiap save/shutdown. Edit DB live lalu
  restart = ketimpa stale. Fix via `/model` di-chat, atau shutdown → edit cold
  → start.
- `NEXT_OPTIONS=--openssl-legacy-provider` wajib untuk `neostudio` dev/build
  (Node 26 vs Next 14) — sudah di `package.json` scripts.
- SIPUTZX menolak request tanpa UA browser; proxy route & apiService kirim UA
  Chrome fixed. Lihat `src/app/api/[...endpoint]/route.js` &
  `src/lib/apiService.js`.
- `apps/web/vite.config.ts` proxy `/api` → `localhost:8787`; CORS hanya di
  ALLOWED_ORIGINS `wrangler.toml` saat deploy.
- Service lain (Discord bot, cron `bansos_ai_watch`/`ai_news_watch`) live di
  gateway — hentikan via `cronjob action=list` dulu sebelum restart total.
