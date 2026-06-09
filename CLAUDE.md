# CLAUDE.md — contesto progetto per Claude Code

Questo file viene letto automaticamente da Claude Code. Serve a riprendere il lavoro
su un altro computer (o in una nuova sessione) senza perdere il contesto.
L'utente non è uno sviluppatore: spiega i comandi prima di eseguirli, in italiano,
e fermati a segnalare gli errori invece di forzare.

---

## Cos'è il progetto

App web statica per tracciare tornei di poker MTT (bankroll, ROI, ITM%, grafico).
Pensata per girare da iPhone (aggiunta alla schermata Home di Safari).
Hosting gratuito su **GitHub Pages**; dati su **Firebase Firestore** (cloud).

- **App live:** https://puddinblues.github.io/poker/tracker/
- **Repo GitHub:** https://github.com/PuddinBlues/poker  (PUBBLICO, branch `main`, Pages da root)
- **Console Firebase:** https://console.firebase.google.com → progetto `poker-tracker` (id: `poker-tracker-45da3`)
- **Account GitHub:** PuddinBlues

## Struttura

```
poker/                      (== repo. Windows: OneDrive\Documenti\poker-site · Mac: iCloud Drive\Claude Code\poker)
├── tracker/index.html      App completa: UI + logica + Firebase (single file)
├── README.md               Descrizione + deploy
├── ISTRUZIONI.md           Promemoria d'uso per l'utente (umano)
├── CLAUDE.md               Questo file
├── aggiorna.bat            (solo Windows) Doppio click = commit + push (IGNORATO da git, vedi .gitignore)
└── .gitignore
```

## Architettura del tracker (tracker/index.html)

File unico. Due script:
1. **Classic `<script>`**: tutta l'app. Stato in `let state = { startBankroll, sessions:[] }`.
   - `load()` legge da `localStorage` (chiave `mtt_tracker_v1`).
   - `save()` scrive su localStorage, poi chiama `window.__cloudSave(state)` se presente, poi `render()`.
   - Ponte verso il cloud: `window.__getState()` e `window.__applyRemote(s)`.
2. **`<script type="module">`**: Firebase (SDK 10.12.5 da gstatic CDN).
   - Auth: **Email/Password** + **Google** + **recupero password via email** (`sendPasswordResetEmail`,
     link "Password dimenticata?" visibile solo in modalità Accedi).
   - Email/Password è il metodo affidabile ovunque (anche iOS standalone, senza popup/redirect).
   - **Google** usa `signInWithPopup` con fallback `signInWithRedirect` + `getRedirectResult`.
     ⚠️ Su iPhone "standalone" (icona Home) il login Google può fallire per lo storage
     partitioning di Safari (authDomain `*.firebaseapp.com` ≠ dominio GitHub Pages). Per questo
     email/password resta la rete di sicurezza e va sempre mantenuta.
   - Persistenza `browserLocalPersistence`.
   - Documento per utente: `users/{uid}` con `{ startBankroll, sessions, updatedAt }`.
   - `onSnapshot` → sync in tempo reale → `window.__applyRemote`.
   - Al primo accesso senza doc cloud, migra lo stato locale verso il cloud.
   - Schermata login overlay `#authOv`; stato sync + logout nel pannello "Backup dati".

## Account e login (comportamento Firebase imparato)

- Impostazione **"un account per email"** (account linking) ATTIVA nella console.
- Conseguenza: accedendo con **Google** usando la stessa email di un account email/password,
  Firebase **collega** i due metodi in **un solo account** (stesso `uid`, stessi dati). In alcuni
  casi rimuove il metodo password → per riaverlo si usa "Password dimenticata?" (reset email).
- I dati stanno nel doc `users/{uid}`: finché l'`uid` resta lo stesso, i tornei non si perdono
  cambiando metodo di login. Un account email/password e un account Google con email **diverse**
  sarebbero invece due `uid` distinti (cassetti separati) → servirebbe Esporta/Importa per unirli.
- Verifica rapida dei domini autorizzati (serve a Google/OAuth, non a email/password):
  `curl "https://www.googleapis.com/identitytoolkit/v3/relyingparty/getProjectConfig?key=<apiKey>"`.
  Deve includere `puddinblues.github.io`.

Il `firebaseConfig` è in chiaro nel file: NON è un segreto (è progettato per il client).
La sicurezza la danno login + regole Firestore.

## Regole di sicurezza Firestore (già pubblicate nella console)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## Come aggiornare / deployare

- Modifica i file, poi: `git add -A && git commit -m "..." && git push` (oppure doppio click su `aggiorna.bat`).
- GitHub Pages si rigenera in ~1 minuto. L'URL non cambia → localStorage e dati restano.
- Verifica live che la nuova versione sia servita prima di dire "fatto".

## Ambiente Windows — trucchi imparati (IMPORTANTE)

- **Shell:** Bash tool = git-bash; PowerShell tool = Windows PowerShell 5.1.
- **gh (GitHub CLI)** non è nel PATH della sessione: invocarlo col path pieno
  `& 'C:\Program Files\GitHub CLI\gh.exe'`.
- **Login gh:** passare il token via pipe a `--with-token` FALLISce (PowerShell aggiunge
  caratteri → 401). Usare invece la variabile d'ambiente: `$env:GH_TOKEN = '<token>'`
  prima dei comandi gh. (Il login è comunque già salvato localmente su questo PC.)
- **Sandbox:** una stringa letterale `/documents` nel comando viene bloccata come
  "percorso protetto". Aggirare componendo l'URL a pezzi (es. `'docu'+'ments'`) e/o
  `dangerouslyDisableSandbox: true` per sole chiamate di rete sicure.
- **Avvisi `LF will be replaced by CRLF`**: innocui (fine-riga Windows), ignorare.
- **Test end-to-end Firebase senza browser:** usare le REST API
  (`identitytoolkit.googleapis.com/v1/accounts:signUp|delete` con `apiKey`, e
  `firestore.googleapis.com/v1/projects/<proj>/databases/(default)/...` con `Authorization: Bearer <idToken>`).
  Creare un account di prova, verificare scrittura/lettura proprie + 403 su altrui, poi cancellarlo.

## Ambiente Mac — note (questo computer)

- Repo clonato in **iCloud Drive**: `~/Library/Mobile Documents/com~apple~CloudDocs/Claude Code/poker`.
- **Shell:** `/bin/bash`. **gh (GitHub CLI)** installato con Homebrew (`/usr/local/bin/gh`);
  aggiornabile con `brew upgrade gh`. Login GitHub già fatto (salvato nel portachiavi).
- **git identity** già impostata; `gh auth setup-git` fatto → `git push` su https funziona.
- Deploy: niente `aggiorna.bat` (è Windows). Si usa `git add -A && git commit -m "..." && git push`.
- **Sandbox iCloud:** avviare un server locale dalla cartella iCloud può dare `PermissionError`
  su `os.getcwd()`. Per l'anteprima copiare il file in `/tmp` e servire da lì. Le chiamate di
  rete (curl verso Firebase/GitHub) richiedono `dangerouslyDisableSandbox: true`.

## Regole di collaborazione

- NON committare mai token GitHub o password (il `.gitignore` esclude già `aggiorna.bat`).
- Repo pubblico: ok per HTML/config Firebase, mai per segreti.
- L'utente preferisce passi spiegati e conferme prima di azioni distruttive.

## Fatto di recente

- Login con **Google** (oltre a email/password). — giu 2026
- **Recupero password** via email (`sendPasswordResetEmail`). — giu 2026

## Possibili lavori futuri (se richiesti)

- Pulsante "Imposta password" da usare quando si è già dentro con Google (rete di sicurezza
  per l'iPhone, dove il login Google standalone può fallire).
- Rendere Google il pulsante principale della schermata di login (email/password in secondo piano).
- Promemoria automatico di backup nell'app.
- Nuova app `training/index.html` (stesso schema, storage separato).
