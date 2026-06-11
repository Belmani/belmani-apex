# SpotCast — Grooming

# GRM-M5 — Mail Service

## Obiettivo

Inviare il file Excel generato come allegato email a tutti i destinatari configurati, con corpo e oggetto localizzati, rendering HTML tramite Handlebars, e gestione robusta degli errori SMTP.

## User Story

*Come utente, voglio che i risultati vengano consegnati automaticamente nella mia inbox ogni mattina, così non devo aprire nessuna applicazione per accedervi.*

---

## Prerequisiti bloccanti (da implementare prima di MailService)

### `src/i18n/format.ts` — utility di formattazione localizzata

Prerequisito di priorità 1. Il rendering del template email richiede date e numeri formattati secondo la lingua attiva. Senza questa utility il template produrrebbe output non localizzato (es. "2026-06-10" invece di "10/06/2026" per l'italiano).

```typescript
// Formatta una data secondo il pattern i18n.formats.date
formatDate(date: Date, i18n: Record<string, unknown>): string

// Formatta un intero con separatore migliaia — per count, totali, contatori
// Output: "1.234" (IT/DE) | "1,234" (EN) | "1 234" (FR)
formatInteger(n: number, i18n: Record<string, unknown>): string

// Formatta un decimale con separatore decimale e migliaia — per rating, valori float
// Output: "4,5" (IT/DE) | "4.5" (EN)
formatDecimal(n: number, i18n: Record<string, unknown>): string
```

I formati sono letti da `i18n.formats` — oggetto già presente in tutti i 10 file lingua da M4+:
```json
"formats": {
  "date": "DD/MM/YYYY",
  "decimal_separator": ",",
  "thousands_separator": "."
}
```

**Test:** suite isolata `tests/i18n/format.test.ts` — verificata prima di procedere con MailService.

---

## Criteri di Accettazione

- [ ] `format.ts` implementato con `formatDate`, `formatInteger`, `formatDecimal`
- [ ] `format.test.ts` verde con copertura ≥ 80%
- [ ] `MailService.ts` implementato ed esportato
- [ ] Invia email tramite Nodemailer con credenziali SMTP da `config.smtp`
- [ ] Allegato: file Excel passato come path assoluto — verificata esistenza prima dell'invio
- [ ] Se il file Excel non esiste: lancia errore descrittivo, logga, non invia email vuota
- [ ] Oggetto email renderizzato da Handlebars con chiave `email_subject` dall'i18n
- [ ] Corpo email renderizzato da `templates/email.html` — Handlebars con variabili `{{date}}`, `{{count}}`, `{{categories}}`, `{{cities}}`, `{{{body}}}`
- [ ] `body` iniettato nel template HTML con `\n` → `<br>` già applicato
- [ ] Primo destinatario in `to`, tutti gli altri in `bcc` (DTR-035)
- [ ] Errori SMTP: wrappati in `try/catch`, loggati con Winston, rilanciati — la pipeline decide se il run è fallito (DTR-036)
- [ ] `MailService` riceve i dizionari i18n già caricati come parametri — non li carica lui
- [ ] Test con `jsonTransport` — nessun invio reale, payload catturato come oggetto JSON

---

## Step di Implementazione

### Step 1 — `format.ts` + `format.test.ts` (TDD)
Prerequisito bloccante. Test scritti prima del codice. Suite isolata in `tests/i18n/`.

### Step 2 — `templates/email.html`
Template HTML statico con wrapper visivo e segnaposti Handlebars. Non richiede test — verificato indirettamente dai test di MailService.

### Step 3 — Test suite `MailService` (TDD)
Suite completa scritta prima del codice di produzione. Usa `jsonTransport`.

### Step 4 — Implementazione `MailService.ts`
Codice di produzione fino a far passare tutti i test.

### Step 5 — Verifica copertura e lint
`pnpm test:coverage` ≥ 80%, `pnpm lint` 0 errori.

---

## Struttura File

```
src/
├── i18n/
│   ├── translate.ts     ← esistente
│   └── format.ts        ← nuovo (Step 1)
├── mailer/
│   └── MailService.ts   ← nuovo (Step 4)

templates/
└── email.html           ← nuovo (Step 2)

tests/
├── i18n/
│   ├── translate.test.ts  ← esistente
│   └── format.test.ts     ← nuovo (Step 1)
└── mailer/
    └── MailService.test.ts ← nuovo (Step 3)
```

---

## Interfaccia Pubblica

```typescript
export class MailService {
  constructor(config: Config) {}

  async send(
    excelFilePath: string,              // path assoluto — da ExcelExporter.export()
    businesses: Business[],             // per estrarre count, categories, cities
    i18n: Record<string, unknown>,      // lingua attiva (include formats)
    fallback: Record<string, unknown>   // en.json — sempre passato
  ): Promise<void>
}
```

**Contratto:**
- Se `excelFilePath` non esiste → lancia `Error` con messaggio descrittivo, logga, non invia
- Se SMTP fallisce → logga l'errore, rilancia — la pipeline decide
- Non decide mai da sola se inviare o meno — questa responsabilità appartiene alla pipeline (M6)

---

## Template HTML — Variabili disponibili

```
{{date}}        — data run formattata con formatDate() secondo i18n.formats.date
{{count}}       — numero aziende formattato con formatInteger() secondo i18n.formats
{{categories}}  — join ", " delle categorie dalla config
{{cities}}      — join ", " delle città dalla config
{{{body}}}      — corpo testuale dall'i18n, \n già convertiti in <br> (triple-stache)
```

Il testo del `body` viene da `i18n.email_body`, compilato con Handlebars con le stesse variabili, poi i `\n` vengono sostituiti con `<br>` prima dell'iniezione nel wrapper HTML.

---

## Gestione Destinatari (DTR-035)

```typescript
const recipients = config.email_to;
const mailOptions = {
  from:    `SpotCast <${config.smtp.user}>`,
  to:      recipients[0],                    // primo destinatario in to
  bcc:     recipients.slice(1).join(', '),   // tutti gli altri in bcc
  subject: renderedSubject,
  html:    renderedHtml,
  attachments: [{
    filename: path.basename(excelFilePath),
    path:     excelFilePath,
  }],
};
```

Se `email_to` ha un solo destinatario, `bcc` è stringa vuota — Nodemailer lo gestisce correttamente.

---

## Wrapping Errori SMTP (DTR-036)

```typescript
try {
  await transporter.sendMail(mailOptions);
  logger.info(`Email sent to ${config.email_to.length} recipient(s)`);
} catch (err) {
  logger.error(`SMTP error: ${(err as Error).message}`);
  throw err; // rilancia — la pipeline in M6 decide se il run è fallito
}
```

Il `try/catch` è obbligatorio su qualsiasi `await` che coinvolga I/O esterno. Senza di esso, un errore SMTP asincrono diventa `UnhandledPromiseRejection` — in Node.js 20+ termina il processo senza log utile.

---

## Test Suite (Vitest)

### `format.test.ts`

| Caso | Descrizione |
|---|---|
| `formatDate` — EN | `2026-06-10` → `"2026-06-10"` |
| `formatDate` — IT | `2026-06-10` → `"10/06/2026"` |
| `formatDate` — DE | `2026-06-10` → `"10.06.2026"` |
| `formatDate` — ZH | `2026-06-10` → `"2026年06月10日"` |
| `formatInteger` — EN | `1234` → `"1,234"` |
| `formatInteger` — IT | `1234` → `"1.234"` |
| `formatInteger` — FR | `1234` → `"1 234"` |
| `formatInteger` — sotto 1000 | `42` → `"42"` (nessun separatore) |
| `formatDecimal` — EN | `4.5` → `"4.5"` |
| `formatDecimal` — IT | `4.5` → `"4,5"` |
| `formatDecimal` — intero passato come float | `4.0` → `"4"` (zero trailing) |
| Fallback formato mancante | `formats` assente → comportamento EN di default |

### `MailService.test.ts`

| Caso | Tipo | Descrizione |
|---|---|---|
| File Excel esistente | Unit | `sendMail` invocato con allegato corretto |
| `to` primo destinatario | Unit | `mailOptions.to` è `email_to[0]` |
| `bcc` altri destinatari | Unit | `mailOptions.bcc` contiene tutti tranne il primo |
| Destinatario singolo | Unit | `bcc` vuoto, nessun errore |
| `subject` renderizzato | Unit | Handlebars sostituisce `{{date}}` e `{{count}}` |
| `body` con `<br>` | Unit | `\n` nel body i18n convertiti in `<br>` nell'HTML |
| `{{cities}}` nel body | Unit | Città dalla config presenti nel corpo email |
| `{{categories}}` nel body | Unit | Categorie dalla config presenti nel corpo email |
| File Excel mancante | Unit | Lancia errore descrittivo, `sendMail` non invocato |
| Errore SMTP | Unit | Logga errore e rilancia — non inghiotte |
| Allegato nome file corretto | Unit | `filename` corrisponde al basename del path |
| `from` formattato correttamente | Unit | `SpotCast <smtp.user>` |
| i18n fallback applicato | Unit | Chiave mancante in active → valore da en.json |

---

## Decisioni Tecniche

**`jsonTransport` per i test:** Nodemailer offre `createTransport({ jsonTransport: true })` — non invia nulla ma cattura il messaggio come oggetto JSON. Permette di verificare il payload completo (to, bcc, subject, html, attachments) senza mock artificiali. Stesso approccio dei file reali in DedupService e ExcelExporter.

**Path vs buffer:** confermato path. Il file Excel persiste in `results/` — artefatto locale utile anche se l'email fallisce. Il buffer richiederebbe refactor del contratto pubblico di `ExcelExporter` (M4).

**Handlebars compile una volta:** stessa decisione di DTR-034 — template compilato fuori da qualsiasi loop.

**`\n` → `<br>`:** applicato al `body` i18n prima dell'iniezione nel wrapper HTML, non nel template stesso — mantiene il file i18n leggibile come testo.

---

## Criticità

| # | Criticità | Mitigazione |
|---|---|---|
| C1 | Errori SMTP asincroni non gestiti → crash processo | `try/catch` obbligatorio su `sendMail`, rilancio esplicito (DTR-036) |
| C2 | File Excel mancante se ExcelExporter ha fallito | Verifica `fs.existsSync` prima di `sendMail` |
| C3 | Gmail richiede App Password, non password normale | Documentato nel README utente — errore più frequente degli utenti finali |
| C4 | Destinatari multipli espongono indirizzi tra loro | Primo in `to`, tutti gli altri in `bcc` (DTR-035) |
| C5 | `formats` assente in un file lingua causa crash | Fallback a comportamento EN di default in `formatDate`/`formatInteger`/`formatDecimal` |

---

## Stima Effort

| Step | Effort |
|---|---|
| Step 1 — `format.ts` + test | 45 min |
| Step 2 — `templates/email.html` | 20 min |
| Step 3 — Test suite MailService | 45 min |
| Step 4 — Implementazione MailService | 60 min |
| Step 5 — Coverage + lint | 15 min |
| **Totale** | **~3 ore** |
