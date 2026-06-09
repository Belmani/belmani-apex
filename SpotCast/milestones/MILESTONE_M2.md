# SpotCast — Riepilogo Milestone 2

## M2 — Google Places Fetcher

**Stato:** ✅ Completata
**Data di chiusura:** 2026-06-04
**Release:** inclusa in v0.1.0

---

## Obiettivo

Implementare il connettore Google Places API con modello dati, logica di fetching, test suite completa e allineamento della toolchain a zero warning ESLint.

---

## Cosa è stato realizzato

### File prodotti

| File | Posizione | Descrizione |
|---|---|---|
| `Business.ts` | `src/fetcher/` | Modello dati `Business`, `EnrichedBusiness`, `Metadata`, `MetadataKey` |
| `GoogleFetcher.ts` | `src/fetcher/` | Connettore Google Places API con `fetchAll()` e `fetchOne()` |
| `ConfigLoader.ts` | `src/config/` | Caricamento e validazione config con Zod, merge env vars |
| `logger.ts` | `src/` | Logger Winston condiviso |
| `GoogleFetcher.test.ts` | `tests/fetcher/` | 17 test cases su mapping, error handling, limit enforcement |
| `ConfigLoader.test.ts` | `tests/config/` | Test su validazione schema, file errors, env var overrides |
| `vitest.config.ts` | root | Configurazione Vitest con coverage thresholds |
| `eslint.config.mjs` | root | ESLint 9 flat config in formato ESM |
| `.npmrc` | root | Configurazione pnpm |

### Modello dati

```typescript
interface Business {
  place_id: string;     // chiave deduplicazione — stabile e univoca
  name: string;
  category: string;     // come da config, non classificazione Google
  city: string;
  country: string;
  address: string;
  phone?: string;
  website?: string;
  rating?: number;
  review_count?: number;
  maps_url: string;
}

interface Metadata {
  key: string;          // MetadataKey enum o stringa custom
  value: string;
  source?: string;
  collected_at?: string;
}

interface EnrichedBusiness extends Business {
  metadata: Metadata[];  // composizione per estensibilità futura
}
```

### GoogleFetcher — comportamento

- Esegue `textSearch` per ogni combinazione `(category × city)` in config
- Rispetta `results_per_run` — si ferma quando il limite è raggiunto
- Gestisce errori API senza crashare — restituisce array parziale
- Filtra risultati senza `place_id` o `name`
- Usa `PlaceData` dell'SDK Google — zero `any`

### ConfigLoader — comportamento

- Valida `config.json` con schema Zod — errori campo per campo
- Override sicuro da variabili d'ambiente per i segreti
- Errori fatali scritti su `process.stderr` — nessuna dipendenza da Winston al boot
- Tipo `Config` inferito automaticamente dallo schema

---

## Qualità del codice

| Metrica | Risultato |
|---|---|
| ESLint errors | 0 |
| ESLint warnings | 0 |
| TypeScript errors | 0 |
| Test cases | 17 (GoogleFetcher) + 16 (ConfigLoader) = 33 |

---

## Decisioni tecniche chiave in M2

- **Composizione vs ereditarietà** — scelto `Metadata[]` per estensibilità (DTR-009)
- **Nessun `IFetcher`** — astrazione speculativa scartata (DTR-002)
- **`PlaceData` dell'SDK** al posto di `any` — type safety totale
- **`process.stderr.write`** in ConfigLoader — evita dipendenza circolare con Winston
- **Mocking Vitest** — `function` obbligatoria per mock di costruttori (non arrow function)
- **pnpm** al posto di npm — migrazione completata (DTR-017)
- **ESLint 9 flat config** in `.mjs` — nessun `require()` nel codebase (DTR-018)
- **Versioni stabili** per tutte le dipendenze — politica di versioning concordata (DTR-016)

---

## Note per M3

M3 implementerà `DedupService` — il servizio di deduplicazione basato su `seen_firms.json` con `place_id` come chiave (DTR-004). Il modello `Business` prodotto in M2 è già compatibile: `place_id` è campo obbligatorio non nullable.
