# Tutorial pnpm
## Migrazione da npm + Guida completa

---

## Cos'è pnpm e perché è la scelta giusta

pnpm (Performant npm) risolve il problema fondamentale di npm e yarn v1: la duplicazione dei `node_modules`. Invece di copiare ogni pacchetto in ogni progetto, pnpm mantiene uno **store globale** sul disco e crea **hard link** da lì verso ogni `node_modules`. Il risultato:

- Install 2-3x più veloci di npm
- `node_modules` fino al 70% più leggeri
- Lockfile (`pnpm-lock.yaml`) deterministico — stesso risultato su qualsiasi macchina
- Supporto monorepo nativo con **workspaces**
- Compatibile con qualsiasi tool che funziona con npm (Vitest, TypeScript, Nodemon, ecc.)

---

## Installazione

```bash
# Metodo raccomandato — via corepack (incluso in Node.js 16+)
corepack enable
corepack prepare pnpm@latest --activate

# Verifica
pnpm --version
```

---

## Migrazione da npm a pnpm in SpotCast

```bash
# 1. Entra nella cartella del progetto
cd C:\Users\aless\source\repos\SpotCast

# 2. Cancella node_modules e il lockfile npm
rm -rf node_modules
rm package-lock.json

# 3. Installa tutto con pnpm (genera pnpm-lock.yaml)
pnpm install

# 4. Aggiungi pnpm-lock.yaml al commit, escludi quello vecchio
# Nel .gitignore verifica che NON ci sia pnpm-lock.yaml
# (il lockfile deve stare nel repo — è garanzia di riproducibilità)
```

Aggiungi questa riga al `.gitignore` per evitare di committare accidentalmente il vecchio lockfile npm:
```
package-lock.json
```

---

## Comandi — confronto npm → pnpm

| npm | pnpm | Descrizione |
|---|---|---|
| `npm install` | `pnpm install` | Installa tutte le dipendenze |
| `npm install zod` | `pnpm add zod` | Aggiunge dipendenza runtime |
| `npm install -D vitest` | `pnpm add -D vitest` | Aggiunge dipendenza dev |
| `npm uninstall zod` | `pnpm remove zod` | Rimuove dipendenza |
| `npm run dev` | `pnpm dev` | Esegue script (il `run` è opzionale) |
| `npm run build` | `pnpm build` | Build |
| `npm test` | `pnpm test` | Test |
| `npm run lint` | `pnpm lint` | Lint |
| `npx vitest run` | `pnpm dlx vitest run` | Esegue binario senza installare |
| `npm audit` | `pnpm audit` | Audit sicurezza |

---

## package.json — nessuna modifica necessaria

Gli script in `package.json` rimangono identici. pnpm li legge e li esegue esattamente come npm. Solo il package manager cambia, non la configurazione.

---

## Monorepo con pnpm workspaces

Anche se SpotCast non è un monorepo puro (backend TS + UI Java comunicano via HTTP, non via codice condiviso), pnpm workspaces è utile se in futuro vorrai estrarre librerie condivise (es. i tipi `Business`, `Config`).

Struttura:
```
SpotCast/
├── pnpm-workspace.yaml     ← definisce i package del monorepo
├── package.json            ← root package (script globali)
├── packages/
│   ├── backend/            ← Node.js/TypeScript
│   │   └── package.json
│   └── shared-types/       ← tipi condivisi (se necessario)
│       └── package.json
```

`pnpm-workspace.yaml`:
```yaml
packages:
  - 'packages/*'
```

Comandi monorepo:
```bash
# Installa tutto il monorepo
pnpm install

# Esegui script in un package specifico
pnpm --filter backend dev
pnpm --filter backend test

# Esegui script in tutti i package
pnpm -r test

# Aggiungi dipendenza a un package specifico
pnpm add zod --filter backend
```

---

## Docker con pnpm — best practices

Il vantaggio principale di pnpm in Docker è la cache dei layer. Struttura ottimale del `Dockerfile`:

```dockerfile
FROM node:20-alpine AS base
RUN corepack enable && corepack prepare pnpm@latest --activate

# ── Dipendenze ────────────────────────────────────────────────────────────────
FROM base AS deps
WORKDIR /app
# Copia SOLO i file di dipendenze prima — questo layer viene cachato
# e non viene reinvalidato se cambia solo il codice sorgente
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

# ── Build ─────────────────────────────────────────────────────────────────────
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm build

# ── Produzione ────────────────────────────────────────────────────────────────
FROM node:20-alpine AS runner
RUN corepack enable && corepack prepare pnpm@latest --activate
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile --prod
COPY --from=builder /app/dist ./dist
COPY templates ./templates
COPY config.example.json ./

EXPOSE 3847
CMD ["node", "dist/SpotCast.js", "--daemon"]
```

**`--frozen-lockfile`** è la flag chiave: se `pnpm-lock.yaml` non corrisponde a `package.json`, il build fallisce invece di aggiornare silenziosamente. Garantisce riproducibilità totale.

---

## `.npmrc` — configurazione pnpm

Crea un file `.npmrc` nella root del progetto per configurare il comportamento di pnpm:

```ini
# Rende gli errori di dipendenze peer più visibili
strict-peer-dependencies=false

# Evita il problema delle dipendenze fantasma (pacchetti usabili
# solo se dichiarati esplicitamente in package.json)
shamefully-hoist=false

# Lockfile sempre aggiornato
lockfile=true
```

---

## Gotcha comuni in migrazione

**Dipendenze fantasma (phantom dependencies)**
npm e yarn v1 permettono di importare pacchetti non dichiarati in `package.json` (perché finiscono comunque in `node_modules`). pnpm lo impedisce. Se dopo la migrazione compaiono errori `Cannot find module 'xxx'`, aggiungi `xxx` esplicitamente:
```bash
pnpm add xxx
```

**Script che usano `npx`**
Sostituisci `npx` con `pnpm dlx` negli script o nella CLI:
```bash
# Prima
npx vitest run
# Dopo
pnpm dlx vitest run
# Oppure, se vitest è già in devDependencies
pnpm exec vitest run
```

**CI/CD**
In GitHub Actions, sostituisci `actions/setup-node` con il setup di pnpm:
```yaml
- uses: pnpm/action-setup@v4
  with:
    version: latest
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'pnpm'
- run: pnpm install --frozen-lockfile
```
