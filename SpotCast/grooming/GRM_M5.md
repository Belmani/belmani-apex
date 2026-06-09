# SpotCast — Grooming

# GRM-M5 — Mail Service

## Obiettivo
Inviare il file Excel generato come allegato email a tutti i destinatari configurati.

## User Story
*Come utente, voglio che i risultati vengano consegnati automaticamente nella mia inbox, così non devo aprire nessuna applicazione per accedervi.*

## Criteri di Accettazione
- [ ] `MailService.ts` implementato ed esportato
- [ ] Invia email tramite Nodemailer con credenziali SMTP da config
- [ ] File Excel allegato con nome file corretto
- [ ] Oggetto e corpo email renderizzati da `templates/email.html` tramite **Handlebars**
- [ ] Variabili template: `{{date}}`, `{{count}}`, `{{categories}}`, `{{cities}}`
- [ ] Invia a tutti gli indirizzi nell'array `email_to` di config
- [ ] In caso di errore SMTP: logga l'errore completo, non crasha il processo
- [ ] Se `count === 0` dopo deduplicazione: skip invio email, log `No new businesses found, skipping email`
- [ ] Test unitari con mock del transport Nodemailer

## Test (Vitest)

| Caso | Tipo | Descrizione |
|---|---|---|
| Payload `to` corretto | Unit | `email_to` dal config passato correttamente a Nodemailer |
| Allegato presente | Unit | `attachments` contiene il file Excel con nome corretto |
| `subject` renderizzato | Unit | Handlebars sostituisce `{{date}}` nell'oggetto |
| `body` renderizzato | Unit | Handlebars sostituisce tutte le variabili nel corpo |
| Errore SMTP | Unit | Logga l'errore, non lancia eccezione non gestita |
| `count === 0` | Unit | `sendMail` non viene invocato, log di skip emesso |
| Destinatari multipli | Unit | Tutti gli indirizzi in `email_to` presenti nel payload |

## Note Tecniche
- **Handlebars** per il rendering del template — vedi `TUTORIAL_stack_tecnico.md`
- Per Gmail: l'utente deve generare una **App Password** — documentato nel tutorial
- Tutto il logging in inglese: `Email sent to 2 recipients`, `SMTP error: ...`

## Stima di Effort
**3–4 ore** (inclusi test)

---