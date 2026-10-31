# Party Survivors — herní obsah a mechaniky

Kdy číst: při změně obsahu nebo pravidel hry (útoky, hrdinové, nepřátelé, spawn, bossové,
level-up, odměny). **Čísla tu nejsou** — žijí v `js/data.js`; u každé mechaniky je jmenovaná
konstanta, ve které je hledat. Metoda balancování a cílová délka runu jsou v
[`../CLAUDE.md`](../CLAUDE.md).

## Princip hry (logika Vampire Survivors)

- **Cíl: přežít co nejdéle.** Jediný výsledek runu je čas přežití. Hráč ovládá **jen pohyb**,
  útoky se spouštějí **automaticky** na cooldown.
- Nepřátelé se valí v houstnoucích vlnách ze všech stran a zraňují dotykem. Z mrtvol padají
  **XP gemy**, sběrem (magnet) hráč leveluje.
- Tlak (počet × HP nepřátel) roste rychleji než DPS hráče → **smrt je nakonec nevyhnutelná**,
  žádné nesmrtelné kombo. Balanc cílí na to, aby se hráč snažil **od první minuty**; cílová délka
  runu je v [`../CLAUDE.md`](../CLAUDE.md).
- Průběh: jméno → výběr hrdiny → kiting hord + sběr XP → level-upy a build → bossové →
  smrt → Game Over s časem, lokálním rekordem a zápisem do globálního žebříčku.

## Level-up

- Level-up pozastaví hru a nabídne 3 karty: **nový útok** (strop `PS.BALANCE.maxWeapons`) /
  **vylepšení vlastněného útoku** (+DMG za level `weaponDmgPerLevel`, strop `weaponMaxLevel`) /
  **perk útoku** (`PS.WEAPON_PERKS`) / **pasivka** (strop `passiveMaxLevel`).
- Výběr je **vážený jako ve VS**: vylepšení vlastněných útoků mají vyšší šanci — v prvních
  levelech výrazně (rozvíjení startovního útoku), s rostoucím levelem se váhy vyrovnávají
  (`weaponUpWeight`).
- XP křivka je **kvadratická** (`xpForLevel`): každý level stojí víc než předchozí, tempo
  levelování klesá celou hru a vyvažuje rostoucí přísun gemů z hustšího spawnu. Hráč za run
  záměrně nestihne vymaxovat vše → volba karet je strategická.

## Hrdinové (`PS.HEROES`)

Každý hrdina má **unikátní startovní útok + pasivku** a vlastní vzhled (obličej/rysy, ne jen
barvu). Síla útoku a pasivky se řídí pravidlem inverzní pasivky (viz [`../CLAUDE.md`](../CLAUDE.md)).
Seznam hrdinů musí znát i backend žebříčku (`HERO_NAMES`) — viz
[`../leaderboard-server/README.md`](../leaderboard-server/README.md).

## Útoky (`PS.ATTACKS`, logika `js/systems/weapons.js`)

Každý útok je jeden **archetyp** v `PS.Weapon` (`init_*`/`tick_*` podle pole `archetype`):
cone / beam / zone / lob / bounce / sweep / ricochet / orbit / slap. Profily se liší ve:

- **míření** (`aim`): na nejbližšího nepřítele (lahváč na náhodného z několika nejbližších) /
  kam je hrdina otočený / pevný vzor kolem hrdiny;
- **DMG/CD profilu**: od „nuke" s dlouhým cooldownem (lahváč) po rychlopalbu (vajgly);
- **multi-target**: pierce, odrazy mezi nepřáteli, neomezené AoE, ricochet, limit cílů;
- **control**: knockback síly 0–4 (pole `knockback`), stun, slow, DoT.

**Perky útoků** (`PS.WEAPON_PERKS`) jsou druhá osa vylepšení: každý útok má 1–2 perky na počty
(další projektil/odraz/cíl/zóna, dvojité útoky, širší úhly, rychlejší tiky), každý s malým
stropem. Efekty čte `PS.Weapon` přes `this.perk(id)`, stav drží `weapon.perkLevels`.

**Pasivky** (`PS.UPGRADES`): max HP, DMG, CD, rychlost, plocha, magnet, regen, armor, DMG na
bosse, krit, XP gain, heal za kill. Základní slabou regeneraci mají všichni (`player.baseRegen`).

## Nepřátelé (`PS.ENEMIES`, `js/systems/spawner.js`)

- **Nekonečná progrese tierů:** nový tier každých `tierSeconds`. Tier *n* → typ `(n−1) % 5`,
  level `⌊(n−1)/5⌋+1` (gufrau → kravaťáci → pikaři → rodiče → policisté → gufrau lvl 2 → …).
  Síla roste plynule: `enemyHp`, `enemyDmg`, `enemySpeed`.
- Každý typ má **vlastní rychlost** (`speedMult`). **Hrdina musí vždy zůstat nejrychlejší** —
  strop rychlosti nepřátel je pod `player.speed`.
- **Odolnost vůči odhozu (anti-AFK):** knockback slábne s levelem nepřítele, bossové odolávají
  nejvíc (`GameScene`, výpočet `resist` u knockbacku) → žádná builda neudrží perimetr donekonečna.
- Strop živých nepřátel `maxEnemies` (přebytek → silnější jedinci).

## Spawn (`PS.Spawner`)

- Spawn probíhá v prstenci za okrajem kamery **s biasem do směru pohybu** — před hráčem nejde
  utíkat donekonečna; příliš vzdálení nepřátelé se teleportují zpět k okraji. Vše je relativní
  k hrdinovi, ne k mapě (`mapSize` výkon ani hustotu neovlivní).
- Proud je **nerovnoměrný**: stálý proud `spawnRate` (roste s časem) + krátké vlnky (`surge`)
  + občas velká telegrafovaná **horda** (`horde`, `spawnHorde` — soustředěný nápor z jednoho
  směru s hláškou „VALÍ SE DAV!", otřesem a zvukem).
- Křivku tlaku v čase ukáže `node tools/spawn-curve.js`.

## Bossové a aréna („ring")

- Bossové (`PS.BOSSES`) mají vlastní mechaniky (projektily, pushwave, melee swing, summoner,
  hypnóza) a přicházejí na **časovém plánu** (`bossFirst`, pak à `bossInterval`); n-tý boss má
  sílu 5n (`bossHp`, `bossDmg`), skiny se cyklí donekonečna.
- Při příchodu bosse se běžní nepřátelé seběhnou na **kruh kolem hrdiny** (`GameScene.startBossFight`,
  poloměr `arenaRadius`, rozestup `arenaSlotGap`) a utvoří **neprostupnou, nezranitelnou zeď** —
  souboj je 1v1 (uvnitř bojují jen vyvolanci bosse summonera).
- Formace probíhá **vždy zvenku** (kdo je uvnitř, radiálně ustoupí na okraj; ostatní obíhají po
  obvodu, nikdy středem) s krátkým sprintem (`ringFormUntil`).
- Hrdina ring nemůže prorazit (geometrický clamp); na okraji dostává normální kontaktní damage.
- Aréna se během souboje **pomalu stahuje** (`arenaShrinkDelay`, `arenaShrinkDur`,
  `arenaMinRadius`) — souboj nejde beztrestně natahovat.
- Nepřátelé ve zdi mají flag `ringWall`: jsou nezranitelní a ignorují je všechny prostorové dotazy
  zbraní i projektily.
- Po dobu souboje **stojí spawn vln, powerupů, časomíra tierů i časomíra přežití** (HUD ukazuje
  „PROBÍHÁ BOSS FIGHT"); smrtí bosse se ring rozpustí a skóre zase běží.

## Odměny na mapě

- **Powerupy („klíčky", `PS.POWERUPS`)** vzácně na mapě (`powerupIntervalMin/Max`): heal, freeze,
  speed, damage, magnet, nesmrtelnost, čistka.
- **Runda panclů** (à la Treasure ve VS): vzácný drop (`rundaIntervalMin/Max`), spawne se
  ~obrazovku od hrdiny (`rundaSpawnMin/Max`); **zlatá šipka na okraji HUD** vede k ní, zůstává na
  mapě, dokud ji hrdina nesebere (max 1 zároveň). Obsahuje náhodné klíče bez duplicit
  (`rundaKeys`) + stálé upgrady vlastněných útoků/pasivek (`rundaUpgrades`; žádné nové útoky,
  respektuje stropy). Sebrání pauzne hru a `RundaScene` postupně odhalí odměny
  (`GameScene.spawnRunda`, šipka v `HUDScene`).

## Výsledek runu

- **Lokální rekord** (`localStorage` `ps_record` = `{time, name}`) se zobrazuje na Game Over
  (v menu už ne — tam je položka LEADERBOARD). Herní jméno se zadává po STARTu (`NameScene`), pamatuje se a předvyplňuje.
- **Globální žebříček** TOP 10 (`PS.LB` v `js/leaderboard.js`, `LeaderboardScene`): při startu
  runu si hra vyžádá anti-cheat token, po smrti odešle čas. Offline-safe — při výpadku sítě se
  tiše přeskočí. Backend: [`../leaderboard-server/README.md`](../leaderboard-server/README.md).
