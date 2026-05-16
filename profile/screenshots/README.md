# Screenshots (from your machine)

These PNGs are **not** checked in as placeholders. Generate them from a running local stack so they always match the real UI.

```powershell
# Terminals: Qdrant (optional), hospital-api :8001, chatbot-ai :8000, then:
cd projects\hospital-ui
npm install
npx playwright install chromium
npm run dev
# another shell:
npm run capture:portfolio
```

Outputs:

| File | Source |
| --- | --- |
| `home.png` | `/` |
| `book-appointment.png` | `/book` |
| `chatbot.png` | `/` with chat FAB opened |
| `admin.png` | `/admin` |

Override base URL: `set BASE_URL=http://127.0.0.1:4173&& npm run capture:portfolio` (e.g. after `npm run build && npm run preview`).

Narrated MP4 + full tour: `npm run capture:portfolio:voice` — see [`../../projects/hospital-ui/scripts/README.md`](../../projects/hospital-ui/scripts/README.md) (`edge-tts`, `ffmpeg`, optional `ADMIN_TOKEN` for ingest).
