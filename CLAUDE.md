# CLAUDE.md — Contesto progetto per Claude Code

Sito statico del **3º Torneo dei Rioni 3vs3** di Monsummano Terme.
📍 Parco Orzali · 🗓️ 10–14 giugno 2026 · cornice «Sportiva — Festa dello Sport».

URL pubblico: **https://alecsal.github.io/torneo-rioni-3vs3/**

## Architettura (single source of truth)

```
dati_torneo.json  →  genera.py  →  index.html
```

- `dati_torneo.json` — unica sorgente di verità (squadre, gironi, calendario, risultati)
- `genera.py` — generatore standalone, Python 3 puro, zero dipendenze
- `index.html` — output autoconsistente (CSS inline, nessun asset esterno)
- `README.md` — istruzioni utente

**Regola d'oro:** mai modificare `index.html` a mano. Si rigenera sempre da `genera.py`.

> ⚠️ **Per Claude / agenti automatici:** NON generare e NON committare tu `index.html`.
> Modifica solo `dati_torneo.json` (e, se serve il layout, `genera.py`) e pusha: è
> l'automazione di GitHub (CI) che rigenera e committa l'HTML dal cambiamento del JSON.
> Puoi eseguire `python3 genera.py` **solo come anteprima locale**, ma non includere
> mai `index.html` nei commit.

## Branch e pubblicazione

- Branch di pubblicazione: **`master`** (default del repo, non `main`)
- GitHub Pages: **Settings → Pages → Deploy from a branch → `master` / `root`** (già attivo)
- Ogni push su `master` ripubblica il sito entro ~1 minuto

## Automazione CI (importante)

File: `.github/workflows/genera-html.yml`

Quando un push su `master` modifica `dati_torneo.json` o `genera.py`:
1. La Action esegue `python3 genera.py`
2. Se l'`index.html` risulta diverso, lo committa con messaggio `Rigenera index.html da dati_torneo.json [skip ci]` e lo pusha
3. Il `[skip ci]` impedisce il loop infinito

Quindi l'utente normalmente **modifica solo `dati_torneo.json` e pusha** — l'HTML lo rigenera la CI.

## Inserire un risultato

In `dati_torneo.json` trova la partita per numero (`"n"`) e aggiorna:

```json
"punti1": 17, "punti2": 14, "giocata": true
```

La classifica del girone si ricalcola da sola (regole FIBA 3x3: vittoria 2 pti,
sconfitta 0; ordinamento: Pti → differenza canestri → canestri fatti).

## Struttura torneo

- 8 squadre da 5 rioni (Centro, Grotta Parlanti, Vergine dei Pini, Bizzarrino, Cintolese)
- 2 gironi da 4, round-robin
- Le squadre A e B dello stesso rione sono sempre in gironi diversi
- Tabellone finale: semifinali incrociate (1°A–2°B / 1°B–2°A), poi finale 3°/4° e 1°/2°

Composizione gironi:
| Girone A | Girone B |
|---|---|
| Centro A | Bizzarrino B |
| Grotta Parlanti A | Cintolese A |
| Vergine dei Pini A | Grotta Parlanti B |
| Bizzarrino A | Centro B |

## Tema grafico

Dark theme rosso/blu/arancio, già nel CSS inline di `genera.py`. Se va modificato,
si tocca il template dentro `genera.py` (non l'HTML).

## Workflow tipico per nuove sessioni

- **Inserire risultati:** edit `dati_torneo.json` → commit → push su `master` → la CI fa il resto.
- **Modificare grafica/layout:** edit `genera.py` → la CI rigenera l'HTML.
- **Modificare struttura torneo (squadre, gironi, calendario):** edit `dati_torneo.json`.
- **Locale (opzionale):** `python3 genera.py` solo per anteprima. **Non committare mai `index.html`**: lo rigenera e committa la CI dal cambiamento del JSON.

## Cose già fatte (storia)

1. Repo inizializzato con i 4 file alla root (commit `053a2e1`)
2. Push su `master`, GitHub Pages attivato manualmente su `master`/`root`
3. Aggiunto workflow CI per rigenerazione automatica (commit `a82bc79`)
