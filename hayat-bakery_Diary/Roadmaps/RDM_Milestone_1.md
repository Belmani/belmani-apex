# RDM_Milestone_1 — Roadmap
# Hayat Bäckerei — Sito Vetrina (Fase 1)

**Versione:** 1.0
**Data:** 2026-05-18
**Deadline:** 15 luglio 2026
**Product Owner:** Gabriela Belmani
**Tech Lead:** Alessandro De Palma

---

## Overview

La Milestone 1 copre la realizzazione del sito vetrina completo, mobile-first, con menu, form torte, multilingua e deploy su dominio reale. Non include e-commerce né AI.

**Durata totale stimata:** ~9 settimane
**Composizione:** Sprint 0 (setup) + 4 Sprint da ~2 settimane

---

## Sprint 0 — Setup & Allineamento
**Durata:** 3-4 giorni
**Obiettivo:** tutto il team lavora dallo stesso punto di partenza

### Attività

- [ ] Gabri crea repo `Belmani/hayat-bakery` (privata) e `Belmani/obsidian-portfolio` (pubblica)
- [ ] Alessandro aggiunto come collaboratore su entrambe
- [ ] Setup monorepo Turborepo + pnpm (`apps/web`, `apps/api`, `packages/shared`)
- [ ] Configurazione base Next.js 14 App Router + TypeScript
- [ ] Configurazione base Fastify + TypeScript
- [ ] Setup Prisma + connessione PostgreSQL (Railway o locale)
- [ ] Setup Tailwind CSS
- [ ] Setup Storybook in `packages/ui`
- [ ] `.gitignore`, `.env.example`, `README.md` iniziale
- [ ] Struttura vault Obsidian `hayat-bakery_Diary/` caricata su GitHub
- [ ] **Sessione data cleaning con cliente:** validazione completa catalogo menu (prezzi mancanti, duplicati, anomalie)

### Dipendenze critiche

- Accesso PostgreSQL (Railway o Neon) attivo
- Catalogo menu validato dal cliente prima dello Sprint 1

---

## Sprint 1 — Fondamenta UI & Menu
**Durata:** ~2 settimane
**Obiettivo:** layout one-page funzionante con menu completo visibile

### Attività

- [ ] Layout one-page con sezioni ancorate (Home, Menu, Torte, Chi siamo, Contatti, Wi-Fi)
- [ ] Bottom navigation bar (mobile) con scroll alle ancore
- [ ] Top navigation (desktop, breakpoint ≥ 768px)
- [ ] Schema DB menu: tabelle `categories`, `products` con Prisma migrations
- [ ] Seed DB con dati validati dal cliente
- [ ] API endpoint `GET /api/menu` (Fastify)
- [ ] Componente menu con categorie (Dolci, Colazione, Specialità Salate, Bevande)
- [ ] Card prodotto con foto, nome, prezzo, unità
- [ ] Lazy loading immagini via `<Image>` Next.js
- [ ] Hero section (foto/video sostituisce il Pexels stock dell'AS-IS)

### Dipendenze critiche

- Catalogo menu validato (da Sprint 0)
- Almeno alcune foto proprietarie dal cliente

---

## Sprint 2 — Multilingua & Contenuti
**Durata:** ~2 settimane
**Obiettivo:** sito fruibile in tedesco, turco e inglese

### Attività

- [ ] Setup `next-intl` con routing per locale (`/de`, `/tr`, `/en`)
- [ ] File traduzioni JSON per DE, TR, EN (tutti i testi statici del sito)
- [ ] Selettore lingua nella navigazione
- [ ] Sezione "Chi siamo" con testi validati dal cliente
- [ ] Sezione Contatti con: indirizzo, orari, mappa Google Maps embed, WhatsApp, telefono
- [ ] Sezione Wi-Fi in-page (password visibile in fondo alla pagina)
- [ ] Alt tag su tutte le immagini
- [ ] Meta tag dinamici per lingua (title, description, OG)

### Dipendenze critiche

- Testi "Chi siamo" approvati dal cliente
- Orari di apertura ufficiali
- Traduzione turca: verificata da cliente o da Gabri

---

## Sprint 3 — Form Torte & Backend
**Durata:** ~2 settimane
**Obiettivo:** form torte funzionante con notifica email al pasticcere

### Attività

- [ ] Componente form "Richiesta Torte" con campi: nome, tipo, dimensione, gusto/tema, data, telefono, note
- [ ] Validazione client-side con Zod
- [ ] Endpoint `POST /api/cake-requests` (Fastify)
- [ ] Schema DB `cake_requests` con Prisma migration
- [ ] Integrazione email: Resend (o Nodemailer) → email formattata al pasticcere
- [ ] Galleria torte esistenti (da foto AS-IS) nella stessa sezione
- [ ] Feedback visivo post-submit (successo / errore)
- [ ] JSON-LD structured data: `LocalBusiness`, `FoodEstablishment`, `Menu`
- [ ] `sitemap.xml` generata automaticamente da Next.js

### Dipendenze critiche

- Account Resend (o SMTP) configurato
- Indirizzo email del pasticcere confermato

---

## Sprint 4 — Polish, SEO & Deploy
**Durata:** ~2 settimane
**Obiettivo:** sito in produzione su dominio reale

### Attività

- [ ] Lighthouse audit: Performance, Accessibility, SEO, Best Practices → target ≥ 90 su mobile
- [ ] Fix eventuali problemi di accessibilità emersi dall'audit
- [ ] Test cross-browser: Chrome, Safari, Firefox (mobile e desktop)
- [ ] Test multilingua completo su tutte le sezioni
- [ ] Test form torte end-to-end (submit → email ricevuta)
- [ ] Setup progetto Railway: web + api + PostgreSQL
- [ ] Configurazione variabili d'ambiente su Railway
- [ ] Deploy staging su URL temporaneo Railway
- [ ] Validazione finale con cliente (Gabri presenta)
- [ ] Migrazione dominio `hayat-baeckerei.de` → Railway
- [ ] Verifica DNS e HTTPS attivo
- [ ] Monitoraggio post-deploy (48h)

### Dipendenze critiche

- Cooperazione VoiceHub per trasferimento dominio
- Approvazione finale cliente prima del go-live

---

## Criteri di Completamento Milestone 1

La Milestone 1 si considera completata quando:

1. Il sito è accessibile su `hayat-baeckerei.de` con HTTPS
2. Menu completo visibile in DE, TR, EN senza errori
3. Form torte invia email al pasticcere correttamente
4. Lighthouse mobile score ≥ 90 su Performance e SEO
5. JSON-LD validato su Google Rich Results Test
6. Nessun alt tag vuoto sulle immagini
7. Sezione Wi-Fi con password corretta visibile in-page
8. Bottom nav funzionante su iPhone e Android (Chrome + Safari)

---

## Note

Le roadmap di dettaglio per ciascuno sprint (task breakdown, assegnazione, story points) saranno prodotte separatamente prima dell'inizio di ogni sprint.

Fasi successive (e-commerce, AI torte) sono documentate in `GRM_Milestone_1.md` sezioni 6 e 7.
