# Poker Tools

Raccolta di strumenti web statici, pensati per girare da iPhone (aggiunti alla schermata Home) con storage stabile.

## App incluse

- **tracker/** — MTT Bankroll Tracker. Inserimento manuale dei tornei, statistiche (ROI, ITM%, profitto, grafico bankroll), buy-in massimo consigliato (1% del bankroll), backup export/import in JSON.

## Deploy su GitHub Pages (da computer)

1. Crea un account su https://github.com (gratis).
2. Crea un nuovo repository **pubblico**, es. nome `poker`.
3. Carica il contenuto di questa cartella nel repository
   (puoi trascinare i file nella pagina del repo, oppure usare git / Claude Code).
4. Vai su **Settings -> Pages**.
5. In "Build and deployment", sorgente **Deploy from a branch**,
   seleziona branch `main` e cartella `/ (root)`. Salva.
6. Dopo ~1 minuto l'app sarà raggiungibile a:
   `https://TUONOME.github.io/poker/tracker/`

## Aggiungere l'app alla Home (iPhone, Safari)

1. Apri l'URL in **Safari**.
2. Tocca **Condividi** (quadrato con freccia in alto).
3. **Aggiungi a Home**.

Da quel momento l'icona apre l'app a un indirizzo fisso: i dati nel
browser restano stabili anche dopo gli aggiornamenti del file.

## Aggiornare un'app mantenendo i dati

Sostituisci il file `index.html` nel repository e fai commit.
L'URL non cambia, quindi lo `localStorage` (i dati) resta intatto.
Per sicurezza, esporta sempre un backup dall'app prima di aggiornare.

## Aggiungere altre app in futuro

Crea una nuova sottocartella con dentro un `index.html`, es. `training/index.html`.
Sarà raggiungibile a `https://TUONOME.github.io/poker/training/`.
Ogni sottocartella ha il suo storage separato (è il comportamento voluto).
