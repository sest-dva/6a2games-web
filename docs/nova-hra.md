# Nová hra — zásady, povinný kontrakt herní stránky, šablona dokumentace

Kdy číst: při přidávání hry a při změně prvků, které mají všechny hry společné (návrat do menu,
mobil, celá obrazovka, náhled odkazu).

## Zásady vývoje

Platí pravidla z kořenového `CLAUDE.md` (bez buildu, česky, mobil na šířku first-class). Navíc:

- **Hratelnost na prvním místě** — hra musí být zábavná a intuitivní, bez frustrace z bugů.
- **Minimální závislosti** — jednoduché hry čistý JS; komplexní smí Phaser 3 přes CDN (jsdelivr,
  s `integrity`).
- **Retro vibe, moderní kvalita** — CRT/arkádová estetika, ale kód čistý a robustní.
- Písmo hry musí umět českou diakritiku.
- **Binární assety na minimu.** Grafiku a zvuky přednostně generovat (Canvas, WebAudio).
  Bitmapy a zvukové klipy zakázané nejsou, když je hra potřebuje — příklad: Pong (fotky, hlasové
  klipy), Party Survivors má jen komprimovanou hudbu.

## Povinný kontrakt herní stránky

Každá hra musí mít všechno z tohoto seznamu. Vzorová řešení: `games/pong/index.html` (čisté JS)
a v Party Survivors (Phaser) `partysurvivors.html` (hlavička, `#rotate`), `js/scenes/MenuScene.js`
(návrat do menu), `js/main.js` (výzva k otočení), `js/scenes/SettingsScene.js` + `js/ui.js`
(celá obrazovka).

- [ ] **Hlavička:** `<html lang="cs">`, viewport s `viewport-fit=cover`, `<title>`,
      `<meta name="description">`.
- [ ] **Ikona:** `<link rel="icon">` a `apple-touch-icon` na `../../favicon.png`.
- [ ] **Web app na mobilu:** `apple-mobile-web-app-capable`, `mobile-web-app-capable`,
      `apple-mobile-web-app-status-bar-style`, `apple-mobile-web-app-title` (běh z plochy bez lišty).
- [ ] **Náhled při sdílení odkazu:** `og:type`, `og:title`, `og:description`,
      `og:url` = absolutní `https://6a2games.cz/games/<hra>/<soubor>.html`,
      `og:image` = `https://6a2games.cz/favicon.png`, `twitter:card` = `summary`.
- [ ] **Zpět do menu:** viditelný návrat na `../../index.html` (na dotyku smí být v herním menu,
      jako v Pongu). Po kliku: je-li v `sessionStorage`
      značka `from-hub` a `history.length > 1`, značku smazat a `history.back()`; jinak nechat
      proběhnout odkaz. Proč — viz [`rozcestnik.md`](rozcestnik.md#návrat-ze-hry-from-hub).
- [ ] **Dotykové ovládání** celé hry včetně menu a nastavení (detekce dotyku → třída/flag).
- [ ] **Výzva k otočení na šířku** (`#rotate`, vzhled shodný s ostatními hrami, písmo Baloo 2) —
      jen na dotykovém zařízení během hraní (menu smí na výšku); hra se mezitím pozastaví.
- [ ] **Celá obrazovka:** Fullscreen API + zámek orientace landscape; kde API chybí (iPhone Safari),
      návod „Sdílet → Přidat na plochu".
- [ ] **Zvuk až po gestu:** AudioContext vytvořit/probudit (`resume()`) při uživatelské akci.
- [ ] **`localStorage`** (používá-li ho hra) vždy v `try/catch` (soukromý režim), klíče
      s prefixem hry (`ps_…`).

## Postup přidání hry

1. Složka `games/<nazev>/`. Jednoduchá hra v jednom `index.html`; složitější smí mít podsložky
   a jiný název vstupního souboru.
2. Implementovat hru podle zásad a kontraktu výše.
3. `games/<nazev>/README.md` a `games/<nazev>/CLAUDE.md` podle šablony níže.
4. Rozcestník: položka `.menu-item` v `<nav>` v `index.html` + doplnit hru do `description`
   a `og:description` (viz [`rozcestnik.md`](rozcestnik.md)).
5. Řádek do seznamu her v kořenovém `README.md`.
6. Proklik lokálně (PC i mobil na šířku: návrat do menu, otočení, celá obrazovka), pak push
   (viz [`provoz.md`](provoz.md)).

## Šablona dokumentace hry

**`README.md`** (kompletní popis, čte se na vyžádání) — pevné nadpisy, nepotřebné vynechat:

```markdown
# <Název hry>
1–3 věty: o co ve hře jde, vstupní soubor, adresa na webu.
## Ovládání            klávesnice + dotyk
## Herní obsah         pravidla a mechaniky (u velké hry raději docs/mechaniky.md)
## Technologie a soubory   stack, strom souborů s rolí každého
## Konvence kódu       jak se v té hře píše (vzory, kam patří čísla, nové assety)
## Nástroje            pomocné skripty a jejich spuštění
## Gotchas             nečekané chování, na které se snadno narazí
```

Velké téma (mechaniky, backend) smí mít vlastní soubor v `games/<nazev>/docs/` — pak musí být
v rozcestníku `CLAUDE.md` té hry.

**`CLAUDE.md`** (načte se automaticky při práci se soubory hry — držet krátké, do ~30 řádků;
víc jen kvůli preferencím uživatele):

```markdown
# <Název hry> — pokyny pro agenta
## Pravidla kódu        jen to, co kousne při KAŽDÉ změně (pořadí skriptů, kam patří čísla…)
## Preference uživatele jak chce uživatel s hrou pracovat (je-li co)
## Dokumentace          rozcestník: „když… → čti" pro README a soubory v docs/
```
