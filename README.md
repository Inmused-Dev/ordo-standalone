# Ordo standalone

**Temporary** public site hosting St Aidan's liturgies for tablet use during
services — live at <https://ordo.staidans.org.au>. No auth, no build step, no
database. This is a deliberate throwaway until the real Ordo app
(st-aidans-apps) is developed; delete this deployment when Ordo lands.

## Layout

```
site/                      ← everything nginx serves (nothing else is exposed)
  index.html               ← liturgy index (Lumen-styled, reads liturgies.json)
  liturgies.json           ← manifest the index renders, newest first
  liturgies/               ← one self-contained HTML page per liturgy
docker-compose.yml         ← nginx:alpine on the armature-prod network
```

Routing lives in the shared Caddyfile (`armature-infra/provision/Caddyfile`):
`ordo.staidans.org.au → reverse_proxy ordo-standalone:80`.

## Adding a liturgy (the Cowork workflow)

1. Compile the liturgy as a **single self-contained HTML page** (same interface
   as the existing ones: previous/next buttons, page indication).
2. Save it as `site/liturgies/YYYY-MM-short-name.html`.
3. Add an entry to the **top** of `site/liturgies.json`:

   ```json
   { "title": "Holy Communion", "date": "August 2026", "href": "liturgies/2026-08-holy-communion.html" }
   ```

4. Commit, push, deploy:

   ```sh
   git add site/ && git commit -m "Add <liturgy> <month year>" && git push
   ssh vps-staidans 'cd /opt/ordo-standalone && git pull --ff-only'
   ```

No container restart needed — `site/` is bind-mounted.

## First deploy / rebuild

Step-by-step execution script: [runbooks/deploy-2026-08.md](runbooks/deploy-2026-08.md)
(clone to `/opt/ordo-standalone`, `docker compose up -d`, add the Caddy route,
verify, then swap the Cloudflare `ordo` record from render.com to the VPS).
