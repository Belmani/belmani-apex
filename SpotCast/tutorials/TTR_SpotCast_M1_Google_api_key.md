# Tutorial — Configurazione Google Places API Key

> Guida passo-passo per ottenere la tua chiave Google Maps API da usare con SpotCast.
> Tempo stimato: 10–15 minuti. Nessuna competenza tecnica richiesta.

---

## Cosa ti serve

- Un account Google (Gmail o Google Workspace)
- Una carta di credito (richiesta da Google per attivare la fatturazione — **non ti verrà addebitato nulla** per l'utilizzo normale di SpotCast, che rimane ampiamente entro la soglia gratuita di $200/mese)

---

## Passo 1 — Apri Google Cloud Console

Vai su: **[https://console.cloud.google.com](https://console.cloud.google.com)**

Accedi con il tuo account Google se richiesto.

---

## Passo 2 — Crea un nuovo progetto

1. In cima alla pagina, clicca sul **selettore di progetto** (mostra il nome del progetto corrente o "Seleziona un progetto")
2. Clicca **Nuovo progetto**
3. Dagli un nome riconoscibile, ad esempio `SpotCast`
4. Clicca **Crea**
5. Aspetta qualche secondo, poi assicurati che il tuo nuovo progetto sia selezionato nella barra in alto

---

## Passo 3 — Abilita la Places API

1. Nel menu a sinistra, vai su **API e servizi → Libreria**
2. Nella barra di ricerca, scrivi `Places API`
3. Vedrai due risultati:
   - **Places API** (legacy)
   - **Places API (New)** ← seleziona questa
4. Clicca **Abilita**

---

## Passo 4 — Configura la fatturazione

Google richiede un account di fatturazione per usare la Places API, anche se il tuo utilizzo sarà gratuito.

1. Nel menu a sinistra, vai su **Fatturazione**
2. Clicca **Collega un account di fatturazione** o **Crea account di fatturazione**
3. Segui i passaggi per aggiungere la tua carta di credito
4. Collega l'account di fatturazione al progetto `SpotCast`

> ℹ️ Google ti offre **$200 di credito gratuito ogni mese**. Un utilizzo tipico di SpotCast (10–50 ricerche al giorno) costa circa $0,02–$0,10 al mese — sostanzialmente gratis. Ti verrà addebitato qualcosa solo se configuri SpotCast per eseguire centinaia di ricerche all'ora, un utilizzo ben oltre quello normale.

---

## Passo 5 — Crea la tua chiave API

1. Nel menu a sinistra, vai su **API e servizi → Credenziali**
2. Clicca **+ Crea credenziali** in alto
3. Seleziona **Chiave API**
4. La tua chiave viene generata — ha un aspetto simile a: `AIzaSyB_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
5. Copiala e conservala in un posto sicuro

---

## Passo 6 — Limita la tua chiave API (consigliato)

Limitare la chiave impedisce che venga usata da altri in caso di perdita.

1. Clicca sulla chiave appena creata per aprirne le impostazioni
2. Sotto **Restrizioni API**, seleziona **Limita chiave**
3. Dal menu a tendina, seleziona **Places API (New)**
4. Clicca **Salva**

Facoltativamente, sotto **Restrizioni applicazione**, puoi limitare la chiave a indirizzi IP specifici se SpotCast gira sempre dalla stessa macchina.

---

## Passo 7 — Aggiungi la chiave a SpotCast

Apri il file `config.json` e incolla la chiave:

```json
{
  "google_api_key": "AIzaSyB_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  ...
}
```

Oppure impostala come variabile d'ambiente (più sicuro):

```bash
# macOS / Linux
export GOOGLE_API_KEY="AIzaSyB_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Windows (Prompt dei comandi)
set GOOGLE_API_KEY=AIzaSyB_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## Passo 8 — Monitora il tuo utilizzo (facoltativo ma consigliato)

Per assicurarti di non superare mai la soglia gratuita accidentalmente:

1. Vai su **API e servizi → Quote e limiti di sistema**
2. Oppure imposta un **avviso di fatturazione** in **Fatturazione → Budget e avvisi**
3. Crea un avviso a $5/mese — riceverai una email se dovesse succedere qualcosa di insolito

---

## Risoluzione dei Problemi

**Errore "REQUEST_DENIED" all'avvio di SpotCast**
→ Verifica che la Places API (New) sia abilitata nel tuo progetto e che la chiave API sia incollata correttamente in `config.json` (senza spazi extra).

**"This API project is not authorized to use this API"**
→ Potresti aver abilitato la versione sbagliata della Places API. Torna alla Libreria e abilita specificamente **Places API (New)**.

**Errore "You must enable billing"**
→ Completa il Passo 4 — Google richiede un account di fatturazione anche per l'utilizzo nella soglia gratuita.

**"API key not valid"**
→ Verifica che la chiave sia stata copiata per intero. Le chiavi sono ~39 caratteri e iniziano con `AIzaSy`.

---

## Quanto costerà?

Per un utilizzo normale di SpotCast:

| Utilizzo | Chiamate API mensili | Costo stimato |
|---|---|---|
| 10 risultati/giorno, 5 categorie, 5 città | ~1.500 | Gratuito (credito $200) |
| 50 risultati/giorno, 10 categorie, 10 città | ~15.000 | Gratuito (credito $200) |
| Utilizzo intenso: 500+ risultati/giorno | 150.000+ | ~$15–30/mese |

La grande maggioranza degli utenti SpotCast non pagherà mai nulla per le API.

---

*Domande? Apri una issue sul [repository GitHub di SpotCast](https://github.com/your-org/spotcast/issues).*
