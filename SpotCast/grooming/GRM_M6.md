# SpotCast — Grooming

# GRM-M6 — Scheduler

## Obiettivo
Automatizzare SpotCast in modo che giri su un orario configurabile senza intervento manuale.

## User Story
*Come utente, voglio che SpotCast parta automaticamente ogni mattina, così i miei lead arrivano senza che io debba ricordarmi di avviarlo.*

## Criteri di Accettazione
- [ ] `Scheduler.ts` implementato ed esportato
- [ ] Legge l'espressione cron da `config.schedule` (default: `"0 8 * * *"`)
- [ ] Quando attivato, esegue la pipeline completa: fetch → dedup → export → send email
- [ ] Ogni esecuzione loggata con timestamp, conteggio, durata e stato
- [ ] Attivato quando SpotCast viene avviato con il flag `--daemon`
- [ ] Senza `--daemon`: esecuzione singola immediata, poi il processo termina
- [ ] Espressione cron non valida: log `Invalid cron expression: "..."` ed exit con codice 1
- [ ] Test unitari con mock della pipeline e del clock

## Test (Vitest)

| Caso | Tipo | Descrizione |
|---|---|---|
| Espressione cron valida | Unit | `startScheduler` non lancia eccezione |
| Espressione cron non valida | Unit | Exit con codice 1 e messaggio descrittivo |
| Esecuzione pipeline al trigger | Unit | Mock pipeline invocato quando il cron scatta |
| Log `RUN STARTED` | Unit | Emesso ad ogni avvio di run |
| Log `RUN COMPLETE` con conteggio e durata | Unit | Emesso al completamento con dati corretti |
| Errore pipeline non propaga | Unit | Scheduler rimane attivo dopo un run fallito |
| Modalità singola run (senza `--daemon`) | Integration | Pipeline eseguita una volta, processo termina |

## Note Tecniche
- **node-cron** per la pianificazione — vedi `TUTORIAL_stack_tecnico.md`
- Esecuzione pipeline in try/catch — lo scheduler non crasha mai per un errore di singolo run
- Formato log (tutto in inglese):
  ```
  [2026-05-28 08:00:03] INFO: RUN STARTED
  [2026-05-28 08:00:07] INFO: RUN COMPLETE — 8 new businesses | duration: 4.2s
  [2026-05-28 08:00:07] WARN: No new businesses found, skipping email
  [2026-05-28 08:00:07] ERROR: RUN FAILED: Google API error: quota exceeded
  ```

## Stima di Effort
**2–3 ore** (inclusi test)

---

## Riepilogo Deadline Venerdì

| Milestone | Effort con TDD | Cumulativo |
|---|---|---|
| M1 — Scaffold | 3–4h | 3–4h |
| M2 — Fetcher | 4–5h | 7–9h |
| M3 — Dedup | 2–3h | 9–12h |
| M4 — Excel | 4–5h | 13–17h |
| M5 — Mailer | 3–4h | 16–21h |
| M6 — Scheduler | 2–3h | 18–24h |

**Effort totale stimato: 18–24 ore** per una coppia (Alessandro + Gabriela).

Sessione di pair domani mattina + giovedì = copertura completa M1→M6.
Venerdì = test di integrazione end-to-end sullo scenario Mac di Onur + preparazione consegna.

**La deadline tiene.** 🎯

---