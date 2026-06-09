# SpotCast — Decisioni Tecniche e Architetturali (DTR)

> Questo documento registra tutte le decisioni tecniche e architetturali significative prese durante il progetto, con motivazione e alternative valutate. Aggiornato man mano che il progetto evolve.

---

## DTR-001 — Linguaggio: TypeScript al posto di Python

**Data:** 2026-05-26
**Stato:** Accettata

**Decisione:** Riscrivere il `google_maps_tracker.py` originale in TypeScript/Node.js invece di mantenere il codebase Python.

**Motivazione:**
- Lo script Python originale è stato fornito dal cliente come riferimento, non come deliverable
- Lo stack primario del team di sviluppo è Node.js/TypeScript
- TypeScript offre tipizzazione forte, miglior supporto IDE e un ecosistema maturo per tutte le dipendenze necessarie
- Node.js gira nativamente su macOS (requisito del cliente confermato)

**Alternative valutate:**
- Mantenere Python: scartato — il team non ha competenze Python, onere di manutenzione inaccettabile
- Migrare a Java: scartato — eccessivo per uno strumento backend di questo scope

---

## DTR-002 — Singolo connettore dati: solo Google Places API

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** Implementare esclusivamente il connettore Google Places API. Rimuovere il connettore Claude API presente nello script di riferimento. Nessuna interfaccia astratta `IFetcher`.

**Motivazione:**
- Lo script Python di riferimento usava Claude per generare dati aziendali fittizi come scorciatoia di sviluppo — non ha valore in produzione
- Lo scopo di SpotCast è la lead generation da dati reali
- Una Google Places API key è disponibile per lo sviluppo fin da subito; Onur creerà la sua alla consegna
- `IFetcher` è architettura speculativa: aggiunge complessità senza un secondo connettore concreto all'orizzonte

**Alternative valutate:**
- Mantenere connettore Claude come fallback dev: scartato — chiave Google disponibile dal giorno uno
- Interfaccia astratta `IFetcher`: rimandato a quando (e se) si renderà necessaria una seconda sorgente

**Nota futura:** se venisse aggiunta una seconda sorgente (es. Yelp, Foursquare), `IFetcher` sarà introdotto tramite refactor in quel momento.

---

## DTR-003 — REST API come bridge verso la GUI (Express su porta 3847)

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** SpotCast espone una REST API locale su `http://localhost:3847` tramite Express, come ponte di comunicazione tra il backend Node.js e la futura GUI JavaFX (ForgeUI).

**Motivazione:**
- La GUI sarà costruita in JavaFX (Java), che non può invocare direttamente gli interni di Node.js
- Due opzioni architetturali valutate:
  - **Child Process:** Java lancia `node SpotCast.js` e legge stdout. Semplice ma unidirezionale, fragile, non interrogabile
  - **REST API:** Node espone endpoint HTTP; Java li chiama. Bidirezionale, JSON strutturato, stato interrogabile, robusto
- REST API scelta per robustezza e orientamento al futuro
- Porta `3847` per evitare conflitti con porte di sviluppo comuni
- Il server REST si avvia solo in modalità `--daemon`

**Alternative valutate:**
- WebSocket: scartato — overkill per interazione request/response
- gRPC: scartato — complessità non giustificata per uno strumento locale

---

## DTR-004 — Chiave di deduplicazione: Google Place ID

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** Usare `place_id` come chiave di deduplicazione in `seen_firms.json`.

**Motivazione:**
- Nomi e indirizzi possono cambiare, causando false rilevazioni di "nuova" attività
- `place_id` è stabile, univoco e fornito dall'API Google Places su ogni risultato
- Garantisce uno storico affidabile nel lungo periodo

---

## DTR-005 — Configurazione: config.json + override da variabili d'ambiente

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** Tutta la configurazione risiede in `config.json`. I campi sensibili (`google_api_key`, `smtp.user`, `smtp.pass`) vengono sovrascritti da variabili d'ambiente (`GOOGLE_API_KEY`, `SMTP_USER`, `SMTP_PASS`) quando presenti.

**Motivazione:**
- `config.json` offre un file di configurazione unico e leggibile per gli utenti finali
- Le variabili d'ambiente seguono la convenzione 12-factor app per i segreti
- `config.json` è gitignored; `config.example.json` (senza segreti) viene committato
- I segreti non toccano mai il repository — non possono essere condivisi accidentalmente

**Integrazione con M10+ Wizard:**
Il wizard grafico scriverà i valori non sensibili in `config.json` e imposterà le variabili d'ambiente sensibili direttamente nel profilo utente del sistema operativo (`~/.zshrc` su macOS, variabili di sistema su Windows, `~/.profile` su Linux). I segreti non toccano mai `config.json`.

---

## DTR-006 — Validazione configurazione: Zod

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** Usare Zod v3 per la validazione di `config.json` e l'inferenza automatica del tipo `Config`.

**Motivazione:**
- `JSON.parse()` restituisce `any` — TypeScript non può garantire nulla sulla struttura di un file esterno
- Zod permette di dichiarare schema, validazione e tipo in un'unica dichiarazione
- Errori campo per campo leggibili dall'utente (es. `google_api_key is required`)
- Il tipo `Config` viene inferito automaticamente — nessuna duplicazione tra schema e interfaccia TypeScript

**Nota versione:** Zod 4 è disponibile ma non ancora stabile per produzione. Coerente con la filosofia di stabilità del progetto (vedi DTR-016).

**Alternative valutate:**
- Validazione manuale: scartato — verboso, ripetitivo, facile da dimenticare
- `io-ts`: scartato — API più complessa, curva di apprendimento più alta per benefici equivalenti

---

## DTR-007 — Logging: Winston

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** Usare Winston come sistema di logging con transport file (`tracker.log`) e console (silenziata in produzione).

**Motivazione:**
- `fs.appendFileSync` è insufficiente per un tool distribuito: nessun livello, nessuna formattazione, nessuna rotazione
- Winston è il logger più maturo dell'ecosistema Node.js, production-ready
- Livelli `error/warn/info/debug` permettono di filtrare il rumore in produzione
- Formato log standardizzato: `[YYYY-MM-DD HH:mm:ss] LEVEL: message` — tutto in inglese
- Console silenziata in produzione (`NODE_ENV=production`) per non disturbare l'utente finale
- Nei moduli critici pre-boot (es. `ConfigLoader`) si usa `process.stderr.write` per evitare dipendenze circolari

---

## DTR-008 — Testing: Vitest

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** Usare Vitest come framework di testing. TDD integrato in ogni milestone — i test non sono opzionali.

**Motivazione:**
- Vitest è ESM nativo, performance superiore a Jest, API identica (drop-in replacement per chi conosce Jest)
- Mocking API pulita e moderna — `vi.mock()`, `vi.fn()`, `vi.spyOn()`
- Integrazione nativa con TypeScript senza configurazione aggiuntiva
- Segnala consapevolezza dell'ecosistema moderno — scelta che comunica seniority
- Coverage provider `v8` tramite `@vitest/coverage-v8`

**Coverage target:** 80% minimo su tutti i moduli.

**Nota mocking:** i mock di classi instanziate con `new` devono usare `function` e non arrow function — le arrow function non possono essere costruttori in JavaScript. Lezione appresa in M2.

**Alternative valutate:**
- Jest: scartato — architettura CommonJS-first, performance inferiore, configurazione TypeScript più verbosa
- Bun Test: scartato — ecosistema ancora giovane, prematuro per un progetto distribuito a terzi

---

## DTR-009 — Modello dati: composizione tramite Metadata

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** Il modello base `Business` viene esteso nelle milestone successive tramite composizione (`EnrichedBusiness extends Business` con array `metadata: Metadata[]`), non tramite ereditarietà multipla o modifica del modello base.

**Motivazione:**
- Il modello base `Business` corrisponde a ciò che Google Places API fornisce oggi — deve rimanere stabile
- Le informazioni aggiuntive variano per sorgente e milestone — l'ereditarietà creerebbe una gerarchia rigida
- La composizione con `Metadata[]` è aperta per definizione: ogni nuova informazione è un `Metadata` in più, zero modifiche al modello base
- Schema analogo ai campi custom in un catalogo e-commerce: chiave/valore estensibile all'infinito
- `MetadataKey` enum estensibile milestone per milestone senza breaking changes

**Struttura:**
```typescript
interface Metadata {
  key: string;           // MetadataKey enum o stringa custom
  value: string;
  source?: string;       // 'google' | 'manual' | 'enrichment_api'
  collected_at?: string; // ISO timestamp
}

interface EnrichedBusiness extends Business {
  metadata: Metadata[];
}
```

---

## DTR-010 — Storico discovery: SQLite con better-sqlite3 (M11+)

**Data:** 2026-05-27
**Stato:** Pianificata (M11+)

**Decisione:** Il file `seen_firms.json` verrà sostituito da un database SQLite gestito tramite `better-sqlite3` per supportare lo storico discovery visualizzabile dalla GUI.

**Motivazione:**
- `seen_firms.json` è sufficiente per la deduplicazione ma non per query storiche
- SQLite è un singolo file `.db` nella cartella del progetto — zero server, zero configurazione, zero installazione aggiuntiva
- `better-sqlite3` è sincrono, performante e tipizzato
- È il database più distribuito al mondo — affidabilità comprovata

**Alternative valutate:**
- MongoDB/CouchDB: scartato — richiede server separato
- LowDB: scartato — non performante per query storiche
- PostgreSQL: scartato — overkill totale

---

## DTR-011 — Internazionalizzazione: 10 lingue di lancio

**Data:** 2026-05-27
**Stato:** Accettata

**Decisione:** SpotCast è distribuito con 10 lingue al lancio, selezionate per massimizzare la copertura del bacino di utenza mondiale.

**Lingue di lancio:** Italiano, Inglese, Tedesco, Francese, Spagnolo, Portoghese, Cinese semplificato, Giapponese, Arabo, Hindi.

**Motivazione:**
- Le 10 lingue coprono oltre l'80% del traffico web mondiale
- Hindi aggiunto per il bacino di utenza (600M+ parlanti)
- Arabo richiede attenzione specifica per il layout RTL nell'export Excel
- Lingue aggiuntive arriveranno nelle milestone successive su richiesta della community

---

## DTR-012 — GUI: JavaFX tramite ForgeUI (M10+)

**Data:** 2026-05-27
**Stato:** Pianificata (M10+)

**Decisione:** La GUI sarà costruita in JavaFX usando il design system ForgeUI, sviluppato in parallelo con NomadSync. SpotCast funge da progetto pilota di ForgeUI.

**Motivazione:**
- JavaFX è nativo, cross-platform e non richiede server
- ForgeUI garantisce coerenza visiva tra SpotCast e NomadSync
- La REST API (DTR-003) fornisce il ponte tra GUI Java e backend Node.js
- L'alternativa web (Next.js + Tailwind) richiederebbe un server — scartata

---

## DTR-013 — Installer grafico con wizard (M10+)

**Data:** 2026-05-27
**Stato:** Pianificata (M10+)

**Decisione:** SpotCast includerà un installer grafico che automatizza l'intera procedura di setup.

**Motivazione:**
- L'installazione manuale è una barriera per gli utenti non tecnici
- La distribuzione su Softonic richiede un'esperienza bubez-proof
- Il wizard è il punto di ingresso naturale per la GUI JavaFX/ForgeUI

---

## DTR-014 — Modello di pricing: Freemium con sblocco Pro dal mese 2

**Data:** 2026-05-27
**Stato:** Accettata (decisione di business)

**Decisione:** SpotCast è gratuito per il primo mese. Dal mese 2, le funzionalità avanzate richiedono un abbonamento Pro a **€3,99/mese** (o €39/anno).

**Piano gratuito:** pipeline base, fino a 5 categorie, fino a 5 città, schedule giornaliero, 10 lingue.

**Piano Pro:** categorie e città illimitate, storico aggregato multi-settimana, template email personalizzabili da GUI, supporto prioritario.

---

## DTR-015 — Distribuzione: MIT su GitHub + Softonic

**Data:** 2026-05-27
**Stato:** Accettata (decisione di business)

**Decisione:** SpotCast è distribuito come open-source (MIT) su GitHub e pubblicato su Softonic e piattaforme freeware equivalenti.

---

## DTR-016 — Politica di versioning delle dipendenze

**Data:** 2026-06-04
**Stato:** Accettata

**Decisione:** Le dipendenze vengono fissate alla versione major stabile più recente e aggiornate deliberatamente, non inseguendo ogni novità. Priorità alla compatibilità con il parco installato reale degli utenti finali.

**Motivazione:**
- L'utente finale di SpotCast non è un developer — il wizard installerà Node per lui
- La stessa filosofia governa Java 21 su NomadSync e ForgeUI: stabilità a lungo termine > feature recenti
- Node.js `>=20.0.0`, pnpm `>=9.0.0` — soglie che coprono il 90%+ delle installazioni attuali
- Express 4.x (non 5 RC), Zod 3.x (non 4 beta): major version in produzione stabile confermata

**Regola operativa:** prima di aggiornare una major version, verificare compatibilità con tutte le dipendenze del progetto e documentare la decisione in un nuovo DTR.

---

## DTR-017 — Package manager: pnpm

**Data:** 2026-06-04
**Stato:** Accettata

**Decisione:** Usare pnpm come package manager al posto di npm.

**Motivazione:**
- npm si era rivelato problematico in progetti precedenti del team (ToDoList) in combinazione con StoryBook
- pnpm è più veloce, deterministico e Docker-friendly
- Store globale con hard link: installazioni più veloci, `node_modules` più leggeri
- Workspaces nativi per eventuale evoluzione monorepo
- `pnpm-lock.yaml` garantisce riproducibilità cross-macchina

**Versione adottata:** pnpm 10.30.2 — ultima versione compatibile con Node.js 18+ (pnpm 11 richiede Node 22).

**Alternative valutate:**
- npm: scartato — storico problematico nel team
- yarn v1: scartato — legacy, non più sviluppato attivamente
- yarn Berry: scartato — plug'n'play crea incompatibilità con alcuni tool
- bun: scartato — ecosistema non sufficientemente maturo per produzione

---

## DTR-018 — ESLint: flat config con eslint.config.mjs

**Data:** 2026-06-04
**Stato:** Accettata

**Decisione:** Usare ESLint 9 con flat config in formato `.mjs` (ES Module) invece del formato legacy `.eslintrc.json`.

**Motivazione:**
- ESLint 9 non supporta più `.eslintrc.*` — migrazione obbligatoria
- `.mjs` permette di usare `import/export` nel file di configurazione indipendentemente dal `"type"` in `package.json`
- Evita di dover aggiungere `"type": "module"` al `package.json`, che richiederebbe estensioni `.js` esplicite in tutti gli import TypeScript — comportamento considerato inaccettabile dal team

**Regola generale:** CommonJS (`require`/`module.exports`) è vietato nel codebase. Se un tool specifico lo richiede obbligatoriamente, si usa l'estensione `.cjs` per quel singolo file (convenzione già adottata in ToDoList per la config di StoryBook).

---

## DTR-019 — Statistiche aggregata discovery: rinviata a M11+

**Data:** 2026-06-08
**Stato:** Rinviata (M11+)

**Decisione:** Nessuna struttura di statistiche aggregate (per categoria, città, totali storici) viene implementata in `seen_firms.json` o altrove prima di M11+.

**Motivazione:**
- Le statistiche per categoria e città richiedono aggregazione dinamica — non modellabile in modo pulito in un JSON flat
- Qualsiasi struttura inventata ora in JSON sarebbe un workaround da buttare alla migrazione SQLite
- Il principio è: zero è meglio di sbagliato

**Implementazione futura (M11+):**
- SQLite con `better-sqlite3` come sorgente primaria
- Statistiche tramite query aggregate sul database
- `seen_firms.json` rimane come fallback offline, senza campi statistici

---

## DTR-020 — `first_seen` nel modello Business

**Data:** 2026-06-08
**Stato:** Accettata — implementata in M3

**Decisione:** Il campo `first_seen` (ISO timestamp) viene aggiunto al modello `Business` come campo opzionale. Viene popolato da `DedupService.markSeen()` al momento della prima rilevazione. Il `GoogleFetcher` non lo conosce e lo lascia `undefined`.

**Motivazione:**
- Le informazioni sull'azienda devono stare nel modello `Business`, non duplicate in `seen_firms.json`
- `first_seen` è un attributo dell'azienda, non del sistema di deduplica
- Disponibile nativamente per M11+ quando i dati migreranno su SQLite

---

## DTR-021 — `seen_firms.json`: responsabilità unica

**Data:** 2026-06-08
**Stato:** Accettata

**Decisione:** `seen_firms.json` ha una sola responsabilità: tenere l'elenco dei `place_id` già inviati. Nessun altro dato vive in questo file — niente `first_seen`, niente categorie, niente statistiche, niente contatori.

**Struttura finale:**
```json
{
  "seen": ["ChIJ...", "ChIJ..."],
  "last_reset": "2026-06-04T08:00:00Z"
}
```

**Motivazione:**
- Separazione netta delle responsabilità: deduplica nel file, dati aziendali nel modello, statistiche in M11+
- Nessuna duplicazione di informazioni già presenti in `Business`
- File minimo, veloce, senza rischio di inconsistenza
