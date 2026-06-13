# SpotCast — Grooming

# GRM-M7 — HereFetcher (HERE Browse API)

## Obiettivo

Sostituire `GoogleFetcher` con `HereFetcher` basato su HERE Browse API.
Il resto della pipeline — `DedupService`, `ExcelExporter`, `MailService`, `SpotCast.ts` — non si tocca.

## User Story

*Come utente, voglio che SpotCast trovi tutte le attività disponibili nella mia area senza limiti arbitrari, così non mi perdo nessun lead potenziale.*

---

## Contesto — perché si cambia fetcher

Google Places Text Search ha un limite strutturale di 20 risultati per query (60 con paginazione). In fase di test end-to-end su Tolmezzo e Socchieve con categorie Bar/Tabacchino/Fabbro, il limite globale `results_per_run: 10` ha restituito esclusivamente tabacchini, nascondendo completamente le altre categorie. Anche rimuovendo il limite globale, il cap di Google rimane un problema per qualsiasi comune non triviale.

HERE Browse API non ha limiti arbitrari per query, usa coordinate geografiche invece di query testuali, e ha copertura capillare Europa. (DTR-038, DTR-040)

---

## Criteri di Accettazione

- [ ] `HereFetcher.ts` implementato ed esportato
- [ ] `HereCategoryMap.ts` con mappa label inglese → codice HERE
- [ ] `ConfigLoader.ts` aggiornato: valida le categorie contro `HereCategoryMap`, emette WARN + skip per quelle non riconosciute, errore fatale se tutte invalide
- [ ] `geocache.json` creato nella root con città europee pre-cachate
- [ ] `GeoCache.ts` gestisce lettura/scrittura della cache geografica
- [ ] Paginazione automatica con `offset` fino ad esaurimento risultati HERE
- [ ] `results_per_run` rimosso da config e da `ConfigLoader` (breaking change documentato)
- [ ] `GoogleFetcher.ts` deprecato con commento — non rimosso
- [ ] `SpotCast.ts` aggiorna solo la riga di import
- [ ] Test suite verde al 100%

---

## File da produrre

```
src/fetcher/HereFetcher.ts
src/fetcher/HereCategoryMap.ts
src/fetcher/GeoCache.ts
geocache.json                      ← committato nel repo
tests/fetcher/HereFetcher.test.ts
tests/fetcher/GeoCache.test.ts
```

---

## Breaking changes in `config.json`

| Campo | Prima | Dopo |
|---|---|---|
| `google_api_key` | presente | rimosso |
| `here_api_key` | assente | obbligatorio |
| `categories` | stringhe libere (`"Tabacchino"`) | label inglesi da mappa (`"Bar"`) |
| `results_per_run` | presente | rimosso (WARN se presente, ignorato) |
| `search_radius_meters` | assente | opzionale, default `15000` |

---

## Architettura `HereFetcher`

### Interfaccia pubblica — identica a GoogleFetcher

```typescript
class HereFetcher {
  fetchAll(): Promise<Business[]>
  fetchOne(categoryCode: string, city: string): Promise<Business[]>
}
```

`SpotCast.ts` cambia solo la riga di import — zero altre modifiche alla pipeline.

### Geocoding con cache

```typescript
// GeoCache.ts
class GeoCache {
  get(city: string, country: string): Coordinates | null
  set(city: string, country: string, coords: Coordinates): void
}
```

Flusso:
1. Controlla `geocache.json` — se la città è presente usa le coordinate senza chiamare HERE
2. Se assente: chiama HERE Geocoding API, salva in `geocache.json`, procede
3. La chiave è `"${city}, ${country}"` — es. `"Berlin, Germany"`

`geocache.json` viene committato nel repo. Non contiene segreti. Pre-caricato con le principali città europee.

### Paginazione HERE Browse

```typescript
async fetchOne(categoryCode: string, city: string): Promise<Business[]> {
  const coords = await this.geoCache.getOrFetch(city, this.config.countries[0]);
  const results: Business[] = [];
  let offset = 0;

  do {
    const response = await this.browseRequest(coords, categoryCode, offset);
    results.push(...this.mapResults(response.items));
    offset += 100;
  } while (response.items.length === 100); // < 100 = ultima pagina

  return results;
}
```

Nessun delay obbligatorio tra le pagine — HERE non ha il problema del `next_page_token` di Google.

### Mapping `HereCategoryMap`

```typescript
// HereCategoryMap.ts
export const HERE_CATEGORY_MAP: Record<string, string> = {
  "Bar":          "100-1000-0000",
  "Restaurant":   "100-1100-0000",
  "Cafe":         "100-1100-0100",
  "Gym":          "400-4100-0141",
  "Pharmacy":     "600-6300-0066",
  "Dentist":      "600-6100-0062",
  "Lawyer":       "700-7400-0246",
  "Accountant":   "700-7400-0244",
  "Plumber":      "700-7400-0249",
  "Locksmith":    "700-7400-0116",
  "Electrician":  "700-7400-0118",
  "Real Estate":  "700-7400-0255",
  "Hairdresser":  "600-6950-0000",
  "Supermarket":  "600-6300-0000",
  "Hotel":        "500-5000-0000",
};
```

La lista viene espansa in base alle esigenze dei clienti — aggiungere una categoria è aggiungere una riga.

### Validazione categorie in `ConfigLoader`

```typescript
// In ConfigLoader.ts — aggiunto alla validazione Zod post-parse
const validCodes: string[] = [];
for (const label of raw.categories) {
  const code = HERE_CATEGORY_MAP[label];
  if (!code) {
    logger.warn(`Unknown category "${label}" — skipping`);
  } else {
    validCodes.push(code);
  }
}
if (validCodes.length === 0) {
  fatal('No valid categories found. Check config.json against supported category labels.');
}
raw.categories = validCodes; // sostituisce le label con i codici
```

---

## Test (Vitest)

Tutti i test usano mock HTTP — nessuna chiamata reale nei test.

| Caso | Tipo | Descrizione |
|---|---|---|
| Geocoding cache hit | Unit | Città già in `geocache.json` → nessuna chiamata HERE Geocoding |
| Geocoding cache miss | Unit | Città assente → chiamata HERE Geocoding → risultato salvato in cache |
| Paginazione una pagina | Unit | HERE restituisce < 100 risultati → un solo request, nessun loop |
| Paginazione multipagina | Unit | HERE restituisce 100 → loop fino a pagina < 100 |
| Categoria non riconosciuta | Unit | WARN nel log + skip, le altre categorie procedono |
| Tutte le categorie invalide | Unit | `fatal()` con messaggio descrittivo |
| `results_per_run` presente in config | Unit | WARN nel log, campo ignorato |
| Mapping Business corretto | Unit | Campi HERE mappati correttamente su modello `Business` |
| Errore HTTP HERE | Unit | Log error + array parziale, nessun crash |

---

## Note Tecniche

### Endpoint HERE Browse
```
GET https://browse.search.hereapi.com/v1/browse
  ?at={lat},{lng}
  &categories={categoryCode}
  &limit=100
  &offset={offset}
  &apiKey={here_api_key}
```

### Endpoint HERE Geocoding
```
GET https://geocode.search.hereapi.com/v1/geocode
  ?q={city},{country}
  &apiKey={here_api_key}
```

### `geocache.json` pre-caricato
```json
{
  "Berlin, Germany":         { "lat": 52.5200, "lng": 13.4050 },
  "Munich, Germany":         { "lat": 48.1351, "lng": 11.5820 },
  "Hamburg, Germany":        { "lat": 53.5753, "lng": 10.0153 },
  "Frankfurt, Germany":      { "lat": 50.1109, "lng":  8.6821 },
  "Niedernhausen, Germany":  { "lat": 50.1731, "lng":  8.3197 },
  "Rome, Italy":             { "lat": 41.9028, "lng": 12.4964 },
  "Milan, Italy":            { "lat": 45.4654, "lng":  9.1859 },
  "Vienna, Austria":         { "lat": 48.2082, "lng": 16.3738 },
  "Zurich, Switzerland":     { "lat": 47.3769, "lng":  8.5417 }
}
```

---

## Stima di Effort
**3–4 ore** inclusi geocaching, paginazione, mappa categorie e test suite
