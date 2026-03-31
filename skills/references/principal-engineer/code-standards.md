# Padrões de Código — Referência Detalhada

Stack de referência: Next.js 14+ (App Router) / TypeScript / Supabase / Tailwind / shadcn/ui

---

## TypeScript

### Tipagem de componentes
```typescript
// ✅ Correto
type Props = {
  userId: string
  onSuccess: (user: User) => void
  isLoading?: boolean
}

export function UserCard({ userId, onSuccess, isLoading = false }: Props) { ... }

// ❌ Evitar
export function UserCard(props: any) { ... }
export function UserCard({ userId, onSuccess, isLoading }: { userId: string, onSuccess: any, isLoading: boolean }) { ... }
```

### Validação com Zod
```typescript
// ✅ Sempre validar dados externos
import { z } from 'zod'

const CreateInvoiceSchema = z.object({
  amount: z.number().positive(),
  customerId: z.string().uuid(),
  dueDate: z.string().datetime(),
})

type CreateInvoiceInput = z.infer<typeof CreateInvoiceSchema>

// Em Server Actions
export async function createInvoiceAction(formData: FormData) {
  const parsed = CreateInvoiceSchema.safeParse({
    amount: Number(formData.get('amount')),
    customerId: formData.get('customerId'),
    dueDate: formData.get('dueDate'),
  })

  if (!parsed.success) {
    return { error: parsed.error.flatten() }
  }
  // ...
}
```

### Error handling
```typescript
// ✅ Retornar erros como valores (não throw em Server Actions)
type Result<T> = { data: T; error: null } | { data: null; error: string }

async function getUser(id: string): Promise<Result<User>> {
  const { data, error } = await supabase.from('users').select().eq('id', id).single()
  if (error) return { data: null, error: error.message }
  return { data, error: null }
}

// ❌ Evitar: throw em lógica de negócio (só em casos realmente excepcionais)
async function getUser(id: string): Promise<User> {
  const { data, error } = await supabase.from('users').select().eq('id', id).single()
  if (error) throw new Error(error.message) // evitar
  return data
}
```

---

## Next.js App Router

### Server vs. Client Components
```typescript
// ✅ Server Component (default) — busca dados, sem interatividade
// app/invoices/page.tsx
export default async function InvoicesPage() {
  const invoices = await getInvoices() // fetch direto, sem useEffect
  return <InvoiceList invoices={invoices} />
}

// ✅ Client Component — só quando necessário
// components/invoices/InvoiceFilter.tsx
'use client'
import { useState } from 'react'

export function InvoiceFilter({ onFilter }: { onFilter: (status: string) => void }) {
  const [status, setStatus] = useState('all')
  // ...
}
```

### Route Handlers (API)
```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  // 1. Autenticar/validar sempre
  const signature = request.headers.get('stripe-signature')
  if (!signature) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // 2. Parsear e validar body
  const body = await request.text()
  // ... validar assinatura

  // 3. Processar
  // ...

  return NextResponse.json({ received: true })
}
```

### Server Actions
```typescript
// actions/invoice.ts
'use server'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createInvoiceAction(formData: FormData) {
  // 1. Autenticar
  const session = await getServerSession()
  if (!session) redirect('/login')

  // 2. Validar
  const parsed = CreateInvoiceSchema.safeParse(Object.fromEntries(formData))
  if (!parsed.success) return { error: parsed.error.flatten() }

  // 3. Executar
  const { error } = await supabase.from('invoices').insert(parsed.data)
  if (error) return { error: error.message }

  // 4. Revalidar e redirecionar
  revalidatePath('/invoices')
  redirect('/invoices')
}
```

---

## Supabase

### Cliente por contexto
```typescript
// lib/supabase/server.ts — para Server Components e Route Handlers
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export function createClient() {
  const cookieStore = cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    { cookies: { get: (name) => cookieStore.get(name)?.value } }
  )
}

// lib/supabase/client.ts — para Client Components
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### Queries tipadas
```typescript
// ✅ Tipar resultado da query
const { data: invoices, error } = await supabase
  .from('invoices')
  .select('id, amount, status, customer:customers(name, email)')
  .eq('user_id', session.user.id)  // sempre filtrar por user_id (RLS como segunda camada)
  .order('created_at', { ascending: false })

type InvoiceWithCustomer = typeof invoices extends (infer T)[] | null ? T : never
```

### Row Level Security — padrão mínimo
```sql
-- Toda tabela com dados de usuário deve ter RLS ativado
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;

-- Policy básica: usuário só acessa seus próprios dados
CREATE POLICY "users_own_invoices" ON invoices
  FOR ALL USING (auth.uid() = user_id);
```

---

## Tailwind + shadcn/ui

### Organização de classes
```typescript
// ✅ Usar cn() para classes condicionais
import { cn } from '@/lib/utils'

function Badge({ variant, className }: { variant: 'success' | 'error', className?: string }) {
  return (
    <span className={cn(
      'px-2 py-1 rounded-full text-sm font-medium',
      variant === 'success' && 'bg-green-100 text-green-800',
      variant === 'error' && 'bg-red-100 text-red-800',
      className
    )}>
      ...
    </span>
  )
}

// ❌ Evitar: template strings com lógica
className={`px-2 ${variant === 'success' ? 'bg-green-100' : 'bg-red-100'}`}
```

### Componentes shadcn/ui
- Nunca editar arquivos em `components/ui/` diretamente — são gerados pelo shadcn/ui CLI
- Para customizar: criar wrapper em `components/[feature]/` que usa o componente base
- Para variantes novas: usar `cva` (class-variance-authority) como os próprios componentes shadcn fazem

---

## Git e Commits

### Formato de commit (Conventional Commits)
```
tipo(escopo): descrição curta em imperativo

[corpo opcional — por que, não o quê]

[referências: closes #123]
```

**Tipos:** `feat` | `fix` | `refactor` | `docs` | `test` | `chore` | `perf`

**Exemplos:**
```
feat(invoices): adicionar filtro por status na listagem
fix(auth): corrigir redirect loop em sessão expirada
refactor(db): extrair queries de invoice para módulo separado
docs(adr): registrar decisão de usar Supabase como banco principal
```

### Branches
```
main              # produção — protegida, só via PR
staging           # ambiente de homologação (se existir)
feat/[slug]       # nova feature
fix/[slug]        # correção de bug
refactor/[slug]   # refactor sem mudança de comportamento
chore/[slug]      # dependências, config, etc.
```

---

## Linting e Formatação

Configuração mínima recomendada:

```json
// package.json scripts
{
  "lint": "next lint",
  "format": "prettier --write .",
  "format:check": "prettier --check .",
  "type-check": "tsc --noEmit"
}
```

```json
// .husky/pre-commit (via lint-staged)
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css}": ["prettier --write"]
  }
}
```

Regras ESLint adicionais recomendadas para Next.js:
- `@typescript-eslint/no-explicit-any`: error
- `@typescript-eslint/no-unused-vars`: error
- `react-hooks/exhaustive-deps`: warn
- `no-console`: warn (para evitar console.log em produção)
