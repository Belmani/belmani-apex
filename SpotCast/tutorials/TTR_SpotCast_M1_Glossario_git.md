# Glossario Git & GitHub

## Per chi parte da zero — o quasi

---

## Concetti fondamentali

### Repository (repo)

È la cartella del progetto sotto controllo di versione. Contiene tutto il codice, la storia di ogni modifica mai fatta, e le impostazioni del progetto. Esiste in due posti contemporaneamente: **in locale** sul tuo computer e **in remoto** su GitHub.

### Git

Il programma che gira sul tuo computer e tiene traccia di tutte le modifiche al codice. Funziona anche senza internet — il collegamento a GitHub è opzionale.

### GitHub

Il sito web che ospita una copia remota della repo. Serve per condividere il codice con altri, fare backup, e collaborare. Git ≠ GitHub: Git è lo strumento, GitHub è il servizio.

### Commit

Una "fotografia" del codice in un momento preciso. Ogni commit ha un messaggio che descrive cosa è cambiato (es. `feat: add Google Places fetcher`). La storia del progetto è una sequenza di commit.

### Branch

Una linea di sviluppo parallela e indipendente. Immagina il codice come un albero: il tronco è `main`, i rami sono i branch. Puoi lavorare su un branch senza toccare il tronco — quando hai finito, "reinnesti" il ramo sul tronco.

### main

Il branch principale — il tronco dell'albero. Contiene sempre il codice stabile e funzionante. Su SpotCast nessuno può scrivere direttamente su `main` senza passare da una PR (vedi sotto).

### Branch di feature

Un branch temporaneo creato per sviluppare una singola funzionalità. Si crea, si lavora, si mergía su `main`, si cancella. Esempio: `feature/google-fetcher`, `feature/excel-export`.

---

## Operazioni Git

### Clone

Scarica una repo da GitHub sul tuo computer per la prima volta.

```bash
git clone https://github.com/Belmani/SpotCast.git
```

### Add

Dice a Git "voglio includere queste modifiche nel prossimo commit".

```bash
git add .          # aggiunge tutto
git add src/app.ts # aggiunge un file specifico
```

### Commit

Scatta la fotografia delle modifiche aggiunte con `add`.

```bash
git commit -m "feat: add config loader"
```

### Push

Carica i tuoi commit locali su GitHub (dal computer al cloud).

```bash
git push origin main
```

### Pull

Scarica da GitHub le modifiche fatte da altri (dal cloud al computer).

```bash
git pull origin main
```

### Merge

Unisce le modifiche di un branch in un altro. Quando una feature è pronta, si "mergía" il branch di feature su `main`.

### Rebase

Alternativa al merge — riscrive i commit del tuo branch come se li avessi scritti partendo dall'ultima versione di `main`. Produce una storia più lineare ma è un'operazione più avanzata.

---

## PR — Pull Request ✅

**PR = Pull Request** (non Push Request).

È una **richiesta formale di integrare le modifiche di un branch in main**. Il flusso è:

```
1. Crei un branch di feature sul tuo computer
2. Sviluppi e fai i tuoi commit
3. Fai push del branch su GitHub
4. Apri una Pull Request su GitHub
   → stai dicendo: "ho finito, qualcuno controlli e approvi"
5. Il collaboratore revisiona il codice
6. Se tutto va bene, la PR viene approvata e mergiata su main
7. Il branch di feature viene cancellato
```

La PR non ha nulla a che fare con il "push" come operazione Git. Il nome viene dal concetto di "tirare" (pull) le modifiche verso main — è main che "tira dentro" le tue modifiche, non tu che le "spingi" direttamente.

**Perché usarle anche in un team di due?**

- Ogni modifica a main passa da una review — anche veloce
- La storia di main rimane pulita e tracciabile
- Si evitano conflitti non gestiti
- È la pratica professionale standard — ci si abitua subito

---

## CI/CD

**CI = Continuous Integration** Ogni volta che viene aperta una PR, un sistema automatico (es. GitHub Actions) esegue i test, il linting e la compilazione. Se qualcosa fallisce, la PR viene bloccata. Su SpotCast useremo Vitest per i test automatici.

**CD = Continuous Deployment** Estensione della CI: se i test passano, il codice viene deployato automaticamente in produzione. Non rilevante per SpotCast (è un tool locale), ma è il passo successivo naturale per applicazioni web.

---

## GitHub Actions

Il sistema di automazione integrato in GitHub. Permette di definire pipeline (chiamate "workflows") che si eseguono automaticamente su eventi specifici (es. "ogni volta che viene aperta una PR, esegui `npm test`"). I workflows sono file `.yml` nella cartella `.github/workflows/`.

---

## Ruleset / Branch Protection

Le regole che governano cosa si può fare su un branch. Su SpotCast abbiamo configurato:

- Nessuno può fare push diretto su `main` — tutto passa da PR
- Nessuno può fare force push (riscrittura forzata della storia)

---

## Force Push

Un push che riscrive la storia di un branch su GitHub, sovrascrivendo i commit precedenti. È un'operazione distruttiva — può cancellare il lavoro di altri. Su `main` è sempre vietato.

---

## Squash

Tecnica che comprime tutti i commit di una PR in un unico commit prima del merge. Utile per tenere la storia di `main` pulita: invece di vedere 15 commit intermedi ("fix typo", "try again", "ok now it works"), se ne vede uno solo con il messaggio finale significativo.

---

## .gitignore

File che dice a Git quali file e cartelle ignorare — non tracciare, non committare. Su SpotCast ignoriamo `config.json` (contiene dati personali), `node_modules/` (troppo grande, si rigenera con `npm install`), e `tracker.log` (log locale).

---

## Semantic Versioning (SemVer)

Convenzione per i numeri di versione: `MAJOR.MINOR.PATCH`

|Parte|Quando si incrementa|Esempio|
|---|---|---|
|`MAJOR`|Cambiamenti incompatibili con versioni precedenti|`1.0.0 → 2.0.0`|
|`MINOR`|Nuove funzionalità compatibili con versioni precedenti|`1.0.0 → 1.1.0`|
|`PATCH`|Bugfix compatibili con versioni precedenti|`1.0.0 → 1.0.1`|

`0.x.x` significa "in sviluppo attivo, non ancora stabile per il pubblico generale". SpotCast parte da `0.1.0` e arriverà a `1.0.0` alla prima release pubblica su Softonic.

---

## Convenzione dei messaggi di commit

Su SpotCast seguiamo la convenzione **Conventional Commits**:

| Prefisso    | Quando usarlo                                      |
| ----------- | -------------------------------------------------- |
| `feat:`     | Nuova funzionalità                                 |
| `fix:`      | Correzione di un bug                               |
| `chore:`    | Manutenzione, dipendenze, configurazione           |
| `docs:`     | Solo documentazione                                |
| `test:`     | Aggiunta o modifica di test                        |
| `refactor:` | Riscrittura di codice senza cambiare comportamento |
| `style:`    | Formattazione, linting (nessun cambiamento logico) |

Esempi:

```
feat: add Google Places fetcher
fix: handle empty results from API
docs: update README installation steps
test: add unit tests for DedupService
chore: upgrade ExcelJS to v4.4.0
```