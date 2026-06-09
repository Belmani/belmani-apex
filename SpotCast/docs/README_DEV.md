# SpotCast — README per Sviluppatori

> Riferimento tecnico per contributori e manutentori.

---

## Stack Tecnologico

|Livello|Tecnologia|Versione|
|---|---|---|
|Runtime|Node.js|>=20.0.0|
|Linguaggio|TypeScript|^5.5.0|
|Gestore pacchetti|pnpm|10.30.2|
|Server HTTP|Express|^4.21.0|
|Google Maps|`@googlemaps/google-maps-services-js`|^3.4.2|
|Generazione Excel|ExcelJS|^4.4.0|
|Invio email|Nodemailer|^6.9.0|
|Pianificatore|node-cron|^3.0.3|
|Validazione config|Zod|^3.23.0|
|Logging|Winston|^3.13.0|
|Testing|Vitest|^2.1.0|
|Linting|ESLint|^9.0.0|

---

## Struttura del Progetto

```
spotcast/
│
├── SpotCast.ts              ← punto di ingresso (modalità CLI e daemon)
├── config.json              ← configurazione utente (in gitignore)
├── config.example.json      ← template incluso nel repo
├── seen_firms.json          ← storico deduplicazione (in gitignore)
├── tracker.log              ← log di esecuzione (in gitignore)
├── eslint.config.mjs        ← configurazione flat ESLint 9
├── vitest.config.ts         ← configurazione Vitest
├── nodemon.json             ← config hot-reload nodemon
│
├── src/
│   ├── config/
│   │   └── ConfigLoader.ts      ← carica, valida e unisce config + variabili d'ambiente
│   ├── fetcher/
│   │   ├── Business.ts          ← modelli Business, EnrichedBusiness, Metadata
│   │   └── GoogleFetcher.ts     ← connettore Google Places API
│   ├── dedup/
│   │   └── DedupService.ts      ← gestione seen_firms.json (M3)
│   ├── excel/
│   │   └── ExcelExporter.ts     ← generazione .xlsx (M4)
│   ├── mailer/
│   │   └── MailService.ts       ← invio email (M5)
│   ├── scheduler/
│   │   └── Scheduler.ts         ← wrapper node-cron (M6)
│   ├── api/
│   │   └── RestServer.ts        ← REST API Express porta 3847 (M7)
│   ├── logger.ts                ← logger Winston condiviso
│   └── i18n/
│       ├── it.json / en.json / de.json / fr.json / es.json
│       └── pt.json / zh.json / ja.json / ar.json / hi.json
│
├── templates/
│   └── email.html               ← template email Handlebars
│
├── results/                     ← output Excel (in gitignore)
├── tests/
│   ├── config/
│   │   └── ConfigLoader.test.ts
│   └── fetcher/
│       └── GoogleFetcher.test.ts
└── docs/
    ├── DTR.md
    ├── DTR_EN.md
    ├── GROOMING_M1_M6.md
    ├── MILESTONE_M1.md
    ├── MILESTONE_M2.md
    └── TUTORIAL_*.md
```

---

## Configurazione dell'Ambiente di Sviluppo

```bash
# Clone
git clone https://github.com/Belmani/SpotCast.git
cd SpotCast

# Installa pnpm se necessario
npm install -g pnpm@10.30.2

# Installa le dipendenze
pnpm install

# Copia la configurazione
cp config.example.json config.json
# Inserisci google_api_key e credenziali SMTP

# Avvia in modalità sviluppo (hot-reload)
pnpm dev

# Build di produzione
pnpm build && pnpm start
```

---

## Script Disponibili

```bash
pnpm dev            # ts-node + hot-reload nodemon
pnpm build          # compila TypeScript → dist/
pnpm start          # esegue la build compilata
pnpm test           # Vitest in modalità watch
pnpm test:run       # esecuzione singola, senza watch
pnpm test:coverage  # report di copertura in /coverage
pnpm lint           # analisi ESLint
pnpm lint:fix       # correzione automatica problemi ESLint
pnpm typecheck      # controllo tipi senza build
pnpm clean          # elimina dist/
```

---

## Variabili d'Ambiente

I valori sensibili sovrascrivono i corrispondenti campi di `config.json` quando presenti:

|Variabile|Campo config|
|---|---|
|`GOOGLE_API_KEY`|`google_api_key`|
|`SMTP_USER`|`smtp.user`|
|`SMTP_PASS`|`smtp.pass`|

```bash
# macOS / Linux
export GOOGLE_API_KEY="AIzaSy..."

# Windows
setx GOOGLE_API_KEY "AIzaSy..."
```

---

## Moduli Principali

### ConfigLoader (`src/config/ConfigLoader.ts`)

Carica `config.json`, valida con lo schema Zod, applica le sovrascritture delle variabili d'ambiente. Gli errori fatali vengono scritti su `process.stderr` — nessuna dipendenza da Winston nella fase di avvio, per evitare importazioni circolari.

```typescript
export type Config = z.infer<typeof ConfigSchema>; // completamente tipizzato, inferito automaticamente
export function loadConfig(configPath = 'config.json'): Config
```

### GoogleFetcher (`src/fetcher/GoogleFetcher.ts`)

Racchiude `@googlemaps/google-maps-services-js`. Per ogni combinazione `(categoria × città)` nella config, esegue `textSearch` e mappa i risultati nel modello interno `Business`. Non lancia mai eccezioni in caso di errore API — restituisce risultati parziali.

```typescript
fetchAll(): Promise<Business[]>           // esecuzione completa rispettando results_per_run
fetchOne(category, city, limit): Promise<Business[]>  // singola combinazione
```

### Modello Business (`src/fetcher/Business.ts`)

```typescript
interface Business {
  place_id: string;     // chiave di deduplicazione
  name: string;
  category: string;     // dalla config, non dalla classificazione di Google
  city: string;
  country: string;
  address: string;
  phone?: string;
  website?: string;
  rating?: number;
  review_count?: number;
  maps_url: string;
}

// Estensibile tramite composizione — nessuna modifica al modello base necessaria per l'arricchimento
interface EnrichedBusiness extends Business {
  metadata: Metadata[];
}
```

### Logger (`src/logger.ts`)

Istanza Winston condivisa. Livelli: `error/warn/info/debug`. Il trasporto su file è sempre attivo; il trasporto su console è disattivato in produzione.

---

## Strategia di Testing (Vitest)

**Obiettivo di copertura: minimo 80% su tutti i moduli.**

```bash
pnpm test:run                    # esegue tutti i test una volta
pnpm test:coverage               # genera report di copertura HTML
```

**Nota sul mocking:** le classi istanziate con `new` devono essere mockate con `function`, non con arrow function. Le arrow function non possono essere usate come costruttori in JavaScript.

```typescript
// Corretto
vi.mock('some-module', () => ({
  SomeClass: function() { return { method: vi.fn() }; }
}));

// Errato — errore "is not a constructor"
vi.mock('some-module', () => ({
  SomeClass: () => ({ method: vi.fn() })
}));
```

---

## Sistema i18n

Tutte le stringhe visibili all'utente sono esternalizzate in `src/i18n/{lang}.json`. La lingua attiva viene letta da `config.json → language`.

**10 lingue al lancio:** `it`, `en`, `de`, `fr`, `es`, `pt`, `zh`, `ja`, `ar`, `hi`

Per aggiungere una nuova lingua:

1. Copia `src/i18n/en.json` → `src/i18n/{lang}.json`
2. Traduci tutti i valori (mantieni le chiavi identiche)
3. Aggiungi il codice lingua all'enum Zod in `ConfigLoader.ts`
4. Apri una PR

---

## Git Flow

```
main          ← solo release stabili, non committare mai direttamente
develop       ← branch di integrazione, tutte le feature si uniscono qui
feature/xxx   ← una feature per branch, parte da develop
release/xxx   ← preparazione release, parte da develop
hotfix/xxx    ← correzioni urgenti su main
```

```bash
git flow feature start my-feature
# ... sviluppo ...
git flow feature finish my-feature  # unisce a develop, elimina il branch
git push origin develop

# Release
git flow release start 0.2.0
git flow release finish -m "v0.2.0 — descrizione" 0.2.0
git push origin main develop --tags
```

---

## Politica di Versioning delle Dipendenze

Le dipendenze sono fissate all'ultima versione **stabile** major. Nessun aggiornamento a RC o versioni beta. Prima di aggiornare una versione major, verifica la compatibilità e documenta la decisione nel DTR.

Vincoli attuali:

- Node.js `>=20.0.0` (pnpm 11 richiede Node 22 — non ancora adottato)
- Express `^4.x` (v5 ancora in RC)
- Zod `^3.x` (v4 non ancora stabile per la produzione)

---

## Checklist di Release

- [ ] Tutti i test passano (`pnpm test:run`)
- [ ] Copertura ≥ 80% (`pnpm test:coverage`)
- [ ] Nessun errore ESLint (`pnpm lint`)
- [ ] Nessun errore TypeScript (`pnpm typecheck`)
- [ ] `config.example.json` aggiornato se sono stati aggiunti nuovi campi
- [ ] `seen_firms.json` e `config.json` in `.gitignore`
- [ ] `CHANGELOG.md` aggiornato
- [ ] Versione aggiornata in `package.json`
- [ ] Riepilogo milestone aggiunto in `docs/`