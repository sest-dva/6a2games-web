# 6&2 Pong

Klasický Pong proti počítači (3 obtížnosti) nebo ve dvou na jedné klávesnici/displeji, ozvláštněný
**mystery boxy**, které mění průběh výměny, a koncovou obrazovkou s fotkami „approved by 6&2" /
„si k hovnu". Celá hra je v jednom souboru `index.html`, na webu
`https://6a2games.cz/games/pong/index.html`.

## Ovládání

- **Menu (klávesnice):** `1` proti počítači / `2` multiplayer / `3` nastavení → u počítače
  obtížnost `1`–`3` → počet výherních bodů `1`–`9`.
- **Hra:** proti počítači hráč ovládá **levou** pálku šipkami; ve dvou hráč 1 (vlevo) `W`/`S`,
  hráč 2 (vpravo) šipky. **Mezerník** = podání, **Esc** = menu. Klávesy jdou přemapovat
  v Nastavení (jen do zavření stránky — neukládají se).
- **Dotyk:** menu jako tlačítka, šipky po stranách displeje (proti počítači jen levé), podání
  klepnutím na hrací plochu, `≡` = herní menu, tlačítko celé obrazovky v Nastavení.

## Mystery boxy

Během výměny se na hřišti objevují krabice (počet, interval i životnost řídí konstanty `BOX_*`; před zmizením blikají).
Míček, který do krabice narazí, spustí náhodný efekt a zazní náhodný hlasový klip (nikdy stejný
dvakrát po sobě):

| Efekt | Co udělá |
|---|---|
| RYCHLEJI! | zrychlí míček |
| OBRAT! | otočí a zpomalí míček (nepadne dvakrát po sobě) |
| ROZDVOJENÍ! | přidá další míček (počet míčků je omezený, viz `applyBoxEffect`) |
| VĚTŠÍ / MENŠÍ PLOŠINA | dočasně zvětší / zmenší pálku toho, kdo míček odehrál naposled |
| ZMĚNA SMĚRU! | pošle míček náhodným směrem |
| PEVNÁ ZEĎ! | krabice zůstane jako překážka, od které se míčky odrážejí, do konce bodu |

Počítač si vybírá nejbližší míček, který letí k němu, a míří s náhodnou odchylkou; obtížnost
mění jeho rychlost, mrtvou zónu a odchylku (`AI_CONFIGS`).

## Technologie a soubory

- Čisté HTML/CSS/JS, kreslení do `<canvas>`, logika v jedné IIFE na konci `index.html`.
  Písmo hry Courier New; Baloo 2 (Google Fonts) jen pro výzvu k otočení a hlášku o celé obrazovce.
- Zvuk: herní pípáky a znělka gólu syntetizované přes WebAudio; hlasové klipy mystery boxů jako MP3.
- **Assety (záměrná výjimka z „minima assetů"):**

```
pong/
  index.html          celá hra (styly, HTML překryvy, logika)
  foto/
    mystery_box.png   obrázek krabice
    win/  lose/       fotky na koncovou obrazovku (vítěz / poražený)
  sounds/             hlasové klipy mystery boxů (MP3)
```

## Konvence kódu

- Laditelné hodnoty (pálky, míček, krabice, AI) jsou pohromadě v bloku `Herní konstanty`.
- Logika je rozdělená komentářovými oddíly `// ─── Název ───` (Audio, Vstupy, Přechody stavů,
  Fyzika míčku, Mystery boxy, Vykreslení, Hlavní smyčka, Mobil…). Stavový automat: `STATE`
  (menu → ready → playing → gameover) a `menuPhase` (mode → difficulty → score / settings).
- **Nová fotka:** soubor do `foto/win/` nebo `foto/lose/` **a** cesta do `WIN_PHOTO_PATHS` /
  `LOSE_PHOTO_PATHS` (velikost písmen v názvu musí sedět — server je rozlišuje).
- **Nový hlasový klip:** soubor do `sounds/` **a** záznam do `BOX_SOUNDS` s hlasitostí `v`
  srovnanou na ~−26 dB (změřit `ffmpeg -af volumedetect`; přepálené klipy jinak přeřvou hru).

## Gotchas

- **Pojmenování pálek:** levá pálka je `player`, pravá `right`. Proti počítači hráč ovládá levou
  pálku klávesami **`p2Up`/`p2Down`** (šipky) a `updatePlayer1()` je ve skutečnosti obsluha
  kláves p2 — ve dvou ovládá p2 pravou pálku a `updatePlayer2()` levou přes `p1` klávesy.
- Fotky a krabice se načítají až při první hře (`preloadAssets`), ne při otevření stránky
  (šetří mobilní data). Chybějící fotka se skryje místo „rozbitého" obrázku.
- AudioContext se po přerušení (iOS na pozadí, hovor, bfcache) probouzí v `ensureAudio()`.
- Při ztrátě fokusu nebo skrytí záložky se nulují stisknuté klávesy (pojistka proti „zaseknuté"
  pálce).
