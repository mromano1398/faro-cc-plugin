# Testing Patterns — Faro

Pattern completi per test unit, integration, E2E e load in progetti Faro.

## VITEST — Setup

`vitest.config.ts`:
```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      include: ['src/lib/**', 'src/services/**', 'src/actions/**'],
      exclude: ['**/*.d.ts', '**/node_modules/**'],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
      },
    },
    pool: 'forks',
    poolOptions: { forks: { singleFork: true } }, // per DB rollback
  },
})
```

`tests/setup.ts`:
```ts
import { beforeAll, afterAll } from 'vitest'
import { db } from '@/lib/db'

beforeAll(async () => {
  await db.$executeRaw`BEGIN`
})

afterAll(async () => {
  await db.$executeRaw`ROLLBACK`
  await db.$disconnect()
})
```

## UNIT TEST — Esempi

```ts
// src/lib/calc.test.ts
import { describe, it, expect } from 'vitest'
import { calcolaIva, formattaEuro } from './calc'

describe('calcolaIva', () => {
  it('applica aliquota 22%', () => {
    expect(calcolaIva(100, 0.22)).toBe(22)
  })

  it('arrotonda al centesimo', () => {
    expect(calcolaIva(10.1, 0.22)).toBe(2.22)
  })

  it('gestisce zero', () => {
    expect(calcolaIva(0, 0.22)).toBe(0)
  })
})

describe('formattaEuro', () => {
  it('formatta con simbolo e decimali', () => {
    expect(formattaEuro(1234.5)).toBe('€ 1.234,50')
  })
})
```

## INTEGRATION TEST — DB rollback

```ts
// tests/integration/ordini.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { db } from '@/lib/db'
import { creaOrdine, evadeOrdine } from '@/services/ordini'

describe('ordini service', () => {
  beforeEach(async () => {
    await db.$executeRaw`SAVEPOINT test`
  })

  afterEach(async () => {
    await db.$executeRaw`ROLLBACK TO SAVEPOINT test`
  })

  it('crea ordine con numerazione atomica', async () => {
    const ordine = await creaOrdine({
      clienteId: 'cli_1',
      righe: [{ prodottoId: 'p1', qta: 2, prezzo: 10 }],
    })
    expect(ordine.numero).toMatch(/^ORD-\d{6}$/)
    expect(ordine.totale).toBe(20)
  })

  it('evasione decrementa giacenza atomicamente', async () => {
    const ordine = await creaOrdine({ /* ... */ })
    await evadeOrdine(ordine.id)
    const giacenza = await db.giacenza.findFirst({ where: { prodottoId: 'p1' } })
    expect(giacenza.quantita).toBe(giacenzaIniziale - 2)
  })

  it('non evade se giacenza insufficiente', async () => {
    await expect(evadeOrdine('ordine_impossibile')).rejects.toThrow('Giacenza insufficiente')
  })
})
```

## PLAYWRIGHT — Setup

`playwright.config.ts`:
```ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html'], ['list']],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'mobile', use: { ...devices['iPhone 13'] } },
  ],
  webServer: {
    command: 'npm run build && npm start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
})
```

## E2E TEST — Flusso login + operazione

```ts
// tests/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test'

test.describe('checkout flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login')
    await page.fill('[data-testid="email"]', 'test@faro.example')
    await page.fill('[data-testid="password"]', 'password123')
    await page.click('[data-testid="submit"]')
    await expect(page).toHaveURL('/dashboard')
  })

  test('utente completa checkout', async ({ page }) => {
    await page.goto('/prodotti')
    await page.click('[data-testid="prodotto-1"]')
    await page.click('[data-testid="add-to-cart"]')
    await page.click('[data-testid="cart"]')
    await page.click('[data-testid="checkout"]')

    await page.fill('[data-testid="indirizzo"]', 'Via Test 1')
    await page.click('[data-testid="conferma"]')

    await expect(page.locator('[data-testid="ordine-confermato"]')).toBeVisible()
    const numero = await page.locator('[data-testid="numero-ordine"]').textContent()
    expect(numero).toMatch(/^ORD-\d{6}$/)
  })

  test('errore se giacenza zero', async ({ page }) => {
    await page.goto('/prodotti/esaurito')
    await page.click('[data-testid="add-to-cart"]')
    await expect(page.locator('[data-testid="error"]')).toContainText('esaurito')
  })
})
```

## K6 — Load test advisory lock

```js
// tests/load/numerazione.js
import http from 'k6/http'
import { check } from 'k6'

export const options = {
  vus: 50,
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
}

export default function () {
  const res = http.post(`${__ENV.BASE_URL}/api/ordini`, JSON.stringify({
    clienteId: 'cli_test',
    righe: [{ prodottoId: 'p1', qta: 1, prezzo: 10 }],
  }), {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${__ENV.TEST_TOKEN}`,
    },
  })

  check(res, {
    'status 201': (r) => r.status === 201,
    'numero ordine presente': (r) => r.json('numero') !== undefined,
  })
}
```

**Post-run check (SQL):**
```sql
-- Zero duplicati?
SELECT numero, COUNT(*)
FROM ordini
WHERE created_at > NOW() - INTERVAL '1 minute'
GROUP BY numero
HAVING COUNT(*) > 1;
-- Atteso: 0 righe
```

## FIXTURE — Factory pattern

```ts
// tests/factories/index.ts
import { faker } from '@faker-js/faker'

faker.seed(42) // deterministico

export function makeUser(overrides = {}) {
  return {
    email: faker.internet.email(),
    name: faker.person.fullName(),
    createdAt: new Date('2026-01-01'),
    ...overrides,
  }
}

export function makeOrdine(overrides = {}) {
  return {
    numero: `ORD-${faker.number.int({ min: 100000, max: 999999 })}`,
    totale: faker.number.float({ min: 10, max: 1000, fractionDigits: 2 }),
    stato: 'BOZZA',
    ...overrides,
  }
}
```

## CI — GitHub Actions

```yaml
name: Test
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: faro_test
        ports: [5432:5432]
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/faro_test
      - run: npm test
      - run: npm run test:coverage

  e2e:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## MOCKING — When & How

**Quando mockare:**
- API esterne (Stripe, provider LLM, email)
- Filesystem in unit test
- Date (`vi.useFakeTimers()`)
- Random (`faker.seed(42)`)

**Quando NON mockare:**
- Database (usa rollback)
- Zod/Prisma (testa la catena reale)
- Business logic interna (sono i tuoi test!)

**Esempio mock Stripe:**
```ts
import { vi } from 'vitest'

vi.mock('@/lib/stripe', () => ({
  stripe: {
    checkout: {
      sessions: {
        create: vi.fn().mockResolvedValue({ id: 'cs_test_123', url: 'https://checkout.test' }),
      },
    },
  },
}))
```

## CONTRACT TEST — OpenAPI + spectral

```ts
// tests/contract/openapi.test.ts
import { describe, it } from 'vitest'
import spectralCore from '@stoplight/spectral-core'
import { Spectral } from '@stoplight/spectral-core'
import spec from '../../openapi.json'

describe('OpenAPI', () => {
  it('valida contro ruleset', async () => {
    const spectral = new Spectral()
    await spectral.setRuleset({ extends: ['spectral:oas'] })
    const results = await spectral.run(spec)
    const errors = results.filter(r => r.severity === 0)
    expect(errors).toEqual([])
  })
})
```
