# Poker Tracker — Istruzioni e promemoria

Promemoria personale per usare, aggiornare e gestire il progetto da qualsiasi computer.
(Non contiene password né token: quelli non vanno mai scritti qui.)

---

## 🔗 Link utili

- **App tracker (da aprire sul telefono):** https://puddinblues.github.io/poker/tracker/
- **Repository GitHub:** https://github.com/PuddinBlues/poker
- **Console Firebase (database):** https://console.firebase.google.com → progetto `poker-tracker`

---

## 📱 Usare il tracker (qualsiasi dispositivo)

1. Apri l'URL dell'app nel browser.
2. **Accedi** in uno di questi modi (è sempre lo stesso account, stessi dati):
   - **Email e password**, oppure
   - **Continua con Google**.
3. Ritrovi tutti i dati sincronizzati in tempo reale.

> Password dimenticata? Nella schermata di accesso scrivi la tua email e tocca **"Password
> dimenticata?"**: ricevi un'email per reimpostarla (controlla anche lo spam).

> ⚠️ Sull'iPhone, **dall'icona della schermata Home**, il login **Google** a volte non va
> (limite di Safari). Se capita, entra con **email e password**: funziona sempre. Per questo
> conviene tenere attivi entrambi i metodi e non dimenticare la password.

Sull'iPhone, per l'icona a tutto schermo: Safari → **Condividi** → **Aggiungi a Home**.

### Dove stanno i dati
- I tornei sono salvati su **Firebase Firestore** (server cloud di Google), non dentro Safari.
- Sopravvivono a: pulizia di Safari, cambio telefono, reinstallazioni.
- Se un giorno i dati "spariscono" dal dispositivo: riapri, **rifai il login** → tornano dal server.
- In basso nell'app, il pannello "Backup dati" mostra lo stato: **"Salvato nel cloud ✓"**.

### Backup extra (consigliato ogni tanto)
- Nell'app: **Esporta backup** → salva il file `.json` in iCloud/File.
- Per ripristinare: **Importa backup** e seleziona quel file.

---

## 🔄 Aggiornare l'app (modificare il codice)

### Su Windows (PC già configurato)
1. Modifica i file nella cartella `poker-site`.
2. Doppio click su **`aggiorna.bat`** → fa commit + push da solo.
3. Dopo ~1 minuto l'app online è aggiornata. L'URL non cambia → i dati restano.

### Su Mac (questo computer)
Niente `aggiorna.bat` (è solo Windows). Si pubblica con i comandi git, oppure chiedendo a
Claude Code di farlo. Dopo ~1 minuto l'app online è aggiornata, l'URL non cambia → i dati restano.

### Comandi git manuali (validi ovunque)
```
git add -A
git commit -m "descrizione modifica"
git push
```

---

## 💻 Continuare da un ALTRO computer

Preparazione una tantum del nuovo computer:

1. **Installa git** (https://git-scm.com) ed eventualmente **GitHub CLI** (https://cli.github.com).
2. **Scarica il progetto:**
   ```
   git clone https://github.com/PuddinBlues/poker.git
   ```
3. **Autenticati a GitHub** (una volta):
   - con GitHub CLI: `gh auth login` (su Mac si installa con Homebrew: `brew install gh`)
   - oppure crea un token su https://github.com/settings/tokens e usalo quando git lo chiede.
4. Da lì modifichi, `git push`, e GitHub Pages si aggiorna.

> Nota: il repo può vivere in una cartella sincronizzata (su Windows **OneDrive**, su Mac
> **iCloud Drive**). Se sull'altro computer hai lo stesso cloud con lo stesso account, la
> cartella compare già sincronizzata e puoi saltare il clone.

---

## 🆕 Aggiungere altre app in futuro

Crea una sottocartella con dentro un `index.html`, es. `training/index.html`.
Sarà raggiungibile a: https://puddinblues.github.io/poker/training/
Ogni sottocartella ha il suo storage separato.

---

## 🔐 Sicurezza (promemoria)

- Il file `firebaseConfig` dentro `tracker/index.html` **non è segreto**: è normale che sia
  pubblico. La protezione dei dati la danno il **login** e le **regole di sicurezza** di Firestore.
- Le regole di sicurezza attuali (in Firebase → Firestore → Rules) garantiscono che
  ogni utente acceda **solo** ai propri dati:
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
- **Non scrivere mai** password o token GitHub dentro i file del progetto.
