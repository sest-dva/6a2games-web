# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Konvence repa: kód, komentáře, UI texty i dokumentace jsou **česky** (s plnou diakritikou).

## Co to je

**6&2 Games** (`6a2games.cz`) — web s retro arkádovými hrami v prohlížeči. `index.html` je
rozcestník, každá hra má vlastní složku `games/<hra>/` (seznam: `ls games/`). Jediná část s Node
závislostmi je leaderboard backend Party Survivors (Cloudflare Worker).

## Platí vždy

- **Žádný build, bundler ani ES moduly** — čisté HTML/CSS/JS přes `<script>` tagy, musí jet
  i z `file://`.
- **`git push` na `main` = okamžité nasazení na ostrý web** (GitHub Pages). Pushovat jen se
  souhlasem uživatele.
- **Repo musí zůstat veřejné**, jinak web spadne (viz `docs/provoz.md`).
- **Mobil na šířku je first-class** — každý nový UI prvek řeš i pro dotyk.
- Lokální spuštění: `python -m http.server` v kořeni → `http://localhost:8000/index.html`.

## Rozcestník dokumentace — když… → čti

| Když | Čti |
|---|---|
| pracuješ na hře (i balanc) | `games/<hra>/CLAUDE.md` — načte se sám, jakmile čteš soubor hry; začínáš-li jinak (grep, spuštění skriptu), přečti ho nejdřív |
| upravuješ úvodní stránku, menu nebo její zvuk | `docs/rozcestnik.md` |
| přidáváš hru / měníš prvky společné všem hrám (návrat do menu, mobil, náhled odkazu) | `docs/nova-hra.md` |
| pushuješ, řešíš doménu, DNS nebo výpadek webu | `docs/provoz.md` |
| navazuješ na rozdělanou práci | `postup.md` (mimo git) |
| potřebuješ účty, DNS hodnoty, Cloudflare | `CLAUDE.local.md` (mimo git) |

## Údržba dokumentace

1. **Každá informace žije na jednom místě**; jinde jen odkaz.
2. **Rozcestníky jsou úplné** — každý dokument je v rozcestníku své úrovně (tady, nebo v CLAUDE.md
   hry) s tím, kdy ho číst. Nový dokument = nový řádek rozcestníku.
3. **Do CLAUDE.md (tady i ve hrách) patří jen to, co kousne při každé změně** v té oblasti;
   ostatní do tematického dokumentu.
4. **Čísla do textu neopisovat** — žijí v kódu; dokumentace popisuje mechanismus a jmenuje
   konstantu. Výjimka: designové cíle a preference uživatele.
5. Nový dokument jen pro samostatně potřebné téma; jinak oddíl existujícího.
6. Nová hra = `README.md` + `CLAUDE.md` podle šablony v `docs/nova-hra.md`.
7. Kořenový `README.md` je jen vizitka pro GitHub — pravidla a postupy do něj nepatří.
