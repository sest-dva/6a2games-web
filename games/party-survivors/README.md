# Party Survivors

Survival ve stylu **Vampire Survivors** v temné klubové atmosféře pro web 6&2 Games: přežij co
nejdéle houstnoucí hordy, útoky se spouštějí samy, ty jen kitíš a skládáš build. Vstupní soubor
`partysurvivors.html`, na webu `https://6a2games.cz/games/party-survivors/partysurvivors.html`.
Herní obsah a mechaniky popisuje [`docs/mechaniky.md`](docs/mechaniky.md).

## Ovládání

- **PC:** pohyb **WASD** (přemapovatelné v Nastavení) + **šipky** vždy jako záloha; pauza **P**,
  zvuk **M**; menu a výběry klávesnicí i myší.
- **Mobil na šířku:** joystick jen pro pohyb — plovoucí (objeví se, kam ťukneš) nebo pevný vpravo
  dole; v HUD dotyková tlačítka pauza a zvuk; jméno přes HTML `<input>`; v Nastavení volby
  joysticku (režim, velikost, průhlednost) a celá obrazovka (na iPhonu návod „Přidat na plochu").

## Technologie a soubory

| Co | Čím |
|---|---|
| Engine | **Phaser 3.87** (arcade physics) přes CDN jsdelivr |
| Jazyk | čistý JavaScript (ES6+), žádný framework, bundler ani ES moduly |
| Grafika | **100 % procedurální** — `BootScene` kreslí vše do Canvas textur jednou při bootu (přechody, měkké stíny, tmavý obrys + dvoutónový rim-light); v repu žádné obrázky |
| Vizuál | temná klubová atmosféra (barevné reflektory + vinětace v `GameScene`), nasvícené postavy s unikátními rysy, signature efekty útoků |
| Zvuk | syntetické SFX přes WebAudio (`PS.Audio`) |
| Hudba | jen ve hře (`GameScene`), tracky sekvenčně podle abecedy ve smyčce s crossfade, stream přes 2 HTML5 Audio (`PS.Music`), respektuje mute |
| Písmo | **Baloo 2** (Google Fonts) — párty styl hry, plná česká diakritika; Permanent Marker jen pro tag „DON" |
| Persistence | `localStorage` (klíče v `PS.STORAGE`): `ps_keys`, `ps_record`, `ps_name`, `ps_muted`, `ps_touch`; `ps_best` je legacy a maže se |
| Platforma | PC + mobil na šířku. `Scale.FIT` na PC a v menu, `ENVELOP` ve hře na mobilu (kritický HUD chrání `PS.UI.safeInset()`); detekce `PS.isTouch` |
| Žebříček | globální TOP 10 přes Cloudflare Worker + D1 ([`leaderboard-server/`](leaderboard-server/README.md)) — jediná část s `package.json` |

```
party-survivors/
  partysurvivors.html      kostra stránky, #rotate, « ZPĚT, <script> tagy (pořadí závazné — CLAUDE.md)
  js/
    data.js                ⭐ jediný zdroj čísel: PS.HEROES, PS.ATTACKS, PS.WEAPON_PERKS, PS.UPGRADES,
                              PS.POWERUPS, PS.ENEMIES, PS.BOSSES, PS.BALANCE, PS.STORAGE, PS.COLORS
    ui.js                  PS.Keys + PS.UI (text/neon helpery, safeInset, PS.isTouch, goFullscreen)
    touch.js               PS.Touch — plovoucí joystick + sdílený stav pohybu
    leaderboard.js         PS.LB — klient globálního žebříčku (offline-safe)
    audio.js               PS.Audio — WebAudio syntéza SFX
    playlist.js            ⚙ AUTOGEN seznam hudby PS.MUSIC (tools/build-music.js) — needitovat
    music.js               PS.Music — hudba na pozadí
    main.js                Phaser config, čekání na font, výzva k otočení, start hry
    systems/
      weapons.js           PS.Weapon — archetypy útoků
      spawner.js           PS.Spawner — tiery, nerovnoměrný spawn, hordy, bossové
    scenes/
      BootScene.js         generování všech textur → Menu
      MenuScene.js         START / VYSVĚTLIVKY / NASTAVENÍ / LEADERBOARD, návrat na web
      LeaderboardScene.js  globální TOP 10
      HelpScene.js         pravidla
      SettingsScene.js     přemapování kláves, dotykové volby, celá obrazovka
      NameScene.js         zadání herního jména
      HeroSelectScene.js   mřížka hrdinů
      GameScene.js         ⭐ herní jádro: pohyb, kolize, damage pipeline, XP, level-up pool, boss ring
      HUDScene.js          overlay: čas, HP/XP bar, ikony zbraní, šipka k Rundě, pauza
      LevelUpScene.js      pauza + 3 karty
      RundaScene.js        odhalení odměn z Rundy panclů
      GameOverScene.js     čas, level, killy, rekord, damage-metr zbraní (zápis do žebříčku odesílá GameScene)
  tools/                   pomocné Node skripty (viz Nástroje)
  music/                   mastery hudby (mimo git) + compressed/*.mp3 (v gitu)
  wording.xlsx             export všech herních textů pro ruční úpravy
  leaderboard-server/      backend žebříčku
  docs/mechaniky.md        herní obsah a mechaniky
  _interni/                pracovní materiály uživatele (mimo git)
```

## Konvence kódu

Pravidla, která platí u každé změny (pořadí skriptů, `window.PS`, čísla jen v `data.js`, tempo
×0,7, `gameTime`), jsou v [`CLAUDE.md`](CLAUDE.md). Dále:

1. **Komentáře a UI texty česky.** Hlavička každého souboru = blokový komentář `// ===` s rolí
   souboru.
2. **Žádné externí assety.** Nová grafika = další `make*()` metoda v `BootScene`, kreslí se do
   Canvas textury (`canvasTex(key, w, h, (ctx) => …)`) se sdílenými helpery `litEdge`, `vGrad`,
   `groundShadow`, `rr`. Nový zvuk = další metoda v `PS.Audio` (oscilátory).
3. **Výkon je first-class** (cíl 60 FPS při plném davu): object pooling přes Phaser groupy
   s `maxSize`, strop živých nepřátel, slučování XP gemů, pool plovoucích čísel poškození,
   throttle hit-zvuků. Při recyklaci pooled objektů **resetovat stavové efekty**
   (slow/DoT/stun/`ringWall` — viz `spawner.js`).
4. **Defenzivní `localStorage`** — vždy v `try/catch` (soukromý režim).
5. **Damage pipeline centrálně:** zbraně počítají jen vlastní base DMG; globální multiplikátory
   (dmgMult, krit, buffy) aplikuje `GameScene.dealDamage()`. Nový útok = nový archetyp
   v `PS.Weapon` (`init_*`/`tick_*` podle `archetype` v datech).

## Nástroje

Node skripty v `tools/` (spouštět ze složky hry, ne z kořene repa); herní data čtou přímo z `js/data.js`
(vm sandbox), takže jsou vždy v synchronu.

| Příkaz | K čemu |
|---|---|
| `node tools/balance-sim.js` | headless simulátor efektivního DPS útoků (3 scénáře) — viz balancování v `CLAUDE.md` |
| `node tools/spawn-curve.js` | křivka tlaku spawnu v čase |
| `node tools/build-wording.js` | export **všech** herních textů do `wording.xlsx` |
| `node tools/read-wording.js [--json]` | vypíše řádky `wording.xlsx` s vyplněným sloupcem „Nový text" (úpravy k zapracování) |
| `node tools/build-music.js` | komprese hudby (potřebuje `ffmpeg` v PATH; nebo dvojklik `tools/compress-music.bat`) |
| `node tools/check-heroes.js` | parita `HERO_NAMES` (backend) ↔ `PS.HEROES`; exit 1 = nesoulad |

- **Úprava textů:** `build-wording.js` → uživatel vyplní sloupec „Nový text" v `wording.xlsx` →
  `read-wording.js` vypíše změny → zapracovat do zdrojových souborů (sloupec „Zdroj") → znovu
  `build-wording.js`.
- **Hudba:** mastery v `music/` (v `.gitignore`) → `build-music.js` je zkomprimuje na 128k MP3 do
  `music/compressed/` a **vygeneruje `js/playlist.js`**. Přidání tracku: master s dalším volným
  číselným prefixem (např. `06_nazev.wav`) do `music/` + spustit skript.

## Gotchas

- **Tempo ×0,7 — co se neškáluje:** velikosti sprajtů a hitboxů, `arenaSlotGap`, kontaktní
  paddingy (+14/+16) v prostorových dotazech.
- `main.js` čeká na font se **vzorkem české diakritiky** — canvas text sám nevynutí stažení
  latin-ext subsetu. Při změnách načítání fontů zachovat.
- Hra startuje přes guard `window.__psGame` (ochrana proti dvojí inicializaci).
- `heroId` se předává do `GameScene` přes `init(data)` s fallbackem na Phaser `registry`.
- **`build-music.js` maže z `music/compressed/` každou MP3 bez masteru.** Mastery jsou mimo git →
  spouštět jen na počítači, kde jsou všechny (po čistém klonu by smazal hudbu ve hře).
- `data.js` vznikl přepisem tabulky `logika.xlsx`, která v repu už není; komentáře v `data.js`
  („list Hrdinové", „sloupce…") na ni odkazují historicky.
- Výzva k otočení (`#rotate`) se ukazuje jen během hry (běží `HUDScene`) a hru pauzne; menu jde
  na výšku i na šířku.
