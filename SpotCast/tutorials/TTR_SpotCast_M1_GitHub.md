## Tutorial Branch Rulesets di GitHub

Quello che stai vedendo è il sistema di **Branch Rulesets** — il meccanismo con cui definisci le regole che governano cosa si può fare su un branch specifico. Vale la pena capirlo bene perché lo userai su ogni progetto serio.

---

### Il concetto di base

Un **Ruleset** è un insieme di regole applicate a uno o più branch. Quando qualcuno tenta un'operazione su quel branch (push, merge, force push), GitHub verifica che rispetti tutte le regole attive. Se non le rispetta, l'operazione viene bloccata.

---

### Le voci che stai vedendo — una per una

**Required approvals (attualmente: 0)** Quante persone devono approvare una PR prima che possa essere mergiata. Con `0` la PR può essere mergiata da chi l'ha aperta senza nessuna review esterna. Per un team di due persone in fase di sviluppo rapido va benissimo — lo alzerai a `1` quando il progetto avrà contributor esterni.

**Dismiss stale pull request approvals when new commits are pushed** Se qualcuno approva una PR e poi l'autore aggiunge altri commit, l'approvazione viene invalidata automaticamente. Utile in team grandi per evitare che si mergino modifiche non revisionate. Per ora: lascia deselezionato.

**Require review from specific teams** Permette di designare team specifici come reviewer obbligatori per certi file. Feature enterprise, non rilevante per SpotCast.

**Require review from Code Owners** Simile al precedente — richiede la review da chi è designato "owner" di certi file tramite un file `CODEOWNERS`. Non rilevante ora.

**Require approval of the most recent reviewable push** L'ultimo push prima del merge deve essere approvato da qualcuno che non sia l'autore del push stesso. Protezione extra contro l'auto-approvazione. Per ora: lascia deselezionato.

**Require conversation resolution before merging** Tutti i commenti aperti su una PR devono essere risolti prima del merge. Buona pratica in team, ma per due persone è overkill. Per ora: lascia deselezionato.

**Allowed merge methods — Merge, Squash, Rebase** Definisce come i commit di una PR possono essere integrati in main. Tre modalità:

| Metodo     | Cosa fa                                      | Quando usarlo                            |
| ---------- | -------------------------------------------- | ---------------------------------------- |
| **Merge**  | Crea un merge commit che unisce i due branch | Mantiene la storia completa              |
| **Squash** | Comprime tutti i commit della PR in uno solo | Storia pulita, ideale per feature branch |
| **Rebase** | Riscrive i commit della PR sopra main        | Storia lineare, nessun merge commit      |

Per SpotCast il consiglio è di tenere tutti e tre disponibili per ora e decidere una convenzione con Alessandro dopo la prima PR reale. Molti team senior preferiscono **Squash** per tenere il log di main leggibile.

**Require status checks to pass** Qui si collegano le pipeline di CI/CD — in pratica dici a GitHub "prima di permettere il merge, assicurati che questi check automatici siano passati" (es. `npm test`). **Non attivarla adesso** perché non abbiamo ancora GitHub Actions configurato. Torneremo qui quando aggiungeremo la CI con Vitest.