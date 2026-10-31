# Rozcestník webu (`index.html` + `menu-ambient.js`)

Kdy číst: při úpravě úvodní stránky, jejího menu nebo zvuku.

## Menu

- Položky her jsou odkazy `.menu-item` v `<nav>`. Popis webu v `<meta name="description">`
  a `og:description` vyjmenovává hry — při přidání hry upravit i ten.
- Písmo **Press Start 2P** nemá české diakritiky → texty rozcestníku jen ASCII (velkými písmeny).

## Zvuk: ambient, ne hudba

- Web **nemá hudbu na pozadí** (dřívější sdílená hudba menu byla zrušena). Jediná hudba na webu je
  Party Survivors ve hře.
- `menu-ambient.js` syntetizuje přes WebAudio tichý ambient („bzučení trafa"), žádný zvukový soubor.
  Při najetí na položku menu (hover/focus) zesílí, ve skryté záložce se ztiší. Laditelné konstanty
  jsou pohromadě nahoře v souboru (ladí se „od ucha").
- Kvůli autoplay pravidlům prohlížeče se zvuk rozezní až po 1. gestu — spouští ho gesture-listener
  v `menu-ambient.js`.

## Úvodní „attract" překryv (`#enter-overlay`) — nemazat

Bez něj by první klik padl rovnou na položku menu a odvedl do hry → ambient by se jen „probzukl".
Překryv „KLIKNI PRO START" zachytí první gesto (nikam nepřesměruje), takže ambient pak hraje jako
trvalá kulisa. Overlay jen schová sám sebe a sundá `inert` z `<nav>` (do té doby brání prokliku
a Tabu na skryté odkazy); zvuk spouští `menu-ambient.js`.

## Návrat ze hry (`from-hub`)

Při kliku na položku menu uloží rozcestník do `sessionStorage` značku `from-hub`. Hra se díky ní
umí vrátit **historií** (`history.back()`) → prohlížeč obnoví rozcestník z bfcache a ambient zase
naběhne sám (čerstvé načtení by kvůli autoplay hrálo až po gestu). Strana hry je povinnou součástí
kontraktu herní stránky — viz [`nova-hra.md`](nova-hra.md).
