# nomad-portfolio

**Nomad Portfolio Vault**

Questa repository è la knowledge base pubblica del percorso di apprendimento e sviluppo di Gabriela Belmani, costruita attorno a progetti reali sviluppati con metodologia Agile e pair programming con il Tech Lead Alessandro De Prato.

Il vault documenta il ciclo di vita completo di ogni progetto: decisioni architetturali, pianificazione degli sprint, retrospettive di milestone e autovalutazioni — strutturati come un vero workflow professionale.

---

## Perché esiste

La maggior parte dei portfolio mostra il prodotto finito. Questo mostra il processo.

Ogni decisione ha una motivazione. Ogni ostacolo ha un post-mortem. Ogni sprint ha un punteggio. L'obiettivo è dimostrare non solo cosa è stato costruito, ma **come** — il ragionamento, i trade-off e la crescita sprint dopo sprint.

---

## Struttura della repository

```
nomad-portfolio/
└── hayat-bakery_Diary/
    ├── Analysis/               ← Materiale pre-sviluppo: analisi AS-IS e requisiti
    │   ├── AS-IS/
    │   │   └── Features.md     ← Analisi sito attuale con gap vs requirements
    │   └── Requirements/       ← Requisiti cliente formalizzati
    ├── Brainstorming/          ← Sessioni di ideazione e valutazione alternative
    │   └── BRN_Milestone_1.md
    ├── Diagrams/               ← Architettura, ER diagram, flussi
    │   └── (in arrivo)
    ├── Groomings/              ← Pianificazione sprint + Architecture Decision Records
    │   └── GRM_Milestone_1.md
    ├── Meeting_Hayat/          ← Materiale preparato per e durante le riunioni col cliente
    │   ├── WFR_Milestone_1_mobile_DE.svg
    │   ├── WFR_Milestone_1_mobile_IT.svg
    │   └── WFR_Milestone_1_mobile.md
    ├── Milestones/             ← Retrospettive di sprint: obiettivi, problemi, cheat-sheet
    │   └── (in arrivo dopo M1)
    ├── Questions/              ← Domande aperte da risolvere col cliente o col team
    │   └── QST_Milestone_1.md
    ├── Roadmaps/               ← Pianificazione implementativa per milestone
    │   └── (in arrivo)
    ├── Troubleshooting/        ← Post-mortem su problemi incontrati
    │   └── (in arrivo)
    ├── Tutorials/              ← Cheat-sheet tecnici acquisiti sprint per sprint
    │   └── (in arrivo)
    └── Wireframes/             ← Wireframe e mockup UI
        └── (vedi Meeting_Hayat/ per i wireframe presentati al cliente)
```

---

## Il progetto: Hayat Bäckerei Website

Rifacimento completo del sito web per **Hayat Bäckerei**, pasticceria turca a Wiesbaden. Il sito attuale è un no-code builder (Zyro/Hostinger) senza codice sorgente recuperabile. Il nuovo sito è mobile-first, one-page, multilingua, con menu da database e form ordini torte strutturato.

**Tech stack:** Next.js 14 (App Router) · TypeScript · Tailwind CSS · Fastify · Prisma · PostgreSQL (Neon) · next-intl · Turborepo · pnpm

**Repo codice:** [github.com/Belmani/hayat-bakery](https://github.com/Belmani/hayat-bakery) *(privata)*

### Punti architetturali salienti

- **Monorepo Turborepo + pnpm** — build caching, tipi TypeScript condivisi tra frontend e backend via `/packages/shared`, nessun problema di hoisting
- **Next.js App Router con SSR** — SEO critico per business locale: JSON-LD `LocalBusiness` + `FoodEstablishment` + `Menu`, sitemap automatica
- **Bottom navigation mobile** — zero hamburger menu per esplicita richiesta del cliente; accesso diretto a tutte le sezioni con 1 tap
- **Fastify + Prisma** — API leggera TypeScript-native; schema PostgreSQL relazionale pensato già per ospitare l'e-commerce in Milestone 2
- **next-intl** — internazionalizzazione DE/TR/EN con routing integrato in App Router; IT e FR pianificati in patch successiva

### Roadmap fasi

| Fase | Scope | Stato |
|---|---|---|
| Milestone 1 | Sito vetrina completo, mobile-first, multilingua, form torte, SEO | 🔄 In corso |
| Milestone 2 | E-commerce: carrello, checkout, Stripe + PayPal | 📅 Post settembre 2026 |
| Milestone 3 | Generatore AI di torte personalizzate (DALL-E 3) | 📅 Da pianificare |

---

## Come leggere questo vault

**1. Inizia con `Analysis/AS-IS/Features.md`** — l'analisi completa del sito esistente: stack tecnico, struttura pagine, dati menu estratti, criticità SEO e performance, gap vs requisiti cliente. È il punto di partenza di tutto.

**2. Poi leggi `Groomings/GRM_Milestone_1.md`** — le scelte architetturali fatte prima di scrivere una riga di codice. Ogni trade-off è documentato con contesto, motivazione e alternative scartate. Include gli obiettivi della milestone con criteri di accettazione misurabili.

**3. Per il contesto cliente**, `Meeting_Hayat/` contiene i wireframe mobile presentati durante le riunioni e le note di confronto. `Questions/QST_Milestone_1.md` raccoglie le domande aperte risolte o in attesa di risposta dal cliente.

**4. Per il brainstorming**, `Brainstorming/BRN_Milestone_1.md` documenta le sessioni di ideazione: alternative valutate, stack confrontati, decisioni di hosting con analisi costi.

A milestone completata, il pattern si arricchisce con: `Roadmaps/` (roadmap implementativa dello sprint) → `Troubleshooting/` (problemi incontrati e post-mortem) → `Tutorials/` (cheat-sheet tecnici acquisiti) → `Milestones/` (retrospettiva finale).

---

## About

**Gabriela Belmani** · Software Engineer
[GitHub](https://github.com/Belmani) · [LinkedIn](https://www.linkedin.com/in/gabriela-da-sa%C3%BAde-belmani-tumfart)

**Alessandro De Prato** · Senior Software Engineer — Tech Lead & Coach
[Portfolio](https://aledep10.github.io/) · [GitHub](https://github.com/AleDeP10) · [LinkedIn](https://www.linkedin.com/in/alessandro-de-prato)
