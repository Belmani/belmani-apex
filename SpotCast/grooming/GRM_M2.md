# SpotCast — Grooming

# GRM-M2 — Google Places Fetcher

## Obiettivo
Implementare il connettore Google Places API che cerca le attività per categoria e città, e mappa i risultati sul modello interno `Business`.

## User Story
*Come SpotCast, voglio interrogare Google Maps per attività locali reali, così che l'output contenga dati di lead utilizzabili.*

## Criteri di Accettazione
- [ ] `GoogleFetcher.ts` implementato ed esportato
- [ ] Per ogni combinazione `(category, city)` in config, esegue una `textSearch` tramite Google Places API
- [ ] Restituisce un array di oggetti `Business` (vedi modello sotto)
- [ ] Rispetta il campo `results_per_run` — limita il totale dei risultati su tutte le combinazioni
- [ ] Gestisce gli errori API in modo elegante: logga l'errore, prosegue con risultati parziali invece di crashare
- [ ] API key letta da config/env — mai hardcodata
- [ ] Test unitari implementati con mock del client Google (vedi sezione Test)

## Modello Business

```typescript
// Modello base — corrisponde a ciò che Google Places fornisce oggi
interface Business {
  place_id: string;         // chiave di deduplicazione — stabile e univoca
  name: string;
  category: string;         // come da config, non la classificazione Google
  city: string;
  country: string;
  address: string;
  phone?: string;
  website?: string;
  rating?: number;
  review_count?: number;
  maps_url: string;         // https://www.google.com/maps/place/?q=place_id:...
}

// Metadato singolo — per arricchimento dati nelle milestone successive
interface Metadata {
  key: string;              // vedi enum MetadataKey
  value: string;
  source?: string;          // 'google' | 'manual' | 'enrichment_api'
  collected_at?: string;    // ISO timestamp
}

// Modello esteso — usato quando i metadata sono disponibili (M10+)
interface EnrichedBusiness extends Business {
  metadata: Metadata[];
}
```

**Nota architetturale:** composizione tramite `metadata[]` per l'estensibilità futura. Ogni informazione aggiuntiva raccolta nelle milestone successive viene aggiunta come `Metadata` senza toccare il modello base — `Business` rimane stabile, `EnrichedBusiness` cresce senza rotture.

## Test (Vitest)

| Caso | Tipo | Descrizione |
|---|---|---|
| Risposta Google valida | Unit | Mapping corretto risposta grezza → `Business` |
| `place_id` sempre presente | Unit | Campo presente e non vuoto su ogni risultato |
| `maps_url` formato corretto | Unit | URL costruito con `place_id` corretto |
| Errore di rete | Unit | Restituisce array vuoto senza crashare, logga l'errore |
| `results_per_run` rispettato | Unit | Non restituisce più risultati del limite configurato |
| Combinazioni multiple category/city | Unit | Esegue una query per ogni combinazione |

```typescript
// Esempio mock Vitest
vi.mock('@googlemaps/google-maps-services-js', () => ({
  Client: vi.fn().mockImplementation(() => ({
    textSearch: vi.fn().mockResolvedValue({
      data: {
        results: [{
          place_id: 'ChIJ123',
          name: 'Studio Dentistico Rossi',
          formatted_address: 'Via Roma 1, Milano',
          rating: 4.5,
          user_ratings_total: 120,
        }],
      },
    }),
  })),
}));
```

## Note Tecniche
- Usare `@googlemaps/google-maps-services-js` — client ufficiale Google, tipizzato
- Formato query `textSearch`: `"${category} in ${city}, ${country}"`
- `maps_url` costruito come `https://www.google.com/maps/place/?q=place_id:${place_id}`
- Il campo `email` non è fornito dalla Places API — arricchimento previsto in milestone successive tramite il sistema `metadata`

## Stima di Effort
**4–5 ore** (inclusi test e primo test live con chiave reale)

---