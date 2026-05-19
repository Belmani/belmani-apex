Grm milestone 1 · MD
Copia

GRM — Milestone 1: Sito Vetrina
Contesto
Il sito attuale di Hayat Bäckerei (hayat-baeckerei.de) è costruito su Hostinger Website Builder (Zyro), piattaforma no-code che non produce codice sorgente recuperabile. Il cliente versa 50€/mese per hosting e manutenzione non erogata. Il contratto passerà al nostro team alla messa in produzione del nuovo sito.

La Milestone 1 copre l'intero sito vetrina: identità turca, layout one-page mobile-first, menu da database, form torte strutturato, multilingua DE/TR/EN, SEO con structured data e deploy su dominio definitivo. Le funzionalità e-commerce (M2) e il generatore AI di torte (M3) sono sviluppi futuri esplicitamente fuori scope.

Obiettivi

# Obiettivo Criterio di accettazione

1 Setup monorepo Turborepo + pnpm funzionante, CI/CD verde su GitHub Actions
2 Layout one-page Tutte le sezioni ancorate navigabili, nessun routing multi-pagina
3 Bottom navigation mobile Visibile su viewport ≤ 768px, scroll-to-anchor funzionante, nessun hamburger
4 Top navigation desktop Visibile su viewport > 768px, link alle sezioni
5 Schema PostgreSQL Tabelle Product, Category, PriceUnit definite e migrate con Prisma
6 API menu GET /api/products con filtro categoria, risposta tipizzata
7 Sezioni menu (4 categorie) Dolci, Colazione, Salate, Bevande caricate da DB e visibili
8 Form Torte anfragen Campi strutturati, validazione Zod, email notification al pasticcere
9 Wi-Fi password in-page Visibile nella sezione Contatti, nessun foglietto fisico necessario
10 Multilingua DE/TR/EN next-intl configurato, language selector funzionante, zero stringa hardcoded
11 Brand turco Zero riferimenti ad "Afghano/Afghanisch" in testi, meta, URL, structured data
12 JSON-LD SEO Schema LocalBusiness + FoodEstablishment + Menu generati correttamente
13 Lighthouse ≥ 85 mobile Performance + SEO + Accessibility su viewport 375px
14 Deploy production Sito live su hayat-baeckerei.de, SSL attivo, dominio migrato
Ordine implementativo
Layer 1 — Setup & Infrastruttura
Step 1 — Monorepo Turborepo + pnpm

Struttura iniziale del repo:

hayat-bakery/
├── apps/
│ ├── web/ → Next.js 14, App Router, TypeScript, Tailwind
│ └── api/ → Fastify, TypeScript, Prisma
├── packages/
│ ├── shared/ → tipi condivisi (Product, Order, CartItem...)
│ └── ui/ → componenti design system (futuro Storybook)
├── turbo.json
└── package.json
Da eseguire prima di qualsiasi sviluppo — impatta tutto il pipeline. Verde su pnpm build prima di proseguire.

Step 2 — GitHub Actions CI

Pipeline minima:

yaml
on: [push, pull_request]
jobs:
build:
steps: - pnpm install - pnpm lint - pnpm build
Step 3 — Provisioning infrastruttura

Servizio Provider Note
App hosting Railway Deploy da GitHub, usage-based ~6€/mese
PostgreSQL Neon Serverless, scale-to-zero, ~3€/mese
Email notification Resend Form torte — free tier sufficiente per M1
Dominio hayat-baeckerei.de Migrazione coordinata con VoiceHub al go-live
Step 4 — Prisma init + schema base

prisma
model Product {
id Int @id @default(autoincrement())
name String
nameDE String?
nameTR String?
nameEN String?
price Float?
priceUnit PriceUnit
category Category
available Boolean @default(true)
imageUrl String?
isFeatured Boolean @default(false)
createdAt DateTime @default(now())
}

enum Category {
SWEETS
BREAKFAST
SAVORY
DRINKS
}

enum PriceUnit {
PER_100G
PRO_STUECK
AB
FIXED
}
Anomalie da risolvere prima del seed — validare col cliente:

Problema Pagina Prodotto
Prezzo mancante Dolci Havuc, Halka, Creme Rolle, Kiwi Trilece, Croissant
Prezzo mancante Colazione Börek
Prezzo mancante Salate Chapli Kebab, Mantu
Dato orfano 5,00€ Salate Nessun nome associato
Duplicato Dolci Şirin Baklava compare 2 volte
Prezzo ambiguo Dolci Donut: 1,20€ o 1,00€
Foto stock Colazione + Salate Brot, Linsensuppe, Bolani — da sostituire
Layer 2 — Layout & Navigation
Step 5 — Struttura one-page Next.js

app/page.tsx con sezioni ancorate:

tsx

<main>
  <HeroSection id="home" />
  <MenuSection id="menu" />
  <CakeRequestSection id="torte" />
  <ContactSection id="kontakt" />
</main>
Nessun routing multi-pagina in M1. Le voci di navigation puntano ad anchor #menu, #torte, #kontakt.

Step 6 — Navigation

Componente BottomNav (mobile, position: fixed; bottom: 0):

┌──────────────────────────────────────────┐
│ 🏠 Home 🍰 Menu 🎂 Torte 📞 Info │
└──────────────────────────────────────────┘
Visibile solo su viewport ≤ 768px. Su desktop: TopNav orizzontale con gli stessi link. Language selector in entrambi.

Step 7 — Design system base

Palette colori, tipografia e spacing definiti come token Tailwind in tailwind.config.ts. Componenti base: Button, Card, Section, ProductCard, PriceTag, NavItem.

Responsive testato sui tre breakpoint: 375px (iPhone SE), 768px (tablet), 1440px (desktop).

Layer 3 — Dati & API
Step 8 — Fastify API

Endpoint M1:

Metodo Path Descrizione
GET /api/products Lista prodotti, query param ?category=SWEETS
GET /api/products/:id Singolo prodotto
POST /api/cake-requests Invia richiesta torta + email notification
Validazione request/response con Zod, tipi condivisi nel package /packages/shared.

Step 9 — Sezioni menu

Componente MenuSection con tab o scroll per categoria. ProductCard mostra: foto (next/image con srcset), nome, prezzo, unità. Dati caricati lato server con fetch in Server Component Next.js.

Step 10 — Form Torte anfragen

Campi:

Campo Tipo Validazione
Nome cliente text required
Tipo torta select: Herzform / Rund / Personalizzata required
Dimensione select: 15cm / 20cm / 30cm required
Gusto / tema textarea required
Data consegna date required, min oggi + 3 giorni
Telefono tel required
Note textarea optional
Submit → POST /api/cake-requests → Resend invia email al pasticcere con tutti i campi formattati.

Layer 4 — i18n & SEO
Step 11 — next-intl

Setup middleware + file JSON traduzioni:

apps/web/
└── messages/
├── de.json ← lingua base, completa
├── tr.json
└── en.json
Language selector nella navigation aggiorna la locale via useRouter(). Zero stringa hardcoded nei componenti — tutto passato via useTranslations().

Step 12 — Brand turco + contenuti

Rimozione di ogni occorrenza "Afghano/Afghanisch" da testi, placeholder, commenti
Meta title/description aggiornati con generateMetadata() per ogni sezione
URL slug puliti — nessun keyword stuffing
Step 13 — JSON-LD structured data

Inserito in app/layout.tsx:

tsx

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Bakery",
  "name": "Hayat Bäckerei",
  "address": { ... },
  "openingHours": [ ... ],
  "hasMenu": { "@type": "Menu", ... }
}
</script>

Sitemap.xml generata automaticamente con next-sitemap. Alt tag su tutte le immagini.

Layer 5 — Deploy & Go-Live
Step 14 — Lighthouse audit

Target prima del go-live:

Categoria Target
Performance ≥ 85
Accessibility ≥ 90
Best Practices ≥ 90
SEO ≥ 95
Step 15 — Migrazione dominio

1. Test completo su staging URL Railway
2. Revisione finale col cliente
3. Abbassare TTL DNS a 300s (coordinare con VoiceHub con anticipo)
4. Deploy production Railway
5. VoiceHub cancella/trasferisce dominio → puntare nameserver a Railway
6. Verifica SSL, redirect, form, traduzioni
7. Submit sitemap su Google Search Console
   Downtime atteso: minuti, non ore.

Rischi
Rischio Probabilità Impatto Mitigazione
Credenziali dominio non trasferibili in tempo Media Alto Sviluppo su staging URL, coordinare VoiceHub con anticipo di 2 settimane
Dati menu incompleti — cliente non ricorda prezzi Media Medio Sessione validazione con cliente prima dello Sprint 2, placeholder visibili
Foto di qualità insufficiente Media Medio Richiedere scatti aggiuntivi al cliente; placeholder branded nel frattempo
Divergenza istanze Claude Alessandro/Gabri Bassa Medio Sincronizzazione Slack ad ogni sessione di grooming, non ad ogni commit
Slittamento timeline post-luglio Bassa Alto Scope M1 volutamente conservativo — zero e-commerce, zero AI
git merge stderr localizzato rompe parsing output Bassa Medio Non applicabile in M1 — nota per M2 con integrazione CI
Sviluppi Futuri (fuori scope M1)
⚠️ Le sezioni seguenti documentano le fasi successive già pianificate. Non sono incluse nella Milestone 1.

Milestone 2 — E-Commerce (Checkout)
Carrello persistente (Zustand), checkout multi-step, integrazione Stripe + PayPal SDK, gestione ordini admin, email conferma con Resend. Schema DB aggiuntivo: Order, OrderItem, Customer. Autenticazione admin con NextAuth.js.

Data inizio prevista: post settembre 2026, dopo il PRIMARY_GOAL microservizi.

Milestone 3 — Generatore AI di Torte
Configuratore visivo con rendering in tempo reale tramite DALL-E 3 (OpenAI) o Replicate. L'utente personalizza forma, colori, decorazioni, testo — ottiene anteprima AI prima di inviare la richiesta. Rate limiting lato Fastify per contenere i costi (~0.04-0.08$/immagine).

Valore di business: differenziazione competitiva unica sul mercato locale, potenziale virale.

Data inizio prevista: da definire post M2.
