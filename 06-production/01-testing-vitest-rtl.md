# Testing with Vitest and RTL

## What

Three layers for this stack: Vitest + React Testing Library (RTL) for components, Vitest for NestJS services and controllers with dependency overrides, and supertest for full HTTP round-trips against the Nest app.

## Why It Matters

The expensive bugs in a fullstack app are contract bugs — the API changed, the frontend did not know — and security bugs — user A reading user B's data. An e2e test suite running the real Nest app against a fresh database catches both classes in one place. Component tests catch rendering regressions cheaply. Together they let you refactor with evidence instead of hope.

## How It Works

### Test the Nest App Over HTTP

```ts
// test/tasks.e2e-spec.ts
import { Test } from '@nestjs/testing'
import { INestApplication, ValidationPipe } from '@nestjs/common'
import * as request from 'supertest'
import mongoose from 'mongoose'
import { AppModule } from '../src/app.module'

describe('Tasks (e2e)', () => {
  let app: INestApplication
  let token: string

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [AppModule]
    }).compile()

    app = moduleRef.createNestApplication()
    app.useGlobalPipes(new ValidationPipe({ whitelist: true }))
    await app.init()

    await mongoose.connect(process.env.DATABASE_URL!)
    await mongoose.connection.dropDatabase()

    const res = await request(app.getHttpServer())
      .post('/auth/signup')
      .send({ email: 'test@example.com', name: 'Test', password: 'password123' })
    token = res.body.token
  })

  afterAll(async () => {
    await mongoose.connection.dropDatabase()
    await mongoose.disconnect()
    await app.close()
  })

  it('creates a task with auth', async () => {
    const res = await request(app.getHttpServer())
      .post('/tasks')
      .set('Authorization', `Bearer ${token}`)
      .send({ title: 'Write tests', priority: 'high' })
      .expect(201)

    expect(res.body.title).toBe('Write tests')
  })

  it('rejects invalid bodies with 400', async () => {
    await request(app.getHttpServer())
      .post('/tasks')
      .set('Authorization', `Bearer ${token}`)
      .send({ title: '' })
      .expect(400)
  })

  it('hides other users tasks', async () => {
    const other = await request(app.getHttpServer())
      .post('/auth/signup')
      .send({ email: 'other@example.com', name: 'Other', password: 'password123' })

    await request(app.getHttpServer())
      .get(`/tasks/${someTaskId}`)
      .set('Authorization', `Bearer ${other.body.token}`)
      .expect(404)
  })
})
```

### Unit-Test Services With Fakes

```ts
const moduleRef = await Test.createTestingModule({
  controllers: [TasksController],
  providers: [
    TasksService,
    { provide: TasksRepository, useValue: fakeRepo }
  ]
}).compile()
```

### Component Tests — RTL

```tsx
// src/features/tasks/__tests__/TaskList.test.tsx
import { render, screen } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { TaskList } from '../TaskList'

function renderWithProviders(ui: React.ReactElement) {
  const qc = new QueryClient({ defaultOptions: { queries: { retry: false } } })
  return render(<QueryClientProvider client={qc}>{ui}</QueryClientProvider>)
}

it('renders task titles', async () => {
  server.use(                                  // msw intercepts fetch
    http.get('*/tasks', () => HttpResponse.json({
      items: [{ id: '1', title: 'Write tests', priority: 'high' }]
    }))
  )

  renderWithProviders(<TaskList />)

  expect(await screen.findByText('Write tests')).toBeInTheDocument()
})
```

| Layer | Tool | What it proves |
|-------|------|----------------|
| Service | Vitest + fakes | Logic without infrastructure |
| API | supertest + fresh DB | Contract, validation, auth, ownership |
| Component | Vitest + RTL + msw | Rendering against the API contract |

Run: `bunx vitest run` (both apps), supertest suite via `bun test` or vitest per config.

## Common Mistakes

- **Mocking the database in e2e tests.** Drop-and-recreate a test database; ownership and scoping bugs only surface against real persistence.
- **Sharing state between e2e tests.** Each suite gets a clean database; order-dependent tests lie.
- **`getByText` brittleness.** Query by role and accessible name (`findByRole('button', { name: 'Sign in' })`) — survives markup changes, checks a11y for free.
- **Skipping the auth-failure tests.** The 401/403/ownership cases are the ones that pay for the suite.
