# SpotCast — Tutorial Tecnici
## Zod · Winston · ts-node · Nodemon · ExcelJS · Handlebars · node-cron · TDD con Vitest

---

# 1. Zod — Validazione e Tipizzazione dello Schema

## Cos'è

Zod è una libreria TypeScript-first per la dichiarazione e validazione di schemi. Risolve un problema classico: `JSON.parse()` restituisce `any`, e TypeScript non può garantire nulla sulla struttura di un file esterno come `config.json`. Zod permette di dichiarare lo schema atteso, validarlo a runtime, e ottenere automaticamente il tipo TypeScript inferito — senza scrivere i tipi a mano.

## Installazione

```bash
npm install zod
```

## Utilizzo base

```typescript
import { z } from 'zod';

const ConfigSchema = z.object({
  language: z.enum(['it', 'en', 'de']).default('it'),
  google_api_key: z.string().min(1, 'google_api_key è obbligatoria'),
  categories: z.array(z.string()).min(1, 'Almeno una categoria richiesta'),
  cities: z.array(z.string()).min(1, 'Almeno una città richiesta'),
  results_per_run: z.number().int().positive().default(10),
  schedule: z.string().default('0 8 * * *'),
  output_dir: z.string().default('results'),
  smtp: z.object({
    host: z.string(),
    port: z.number().default(587),
    user: z.string().email(),
    pass: z.string(),
  }),
  email_to: z.array(z.string().email()).min(1),
});

// Il tipo viene inferito automaticamente — niente duplicazione
type Config = z.infer<typeof ConfigSchema>;

// Validazione
const raw = JSON.parse(fs.readFileSync('config.json', 'utf-8'));
const result = ConfigSchema.safeParse(raw);

if (!result.success) {
  console.error('Configurazione non valida:');
  console.error(result.error.format()); // errori leggibili, campo per campo
  process.exit(1);
}

const config: Config = result.data; // pienamente tipizzato
```

## Perché è migliore dell'alternativa manuale

Senza Zod, il codice di validazione è verboso, ripetitivo e facile da dimenticare. Zod centralizza schema, validazione e tipo in un'unica dichiarazione. Se aggiungi un campo allo schema, il tipo si aggiorna automaticamente ovunque.

---

# 2. Winston — Logging Strutturato

## Cos'è

Winston è il logger più diffuso nell'ecosistema Node.js. Gestisce livelli di log (error, warn, info, debug), trasporti multipli (console, file, servizi remoti), formattazione personalizzabile e rotazione automatica dei file di log. Molto più robusto di `fs.appendFileSync`, senza la complessità di soluzioni enterprise.

Il nome non ha nulla a che fare con le sigarette di Marty McFly — ma l'associazione è memorabile 😄

## Installazione

```bash
npm install winston
```

## Configurazione per SpotCast

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.printf(({ timestamp, level, message }) => {
      return `[${timestamp}] ${level.toUpperCase()}: ${message}`;
    })
  ),
  transports: [
    // Scrive tutto su file
    new winston.transports.File({ filename: 'tracker.log' }),
    // Mostra in console durante lo sviluppo
    new winston.transports.Console({
      silent: process.env.NODE_ENV === 'production',
    }),
  ],
});

export default logger;
```

## Utilizzo nei moduli

```typescript
import logger from './logger';

logger.info('RUN STARTED');
logger.info(`RUN COMPLETE — 8 new businesses | duration: 4.2s`);
logger.warn('No new businesses found, skipping email');
logger.error(`Google API error: ${err.message}`);
```

## Livelli di log

| Livello | Quando usarlo |
|---|---|
| `error` | Errori che impediscono l'esecuzione |
| `warn` | Situazioni anomale ma recuperabili |
| `info` | Flusso normale — avvio, completamento, conteggi |
| `debug` | Dettagli di sviluppo (disabilitato in produzione) |

---

# 3. ts-node e Nodemon — Sviluppo senza Build Step

## ts-node

ts-node è un runtime che esegue TypeScript direttamente, senza compilare prima in JavaScript. In sviluppo elimina il ciclo `tsc → node dist/SpotCast.js` e permette di lanciare `ts-node src/SpotCast.ts` direttamente.

```bash
npm install -D ts-node typescript @types/node
```

Aggiungere in `package.json`:
```json
{
  "scripts": {
    "dev": "ts-node src/SpotCast.ts",
    "build": "tsc",
    "start": "node dist/SpotCast.js"
  }
}
```

## Nodemon — Hot Reload

Nodemon monitora i file del progetto e riavvia automaticamente il processo quando rileva modifiche. Combinato con ts-node, ogni salvataggio riavvia SpotCast con le modifiche applicate — zero interruzioni del flusso di sviluppo.

```bash
npm install -D nodemon
```

Configurazione `nodemon.json`:
```json
{
  "watch": ["src"],
  "ext": "ts,json",
  "ignore": ["src/**/*.spec.ts"],
  "exec": "ts-node src/SpotCast.ts"
}
```

Aggiungere in `package.json`:
```json
{
  "scripts": {
    "dev": "nodemon"
  }
}
```

## Come funziona il hot reload

```
Salvi un file .ts
       ↓
Nodemon rileva la modifica (via inotify/FSEvents)
       ↓
Termina il processo Node corrente (SIGTERM)
       ↓
Rilancia ts-node con il file aggiornato
       ↓
Il processo riparte da zero con le modifiche attive
```

**Nota:** hot reload qui significa riavvio del processo, non sostituzione a caldo dei moduli (come in React). Per uno strumento backend come SpotCast è il comportamento corretto e atteso.

---

# 4. TDD in TypeScript — Le Tre Alternative

## Criterio di scelta

Per un progetto che vuole segnalare seniority, la scelta del framework di testing non è solo tecnica — è anche un segnale culturale. I tre candidati principali per TypeScript sono:

---

## Jest

**Diffusione:** altissima — è il framework di default per React e la maggioranza dei progetti Node.js degli ultimi 10 anni.

**Curva di apprendimento:** bassa. API familiare (`describe`, `it`, `expect`), documentazione eccellente, enormi community e risorse.

**Configurazione per TypeScript:**
```bash
npm install -D jest ts-jest @types/jest
```
```json
// jest.config.ts
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
};
```

**Giudizio degli architetti Senior:** rispettato ma percepito come *legacy-leaning*. Architettura interna CommonJS-first, performance inferiore ai competitor moderni, mocking API verbose e a tratti controintuitiva. Scelta sicura, non scelta ambiziosa.

---

## Vitest ✅ — La scelta raccomandata

**Diffusione:** in fortissima crescita — nato nel 2021, è diventato rapidamente il riferimento per progetti Vite/TypeScript moderni. Adottato da Vue, Nuxt e un numero crescente di progetti enterprise.

**Curva di apprendimento:** quasi zero per chi conosce Jest — API identica (`describe`, `it`, `expect`), drop-in replacement nella maggioranza dei casi.

**Configurazione per TypeScript:**
```bash
npm install -D vitest
```
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
export default defineConfig({
  test: {
    environment: 'node',
    coverage: { provider: 'v8' },
  },
});
```

**Esempio test con mocking:**
```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { GoogleFetcher } from '../src/fetcher/GoogleFetcher';

// Mock del modulo Google Maps
vi.mock('@googlemaps/google-maps-services-js', () => ({
  Client: vi.fn().mockImplementation(() => ({
    textSearch: vi.fn().mockResolvedValue({
      data: {
        results: [
          {
            place_id: 'ChIJ123',
            name: 'Studio Dentistico Rossi',
            formatted_address: 'Via Roma 1, Milano',
            rating: 4.5,
            user_ratings_total: 120,
          },
        ],
      },
    }),
  })),
}));

describe('GoogleFetcher', () => {
  let fetcher: GoogleFetcher;

  beforeEach(() => {
    fetcher = new GoogleFetcher({ google_api_key: 'test-key' } as any);
  });

  it('mappa correttamente la risposta Google sul modello Business', async () => {
    const results = await fetcher.fetch('Dentist', 'Milan', 'Italy');

    expect(results).toHaveLength(1);
    expect(results[0]).toMatchObject({
      place_id: 'ChIJ123',
      name: 'Studio Dentistico Rossi',
      category: 'Dentist',
      city: 'Milan',
    });
  });

  it('restituisce array vuoto in caso di errore API, senza crashare', async () => {
    vi.mocked(fetcher['client'].textSearch).mockRejectedValueOnce(
      new Error('Network error')
    );
    const results = await fetcher.fetch('Dentist', 'Milan', 'Italy');
    expect(results).toEqual([]);
  });
});
```

**Giudizio degli architetti Senior:** è la scelta che segnala consapevolezza dell'ecosistema moderno. ESM nativo, performance superiore a Jest (esecuzione parallela reale, nessun overhead di transpilazione), DX eccellente. Chi vede Vitest in un progetto capisce subito che il team è aggiornato.

---

## Bun Test

**Diffusione:** emergente — Bun è il runtime alternativo a Node.js, il suo test runner è integrato nativamente.

**Curva di apprendimento:** API simile a Jest/Vitest, ma l'ecosistema è ancora giovane e alcune integrazioni TypeScript avanzate richiedono workaround.

**Giudizio degli architetti Senior:** ottimo come signal di innovazione, ma prematuro per un progetto destinato a essere distribuito e mantenuto da terzi. Riservatelo a progetti sperimentali o interni.

---

## Verdetto per SpotCast

**Vitest** — segnala seniority, performance eccellente, zero overhead di configurazione con TypeScript, mocking API pulita e moderna. È la scelta che fa dire "ah, sanno quello che fanno" a chiunque apra il repo.

---

# 5. ExcelJS — Generazione Excel Professionale

## Cos'è

ExcelJS è la libreria Node.js più completa per creare e manipolare file `.xlsx`. Supporta fogli multipli, stili, formule, immagini, larghezze colonne automatiche e streaming per file grandi.

## Installazione

```bash
npm install exceljs
```

## Utilizzo per SpotCast

```typescript
import ExcelJS from 'exceljs';

const workbook = new ExcelJS.Workbook();
workbook.creator = 'SpotCast';
workbook.created = new Date();

// Foglio 1 — Attività
const sheet = workbook.addWorksheet('Businesses');

// Definizione colonne con larghezza automatica
sheet.columns = [
  { header: 'Company Name', key: 'name',          width: 30 },
  { header: 'Category',     key: 'category',      width: 20 },
  { header: 'City',         key: 'city',          width: 15 },
  { header: 'Address',      key: 'address',       width: 35 },
  { header: 'Phone',        key: 'phone',         width: 18 },
  { header: 'Website',      key: 'website',       width: 30 },
  { header: 'Rating',       key: 'rating',        width: 10 },
  { header: 'Reviews',      key: 'review_count',  width: 10 },
  { header: 'Maps URL',     key: 'maps_url',      width: 40 },
];

// Stile riga intestazione
const headerRow = sheet.getRow(1);
headerRow.font = { bold: true, color: { argb: 'FFFFFFFF' } };
headerRow.fill = {
  type: 'pattern',
  pattern: 'solid',
  fgColor: { argb: 'FF2E75B6' },
};
headerRow.alignment = { vertical: 'middle', horizontal: 'center' };

// Inserimento dati con colori alternati
businesses.forEach((biz, index) => {
  const row = sheet.addRow(biz);
  if (index % 2 === 1) {
    row.fill = {
      type: 'pattern',
      pattern: 'solid',
      fgColor: { argb: 'FFEBF3FB' },
    };
  }
});

// Salvataggio
await workbook.xlsx.writeFile(`results/SpotCast_${date}.xlsx`);
```

---

# 6. Handlebars — Template Engine

## Cos'è

Handlebars è un motore di template logic-less: riceve un template HTML con segnaposto `{{variabile}}` e un oggetto dati, e produce la stringa finale con i valori sostituiti. È l'evoluzione diretta di Mustache — più potente, stessa filosofia.

Chi viene da Freemarker troverà la transizione immediata: la sintassi `{{variabile}}` è più pulita di `${variabile}`, e la mancanza di logica nel template è una feature, non un limite — forza la separazione tra dati e presentazione.

## Installazione

```bash
npm install handlebars
npm install -D @types/handlebars
```

## Utilizzo per SpotCast

Template `templates/email.html`:
```html
<h2>SpotCast — New leads for {{date}}</h2>
<p>Hi,<br>
attached you'll find today's results: <strong>{{count}} new businesses</strong>
found in {{cities}} for the following categories: {{categories}}.</p>
<p>Good hunting! 🎯</p>
```

Rendering nel codice:
```typescript
import Handlebars from 'handlebars';
import fs from 'fs';

const templateSource = fs.readFileSync('templates/email.html', 'utf-8');
const template = Handlebars.compile(templateSource);

const html = template({
  date: '2026-05-28',
  count: 8,
  cities: 'Milan, Rome',
  categories: 'Dentist, Gym',
});
// html è la stringa HTML pronta da passare a Nodemailer
```

## Helper utili (opzionali)

```typescript
// Blocco condizionale
Handlebars.registerHelper('ifZero', (value, options) =>
  value === 0 ? options.fn(this) : options.inverse(this)
);

// Nel template:
// {{#ifZero count}}No new businesses found today.{{else}}{{count}} businesses found.{{/ifZero}}
```

---

# 7. node-cron — Scheduling Integrato

## Cos'è

node-cron è un task scheduler per Node.js che usa la sintassi cron standard. Non dipende dal cron di sistema — gira interamente nel processo Node, il che lo rende portabile su qualsiasi OS senza configurazione aggiuntiva.

## Installazione

```bash
npm install node-cron
npm install -D @types/node-cron
```

## Sintassi Cron — Riferimento Rapido

```
┌─────────── minuto (0-59)
│ ┌───────── ora (0-23)
│ │ ┌─────── giorno del mese (1-31)
│ │ │ ┌───── mese (1-12)
│ │ │ │ ┌─── giorno della settimana (0-7, 0 e 7 = domenica)
│ │ │ │ │
* * * * *

'0 8 * * *'      → ogni giorno alle 8:00
'0 8 * * 1-5'    → lunedì-venerdì alle 8:00
'*/30 * * * *'   → ogni 30 minuti
'0 8,20 * * *'   → alle 8:00 e alle 20:00
```

## Utilizzo per SpotCast

```typescript
import cron from 'node-cron';
import { runPipeline } from './pipeline';
import logger from './logger';

export function startScheduler(expression: string): void {
  if (!cron.validate(expression)) {
    logger.error(`Invalid cron expression: "${expression}"`);
    process.exit(1);
  }

  logger.info(`Scheduler started — expression: "${expression}"`);

  cron.schedule(expression, async () => {
    const start = Date.now();
    logger.info('RUN STARTED');
    try {
      const count = await runPipeline();
      const duration = ((Date.now() - start) / 1000).toFixed(1);
      logger.info(`RUN COMPLETE — ${count} new businesses | duration: ${duration}s`);
    } catch (err) {
      logger.error(`RUN FAILED: ${(err as Error).message}`);
    }
  });
}
```

---

# 8. Modello Dati — Ereditarietà vs Composizione

## La domanda

La risposta che stavi cercando mentre dettagliavi le proposte è **composizione** — e hai ragione.

## Perché composizione vince su ereditarietà

L'ereditarietà (`EnrichedActivity extends Activity`) funziona finché i layer sono prevedibili e stabili. Non appena i dati aggiuntivi variano per sorgente, per milestone o per feature opzionale, la gerarchia diventa rigida e difficile da estendere senza rotture.

La composizione con `ActivityMetadata` è invece aperta per definizione:

```typescript
// Modello base — stabile, corrisponde a ciò che Google Places fornisce oggi
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
}

// Chiavi metadata conosciute — enum estensibile milestone per milestone
enum MetadataKey {
  EMAIL            = 'email',           // M99+ — email enrichment
  LINKEDIN_URL     = 'linkedin_url',    // M99+
  AD_BUDGET_EUR    = 'ad_budget_eur',   // M99+ — stima budget Google Ads
  LAST_SEEN        = 'last_seen',       // M10+ — storico discovery
  NOTES            = 'notes',           // libero — note utente dalla GUI
}

// Singolo metadato
interface Metadata {
  key: MetadataKey | string;  // string per chiavi custom non ancora in enum
  value: string;
  source?: string;            // 'google' | 'manual' | 'enrichment_api'
  collected_at?: string;      // ISO timestamp
}

// Modello completo
interface EnrichedBusiness extends Business {
  metadata: Metadata[];
}

// Utilizzo
const biz: EnrichedBusiness = {
  place_id: 'ChIJ123',
  name: 'Studio Dentistico Rossi',
  // ... campi base ...
  metadata: [
    { key: MetadataKey.EMAIL, value: 'info@rossi.it', source: 'enrichment_api' },
    { key: MetadataKey.AD_BUDGET_EUR, value: '450', source: 'google' },
  ],
};

// Accesso a un metadato specifico
const email = biz.metadata.find(m => m.key === MetadataKey.EMAIL)?.value;
```

Questo schema funziona indipendentemente da quante fonti di dati aggiungerai in futuro — ogni nuova informazione è un `Metadata` in più, non una modifica al modello base. Esattamente come i campi custom in un catalogo e-commerce.
