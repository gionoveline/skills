# Segurança, LGPD e Conformidade Enterprise

## Checklist de Segurança por Estágio

### MVP (pré-revenue)
- [ ] Auth via provider estabelecido (Supabase/Clerk/Auth0) — nunca custom
- [ ] HTTPS em todas as rotas
- [ ] Variáveis de ambiente para secrets — nunca hardcoded
- [ ] Validação de inputs server-side (Zod ou similar)
- [ ] Rate limiting nas rotas de API públicas
- [ ] Sem dados sensíveis em logs

### Produto em Crescimento (primeiros clientes)
- [ ] Row Level Security (RLS) no banco — usuário só vê seus dados
- [ ] CORS configurado explicitamente
- [ ] Headers de segurança (CSP, HSTS, X-Frame-Options)
- [ ] Dependências sem vulnerabilidades conhecidas (`npm audit`)
- [ ] Backups automáticos do banco
- [ ] Política de senha / MFA disponível

### Enterprise / B2B
- [ ] SSO/SAML (via Clerk ou Auth0)
- [ ] Logs de auditoria (quem fez o quê e quando)
- [ ] DPA (Data Processing Agreement) assinável
- [ ] Penetration test documentado
- [ ] SLA definido e monitorado
- [ ] Disaster recovery plan
- [ ] SOC2 roadmap (se exigido pelo cliente)

---

## LGPD — Checklist Mínimo

### Coleta e Consentimento
- [ ] Política de Privacidade publicada e linkada no produto
- [ ] Consentimento explícito antes de coletar dados pessoais
- [ ] Finalidade declarada para cada tipo de dado coletado
- [ ] Opt-in para comunicações de marketing (não opt-out)

### Direitos do Titular
- [ ] Direito de acesso: usuário pode ver seus dados
- [ ] Direito de correção: usuário pode editar seus dados
- [ ] Direito de exclusão: "deletar conta" deve apagar ou anonimizar dados pessoais
- [ ] Direito de portabilidade: exportação de dados em formato legível (JSON/CSV)

### Dados Sensíveis (atenção extra)
Dados biométricos, saúde, orientação sexual, religião, político-filosóficos precisam de:
- Consentimento específico e destacado
- Medidas técnicas adicionais de proteção
- Avaliação de impacto (RIPD)

### Incidentes
- [ ] Processo definido para notificar a ANPD em até 72h
- [ ] Notificação aos titulares afetados documentada

### Fornecedores / Sub-processadores
- [ ] Contrato ou DPA com cada fornecedor que processa dados pessoais
- [ ] Verificar se fornecedores estão em países com adequação reconhecida pela ANPD

---

## Riscos Comuns em Projetos Next.js

| Risco | Onde aparece | Mitigação |
|-------|-------------|-----------|
| API Routes sem autenticação | `/api/*` sem verificar token | Middleware de auth em todas as rotas privadas |
| Secrets no frontend | `NEXT_PUBLIC_*` com dados sensíveis | Só usar `NEXT_PUBLIC_` para dados verdadeiramente públicos |
| Server Actions sem validação | Form actions diretas | Zod em todos os inputs + verificar sessão |
| IDOR (Insecure Direct Object Reference) | `GET /api/invoice/123` sem verificar dono | Sempre filtrar por `userId` na query |
| N+1 queries expondo dados | Prisma include aninhado | Revisar queries geradas; usar RLS como segunda camada |

---

## Comunicação com Clientes Enterprise

Quando Giovanni precisar responder perguntas de segurança de um cliente enterprise, o framing padrão é:

**Perguntas sobre onde os dados ficam:**
> "Os dados são armazenados em [Supabase/AWS] na região [us-east-1 / sa-east-1]. Todos os dados em repouso são criptografados com AES-256 e em trânsito via TLS 1.2+."

**Perguntas sobre acesso:**
> "O acesso é controlado por [Supabase RLS / permissões de role]. Mantemos logs de acesso e podemos fornecer relatório de auditoria mediante solicitação."

**Perguntas sobre LGPD:**
> "Operamos em conformidade com a LGPD. Podemos assinar um DPA e estamos disponíveis para responder ao questionário de segurança da sua equipe."
