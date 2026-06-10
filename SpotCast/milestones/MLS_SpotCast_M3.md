# SpotCast — Riepilogo Milestone 3

## M3 — Servizio di Deduplicazione

**Stato:** ✅ Completata
**Data di chiusura:** 2026-06-10
**Release:** inclusa in v0.1.0

---

## Obiettivo

Implementare il layer di deduplicazione che impedisce di reinviare aziende già viste nelle esecuzioni precedenti, aggiornare il modello `Business` con i campi temporali di tracciamento, e garantire una gestione robusta del file di storico `seen_firms.json`.

---

## Cosa è stato realizzato

### File prodotti

| File | Posizione | Tipo | Descrizione |
|---|---|---|---|
| `Business.ts` | `src/fetcher/` | Modifica | Aggiunta `first_seen?` e `last_seen?` al modello base |
| `DedupService.ts` | `src/dedup/` | Nuovo | Servizio di deduplicazione con `filter`, `markSeen`, `reset` |
| `DedupService.test.ts` | `tests/dedup/` | Nuovo | Suite completa — 17 test cases |

### Refactoring `Business.ts`

Aggiunta di due campi opzionali al modello base `Business`, senza breaking changes su `GoogleFetcher` o sui test di M2:

```typescript
interface Business {
  // ... campi esistenti invariati ...
  first_seen?: string;   // ISO timestamp — prima rilevazione, mai sovrascritto
  last_seen?: string;    // ISO timestamp — aggiornato ad ogni esecuzione
}
```

**Invariante:** `first_seen` è scritto una volta sola da `DedupService.markSeen()`. `last_seen` viene aggiornato ad ogni chiamata. `GoogleFetcher` non conosce né l'uno né l'altro — entrambi rimangono `undefined` prima della deduplicazione.

### DedupService — Interfaccia pubblica

```typescript
class DedupService {
  constructor(filePath?: string)              // default: cwd/seen_firms.json
  filter(businesses: Business[]): Business[] // solo i non ancora visti
  markSeen(businesses: Business[]): void     // segna come visti + aggiorna timestamp
  reset(): void                              // svuota seen[], aggiorna last_reset
}
```

### Struttura `seen_firms.json`

```json
{
  "seen": ["ChIJN1t_tDeuEmsRUsoyG83frY4", "ChIJdd4hrwug2EcRmSrV3Vo6llQ"],
  "last_reset": null
}
```

Responsabilità unica (DTR-021): solo `place_id` e timestamp di reset. Nessun altro dato.

### Comportamenti garantiti

| Comportamento | Implementazione |
|---|---|
| Lazy loading | File letto al primo utilizzo, non nel costruttore |
| File mancante | Struttura vuota creata automaticamente — nessun crash |
| JSON corrotto | Warning su logger, riparte da struttura vuota — nessun crash |
| Scrittura sincrona | `fs.writeFileSync` — dati su disco anche su crash immediato |
| `first_seen` immutabile | Preservato se già valorizzato — non sovrascritto |
| `last_seen` aggiornabile | Sempre impostato al timestamp dell'esecuzione corrente |
| No duplicati in `seen[]` | Set in memoria durante `markSeen()` — deduplicazione idempotente |

---

## Test Suite — 17 casi, 0 failure

```
DedupService
  filter
    ✓ returns all businesses when store is empty
    ✓ returns empty array when all businesses are already seen
    ✓ returns only unseen businesses on partial overlap
  markSeen
    ✓ persists place_ids to disk — verified by a fresh instance
    ✓ sets first_seen on each business
    ✓ sets last_seen on each business
    ✓ does NOT overwrite first_seen on second call
    ✓ updates last_seen on second call while preserving first_seen
    ✓ does not add duplicate place_ids to seen array
  reset
    ✓ empties the seen array
    ✓ sets last_reset to a valid ISO timestamp
    ✓ persists the empty state so a fresh instance sees it
  file handling
    ✓ creates seen_firms.json automatically when missing
    ✓ recovers from corrupt JSON without crashing
    ✓ logs a warning when JSON is corrupt
    ✓ writes correctly after recovering from corrupt file
    ✓ treats seen as empty when file has invalid structure
```

**Strategia di test:** file reali in `os.tmpdir()` — nessun mock di `fs`. Ogni test ha una directory temporanea isolata, eseguibili in parallelo senza interferenze.

---

## Decisioni tecniche chiave in M3

| DTR | Decisione |
|---|---|
| DTR-020 | `first_seen` nel modello `Business` — scritto da `DedupService`, mai da `GoogleFetcher` |
| DTR-021 | `seen_firms.json` con responsabilità unica — solo `place_id` |
| DTR-022 | `last_seen` nel modello `Business` — aggiornato ad ogni run (nuovo in M3) |
| DTR-023 | Path iniettabile nel costruttore — testability senza mock del filesystem |
| DTR-024 | Lazy loading + scrittura sincrona — robustezza e semplicità |

**Lezione appresa (aggiornamento DTR-008):** le callback `it()` che contengono `await` (es. import dinamici) devono essere dichiarate `async`. Errore rilevato e corretto al primo run dei test.

---

## Ordine delle operazioni nella pipeline (vincolo per M6)

```typescript
const fresh = dedup.filter(fetched);   // 1. filtra i nuovi
await mailer.send(fresh);              // 2. invia
dedup.markSeen(fresh);                 // 3. segna come visti — SEMPRE dopo l'invio
```

`markSeen()` riceve solo i Business effettivamente inviati — mai il risultato grezzo del `GoogleFetcher`. Questo vincolo è documentato in DTR-020 e dovrà essere rispettato dall'orchestratore in M6.

---

## Note per M4

M4 implementerà `ExcelExporter`. I campi `first_seen` e `last_seen` ora disponibili nel modello `Business` possono essere inclusi come colonne opzionali nel foglio **Businesses** del file Excel, fornendo all'utente finale visibilità sulla storia dei lead.

Il `DedupService` è pronto per essere integrato nella pipeline — interfaccia stabile, nessuna dipendenza da M4/M5.
