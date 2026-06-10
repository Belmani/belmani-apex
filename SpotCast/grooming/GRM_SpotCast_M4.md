# SpotCast — Grooming

# GRM-M4 — Esportatore Excel

## Obiettivo
Generare un file `.xlsx` formattato professionalmente con tre fogli dai risultati filtrati.

## User Story
*Come utente, voglio ricevere un file Excel pulito e pronto all'uso, così da poter lavorare subito con i lead.*

## Criteri di Accettazione
- [ ] `ExcelExporter.ts` implementato ed esportato
- [ ] Genera un file `.xlsx` nella cartella `output_dir` configurata
- [ ] Nome file: `SpotCast_YYYY-MM-DD.xlsx`
- [ ] **Sheet 1 — Businesses:** una riga per attività, colonne secondo il modello `Business`, riga intestazione stilizzata (grassetto, sfondo `#2E75B6`, testo bianco)
- [ ] **Sheet 2 — Email Templates:** una riga per attività con corpo email di contatto pre-compilato nella lingua attiva
- [ ] **Sheet 3 — Run Summary:** `run_date`, `total_found`, `duplicates_skipped`, `data_source`, `language`
- [ ] Larghezze colonne adattate al contenuto
- [ ] Tutte le stringhe visibili provengono dal file lingua i18n attivo — nessun testo hardcodato
- [ ] Test unitari implementati con lettura del file generato

## Test (Vitest)

| Caso | Tipo | Descrizione |
|---|---|---|
| Tre fogli presenti | Unit | Il file generato contiene esattamente 3 fogli |
| Nomi fogli corretti | Unit | Nomi fogli corrispondono al file lingua attivo |
| Riga intestazione Sheet 1 | Unit | Prima riga con tutti i campi attesi di `Business` |
| Dati corretti Sheet 1 | Unit | Ogni riga corrisponde al business in input |
| Sheet 3 — run_date | Unit | Data nel formato `YYYY-MM-DD` |
| Sheet 3 — conteggi | Unit | `total_found` e `duplicates_skipped` corretti |
| Cartella output mancante | Unit | Viene creata automaticamente senza errori |
| Array business vuoto | Unit | File generato con soli header, nessun crash |

## Note Tecniche
- Usare **ExcelJS** — vedi `TUTORIAL_stack_tecnico.md` per esempio completo
- Colori alternati righe: bianco / `#EBF3FB`
- Cartella output creata automaticamente: `fs.mkdirSync(outputDir, { recursive: true })`
- I test generano il file in `os.tmpdir()` e lo rileggono con ExcelJS reader per le asserzioni

## Stima di Effort
**4–5 ore** (inclusi test)

---