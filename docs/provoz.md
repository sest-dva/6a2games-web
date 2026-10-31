# Provoz webu — nasazení, GitHub Pages, doména

Kdy číst: před `git push`, při práci s doménou nebo DNS, když web nejede.

## Nasazení

- **GitHub Pages servíruje repo přímo z větve `main` (kořen)**, vlastní doména přes soubor `CNAME`.
  Žádný build krok ani CI → **deploy = `git push` na `main`**. Push je tedy rovnou ostrý web
  (změna se projeví během pár minut).
- Leaderboard backend Party Survivors se nasazuje zvlášť (Cloudflare) —
  viz [`games/party-survivors/leaderboard-server/README.md`](../games/party-survivors/leaderboard-server/README.md).

## Repo musí zůstat veřejné

Na free tarifu GitHub Pages z privátního repa nejede. Přepnutím na soukromé (nebo koncem GitHub Pro)
GitHub Pages vypne **i se zapomenutím vlastní domény** a web vrací 404 (stalo se 10/2026).

**Obnova po výpadku:**
1. Zveřejnit repo.
2. Settings → Pages: zdroj `main` / kořen.
3. Custom domain `6a2games.cz`, zapnout **Enforce HTTPS**.
4. Hlásí-li GitHub „domain is already taken", ověřit doménu u účtu (Settings → Pages → Add a domain,
   TXT záznam u DNS správce) — viz níže.

## Doména `6a2games.cz`

- Apex má A záznamy GitHub Pages, `www` je CNAME na GitHub Pages účtu.
- Doména je u GitHub účtu **ověřená** TXT záznamem `_github-pages-challenge-6a2` u DNS
  správce — **záznam nemazat**, jinak ji může zabrat cizí Pages web a GitHub pak hlásí
  „domain is already taken".
- Konkrétní DNS správce, hodnoty záznamů a účty jsou v `CLAUDE.local.md` (mimo git).
