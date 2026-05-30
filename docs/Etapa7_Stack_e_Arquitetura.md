# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 7 — Stack e Arquitetura (v1.0)

---

### 1. Stack Tecnológica Completa

| Camada | Tecnologia | Versão | Justificativa |
|--------|-----------|--------|---------------|
| **Framework** | Next.js (App Router) | `15.5.18` | LTS estável, Server Actions, Route Handlers, deploy fácil na Vercel |
| **Runtime** | Node.js | `20.x LTS` | Compatível com Next.js 15, suporte longo |
| **Linguagem** | TypeScript | `5.8.3` | Tipagem completa, DX excelente |
| **React** | React | `19.0.6` | Requerido pelo Next.js 15, Concurrent Features |
| **Estilização** | Tailwind CSS | `3.4.17` | Estável, utility-first, não reinventar CSS |
| **Componentes** | shadcn/ui | `latest` | Base acessível (Radix), altamente customizável |
| **Ícones** | Lucide React | `0.487.0` | Ícones consistentes, tree-shakeable |
| **ORM** | Prisma | `5.22.0` | Type-safe, migrations automáticas, excelente DX |
| **Banco** | PostgreSQL | `16` (Neon) | Neon free tier, serverless, pooling automático |
| **Auth** | NextAuth.js v4 | `4.24.11` | Stable, Credentials Provider para login interno, sessão segura |
| **Validação** | Zod | `3.24.2` | Schemas TypeScript-first, integração nativa com RHF |
| **Forms** | React Hook Form | `7.54.2` | Performance, menos re-renders |
| **Cache/Estado Servidor** | TanStack Query | `5.74.0` | Cache inteligente, stale-while-revalidate |
| **Datas** | date-fns | `4.1.0` | Manipulação de datas imutável |
| **Tabelas** | TanStack Table | `8.21.0` | Headless, sorting, filtering, pagination |
| **Upload** | UploadThing | `7.x` | Upload simplificado para Next.js, free tier generoso |
| **PDF/Excel** | jspdf + xlsx | `2.5.x / 0.18.x` | Geração client-side de relatórios |
| **Toast** | Sonner | `1.7.x` | Toast elegante, usado pelo shadcn |
| **Class Utils** | clsx + tailwind-merge | `latest` | Utilitário `cn()` para classes condicionais |

### 2. Por que NÃO separar front e back?

| Separação | Monolito Next.js |
|-----------|------------------|
| 2 deploys para manter | 1 deploy na Vercel |
| CORS para configurar | Zero CORS (mesma origem) |
| Autenticação duplicada | Sessão compartilhada |
| Tipos duplicados (DTOs) | Tipos Prisma compartilhados |
| Mais complexo para 1 dev | Mais simples para MVP |

**Decisão:** Monolito Next.js com App Router. Se um dia precisar de API para app mobile, os Route Handlers já são uma API REST pronta.

### 3. Arquitetura de Pastas

```
sga-abrigos-recife/
├── .env.local
├── .env.example
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── components.json          # config do shadcn
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   └── login/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx          # Header + Sidebar + AuthGuard
│   │   │   ├── page.tsx            # redirect por perfil
│   │   │   ├── rede/
│   │   │   │   └── page.tsx
│   │   │   ├── abrigo/
│   │   │   │   └── page.tsx
│   │   │   ├── acolhidos/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx
│   │   │   ├── prontuario/
│   │   │   │   ├── registrar/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── consultar/
│   │   │   │       └── page.tsx
│   │   │   ├── medicacoes/
│   │   │   │   ├── esquemas/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── separacao/
│   │   │   │       └── page.tsx
│   │   │   ├── agendamentos/
│   │   │   │   └── page.tsx
│   │   │   ├── documentos/
│   │   │   │   ├── tecnicos/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── instrumentais/
│   │   │   │       └── page.tsx
│   │   │   ├── equipe/
│   │   │   │   └── page.tsx
│   │   │   ├── transferencias/
│   │   │   │   └── page.tsx
│   │   │   └── relatorios/
│   │   │       └── page.tsx
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   │   └── [...nextauth]/
│   │   │   │       └── route.ts
│   │   │   ├── upload/
│   │   │   │   └── route.ts
│   │   │   └── relatorios/
│   │   │       └── route.ts
│   │   └── actions/
│   │       ├── auth-actions.ts
│   │       ├── acolhido-actions.ts
│   │       ├── documento-actions.ts
│   │       ├── prescricao-actions.ts
│   │       ├── agendamento-actions.ts
│   │       ├── transferencia-actions.ts
│   │       └── ocorrencia-actions.ts
│   ├── components/
│   │   ├── ui/               # shadcn/ui puro (não editar)
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── table.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── select.tsx
│   │   │   ├── toast.tsx
│   │   │   └── ...
│   │   ├── base/             # SEUS wrappers customizados
│   │   │   ├── BaseButton.tsx
│   │   │   ├── BaseInput.tsx
│   │   │   ├── BaseSelect.tsx
│   │   │   ├── BaseTable.tsx
│   │   │   ├── BaseCard.tsx
│   │   │   ├── BaseModal.tsx
│   │   │   ├── BaseDrawer.tsx
│   │   │   ├── BaseBadge.tsx
│   │   │   ├── BaseDatePicker.tsx
│   │   │   ├── BaseSearch.tsx
│   │   │   ├── BaseEmptyState.tsx
│   │   │   └── BaseSkeleton.tsx
│   │   ├── layout/
│   │   │   ├── AppHeader.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   ├── SidebarItem.tsx
│   │   │   ├── MobileMenu.tsx
│   │   │   └── Breadcrumb.tsx
│   │   ├── forms/
│   │   │   ├── AcolhidoForm.tsx
│   │   │   ├── PrescricaoForm.tsx
│   │   │   ├── DocumentoForm.tsx
│   │   │   ├── AgendamentoForm.tsx
│   │   │   └── OcorrenciaForm.tsx
│   │   └── sections/
│   │       ├── VagasCard.tsx
│   │       ├── AgendamentosHoje.tsx
│   │       ├── AcolhidosAtivos.tsx
│   │       ├── DocumentosRecentes.tsx
│   │       └── AlertasOperacionais.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useAcolhido.ts
│   │   ├── useToast.ts
│   │   └── usePermissao.ts       # verifica acesso por perfil
│   ├── lib/
│   │   ├── prisma.ts             # singleton PrismaClient
│   │   ├── auth.ts               # config NextAuth + callbacks
│   │   ├── utils.ts              # cn(), formatDate, calcularMedicacao, etc
│   │   ├── constants.ts          # perfis, status, tipos de documento, cores
│   │   └── permissoes.ts         # matriz de permissões em código
│   ├── types/
│   │   ├── next-auth.d.ts        # extensão Session/User
│   │   └── index.ts              # tipos globais compartilhados
│   └── styles/
│       └── globals.css
├── public/
│   └── logo-prefeitura.svg
└── package.json
```

### 4. Padrões Arquiteturais

#### Server Actions vs Route Handlers
| Caso | Usar |
|------|------|
| Form submission (criar, editar, excluir) | Server Action |
| Query simples com cache | Server Action |
| Upload de arquivo multipart | Route Handler (`api/upload`) |
| Streaming de PDF/Excel | Route Handler (`api/relatorios`) |
| Endpoint para integração futura | Route Handler |

#### Estratégia de Dados
- **Server Components (default):** Buscar dados diretamente via Prisma no `page.tsx`.
- **Client Components:** Apenas quando necessário (interatividade, forms, hooks browser).
- **TanStack Query:** Usar em Client Components para cache e revalidação otimista.

#### Segurança
- `middleware.ts` no root: protege rotas, redireciona não autenticados para `/login`, redireciona por perfil.
- Server Actions: verificar `session.user.perfil` antes de executar.
- Prisma: nunca expor queries brutas; sempre usar abstração em `actions/`.

### 5. Configurações Iniciais

#### next.config.ts
```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      { hostname: "utfs.io" },      // UploadThing
      { hostname: "res.cloudinary.com" }, // Cloudinary fallback
    ],
  },
  experimental: {
    // serverActions: { bodySizeLimit: "2mb" } // se necessário
  },
};

export default nextConfig;
```

#### tsconfig.json (paths)
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/types/*": ["./src/types/*"]
    }
  }
}
```

### 6. Deploy

| Serviço | Uso | Custo |
|---------|-----|-------|
| **Vercel** | Hosting Next.js | Free tier (hobby) |
| **Neon** | PostgreSQL serverless | Free tier (500MB, 100h compute) |
| **UploadThing** | Storage de arquivos | Free tier (2GB) |
| **GitHub** | Repositório + CI/CD | Free |

**Variáveis de ambiente:**
```
DATABASE_URL="postgresql://...neon.tech/..."
NEXTAUTH_SECRET="..."
NEXTAUTH_URL="http://localhost:3000" # ou domínio Vercel
UPLOADTHING_TOKEN="..."
```
