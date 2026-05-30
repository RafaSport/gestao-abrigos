# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 1 — Visão Geral do Projeto (Refatorado v2.0)

---

### 1. Identificação do Projeto

| Campo | Valor |
|-------|-------|
| **Nome** | SGA — Sistema de Gestão de Abrigos |
| **Organização** | Prefeitura do Recife — Secretaria de Assistência Social |
| **Abrangência** | Rede de Abrigos Institucionais do Recife |
| **Público Atendido** | Pessoas em situação de acolhimento institucional (crianças, adolescentes, adultos, idosos, famílias) |
| **Versão** | 2.0 — Base para desenvolvimento |
| **Data** | Maio de 2026 |

---

### 2. Contexto e Problema

Hoje a gestão dos abrigos depende de:
- Fichas físicas e registros em papel
- Dificuldade de acesso ao histórico completo do acolhido
- Risco de perda, deterioração ou extravio de documentos
- Impossibilidade de compartilhamento em tempo real entre abrigos da rede
- Retrabalho na produção de relatórios para fiscalização (SUAS, MDS, Justiça)
- Falta de visão consolidada da ocupação e perfil da rede
- Controle de medicações e agendamentos feito manualmente, sujeito a erro de cálculo e esquecimento

---

### 3. Objetivos

#### Geral
Sistema web responsivo para gestão integrada da rede de abrigos, garantindo rastreabilidade, segurança dos dados (LGPD) e continuidade do atendimento independente de qual unidade o acolhido esteja.

#### Específicos
1. **Prontuário digital único e imutável:** dossiê vivo do acolhido, portável entre abrigos, com rastreabilidade de autoria e erratas.
2. **Gestão de documentos técnicos e instrumentais:** produção, consulta, impressão e controle de versões.
3. **Controle farmacêutico:** prescrição com esquema posológico (M+T+N), cálculo automático de separação semanal e controle de estoque.
4. **Agendamentos e alertas:** cadastro de compromissos com notificação antecipada e vinculação a relatório de retorno.
5. **Movimentação de pessoas:** entrada, transferência entre abrigos e desligamento com dupla autorização.
6. **Painel gerencial:** visão operacional do abrigo e visão consolidada da coordenação da rede.
7. **Relatórios estatísticos:** censo SUAS, produtividade técnica, consumo de medicamentos.
8. **Controle de acesso por perfil:** princípio do menor privilégio, isolamento por abrigo (exceto coordenação).

---

### 4. Usuários do Sistema

| Perfil | Função Principal | Abrangência | Nível de Acesso |
|--------|------------------|-------------|-----------------|
| **Coordenação da Rede** | Supervisão estratégica, cadastro de abrigos, relatórios consolidados | Toda a rede | Completo |
| **Gerente** | Gestão da unidade, autorizações, visão geral | Abrigo próprio | Completo na unidade |
| **Assistente Social** | Prontuário, relatórios técnicos, PIA | Abrigo próprio | Técnico |
| **Psicólogo(a)** | Atendimentos clínicos, relatórios técnicos | Abrigo próprio | Técnico |
| **Assistente Administrativo** | Medicações, agendamentos, documentos instrumentais, estoque | Abrigo próprio | Administrativo |
| **Educador Social** | Registro de ocorrências, cotidiano, atividades | Abrigo próprio | Operacional |
| **Cuidador** | Registro de intercorrências, administração de medicações (física) | Abrigo próprio | Operacional limitado |

---

### 5. Escopo

#### Dentro do escopo (MVP)
- Cadastro de abrigos (coordenação) e acolhidos
- Prontuário digital com documentos técnicos e digitalizados
- Prescrição de medicamentos e relatório de separação semanal
- Agendamentos com alerta e relatório de retorno
- Transferência e desligamento de acolhidos
- Gestão de equipe (funcionários) por abrigo
- Painel gerencial (abrigo e rede)
- Relatórios estatísticos básicos (exportação PDF/Excel)
- Autenticação e controle de acesso por perfil
- Responsividade para desktops e tablets

#### Fora do escopo (MVP — podem entrar em fases futuras)
- Integração com sistemas externos (e-SUAS, prontuário da Secretaria de Saúde, Sistema Escolar)
- Aplicativo nativo mobile (iOS/Android)
- Módulo financeiro (folha, compras, licitações)
- Emissão de documentos oficiais com assinatura digital ICP-Brasil
- Registro diário de plantão (turnos de educador/cuidador)
- Chat/mensagens internas
- Biometria ou reconhecimento facial

---

### 6. Premissas e Restrições

| ID | Premissa/Restrição | Impacto |
|----|-------------------|---------|
| P01 | Internet disponível em cada abrigo (mesmo que instável) | Interface deve ser resiliente; considerar fila offline futura |
| P02 | Usuários sem formação técnica em informática | UX deve ser extremamente simples, com termos do dia a dia |
| P03 | LGPD obrigatória | Logs de auditoria, criptografia, anonimização futura |
| P04 | Desenvolvimento incremental | Priorizar MVP funcional; funcionalidades complexas em fases |
| P05 | Recursos limitados (deploy sem custos) | Stack otimizada para Vercel + Neon free tier |
| P06 | Computadores simples nos abrigos | Performance leve; evitar animações pesadas |
| P07 | Cada abrigo tem capacidade fixa e perfil qualificado | Controle de vagas não é só numérico; é por faixa etária e gênero |

---

### 7. Glossário do Domínio

| Termo | Significado |
|-------|-------------|
| **Acolhido** | Pessoa em situação de acolhimento institucional |
| **Passagem** | Período de estadia de um acolhido em um abrigo (do entrada ao desligamento/transferência) |
| **Prontuário** | Conjunto completo de documentos e arquivos de um acolhido |
| **Documento Técnico** | Relatório produzido no sistema por técnico (escuta, acompanhamento, etc.) |
| **Documento Instrumental** | Modelo pré-definido preenchido pelo administrativo (ofícios, declarações) |
| **Errata** | Correção vinculada a um documento técnico já salvo; preserva o original |
| **Esquema Posológico** | Notação M+T+N (manhã + tarde + noite) de administração de medicamento |
| **Separação Semanal** | Processo de separação dos comprimidos nas caixinhas dos acolhidos |
| **PIA** | Plano Individual de Atendimento (exigência SUAS) |
| **RMA** | Relatório de Gestão do Serviço (exigência SUAS) |

---

### 8. Documentos Filhos

| Etapa | Documento | Status |
|-------|-----------|--------|
| 2 | Levantamento de Requisitos Detalhados | ✅ Refatorado |
| 3 | Mapeamento de Perfis e Permissões | ✅ Refatorado |
| 4 | Modelagem de Dados Completa | ✅ Refatorado |
| 5 | Fluxos e Processos Detalhados | ✅ Refatorado |
| 6 | Protótipo de Telas e UX | ✅ Refatorado |
| 7 | Stack e Arquitetura | 🆕 Novo |
| 8 | Regras de Negócio | 🆕 Novo |
| 9 | LGPD e Auditoria | 🆕 Novo |
| 10 | Plano de Implementação | 🆕 Novo |
