# SpotCast — Riepilogo Milestone 6
## M6 — Scheduler + Pipeline Completa

**Stato:** ✅ Completata con errata corrige
**Data di chiusura:** 2026-06-12
**Release:** inclusa in v0.1.0-beta

---

## Obiettivo

Implementare lo scheduler cron e orchestrare la pipeline completa M1→M6:
fetch → dedup → export Excel → invio email → markSeen.

---

## Cosa è stato realizzato

### File prodotti

| File | Posizione | Descrizione |
|---|---|---|
| `SpotCast.ts` | `src/` | Entry point, orchestratore pipeline, gestione CLI |
| `SpotCast.test.ts` | `tests/` | 6 test di integrazione sulla pipeline |

### Modalità CLI implementate

```bash
node SpotCast.js                  # single run, termina
node SpotCast.js --daemon         # daemon con cron, resta in vita
node SpotCast.js --daemon --now   # daemon + esecuzione immediata al primo avvio
node SpotCast.js --reset          # azzera seen_firms.json
```

### Pipeline orchestrata

```
1. GoogleFetcher.fetchAll()        → Business[]
2. DedupService.filter()           → Business[] nuovi
3. Se array vuoto → log + stop
4. ExcelExporter.export()          → path .xlsx
5. MailService.send()              → email con allegato
6. DedupService.markSeen()         → aggiorna seen_firms.json
7. Log riassuntivo
```

### Gestione errori

- Single run → `process.exit(1)` su errore fatale
- Daemon → logga l'errore, resta in vita per il run successivo
- `markSeen` chiamato **solo dopo** l'invio email — mai prima

---

## Test

| Metrica | Risultato |
|---|---|
| Test suite complessiva | 159 test, tutti verdi |
| File di test | 9 suite |
| Durata | 4.56s |

---

## Problemi riscontrati in fase di test end-to-end

### Problema 1 — SMTP Outlook blocca l'autenticazione base

Microsoft ha disabilitato l'autenticazione SMTP base (`535 5.7.139 Authentication unsuccessful`) per tutti gli account `@outlook.com` personali dal 2024. Richiede OAuth2 che comporta registrazione app su Azure Active Directory — effort non giustificato per il test.

**Soluzione adottata:** Gmail con App Password per il test end-to-end. Onur in fase di consegna utilizzerà le proprie credenziali SMTP.

### Problema 2 — `results_per_run` globale nasconde categorie

Con `results_per_run: 10` impostato come limite globale di pipeline, la prima categoria interrogata (es. Tabacchino) esauriva il budget di risultati prima che Bar e Fabbro venissero interrogati. Test su Tolmezzo e Socchieve: 10 risultati, tutti tabacchini, zero bar e fabbri nonostante esistano.

**Causa:** design del limite come cap globale invece che per combinazione categoria×città.

### Problema 3 — Google Places API: limite strutturale di 20 risultati per query

Google Places Text Search restituisce massimo 20 risultati per query. Con paginazione (`next_page_token`) si arriva a 60 — tre pagine fisse, delay obbligatorio tra le pagine, costo triplicato. Per comuni con molte attività (es. Berlin) il limite è insufficiente. Per comuni piccoli (es. Socchieve) è sufficiente ma l'architettura rimane fragile.

Questo limite non è aggirabile con la Text Search API — è una limitazione strutturale di Google imposta lato server.

**Decisione:** abbandono di Google Places API in favore di HERE Browse API (vedi M7). HERE non ha limiti arbitrari per query, supporta paginazione reale con `offset`, ha copertura capillare Europa e dati stabili garantiti da contratti enterprise.

---

## Note per M7

`GoogleFetcher.ts` viene deprecato ma non rimosso — utile per confronto e rollback. La sostituzione con `HereFetcher.ts` è l'obiettivo di M7 e non richiede modifiche al resto della pipeline.
