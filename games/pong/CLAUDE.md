# 6&2 Pong — pokyny pro agenta

## Pravidla kódu (kousnou i u malé změny)

- **Celá hra je v jednom `index.html`** — žádné další JS soubory nezakládat bez dobrého důvodu.
- **Pozor na pojmenování pálek:** proti počítači ovládá hráč levou pálku (`player`) klávesami
  `p2Up`/`p2Down`; detail v `README.md` (Gotchas).
- Fotka nebo hlasový klip = soubor **a** záznam v poli v kódu (`WIN_PHOTO_PATHS`,
  `LOSE_PHOTO_PATHS`, `BOX_SOUNDS`), u klipu i srovnaná hlasitost — postup v `README.md`.

## Dokumentace — když… → čti

| Když | Čti |
|---|---|
| měníš cokoli ve hře (pravidla, mystery boxy, ovládání, assety) | `README.md` |
| měníš prvky společné všem hrám (návrat do menu, otočení, celá obrazovka, náhled odkazu) | `../../docs/nova-hra.md` |
