# Provoz webu — nasazení, GitHub Pages, doména

Kdy číst: před `git push`, při práci s doménou nebo DNS, když web nejede.

## Nasazení

- Repo je `sest-dva/6a2games-web` (GitHub organizace `sest-dva`).
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
4. Hlásí-li GitHub „domain is already taken", ověřit doménu u organizace (org Settings → Pages →
   Add a domain, TXT záznam u DNS správce) — viz níže. Jednu doménu nejde ověřit u dvou účtů naráz.

## Repo nenese jméno autora

Web nemá jít snadno spojit s konkrétní osobou. Proto:
- Commity mají neutrálního autora **`6&2 <noreply@6a2games.cz>`** (adresa nikam nevede). Nastavuje
  ho lokální `git config user.name` / `user.email`; v novém klonu ho nastavit dřív než první commit
  a ověřit `git log -1 --format='%an <%ae>'`.
- Do repa nepsat osobní jména, e-maily ani uživatelské jméno GitHub účtu — účtové hodnoty patří
  do `CLAUDE.local.md`.

## Doména `6a2games.cz`

- Apex má A záznamy GitHub Pages, `www` je CNAME na `sest-dva.github.io`.
- Doména je u organizace **ověřená** TXT záznamem `_github-pages-challenge-sest-dva` u DNS
  správce — **záznam nemazat**, jinak ji může zabrat cizí Pages web a GitHub pak hlásí
  „domain is already taken".
- Konkrétní DNS správce, hodnoty záznamů a účty jsou v `CLAUDE.local.md` (mimo git).
