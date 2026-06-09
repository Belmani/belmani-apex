# Tutorial Vitest
## Per chi viene da JUnit5 + Mockito + AssertJ

---

## Concetto generale

Vitest è il JUnit5 dell'ecosistema TypeScript moderno. L'analogia è quasi 1:1:

| JUnit5 / Mockito | Vitest | Nota |
|---|---|---|
| `@Test` | `it('descrizione', () => {})` | La descrizione è il nome del test |
| `@Nested` | `describe('gruppo', () => {})` | Annidabile liberamente |
| `@BeforeEach` | `beforeEach(() => {})` | Stesso identico concetto |
| `@AfterEach` | `afterEach(() => {})` | Idem |
| `@BeforeAll` | `beforeAll(() => {})` | Idem |
| `Mockito.mock(Classe.class)` | `vi.mock('modulo')` | Mock a livello di modulo |
| `Mockito.when(...).thenReturn(...)` | `mockFn.mockReturnValue(...)` | |
| `Mockito.when(...).thenThrow(...)` | `mockFn.mockRejectedValueOnce(...)` | Per Promise/async |
| `verify(mock).metodo(...)` | `expect(mockFn).toHaveBeenCalledWith(...)` | |
| `AssertJ: assertThat(x).isEqualTo(y)` | `expect(x).toBe(y)` | |
| `AssertJ: assertThat(list).hasSize(n)` | `expect(arr).toHaveLength(n)` | |
| `AssertJ: assertThat(x).isNull()` | `expect(x).toBeNull()` | |

---

## Struttura base

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';

describe('NomeClasse', () => {

  beforeEach(() => {
    // reset prima di ogni test — equivalente a @BeforeEach
  });

  describe('nomeMetodo — caso felice', () => {
    it('fa la cosa giusta', () => {
      // arrange
      const input = 'hello';
      // act
      const result = input.toUpperCase();
      // assert
      expect(result).toBe('HELLO');
    });
  });

  describe('nomeMetodo — gestione errori', () => {
    it('lancia eccezione su input nullo', () => {
      expect(() => rischioso(null)).toThrow('messaggio atteso');
    });
  });
});
```

---

## Assertions — da AssertJ a Vitest

```typescript
// Uguaglianza primitiva (===)
expect(result).toBe(42);

// Uguaglianza strutturale (deep equals — come AssertJ .isEqualTo su oggetti)
expect(obj).toEqual({ name: 'Rossi', city: 'Milan' });

// Subset — l'oggetto contiene almeno queste proprietà
expect(obj).toMatchObject({ name: 'Rossi' });

// Lunghezza array
expect(arr).toHaveLength(3);

// Array contiene un elemento
expect(arr).toContain('Milan');

// Null / undefined
expect(x).toBeNull();
expect(x).toBeUndefined();
expect(x).toBeDefined();

// Truthy / falsy
expect(x).toBeTruthy();
expect(x).toBeFalsy();

// Numeri
expect(n).toBeGreaterThan(0);
expect(n).toBeLessThanOrEqual(10);

// Stringhe
expect(str).toContain('SpotCast');
expect(str).toMatch(/^\d{4}-\d{2}-\d{2}$/); // regex

// Eccezioni sincrone
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('messaggio');
expect(() => fn()).toThrow(TypeError);

// Eccezioni async (Promise rejected)
await expect(asyncFn()).rejects.toThrow('errore');

// Promise risolta
await expect(asyncFn()).resolves.toBe('valore atteso');
```

---

## Mocking — da Mockito a Vitest

### Mock di un modulo intero

In JUnit mockeresti una dipendenza iniettandola nel costruttore.
In Vitest, per i moduli Node.js, si mocka l'intero `import`:

```typescript
// Dice a Vitest: quando qualcuno importa questo modulo, usa questa versione falsa
vi.mock('@googlemaps/google-maps-services-js', () => ({
  // IMPORTANTE: se la classe viene usata con `new`, deve essere una function, non arrow
  Client: function () {
    return { textSearch: vi.fn() };
  },
}));
```

**Perché `function` e non arrow `() => {}`?**
Le arrow function in JavaScript non possono essere usate come costruttori (`new`).
Se il codice fa `new Client()`, il mock deve essere una `function` normale.
È esattamente l'errore che abbiamo incontrato — e il motivo per cui tutti i test fallivano.

---

### Mock di una singola funzione (vi.fn)

Equivalente di `Mockito.mock()` su un singolo metodo:

```typescript
const mockFn = vi.fn();

// Imposta il valore di ritorno (come Mockito.when().thenReturn())
mockFn.mockReturnValue('valore');

// Per funzioni async (come thenReturn su un CompletableFuture)
mockFn.mockResolvedValue({ data: { results: [] } });

// Solo per la prossima chiamata, poi torna al comportamento di default
mockFn.mockResolvedValueOnce({ data: { results: [mockPlace] } });

// Simula un'eccezione (come Mockito.when().thenThrow())
mockFn.mockRejectedValueOnce(new Error('Network error'));
```

---

### Verifiche sulle chiamate (come Mockito.verify)

```typescript
// È stato chiamato almeno una volta?
expect(mockFn).toHaveBeenCalled();

// È stato chiamato esattamente N volte?
expect(mockFn).toHaveBeenCalledTimes(2);

// È stato chiamato con questi argomenti?
expect(mockFn).toHaveBeenCalledWith('Dentist', 'Milan', 5);

// Non è mai stato chiamato?
expect(mockFn).not.toHaveBeenCalled();
```

---

### Reset tra un test e l'altro

```typescript
beforeEach(() => {
  vi.clearAllMocks(); // azzera chiamate e return values — usa sempre questo in beforeEach
  // vi.resetAllMocks()  — come clearAllMocks ma rimuove anche le implementazioni
  // vi.restoreAllMocks() — ripristina i mock creati con vi.spyOn
});
```

---

## Test asincroni

Vitest gestisce Promise nativamente — basta `async/await`:

```typescript
it('fetches data from API', async () => {
  mockFn.mockResolvedValue({ data: { results: [mockPlace] } });

  const results = await fetcher.fetchOne('Dentist', 'Milan', 5);

  expect(results).toHaveLength(1);
  expect(results[0].name).toBe('Studio Dentistico Rossi');
});
```

---

## vi.spyOn — spiare metodi reali

Equivalente di `Mockito.spy()`:

```typescript
// Spia un metodo esistente senza sostituirlo completamente
const spy = vi.spyOn(oggetto, 'metodo');

// Oppure con override del comportamento
const spy = vi.spyOn(oggetto, 'metodo').mockReturnValue('fake');

// Verifica
expect(spy).toHaveBeenCalledTimes(1);

// Ripristina il comportamento originale dopo il test
spy.mockRestore();
```

---

## Struttura dei file in SpotCast

```
tests/
├── config/
│   └── ConfigLoader.test.ts    ← testa src/config/ConfigLoader.ts
├── fetcher/
│   └── GoogleFetcher.test.ts   ← testa src/fetcher/GoogleFetcher.ts
├── dedup/
│   └── DedupService.test.ts    ← testa src/dedup/DedupService.ts
└── ...
```

Convenzione: ogni file `src/X/Y.ts` ha il suo specchio `tests/X/Y.test.ts`.

---

## Comandi utili

```bash
npm test                    # esegue tutti i test in watch mode
npm run test:coverage       # genera report di copertura
npx vitest run              # esegue una volta sola (no watch) — utile in CI
npx vitest run --reporter=verbose  # output dettagliato test per test
```

**Watch mode** (quello che vedi quando premi invio): Vitest rimane in ascolto
e riesegue i test ogni volta che salvi un file. Per uscire: `q` + invio.
Per eseguire una volta sola senza watch: `npx vitest run`.
