# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 10 — Plano de Implementação (v1.0)

---

### 1. Introdução

Roadmap de desenvolvimento em fases (sprints). Cada fase entrega valor operacional independente.

---

### 2. Fase 0 — Fundação (Semanas 1–2)

**Objetivo:** Ambiente pronto para desenvolvimento.

- [ ] Configurar projeto Next.js 15 + TypeScript + Tailwind
- [ ] Instalar e configurar shadcn/ui
- [ ] Configurar Prisma + Neon PostgreSQL
- [ ] Criar schema.prisma completo com enums e relacionamentos
- [ ] Rodar primeira migration
- [ ] Criar seed.ts com 2 abrigos + 7 perfis de funcionários
- [ ] Configurar NextAuth.js v4 com Credentials Provider
- [ ] Criar middleware.ts para proteção de rotas
- [ ] Configurar variáveis de ambiente (.env.local + .env.example)
- [ ] Criar estrutura de pastas base (app, components, lib, hooks)
- [ ] Criar componentes base (BaseButton, BaseInput, BaseTable, BaseCard)
- [ ] Criar sistema de permissões (lib/permissoes.ts)

**Entregável:** Projeto rodando localmente com login funcional e seed aplicado.

---

### 3. Fase 1 — Autenticação e Layout (Semanas 3–4)

**Objetivo:** Usuários conseguem logar e navegar conforme perfil.

- [ ] Tela de login funcional (validação Zod, feedback visual)
- [ ] Layout padrão (Header + Sidebar) com controle por perfil
- [ ] Redirecionamento pós-login (coord → /rede, outros → /abrigo)
- [ ] Sidebar com itens ocultos por perfil
- [ ] Header com abrigo atual, avatar e logout
- [ ] Página 403 (acesso negado) e 404
- [ ] Hook usePermissao para verificar acesso em componentes
- [ ] Seed de funcionários testáveis (senhas simples para teste)

**Entregável:** Sistema navegável com login e menus por perfil.

---

### 4. Fase 2 — Cadastro e Painel do Abrigo (Semanas 5–7)

**Objetivo:** Gestão básica de acolhidos e visão operacional.

- [ ] CRUD de abrigos (coordenação)
- [ ] Configuração de vagas qualificadas por abrigo
- [ ] Cadastro de acolhido (novo e provisório)
- [ ] Busca de duplicidade (fuzzy)
- [ ] Listagem de acolhidos com BaseTable
- [ ] Drawer lateral com resumo do acolhido
- [ ] Painel do Abrigo:
  - Cards de vagas (totais, ocupadas, disponíveis)
  - Lista de acolhidos ativos
  - Alertas operacionais (provisórios > 7 dias)
- [ ] Painel da Coordenação (cards de abrigos, busca rede)

**Entregável:** Coordenação e gerentes conseguem ver ocupação e cadastrar acolhidos.

---

### 5. Fase 3 — Prontuário e Documentos (Semanas 8–11)

**Objetivo:** Núcleo do sistema — dossiê vivo do acolhido.

- [ ] Modelo Documento com editor de texto simples
- [ ] Criação de documentos técnicos (escuta inicial, acompanhamentos)
- [ ] Bloqueio após salvar (imutabilidade)
- [ ] Sistema de erratas (vinculação ao original)
- [ ] Consulta ao prontuário (timeline)
- [ ] Controle de sigilo (documentos psicológicos)
- [ ] Upload de arquivos digitalizados (UploadThing)
- [ ] PIA (modelo básico)
- [ ] Documentos instrumentais (templates com placeholders)

**Entregável:** Técnicos conseguem registrar e consultar prontuários completos.

---

### 6. Fase 4 — Medicações (Semanas 12–14)

**Objetivo:** Controle farmacêutico operacional.

- [ ] Catálogo de medicamentos
- [ ] Prescrição com esquema M+T+N
- [ ] Validação de esquema (inteiros ≥ 0, soma > 0)
- [ ] Relatório de separação semanal (cálculo automático)
- [ ] Confirmação de separação
- [ ] Controle de estoque básico (entrada/saída)
- [ ] Alertas de vencimento e estoque baixo

**Entregável:** Administrativo gera relatório de separação semanal sem fazer cálculos.

---

### 7. Fase 5 — Agendamentos e Ocorrências (Semanas 15–16)

**Objetivo:** Gestão de compromissos e registro cotidiano.

- [ ] Cadastro de agendamentos
- [ ] Calendário semanal visual
- [ ] Alerta no login do dia anterior
- [ ] Relatório de retorno vinculado a agendamento
- [ ] Registro de ocorrências (educador/cuidador)
- [ ] Notificação de ocorrências graves ao gerente
- [ ] Registro de visitas familiares

**Entregável:** Administrativo agenda, técnico registra retorno, educador registra ocorrências.

---

### 8. Fase 6 — Movimentações (Semanas 17–18)

**Objetivo:** Transferência e desligamento com autorização.

- [ ] Transferência de acolhido (fluxo dupla autorização)
- [ ] Desligamento de acolhido (técnico cria + gerente autoriza)
- [ ] Histórico de passagens (timeline)
- [ ] Transferência de funcionário entre abrigos
- [ ] Desligamento de funcionário

**Entregável:** Fluxos de movimentação funcionando com notificações.

---

### 9. Fase 7 — Relatórios e Fechamento (Semanas 19–20)

**Objetivo:** Visão gerencial e exportação.

- [ ] Painel gerencial com indicadores
- [ ] Relatório Censo SUAS
- [ ] Relatório de produtividade técnica
- [ ] Relatório de consumo de medicamentos
- [ ] Exportação PDF (jspdf)
- [ ] Exportação Excel (xlsx)
- [ ] Tela de auditoria básica (logs)
- [ ] Testes de integração principais
- [ ] Documentação de deploy

**Entregável:** Sistema completo para MVP, pronto para deploy na Vercel.

---

### 10. Fases Futuras (Pós-MVP)

| Fase | Descrição |
|------|-----------|
| 8 | Offline-first / PWA para internet instável |
| 9 | Integração com e-SUAS / RMA |
| 10 | Módulo de registro diário de plantão (educadores) |
| 11 | App mobile simplificado |
| 12 | Assinatura digital ICP-Brasil |
| 13 | BI avançado (Power BI / Metabase) |
| 14 | Chat interno e notificações push |

---

### 11. Critérios de Aceitação por Fase

- Cada fase só avança quando:
  1. Funcionalidades principais passam em testes manuais.
  2. Não há bugs críticos (impedem uso).
  3. Seed de dados permite demonstrar a fase.
  4. Documentação da fase está atualizada.
