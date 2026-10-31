# Party Survivors — pokyny pro agenta

## Pravidla kódu (kousnou i u malé změny)

- **Pořadí `<script>` tagů v `partysurvivors.html` je závazné** (Phaser → data → ui → touch →
  leaderboard → audio → playlist → music → systems → scenes → main), žádné ES moduly. Nový soubor
  = tag na správné místo, jinak nic nejede.
- **Vše visí na `window.PS`** (každý soubor začíná `window.PS = window.PS || {};`); scény se věší
  na `window` a samy registrují do `PS.scenes`, `main.js` z toho staví hru.
- **Čísla jen v `js/data.js`** (`PS.ATTACKS`, `PS.BALANCE`, `PS.HEROES`, `PS.ENEMIES`,
  `PS.WEAPON_PERKS`…), nikdy natvrdo v logice.
- **Globální tempo ×0,7:** rychlosti *i vzdálenosti* jsou škálované 0,7 (časy ne) → rychlost
  a vzdálenost lad vždy **spolu**.
- **Gameplay časy běží proti `GameScene.gameTime`** (akumulovaná delta), ne `this.time.now`
  (ten po pauze skočí dopředu a rozbíjí buffy z klíčů).

## Balancování útoků — jak to chce uživatel

Uživatel ladí podle **pocitu ze hry**; cíl je **užší rozptyl** mezi útoky, ne plochý. Drž se:

- **Slabé útoky vybuff** na konkurenceschopné, ale **ne nejsilnější**; silné jen jemně srovnej,
  **žádné velké nerfy** toho, co už je dobré.
- **Metrika = efektivní DPS na roj**; slow/stun/knockback se počítají jako utility bonus
  (control-heavy útoky smí mít ~20–35 % nižší surové DPS). Lad útoky **samostatně** — ignoruj
  pasivky hrdinů (jakýkoli útok si může vzít kdokoli přes level-up).
- Útoky **na dálku** mají mít v simu **záměrně ~25–35 % nižší skóre než melee** (sim neumí změřit
  bezpečí distance).
- **Cílová sim-pásma:** melee/self ~90–125, ranged ~55–85, control-heavy ~45.
- **Inverzní pasivka:** silný primární útok hrdiny ⇒ slabá startovní pasivka a naopak
  (`PS.HEROES`).
- **Anti-AFK:** nepřátelé vyšších úrovní odolávají odhozu → žádná builda neudrží perimetr
  donekonečna. Neruš to.
- **Cílová délka runu:** většina runů končí ~13.–18. min, 18–23 min velmi dobré, 23+ insane
  (stejný cíl je v komentáři u `PS.BALANCE.spawnRate`; při změně upravit obojí).
- Postup (příkazy ze složky hry): uprav `js/data.js` → `node tools/balance-sim.js` → iteruj
  (effDPS ve 3 scénářích: hustý roj / řídký proud / slabí nepřátelé). Křivku tlaku spawnu ukáže
  `tools/spawn-curve.js`.

## Dokumentace — když… → čti

| Když | Čti |
|---|---|
| měníš kód: architektura, soubory, konvence, nástroje, gotchas | `README.md` |
| měníš herní texty (kód zastará `wording.xlsx`) nebo hudbu | `README.md`, oddíl Nástroje |
| měníš herní obsah nebo mechaniky (útoky, hrdinové, nepřátelé, spawn, bossové, level-up) | `docs/mechaniky.md` |
| sáhneš na žebříček nebo hrdiny (backend musí znát jejich seznam) | `leaderboard-server/README.md` |
| měníš prvky společné všem hrám (návrat do menu, otočení, celá obrazovka, náhled odkazu) | `../../docs/nova-hra.md` |
