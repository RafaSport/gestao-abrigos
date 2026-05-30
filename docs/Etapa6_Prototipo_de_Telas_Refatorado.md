# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 6 — Protótipo de Telas e UX (v2.0)

---

### 1. Introdução

Este documento descreve as telas do SGA, seus estados, comportamentos e decisões de UX. Não são wireframes visuais, mas especificações de interface para implementação.

---

### 2. Stack de Interface

- **Framework:** Next.js 15 App Router
- **Estilização:** Tailwind CSS 3.4
- **Componentes base:** shadcn/ui (Radix + Tailwind)
- **Ícones:** Lucide React
- **Forms:** React Hook Form + Zod
- **Tabelas:** TanStack Table
- **Datas:** date-fns
- **Toast/Feedback:** Sonner (shadcn)

---

### 3. Layout Padrão (Dashboard)

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER FIXO                                                │
│  [Logo Prefeitura]  Secretaria de Assistência Social   [👤] │
├──────────┬──────────────────────────────────────────────────┤
│          │                                                  │
│ SIDEBAR  │              ÁREA DE CONTEÚDO                   │
│ FIXO     │              (dinâmica por rota)                │
│          │                                                  │
│ [Menu    │                                                  │
│  com     │                                                  │
│  itens   │                                                  │
│  por     │                                                  │
│  perfil] │                                                  │
│          │                                                  │
└──────────┴──────────────────────────────────────────────────┘
```

#### Header
- **Esquerda:** Logo da Prefeitura do Recife + nome da Secretaria
- **Centro:** Nome do abrigo atual (exceto coordenação) + badge do perfil
- **Direita:** Avatar + nome do usuário logado + botão de logout

#### Sidebar
- **Grupo GERAL:** Painel, Acolhidos, Agendamentos
- **Grupo TÉCNICO:** Prontuário (submenu: Registrar / Consultar), Documentos Técnicos
- **Grupo ADMINISTRATIVO:** Medicações (submenu: Esquemas / Separação), Documentos Instrumentais
- **Grupo EQUIPE:** Equipe, Transferências
- **Grupo RELATÓRIOS:** Relatórios (coordenação e gerente)
- **Itens ocultos:** Perfil sem acesso nem vê o item (não fica cinza)
- **Submenus:** Hover ou click no item pai (decisão de UX: click para mobile, hover para desktop)

---

### 4. Telas Detalhadas

#### Tela 1 — Login
- **Rota:** `/login`
- **Layout:** Sem header/sidebar (layout isolado)
- **Elementos:**
  - Logo centralizada
  - Campo e-mail
  - Campo senha
  - Botão "Entrar"
  - Link "Esqueci minha senha" (futuro)
- **Comportamento:**
  - Validação com Zod (e-mail válido, senha ≥ 6 caracteres)
  - Loading state no botão
  - Erro: toast vermelho "E-mail ou senha inválidos"
  - Sucesso: redirect conforme perfil (coord → /rede, outros → /abrigo)

---

#### Tela 2 — Painel da Coordenação (Rede)
- **Rota:** `/rede`
- **Acesso:** COORDENACAO
- **Layout:** Cards grid + sidebar lateral de filtros
- **Elementos:**
  - **Cards de Abrigos (grid 3 colunas):**
    - Nome do abrigo
    - Perfil (ícone + texto: "Adultos Masculinos")
    - Vagas: `ocupadas / total` (barra de progresso colorida)
    - Alerta se ocupação > 90% (vermelho)
    - Clique no card → expande mini-dashboard do abrigo ou navega para `/abrigo/[id]`
  - **Barra superior:**
    - Busca de acolhido na rede (input com busca fuzzy)
    - Filtros: faixa etária, gênero, com vaga disponível
  - **Indicadores rápidos (topo):**
    - Total de acolhidos na rede
    - Transferências pendentes
    - Ocorrências graves nas últimas 24h

---

#### Tela 3 — Painel do Abrigo
- **Rota:** `/abrigo` ou `/abrigo/[id]`
- **Acesso:** Todos (exceto COORDENACAO que vê via `/rede`)
- **Layout:** Duas colunas (60/40)
- **Coluna Esquerda — Situação do Abrigo:**
  - **Cards de vagas (horizontal):**
    - Vagas totais | Ocupadas | Disponíveis
    - Cores: verde (disponível), amarelo (atenção), vermelho (lotado)
  - **Lista de Acolhidos Ativos:**
    - Tabela paginada: foto miniatura, nome, idade, tempo de acolhimento, status
    - Filtros rápidos: chegaram esta semana, documentos pendentes, medicação ativa
    - Clique na linha → abre Drawer lateral com ficha resumida
  - **Alertas operacionais (badge vermelho):**
    - Acolhidos provisórios > 7 dias sem escuta inicial
    - Medicamentos vencendo em 30 dias
    - Agendamentos sem retorno registrado
- **Coluna Direita — Agenda e Movimentação:**
  - **Agendamentos de Hoje:** lista com horário, acolhido, tipo, local
  - **Agendamentos de Amanhã:** destaque para preparativos
  - **Documentos Recentes:** últimos 5 documentos técnicos criados na unidade
  - **Ocorrências Graves:** lista das não lidas pelo gerente

---

#### Tela 4 — Acolhidos (Listagem)
- **Rota:** `/acolhidos`
- **Acesso:** Todos (exceto CUIDADOR que vê apenas via Painel)
- **Elementos:**
  - **Tabela base** (BaseTable component):
    - Colunas: foto, nome, idade, gênero, unidade atual, status, tempo de acolhimento, ações
    - Ordenação por cabeçalho
    - Busca global (nome, CPF, nome da mãe)
    - Filtros laterais: status, faixa etária, com documentos pendentes
  - **Botão "Novo Acolhido"** (visível apenas para GERENTE, AS, PSICOLOGO)
  - **Paginação:** 20 itens por página
- **Drawer lateral (ao clicar na linha):**
  - Foto grande + dados pessoais
  - Tabs: Resumo | Prontuário | Medicações | Agendamentos | Ocorrências | Passagens
  - Atalhos rápidos (contexto do perfil logado):
    - Técnico: "Novo Documento", "Nova Prescrição"
    - Admin: "Novo Agendamento", "Ver Separação"
    - Educador: "Nova Ocorrência"

---

#### Tela 5 — Prontuário (Registrar)
- **Rota:** `/prontuario/registrar`
- **Acesso:** GERENTE, ASSISTENTE_SOCIAL, PSICOLOGO
- **Elementos:**
  - **Passo 1:** Seleção do acolhido (busca com autocomplete)
  - **Passo 2:** Seleção do tipo de documento (dropdown)
  - **Passo 3:** Editor de texto
    - Toolbar simples: negrito, itálico, lista, tabela 2x2
    - Área principal com altura mínima 400px
    - Contador de caracteres
  - **Passo 4:** Flags
    - Sigiloso? (checkbox, default false, só PSICOLOGO vê)
    - Vincular a agendamento? (se houver agendamento pendente)
  - **Ações:**
    - "Salvar Rascunho" (salva, mantém editável)
    - "Salvar e Bloquear" (salva, status = ATIVO, redireciona para consulta)
- **Validação:**
  - Título obrigatório
  - Conteúdo mínimo 50 caracteres
  - Se tipo = ESCUTA_INICIAL e já existe para esta passagem: bloquear

---

#### Tela 6 — Prontuário (Consultar)
- **Rota:** `/prontuario/consultar?acolhido=[id]`
- **Acesso:** Todos (com restrição de sigilo)
- **Elementos:**
  - **Timeline vertical:** documentos ordenados por data (mais recente no topo)
  - **Card de documento:**
    - Header: tipo (badge colorido), título, autor, data, unidade de origem
    - Body: conteúdo (com scroll se muito longo)
    - Footer: se ATIVO e usuário = autor → botão "Criar Errata"
    - Se CORRIGIDO → banner amarelo "Documento corrigido por errata", link para errata
  - **Filtros:** por tipo, por autor, por período
  - **Abas:** Documentos Técnicos | Documentos Digitalizados | Erratas

---

#### Tela 7 — Medicações — Esquemas
- **Rota:** `/medicacoes/esquemas`
- **Acesso:** GERENTE (C), AS (C), PSICOLOGO (C), ADMINISTRATIVO (V)
- **Elementos:**
  - **Tabela de prescrições ativas:**
    - Acolhido | Medicamento | Esquema (M+T+N) | Tipo de uso | Data início | Ações
  - **Botão "Nova Prescrição":**
    - Modal com formulário:
      - Select acolhido
      - Select medicamento (ou cadastro rápido)
      - Inputs numéricos: Manhã, Tarde, Noite
      - Select tipo de uso
      - Date picker início (e fim se tratamento)
  - **Ações na linha:**
    - Editar (só se não houver separação confirmada)
    - Suspender (desativa, preserva histórico)

---

#### Tela 8 — Medicações — Separação Semanal
- **Rota:** `/medicacoes/separacao`
- **Acesso:** GERENTE (V), ADMINISTRATIVO (C)
- **Elementos:**
  - **Configuração do período:**
    - Date picker início e fim (default: próximos 7 dias)
    - Se feriado no meio: checkbox "estender período"
  - **Botão "Gerar Relatório":**
    - Sistema calcula e exibe preview
  - **Preview (tabela):**
    - Agrupado por acolhido
    - Cada medicamento com: esquema, dias, total calculado
    - Destaque em vermelho se medicamento controlado
  - **Botão "Confirmar Separação":**
    - Registra data/hora
    - Se estoque ativo: baixa automática
    - Gera PDF para impressão (futuro)

---

#### Tela 9 — Agendamentos
- **Rota:** `/agendamentos`
- **Acesso:** Todos (exceto CUIDADOR que vê apenas "Agendamentos de Hoje")
- **Elementos:**
  - **Visualização em calendário semanal** (default) ou lista
  - **Dias passados:** esmaecidos, não editáveis
  - **Dia atual:** destacado em azul
  - **Eventos:**
    - Cor por tipo (médico = verde, audiência = vermelho...)
    - Clique → modal de detalhes + ação "Registrar Retorno" (técnicos)
  - **Botão "Novo Agendamento"** (admin, gerente, técnico):
    - Formulário: acolhido, tipo, data, hora, local, profissional, obs
  - **Alerta no topo (se amanhã tem agendamento):**
    - Banner amarelo fixo: "Amanhã há X agendamentos. Clique para ver preparativos."

---

#### Tela 10 — Documentos Técnicos (Consulta Centralizada)
- **Rota:** `/documentos/tecnicos`
- **Acesso:** Todos (com restrição de sigilo)
- **Elementos:**
  - Similar ao Prontuário/Consultar, mas visão agregada da unidade
  - Filtros avançados: por acolhido, por autor, por tipo, por período
  - Indicador de documentos sigilosos (cadeado para quem não tem acesso)

---

#### Tela 11 — Documentos Instrumentais
- **Rota:** `/documentos/instrumentais`
- **Acesso:** GERENTE (C), ADMINISTRATIVO (C)
- **Elementos:**
  - **Abas:** Modelos | Emitidos
  - **Modelos:**
    - Lista de templates criados pelo administrativo
    - Botão "Novo Modelo" (editor com placeholders `{{variaveis}}`)
  - **Emitir:**
    - Seleciona modelo → preenche variáveis → preview → imprimir/salvar PDF

---

#### Tela 12 — Equipe
- **Rota:** `/equipe`
- **Acesso:** GERENTE (C), COORDENACAO (V)
- **Elementos:**
  - Tabela: nome, perfil, cargo, status, último acesso
  - Ações: editar, transferir, desligar
  - Botão "Novo Colaborador"

---

#### Tela 13 — Transferências
- **Rota:** `/transferencias`
- **Acesso:** GERENTE (C), COORDENACAO (V)
- **Elementos:**
  - **Abas:** Acolhidos | Funcionários
  - **Status visual:** timeline de cada transferência (pendente → autorizada → concluída)
  - Ações por status:
    - PENDENTE_ORIGEM: botão "Autorizar Saída" (gerente origem)
    - PENDENTE_DESTINO: botão "Autorizar Chegada" (gerente destino)

---

#### Tela 14 — Relatórios
- **Rota:** `/relatorios`
- **Acesso:** COORDENACAO (C), GERENTE (V da própria unidade)
- **Elementos:**
  - Cards de relatórios disponíveis:
    - Censo SUAS
    - Produtividade Técnica
    - Consumo de Medicamentos
    - Movimentação (entradas/saídas)
  - Filtros: período, unidade (coordenação vê todas)
  - Botão exportar: PDF | Excel

---

### 5. Componentes Base Reutilizáveis

| Componente | Props principais | Uso |
|------------|------------------|-----|
| `BaseButton` | variant, size, isLoading, leftIcon, onClick | Todo o sistema |
| `BaseInput` | label, error, register (RHF), icon | Forms |
| `BaseSelect` | options, label, placeholder | Forms |
| `BaseTable` | columns, data, pagination, sorting, onRowClick | Listagens |
| `BaseCard` | title, subtitle, children, footer | Dashboards |
| `BaseModal` | open, onClose, title, children | Ações rápidas |
| `BaseDrawer` | open, onClose, title, children | Detalhes do acolhido |
| `BaseBadge` | variant (success, warning, danger, info), label | Status |
| `BaseToast` | type, message | Feedback (Sonner) |
| `BaseAvatar` | src, fallback, size | Usuários e acolhidos |
| `BaseDatePicker` | value, onChange, label | Agendamentos, períodos |
| `BaseSearch` | placeholder, onSearch, loading | Busca fuzzy |
| `BaseEmptyState` | icon, title, description | Listas vazias |
| `BaseSkeleton` | rows, columns | Loading states |

---

### 6. Decisões de UX Críticas

1. **Mobile-first não, mas tablet-friendly:** Abrigos usam desktops e tablets. Layout deve funcionar em 1024px.
2. **Feedback imediato:** Toda ação destrutiva (bloquear documento, desligar acolhido) exige modal de confirmação.
3. **Prevenção de erro:** Campos numéricos de medicação têm validação de máximo (ex: não permitir 99 comprimidos de uma vez).
4. **Acessibilidade:** Cores não são o único indicador (ícones + texto sempre).
5. **Performance:** Imagens de acolhidos lazy-loaded; tabelas paginadas server-side.
6. **Offline awareness:** Se conexão cair durante preenchimento de documento, avisar usuário e sugerir copiar conteúdo.
