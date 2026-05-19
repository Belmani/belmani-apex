# Wireframe — Hayat Bäckerei
## Mobile-first · One-page · Milestone 1 · v3.0

> **⚠️ IMPORTANTE — HERO E FOOTER SONO FISSI**
> L'hero in cima e il footer (bottom nav) in fondo sono elementi **position: fixed** — rimangono sempre visibili mentre l'utente scorre la pagina. L'intera single-page scorre nella fascia intermedia tra i due elementi fissi.

| Versione | Link | Destinatario |
|---|---|---|
| 🇮🇹 Italiano | [WFR_Milestone_1_mobile_IT.svg](./WFR_Milestone_1_mobile_IT.svg) | Team di sviluppo |
| 🇩🇪 Tedesco | [WFR_Milestone_1_mobile_DE.svg](./WFR_Milestone_1_mobile_DE.svg) | Cliente |

---

## Struttura sezioni

| # | Sezione | Ancora | Stato |
|---|---|---|---|
| — | **Hero (FISSO)** | — | ID business · lingua · tema · auth · carrello M2 |
| 1 | Menu | `#menu` | 4 fasce Netflix: Dolci · Colazione · Salate · Bevande |
| 2 | Personalizza | `#torte` | Form torta (M1) · Generatore AI (M3, hidden) |
| 3 | Chi siamo | `#about` | Foto + testo breve |
| 4 | Raggiungici | `#location` | 3 step: Indirizzo → Mappa → Wi-Fi |
| 5 | Contatti | `#contatti` | Telefono · WhatsApp · email · social · orari |
| — | **Footer / Bottom nav (FISSO)** | — | Navigazione immediata alle sezioni |

---

## Hero — elementi e gerarchia

**① Identità business** — logo + nome + orari, sempre visibili.

**② Selettore lingua — controllo primario**
```
[ 🌐  DE ▾ ]   pill nera (#1c1917), testo bianco
```
Pill scura a pieno riempimento. Il peso visivo comunica priorità massima: questa scelta determina se l'utente capisce il sito. Rilevamento automatico da `navigator.language` al primo render, persistito su `localStorage`.
Lingue al go-live M1: **DE · TR · EN** — IT e FR in patch successiva.

**③ Selettore tema — controllo subalterno**
```
[ ☀️ · 🌙 · auto ]   bordo sottile, sfondo bianco
```
Rettangolo con solo bordo sottile. Stessa area del lang switcher, peso visivo dimezzato. La differenza di forma (pill vs rettangolo) comunica la subalternità senza testo aggiuntivo. Opzioni: chiaro · scuro · segui sistema (default). Persistito su `localStorage`.

**④ ComboButton autenticazione + carrello M2**

Componente riutilizzabile del design system. Riunisce un'azione principale e azioni correlate in un unico controllo, accessibili tramite il selettore `▾` sulla destra — pattern analogo alle ComboBox, con icona e contenuto del popup completamente personalizzabili dall'esterno come props del componente Next.js.

*Stato non autenticato:*
```
[ 🔑 | ▾ ]   → click icona: modal login / click ▾: dropdown login + registrati
```

*Stato autenticato:*
```
[ [AD] | ▾ ]   → avatar + dropdown contestuale
```

Il carrello appare solo se `session && FEATURES.ORDERS === true` — invisibile in M1 per utenti non loggati, visibile ma disabilitato per utenti loggati (badge M2, bordo tratteggiato giallo).

---

## Avatar — opzioni configurabili al signup

| Opzione | Servizio | Esempio URL | Note |
|---|---|---|---|
| Iniziali | DiceBear Initials | `https://api.dicebear.com/8.x/initials/svg?seed=AD` | Generato da nome/cognome |
| Foto caricata | — | upload diretto | Opzionale |
| Robot | Robohash set1 | `https://robohash.org/{userId}?set=set1` | Unico per userId |
| Mostri | Robohash set2 | `https://robohash.org/{userId}?set=set2` | Unico per userId |
| Gatti | Robohash set4 | `https://robohash.org/{userId}?set=set4` | Unico per userId |
| Pixel art | DiceBear Pixel Art | `https://api.dicebear.com/8.x/pixel-art/svg?seed={name}` | Stabile per seed |
| Astratto | DiceBear Shapes | `https://api.dicebear.com/8.x/shapes/svg?seed={name}` | Stabile per seed |

Tutti gratuiti, nessuna API key, nessun rate limit rilevante. Il `seed` garantisce stabilità: stesso utente → stesso avatar sempre.

---

## Popup utente loggato — struttura

```
┌─────────────────────────┐
│  @username              │  ← label, non cliccabile
│  mail@esempio.com       │  ← label, non cliccabile
├─────────────────────────┤
│  👤  Il mio profilo     │  ← impostazioni account e avatar
│  🛒  I miei ordini [M2] │  ← storico ordini (visibile solo se FEATURES.ORDERS)
│  🎂  Le mie torte       │  ← richieste torte inviate
├─────────────────────────┤
│  🌐  Lingua             │  ← alias del lang switcher hero
├─────────────────────────┤
│  🚪  Logout             │  ← azione distruttiva, sempre in fondo
└─────────────────────────┘
```

Nota: Lingua nel popup è alias dello stesso switcher dell'hero — utile su desktop quando l'hero può essere fuori viewport durante lo scroll.

---

## Footer / Bottom navigation — FISSO

4 voci fisse sempre visibili. Navigazione immediata alle sezioni principali via `scrollIntoView({ behavior: 'smooth' })`.

| Voce | Ancora | Note |
|---|---|---|
| 🍽 Menu | `#menu` | attivo di default |
| 🎂 Torte | `#torte` | — |
| 📍 Dove | `#location` | — |
| 📞 Contatti | `#contatti` | — |

In M2: si aggiunge voce **🛒 Carrello** quando `FEATURES.ORDERS === true`.
Su desktop (≥768px): il footer diventa top navigation orizzontale.

---

## Menu — pattern Netflix

Una fascia orizzontale per categoria. Scroll laterale. Card con foto, nome, prezzo, tasto "Acquista" (disabilitato M1, attivo M2). Colori per categoria:

| Categoria | Colore sfondo card |
|---|---|
| Dolci | `#f0fdf4` verde chiaro |
| Colazione | `#fefce8` giallo chiaro |
| Specialità Salate | `#fef2f2` rosso chiaro |
| Bevande | `#eff6ff` blu chiaro |

Dati da: `GET /api/menu?category={slug}`

---

## Feature flags

```typescript
// apps/web/config/features.ts
export const FEATURES = {
  ORDERS:  false,  // M2 — carrello, checkout, pagamenti Stripe/PayPal
  AI_CAKE: false,  // M3 — generatore torte DALL-E / Replicate
}
```

| Componente | M1 | M2 | M3 |
|---|---|---|---|
| Tasto Acquista | visibile, disabilitato | attivo | attivo |
| Carrello hero | visibile, disabilitato | attivo (solo login) | attivo |
| Carrello footer | assente | presente | presente |
| "I miei ordini" nel popup | assente | presente | presente |
| Generatore AI | placeholder | placeholder | attivo |

---

*Wireframe v3.0 · 2026-05-19 · Hayat Bäckerei*
*Product Owner: Gabriela Belmani · Tech Lead: Alessandro De Prato*
