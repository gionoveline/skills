# Stack Defaults — CTO Co-Founder

Estas são as escolhas padrão recomendadas para os projetos de Giovanni, salvo razão explícita para desviar.

## Frontend
| Decisão | Default | Quando desviar |
|---------|---------|----------------|
| Framework | Next.js (App Router) | Nunca em projeto novo |
| Linguagem | TypeScript | Apenas se time não conhece TS |
| Styling | Tailwind CSS | Se design system já existe em outro formato |
| UI Components | shadcn/ui | Se precisar de componentes altamente customizados |
| Estado global | Zustand | Redux só se migração de projeto legado |
| Forms | React Hook Form + Zod | |
| Data fetching | React Query (TanStack) | Server Components podem eliminar necessidade |

## Backend / BFF
| Decisão | Default | Quando desviar |
|---------|---------|----------------|
| API Layer | Next.js API Routes / Route Handlers | Serviço separado só se time > 3 devs ou microserviço justificado |
| Validação | Zod | |
| ORM | Prisma | Drizzle se performance crítica |
| Background jobs | Vercel Cron / Trigger.dev | Quando precisar de filas robustas |

## Banco de Dados
| Decisão | Default | Quando desviar |
|---------|---------|----------------|
| Banco principal | Supabase (PostgreSQL) | PlanetScale se escala massiva; Firebase se time mobile-first |
| Cache | Upstash Redis | Só se volume justifica |
| Search | Supabase full-text ou Algolia | Se search é core feature |
| Storage | Supabase Storage | S3 direto se >100GB ou custo relevante |

## Auth
| Decisão | Default | Quando desviar |
|---------|---------|----------------|
| Auth provider | Supabase Auth | Clerk se UX de auth é diferencial; Auth0 se Enterprise SSO from day 1 |
| Social login | Google + Email magic link | |
| Enterprise SSO | Clerk ou Auth0 (SAML/OIDC) | Nunca implementar custom |

## Infraestrutura / Deploy
| Decisão | Default | Quando desviar |
|---------|---------|----------------|
| Hosting | Vercel | Railway se precisar de containers; AWS se Enterprise com compliance |
| CI/CD | GitHub Actions | |
| Monitoramento | Vercel Analytics + Sentry | Datadog se Enterprise |
| Feature flags | Vercel Feature Flags ou Unleash | |

## Integrações Comuns
| Serviço | Default |
|---------|---------|
| Email transacional | Resend |
| Pagamentos BR | Stripe (internacional) ou Pagar.me (BR nativo) |
| WhatsApp API | Meta Cloud API ou Twilio |
| Analytics produto | PostHog (self-hosted ou cloud) |
| LLM | Anthropic API (Claude) ou OpenAI — via Vercel AI SDK |

---

## Regras de Ouro

1. **Não construa o que pode comprar** — especialmente auth, email, pagamentos
2. **Supabase resolve 80% dos casos de backend** para produtos early-stage
3. **TypeScript não é opcional** — erro de tipo em runtime é custo de produto
4. **Monorepo só quando tiver > 2 apps** que compartilham código real
5. **Serverless-first** — evite gerenciar servidores até ter razão clara para isso
