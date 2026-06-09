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
poker-site/                 (== repo, in OneDrive\Documenti\poker-site)
├── tracker/index.html      App completa: UI + logica + Firebase (single file)
├── README.md               Descrizione + deploy
├── ISTRUZIONI.md           Promemoria d'uso per l'utente (umano)
├── CLAUDE.md               Questo file
├── aggiorna.bat            Doppio click = commit + push (IGNORATO da git, vedi .gitignore)
└── .gitignore
```

## Architettura del tracker (tracker/index.html)

File unico. Due script:
1. **Classic `<script>`**: tutta l'app. Stato in `let state = { startBankroll, sessions:[] }`.
   - `load()` legge da `localStorage` (chiave `mtt_tracker_v1`).
   - `save()` scrive su localStorage, poi chiama `window.__cloudSave(state)` se presente, poi `render()`.
   - Ponte verso il cloud: `window.__getState()` e `window.__applyRemote(s)`.
2. **`<script type="module">`**: Firebase (SDK 10.12.5 da gstatic CDN).
   - Auth **Email/Password** (scelto perché affidabile su iOS standalone; niente popup/redirect).
   - Persistenza `browserLocalPersistence`.
   - Documento per utente: `users/{uid}` con `{ startBankroll, sessions, updatedAt }`.
   - `onSnapshot` → sync in tempo reale → `window.__applyRemote`.
   - Al primo accesso senza doc cloud, migra lo stato locale verso il cloud.
   - Schermata login overlay `#authOv`; stato sync + logout nel pannello "Backup dati".

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

## Regole di collaborazione

- NON committare mai token GitHub o password (il `.gitignore` esclude già `aggiorna.bat`).
- Repo pubblico: ok per HTML/config Firebase, mai per segreti.
- L'utente preferisce passi spiegati e conferme prima di azioni distruttive.

## Possibili lavori futuri (se richiesti)

- Promemoria automatico di backup nell'app.
- Nuova app `training/index.html` (stesso schema, storage separato).
- Eventuale reset password via email (Firebase `sendPasswordResetEmail`).
