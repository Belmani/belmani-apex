# SpotCast — Grooming

# GRM-M1 — Scaffold e Configurazione

## Obiettivo
Impostare lo scheletro del progetto, la configurazione TypeScript, la gestione delle dipendenze, il caricamento della configurazione, la gestione delle variabili d'ambiente e il logging di base. Tutto il resto costruisce su questa fondamenta.

## User Story
*Come sviluppatore, voglio una struttura di progetto pulita e tipizzata, così da poter costruire ogni modulo con fiducia e coerenza.*

## Criteri di Accettazione
- [ ] `package.json` con tutte le dipendenze necessarie dichiarate
- [ ] `tsconfig.json` configurato per Node.js 18, strict mode abilitato
- [ ] Entry point `SpotCast.ts` esiste, compila ed esegue senza errori
- [ ] `ConfigLoader.ts` carica `config.json`, valida i campi obbligatori con **Zod**, lancia errori descrittivi su valori mancanti o non validi
- [ ] Le variabili d'ambiente `GOOGLE_API_KEY`, `SMTP_USER`, `SMTP_PASS` sovrascrivono i campi corrispondenti in config quando presenti
- [ ] `config.example.json` committato nel repo con tutti i campi, nessun valore reale
- [ ] `config.json` e `seen_firms.json` presenti nel `.gitignore`
- [ ] Logger **Winston** configurato: scrive su `tracker.log` e su console (console silenziata in produzione)
- [ ] Formato log: `[YYYY-MM-DD HH:mm:ss] LEVEL: message` — tutto in inglese
- [ ] `npm run dev` avvia l'app tramite **ts-node** + **nodemon** senza step di compilazione
- [ ] `npm run build` compila in `/dist` senza errori
- [ ] `npm start` esegue il build compilato
- [ ] `npm test` esegue la suite Vitest e stampa il report di copertura

## Test (Vitest)

| Caso | Tipo | Descrizione |
|---|---|---|
| Config valida completa | Unit | `ConfigLoader` restituisce oggetto tipizzato correttamente |
| Campo obbligatorio mancante | Unit | Lancia errore con nome del campo mancante |
| `google_api_key` vuota | Unit | Lancia errore descrittivo |
| Override da variabile d'ambiente | Unit | `GOOGLE_API_KEY` env sovrascrive il valore in config |
| `language` non supportata | Unit | Zod rifiuta con enum error |
| `email_to` array vuoto | Unit | Zod rifiuta con messaggio esplicito |

## Dipendenze npm

```bash
# Runtime
npm install zod winston express node-cron nodemailer exceljs handlebars
npm install @googlemaps/google-maps-services-js

# Dev
npm install -D typescript ts-node nodemon vitest @types/node
npm install -D @types/nodemailer @types/node-cron @types/handlebars
```

## Note Tecniche
- **Zod** per la validazione — schema dichiarativo, tipo inferito automaticamente, errori campo per campo
- **Winston** per il logging — livelli `error/warn/info/debug`, transport file + console
- **ts-node** + **nodemon** — hot-reload in sviluppo, riavvio automatico ad ogni salvataggio
- Tutti gli identificatori, nomi di proprietà e variabili d'ambiente in inglese — nessuna traduzione

## Stima di Effort
**3–4 ore** (inclusi test)

---