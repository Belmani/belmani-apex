# SpotCast — Riepilogo Milestone 1
## M1 — Scaffold e Configurazione

**Stato:** ✅ Completata
**Data di chiusura:** 2026-06-04
**Release:** inclusa in v0.1.0

---

## Obiettivo

Costruire le fondamenta del progetto: struttura cartelle, configurazione TypeScript, gestione dipendenze, caricamento e validazione della configurazione, logging di base e toolchain di sviluppo.

---

## Cosa è stato realizzato

### Struttura del progetto
```
SpotCast/
├── src/
│   ├── api/
│   ├── config/
│   ├── dedup/
│   ├── excel/
│   ├── fetcher/
│   ├── i18n/
│   ├── mailer/
│   ├── scheduler/
│   └── SpotCast.ts
├── templates/
├── tests/
├── results/
├── .gitignore
├── config.example.json
├── nodemon.json
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

### Dipendenze installate

**Runtime:** `zod`, `winston`, `express`, `node-cron`, `nodemailer`, `exceljs`, `handlebars`, `@googlemaps/google-maps-services-js`

**Dev:** `typescript`, `ts-node`, `nodemon`, `vitest`, `@vitest/coverage-v8`, `eslint`, `@typescript-eslint/*`, `rimraf`

### Decisioni tecniche chiave
- **TypeScript** con strict mode — tipizzazione forte, zero `any` tollerati
- **pnpm 10.30.2** come package manager — migrazione da npm (DTR-017)
- **Zod v3** per validazione configurazione (DTR-006)
- **Winston** per logging strutturato (DTR-007)
- **Vitest** per testing con coverage 80% (DTR-008)
- **ESLint 9** con flat config `.mjs` (DTR-018)
- **nodemon + ts-node** per hot-reload in sviluppo

### Configurazione
- `config.example.json` committato come template
- `config.json` gitignored — mai nel repository
- Override da variabili d'ambiente: `GOOGLE_API_KEY`, `SMTP_USER`, `SMTP_PASS`

---

## Comandi disponibili dopo M1

```bash
pnpm dev          # avvio sviluppo con hot-reload
pnpm build        # compilazione TypeScript → dist/
pnpm start        # avvio build compilato
pnpm test         # test in watch mode
pnpm test:run     # test esecuzione singola
pnpm test:coverage # report copertura
pnpm lint         # analisi statica ESLint
pnpm lint:fix     # correzione automatica
pnpm typecheck    # verifica tipi senza build
pnpm clean        # pulizia cartella dist
```

---

## Repository

- **Branch:** `main` protetto, `develop` attivo
- **Git Flow:** inizializzato
- **Release:** `v0.1.0` taggata su main
- **Collaboratori:** AleDeP10 (tech lead), Belmani (product owner)
