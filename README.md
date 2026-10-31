# 6&2 Games

Retro arkádové hry s CRT estetikou, které běží přímo v prohlížeči — na počítači i na mobilu
(na šířku). Hraj na **[6a2games.cz](https://6a2games.cz)**.

## Hry

| Hra | O čem je | Složka |
|---|---|---|
| **6&2 Pong** | klasický Pong proti počítači nebo ve dvou, s mystery boxy, které mění průběh hry | [`games/pong/`](games/pong/) |
| **Party Survivors** | survival ve stylu Vampire Survivors v temném klubu: přežij co nejdéle hordy nepřátel, s globálním žebříčkem | [`games/party-survivors/`](games/party-survivors/) |

## Spuštění na počítači

Žádná instalace ani build. Stačí otevřít `index.html`, nebo (doporučeno) spustit statický server
v kořeni repa:

```
python -m http.server
# → http://localhost:8000/index.html
```

Hry načítají písma (a Party Survivors engine Phaser) z internetu.

## Dokumentace

Postupy pro vývoj jsou v [`docs/`](docs/), popis každé hry v `README.md` její složky.
Vstupním bodem pro AI agenta je [`CLAUDE.md`](CLAUDE.md).
