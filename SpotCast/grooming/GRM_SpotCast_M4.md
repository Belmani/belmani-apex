# SpotCast — Grooming

# GRM-M4 — Esportatore Excel

## Obiettivo

Generare un file `.xlsx` formattato professionalmente con tre sheet dai risultati filtrati. L'output è il prodotto visibile ad Onur ogni mattina — qualità e leggibilità sono requisiti primari, non opzionali.

## User Story

*Come utente, voglio ricevere un file Excel pulito e pronto all'uso, così da poter lavorare subito con i lead senza dover riformattare nulla.*

---

## Criteri di Accettazione

- [ ] `ExcelExporter.ts` implementato ed esportato come named export
- [ ] Genera un file `.xlsx` nella cartella `output_dir` da config, risolta con `path.resolve()` rispetto a `process.cwd()`
- [ ] La cartella `output_dir` viene creata automaticamente se non esiste — `fs.mkdirSync(outputDir, { recursive: true })`
- [ ] Nome file: `SpotCast_YYYY-MM-DD.xlsx` — due run nello stesso giorno sovrascrivono il file (comportamento atteso e documentato)
- [ ] **Sheet 1 — Businesses:** colonne configurate da `excel.json`, ordine rispettato, campi obbligatori protetti
- [ ] **Sheet 2 — Email Templates:** generato solo se `excel.json → include_email_templates: true`
- [ ] **Sheet 3 — Run Summary:** sempre presente, 5 campi fissi con label dall'i18n
- [ ] Riga intestazione Sheet 1: font bold, sfondo `#2E75B6`, testo bianco
- [ ] Righe dati Sheet 1: colori alternati bianco / `#EBF3FB`
- [ ] Larghezze colonne adattate al contenuto (calcolo post-inserimento dati)
- [ ] Tutte le stringhe visibili provengono da `translate.ts` — zero hardcode
- [ ] `ExcelExporter` riceve il dizionario i18n già caricato come parametro — non lo carica lui
- [ ] Se un campo obbligatorio (`name`, `category`, `city`, `address`) è disabilitato in `excel.json` → errore descrittivo, nessun file generato
- [ ] Test suite completa con lettura del file generato tramite ExcelJS reader

---

## Step di Implementazione

### Step 1 — Scaffolding i18n
Prerequisito bloccante. Creare `src/i18n/en.json` con tutte le chiavi necessarie. Creare stub per le altre 9 lingue con le stesse chiavi (valori in inglese come placeholder). Senza questo step nessun test può essere scritto correttamente.

### Step 2 — `translate.ts`
Utility condivisa con tre livelli di fallback (DTR-027). Implementata e testata in isolamento prima di procedere.

### Step 3 — `excel.example.json` + `ExcelConfigLoader.ts`
File di configurazione esempio committabile. Loader con validazione Zod — stesso pattern di `ConfigLoader`. Campi obbligatori protetti, ordine colonne rispettato.

### Step 4 — Test suite `ExcelExporter` (TDD)
Suite completa scritta **prima** del codice di produzione. I test usano `os.tmpdir()` e file reali — nessun mock di ExcelJS.

### Step 5 — Implementazione `ExcelExporter.ts`
Codice di produzione fino a far passare tutti i test. Sheet 1, Sheet 2 (condizionale), Sheet 3. Styling, auto-width, Handlebars, i18n via `translate.ts`.

### Step 6 — Verifica copertura e lint
`pnpm test:coverage` ≥ 80%, `pnpm lint` 0 errori, `pnpm typecheck` 0 errori.

---

## Struttura File

```
src/
├── i18n/
│   ├── translate.ts          ← utility condivisa (Step 2)
│   ├── en.json               ← lingua di riferimento completa (Step 1)
│   ├── it.json               ← stub con chiavi M4 (Step 1)
│   ├── de.json / fr.json / es.json / pt.json
│   └── zh.json / ja.json / ar.json / hi.json
├── excel/
│   └── ExcelExporter.ts      ← esportatore (Step 5)
├── config/
│   └── ExcelConfigLoader.ts  ← loader excel.json (Step 3)

excel.example.json             ← template committato (Step 3)

tests/
├── i18n/
│   └── translate.test.ts     ← test translate.ts (Step 2)
├── excel/
│   └── ExcelExporter.test.ts ← suite principale (Step 4)
└── config/
    └── ExcelConfigLoader.test.ts (Step 3)
```

---

## Interfaccia Pubblica

```typescript
// ExcelExporter
export class ExcelExporter {
  constructor(config: Config, excelConfig: ExcelConfig) {}

  async export(
    businesses: Business[],
    duplicatesSkipped: number,
    i18n: Record<string, string>,
    fallback: Record<string, string>   // en.json — sempre passato
  ): Promise<string>                   // restituisce il path del file generato
}

// ExcelConfig — inferito da Zod
interface ExcelConfig {
  columns: Record<string, boolean>
  include_email_templates: boolean
}
```

---

## Chiavi i18n — Contratto completo

### Sheet 1 — Businesses
```json
"sheet_businesses":   "Businesses",
"col_place_id":       "Place ID",
"col_name":           "Company Name",
"col_category":       "Category",
"col_city":           "City",
"col_country":        "Country",
"col_address":        "Address",
"col_phone":          "Phone",
"col_website":        "Website",
"col_rating":         "Rating",
"col_review_count":   "Reviews",
"col_maps_url":       "Maps URL",
"col_first_seen":     "First Seen"
```

### Sheet 2 — Email Templates
```json
"sheet_email_templates":  "Email Templates",
"col_email_subject":      "Subject",
"col_email_body":         "Body",
"email_subject_template": "Partnership opportunity — {{name}}",
"email_body_template":    "Dear {{name}},\n\nI noticed your business in {{city}} and would love to connect.\n\nBest regards"
```

### Sheet 3 — Run Summary
```json
"sheet_run_summary":      "Run Summary",
"summary_run_date":       "Run Date",
"summary_total_found":    "Total Found",
"summary_duplicates":     "Duplicates Skipped",
"summary_data_source":    "Data Source",
"summary_language":       "Language"
```

---

## `excel.example.json`

```json
{
  "columns": {
    "place_id":     false,
    "name":         true,
    "category":     true,
    "city":         true,
    "country":      true,
    "address":      true,
    "phone":        true,
    "website":      true,
    "rating":       true,
    "review_count": true,
    "maps_url":     false,
    "first_seen":   false
  },
  "include_email_templates": true
}
```

---

## Test Suite (Vitest)

### `translate.test.ts`

| Caso | Descrizione |
|---|---|
| Chiave presente in i18n attivo | Restituisce il valore della lingua attiva |
| Chiave mancante in attivo, presente in fallback | Restituisce il valore di `en.json` |
| Chiave mancante in entrambi | Restituisce la chiave stessa |
| Chiave presente con valore vuoto in attivo | Usa il fallback — stringa vuota non è valore valido |

### `ExcelConfigLoader.test.ts`

| Caso | Descrizione |
|---|---|
| Config valida completa | Restituisce oggetto tipizzato |
| Campo obbligatorio disabilitato | Lancia errore descrittivo |
| File mancante | Exit con messaggio chiaro |
| JSON corrotto | Exit con messaggio chiaro |
| Ordine colonne rispettato | L'array colonne attive segue l'ordine delle chiavi |

### `ExcelExporter.test.ts`

| Caso | Tipo | Descrizione |
|---|---|---|
| Tre sheet presenti | Unit | Il file generato contiene esattamente 3 sheet |
| Nomi sheet dall'i18n | Unit | Nomi corrispondono alle chiavi `sheet_*` dell'i18n attivo |
| Header Sheet 1 corretti | Unit | Prima riga con tutte le colonne abilitate in `excel.json` |
| Ordine colonne rispettato | Unit | L'ordine delle colonne segue `excel.json` |
| Dati Sheet 1 corretti | Unit | Ogni riga corrisponde al business in input |
| Colonne disabilitate assenti | Unit | `place_id: false` → colonna non presente nel file |
| Campi obbligatori protetti | Unit | Disabilitare `name` → errore, nessun file generato |
| Sheet 2 presente se abilitato | Unit | `include_email_templates: true` → sheet presente |
| Sheet 2 assente se disabilitato | Unit | `include_email_templates: false` → sheet non presente |
| Sheet 2 template renderizzato | Unit | `{{name}}` sostituito con il nome del business |
| Sheet 2 wrapText su email_body | Unit | Alignment della colonna body ha `wrapText: true` |
| Sheet 3 run_date formato corretto | Unit | Data nel formato `YYYY-MM-DD` |
| Sheet 3 conteggi corretti | Unit | `total_found` e `duplicates_skipped` corretti |
| Sheet 3 label dall'i18n | Unit | Label di riga corrispondono alle chiavi `summary_*` |
| Cartella output autocreata | Unit | Output in directory inesistente → creata automaticamente |
| Array businesses vuoto | Unit | File generato con soli header, nessun crash |
| Fallback i18n applicato | Unit | Chiave mancante nella lingua attiva → valore da `en.json` |
| Auto-width applicato | Unit | Larghezza colonne > 0 e proporzionale al contenuto |
| Path restituito corretto | Unit | Il metodo `export()` restituisce il path assoluto del file generato |

---

## Decisioni Tecniche

**Auto-width:** ExcelJS non ha auto-fit nativo. Calcolo manuale post-inserimento dati:
```typescript
column.width = Math.max(header.length, ...rows.map(r => String(r ?? '').length)) + 2;
```

**Handlebars per Sheet 2:** `Handlebars.compile(template)(businessData)` — già in `package.json`, nessuna nuova dipendenza.

**Sovrascrittura stesso giorno:** comportamento atteso e documentato — SpotCast è un report giornaliero, non un archivio storico.

**`output_dir` relativo:** sempre risolto con `path.resolve(process.cwd(), config.output_dir)` — comportamento determinisitco indipendentemente da dove viene lanciato il processo.

---

## Criticità

| # | Criticità | Mitigazione |
|---|---|---|
| C1 | Chiavi i18n mancanti causano fallback visibili nell'output | Step 1 bloccante — chiavi definite prima di scrivere codice |
| C2 | `output_dir` relativo si comporta diversamente per percorso di lancio | Sempre `path.resolve(process.cwd(), ...)` |
| C3 | Sovrascrittura file stesso giorno | Comportamento documentato in DTR-025 e README |
| C4 | Campi obbligatori disabilitati per errore | Validazione in `ExcelConfigLoader` con errore descrittivo |

---

## Stima Effort

| Step | Effort |
|---|---|
| Step 1 — Scaffolding i18n | 30 min |
| Step 2 — `translate.ts` + test | 30 min |
| Step 3 — `excel.json` + loader + test | 45 min |
| Step 4 — Test suite ExcelExporter | 60 min |
| Step 5 — Implementazione ExcelExporter | 90 min |
| Step 6 — Coverage + lint | 15 min |
| **Totale** | **~4.5 ore** |

Coerente con la stima GRM originale (4–5 ore).
