# SpotCast — Riepilogo Milestone 4

## M4 — Esportatore Excel

**Stato:** ✅ Completata
**Data di chiusura:** 2026-06-10
**Release:** inclusa in v0.1.0

---

## Obiettivo

Generare un file `.xlsx` formattato professionalmente con tre sheet dai risultati filtrati. L'output è il prodotto visibile ad Onur ogni mattina — qualità, leggibilità e configurabilità erano requisiti primari, non opzionali.

---

## Cosa è stato realizzato

### File prodotti

| File | Posizione | Tipo | Descrizione |
|---|---|---|---|
| `en.json` | `assets/i18n/` | Nuovo | Lingua di riferimento — 31 chiavi, fallback universale |
| `it.json` | `assets/i18n/` | Nuovo | Italiano — 31 chiavi tradotte |
| `de.json` | `assets/i18n/` | Nuovo | Tedesco — 31 chiavi tradotte |
| `fr.json` | `assets/i18n/` | Nuovo | Francese — 31 chiavi tradotte |
| `es.json` | `assets/i18n/` | Nuovo | Spagnolo — 31 chiavi tradotte |
| `pt.json` | `assets/i18n/` | Nuovo | Portoghese — 31 chiavi tradotte |
| `zh.json` | `assets/i18n/` | Nuovo | Cinese semplificato — 31 chiavi tradotte |
| `ja.json` | `assets/i18n/` | Nuovo | Giapponese — 31 chiavi tradotte |
| `ar.json` | `assets/i18n/` | Nuovo | Arabo — 31 chiavi tradotte |
| `tr.json` | `assets/i18n/` | Nuovo | Turco — 31 chiavi tradotte |
| `translate.ts` | `src/i18n/` | Nuovo | Utility t() con fallback a cascata |
| `ExcelConfigLoader.ts` | `src/config/` | Nuovo | Loader e validatore Zod per excel.json |
| `ExcelExporter.ts` | `src/excel/` | Nuovo | Generatore .xlsx con tre sheet |
| `excel.example.json` | root | Nuovo | Template di configurazione committato |
| `excel.json` | root | Nuovo | Configurazione operativa (gitignored) |
| `translate.test.ts` | `tests/i18n/` | Nuovo | Suite translate — 11 test |
| `ExcelConfigLoader.test.ts` | `tests/config/` | Nuovo | Suite loader — 13 test |
| `ExcelExporter.test.ts` | `tests/excel/` | Nuovo | Suite exporter — 30 test |

---

## Architettura i18n

### Separazione codice / asset (DTR-030)

```
assets/
└── i18n/              ← risorse statiche — 10 file JSON lingua
    ├── en.json        ← riferimento e fallback universale
    ├── it.json
    └── ...

src/
└── i18n/
    └── translate.ts   ← codice — utility condivisa da ExcelExporter, MailService, Scheduler
```

### Fallback a cascata — `translate.ts` (DTR-027)

```typescript
export function t(key, i18n, fallback): string {
  return i18n[key] || fallback[key] || key;
}
```

Tre livelli: lingua attiva → `en.json` → chiave stessa. La chiave come ultimo livello rende immediatamente visibili le traduzioni mancanti in fase di QA. Stringa vuota in active cade al livello successivo — non è una traduzione utile.

### Chiavi i18n — 31 per file, divise per dominio

| Dominio | Chiavi | Esempi |
|---|---|---|
| Sheet 1 — colonne | 13 | `col_name`, `col_category`, `sheet_businesses` |
| Sheet 2 — email | 5 | `email_subject_template`, `email_body_template` |
| Sheet 3 — summary | 6 | `summary_run_date`, `summary_total_found` |
| Log / runtime | 4 | `log_run_start`, `log_run_complete` |
| Email invio | 3 | `email_subject`, `email_body_intro` |

Parità verificata automaticamente: tutti i 10 file hanno esattamente le stesse 31 chiavi di `en.json`.

---

## Configurazione Excel — `excel.json`

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

**Campi obbligatori (non disabilitabili):** `name`, `category`, `city`, `address` — `ExcelConfigLoader` lancia errore descrittivo se disabilitati.

**Ordine colonne:** determinato dall'ordine delle chiavi nel JSON — nessun hardcode in `ExcelExporter` (DTR-032).

---

## ExcelExporter — Interfaccia pubblica

```typescript
export class ExcelExporter {
  constructor(config: Config, excelConfig: ExcelConfig)

  async export(
    businesses: Business[],
    duplicatesSkipped: number,
    i18n: Record<string, string>,
    fallback: Record<string, string>
  ): Promise<string>  // path assoluto del file generato
}
```

### Sheet 1 — Businesses

- Colonne: attive da `excel.json`, nell'ordine delle chiavi
- Header: font bold, sfondo `#2E75B6`, testo bianco
- Righe dati: alternanza bianco / `#EBF3FB`
- Auto-width: `max(MIN_COL_WIDTH=10, headerLen, maxDataLen) + COL_PADDING=2` (DTR-033)

### Sheet 2 — Email Templates (opzionale)

- Abilitato da `excel.json → include_email_templates`
- Colonne: `name | category | city | subject | body`
- Template Handlebars dall'i18n attivo — variabili disponibili: tutti i campi `Business`
- `wrapText: true` sulla colonna `body`
- Handlebars compilato una volta fuori dal loop — non ad ogni iterazione (DTR-034)

### Sheet 3 — Run Summary

- Sempre presente, 5 righe fisse: run_date, total_found, duplicates_skipped, data_source, language
- Label dall'i18n — zero hardcode
- `data_source` sempre `"google"` in questa milestone

---

## Test Suite — 54 nuovi test, 0 failure

```
translate.test.ts (11 test)
  level 1 — active language         ✓ ✓
  level 2 — en.json fallback         ✓ ✓
  level 3 — key as last resort       ✓ ✓
  edge cases                         ✓ ✓ ✓ ✓ ✓

ExcelConfigLoader.test.ts (13 test)
  valid configuration                ✓ ✓ ✓
  mandatory fields protection        ✓ ✓ ✓ ✓ ✓
  column order                       ✓ ✓
  file errors                        ✓ ✓ ✓

ExcelExporter.test.ts (30 test)
  file generation                    ✓ ✓ ✓ ✓
  sheet structure                    ✓ ✓ ✓ ✓ ✓
  Sheet 1 — Businesses               ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✓
  Sheet 2 — Email Templates          ✓ ✓ ✓ ✓ ✓
  Sheet 3 — Run Summary              ✓ ✓ ✓ ✓ ✓
  i18n fallback                      ✓
```

**Suite cumulativa al termine di M4: 71/71** — nessuna regressione sulle milestone precedenti.

**Strategia di test:** file reali in `os.tmpdir()`, letti con `ExcelJS.Workbook.readFile()` — nessun mock della libreria. Stesso principio adottato in M3 per `DedupService`.

---

## Decisioni Tecniche Chiave in M4

| DTR | Decisione |
|---|---|
| DTR-027 | `translate.ts` utility condivisa — fallback a cascata in tre livelli |
| DTR-029 | `excel.json` file separato — policy configurazioni per dominio confermata |
| DTR-030 | `assets/i18n/` per i JSON — separazione codice/asset |
| DTR-031 | Import JSON dinamici con `.default` — lezione appresa da errore TypeScript ts(2352) |
| DTR-032 | Colonne Excel guidate da `excel.json` — ordine chiavi JSON = ordine colonne foglio |
| DTR-033 | Auto-width manuale post-inserimento — limitazione ExcelJS, parametri documentati |
| DTR-034 | Handlebars compile fuori dal loop — performance O(1) invece di O(n) |

---

## Cambio lingua Turco al posto di Hindi

In M4 è stato formalizzato il cambio della decima lingua di lancio: Hindi (`hi`) sostituito con Turco (`tr`). Motivazione: rilevanza diretta del mercato di riferimento (cliente iniziale Onur, mercato turco) e dimensione del bacino (85M+ parlanti). Aggiornato DTR-011 in italiano e inglese.

---

## Note per M5

M5 implementerà `MailService`. Il modulo riceverà:

- Il file `.xlsx` generato da `ExcelExporter` come allegato
- Il template `templates/email.html` renderizzato con Handlebars
- I dizionari i18n già caricati — stesso pattern adottato in M4

Le chiavi i18n necessarie a `MailService` (`email_subject`, `email_body_intro`, `email_body_count`) sono già presenti in tutti e 10 i file lingua — nessuna aggiunta necessaria per il prossimo step.
