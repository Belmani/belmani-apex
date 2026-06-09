# SpotCast — Grooming

# GRM-M3 — Servizio di Deduplicazione (aggiornato post-grooming)

## Obiettivo
Implementare il layer di deduplicazione che impedisce alla stessa azienda di apparire in più esecuzioni. Prima di scrivere `DedupService`, applicare il refactoring di `Business.ts` per aggiungere `first_seen`.

## User Story
*Come utente, voglio ricevere solo le aziende nuove che non ho ancora visto, così la mia inbox non si riempie di ripetizioni.*

---

## Step 0 — Refactoring `Business.ts` (prima di tutto il resto)

Aggiungere `first_seen` al modello `Business`:

```typescript
interface Business {
  place_id: string;
  name: string;
  category: string;
  city: string;
  country: string;
  address: string;
  phone?: string;
  website?: string;
  rating?: number;
  review_count?: number;
  maps_url: string;
  first_seen?: string;    // ISO timestamp — undefined se appena arrivato dal Fetcher
                          // valorizzato da DedupService.markSeen()
}
```

**Perché `string` e non `Date`:**
Tutto il sistema serializza su JSON — stringhe ISO ovunque, zero conversioni inutili.

**Perché opzionale:**
Il `GoogleFetcher` non conosce `first_seen`. Lo aggiunge solo `DedupService.markSeen()` al momento della prima rilevazione. Renderlo obbligatorio forcerebbe il Fetcher a valorizzarlo con `undefined` esplicitamente — rumore inutile.

---

## Criteri di Accettazione

- [ ] `Business.ts` aggiornato con `first_seen?: string`
- [ ] `DedupService.ts` implementato ed esportato
- [ ] `filter(businesses)` restituisce solo i `Business` il cui `place_id` non è in `seen_firms.json`
- [ ] `markSeen(businesses)` aggiunge i `place_id` a `seen_firms.json` e valorizza `first_seen` su ogni `Business`
- [ ] `reset()` svuota `seen[]` e aggiorna `last_reset`
- [ ] Se `seen_firms.json` non esiste → creato automaticamente con struttura vuota
- [ ] Se `seen_firms.json` è corrotto → warning loggato, si riparte da struttura vuota senza crashare
- [ ] `markSeen` viene chiamato **dopo** l'invio email — mai prima
- [ ] Test suite completa (vedi sezione Test)

---

## Struttura `seen_firms.json`

```json
{
  "seen": [
    "ChIJN1t_tDeuEmsRUsoyG83frY4",
    "ChIJdd4hrwug2EcRmSrV3Vo6llQ"
  ],
  "last_reset": "2026-06-04T08:00:00Z"
}
```

**`seen`** — array di `place_id` già inviati. Unica responsabilità del file.
**`last_reset`** — timestamp dell'ultimo `--reset`. Solo informativo.

Nessun altro campo. Statistiche e aggregazioni rimandare a M11+ con SQLite (DTR-019).

---

## Interfaccia pubblica

```typescript
class DedupService {
  filter(businesses: Business[]): Business[]
  markSeen(businesses: Business[]): void
  reset(): void
}
```

---

## Note Tecniche

### Caricamento — lazy, non nel costruttore
Il file viene letto al primo utilizzo, non alla costruzione dell'istanza.
- File mancante → crea struttura vuota automaticamente
- File corrotto → logga warning, riparte da struttura vuota

### Scrittura — sincrona
`fs.writeFileSync` — il file è piccolo (< 1 MB anche dopo anni), la scrittura è istantanea. La sincronia garantisce che i dati siano su disco anche se il processo viene interrotto immediatamente dopo.

### Struttura in memoria
```typescript
interface SeenStore {
  seen: string[];
  last_reset: string | null;
}
```
Caricata una volta, tenuta in memoria, scritta su disco ad ogni modifica. Nessuna rilettura del file ad ogni chiamata.

### Ordine delle operazioni nella pipeline
```typescript
const fresh = dedup.filter(fetched);   // 1. filtra i nuovi
await mailer.send(fresh);              // 2. invia
dedup.markSeen(fresh);                 // 3. segna come viste — SEMPRE dopo l'invio
```
`markSeen` riceve solo i Business effettivamente inviati — mai il risultato grezzo del Fetcher.

---

## Test (Vitest)

I test usano directory temporanee (`os.tmpdir()`) — isolati, nessun file reale toccato, eseguibili in parallelo.

**Perché file reali e non mock di `fs`:**
Mockare il filesystem nasconde bug reali di serializzazione JSON. File temporanei reali sono veloci e affidabili.

| Caso | Tipo | Descrizione |
|---|---|---|
| Storico vuoto | Unit | `filter` restituisce tutti i Business in input |
| Tutti già visti | Unit | `filter` restituisce array vuoto |
| Sovrapposizione parziale | Unit | `filter` restituisce solo i non ancora visti |
| `markSeen` persiste su disco | Unit | dopo `markSeen`, gli stessi Business vengono filtrati in una nuova istanza |
| `markSeen` valorizza `first_seen` | Unit | ogni Business ricevuto ha `first_seen` valorizzato con un ISO timestamp |
| `first_seen` non viene sovrascritto | Unit | se `markSeen` viene chiamato due volte sullo stesso Business, `first_seen` rimane la data originale |
| `reset` svuota `seen` | Unit | dopo `reset`, `filter` restituisce tutti i Business come nuovi |
| `reset` aggiorna `last_reset` | Unit | `last_reset` è un ISO timestamp valido dopo il reset |
| File mancante | Unit | prima esecuzione crea `seen_firms.json` automaticamente |
| File corrotto | Unit | JSON malformato → warning loggato, nessun crash, si riparte da struttura vuota |
| File corrotto non perde i dati successivi | Unit | dopo recovery da file corrotto, `markSeen` scrive correttamente |

---

## Stima di Effort
**2–3 ore** (inclusi refactoring `Business.ts` e test)