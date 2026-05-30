# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 3 — Mapeamento de Perfis e Permissões (v2.0)

---

### 1. Introdução

Este documento define os 7 perfis de usuário, as permissões granulares por funcionalidade e as regras de isolamento de dados entre abrigos.

---

### 2. Perfis de Usuário

| Perfil | Sigla | Descrição |
|--------|-------|-----------|
| Coordenação da Rede | `COORDENACAO` | Supervisão estratégica; acesso a todos os abrigos e dados agregados |
| Gerente | `GERENTE` | Gestão operacional da unidade; autorizações críticas |
| Assistente Social | `ASSISTENTE_SOCIAL` | Atendimento técnico; prontuário e PIA |
| Psicólogo(a) | `PSICOLOGO` | Atendimento clínico; prontuário sigiloso |
| Assistente Administrativo | `ADMINISTRATIVO` | Medicações, agendamentos, documentos instrumentais, estoque |
| Educador Social | `EDUCADOR_SOCIAL` | Cotidiano; ocorrências e atividades |
| Cuidador | `CUIDADOR` | Cuidados diretos; registro limitado de ocorrências |

---

### 3. Níveis de Permissão

Cada funcionalidade pode ter um dos seguintes níveis por perfil:

| Nível | Sigla | Significado |
|-------|-------|-------------|
| **Completo** | `C` | Pode criar, ler, editar e excluir (soft delete) |
| **Técnico** | `T` | Pode criar e ler documentos técnicos; editar apenas rascunhos |
| **Administrativo** | `A` | Pode criar e gerenciar agendamentos, medicações, documentos instrumentais |
| **Operacional** | `O` | Pode criar e ler ocorrências e registros cotidianos |
| **Operacional Limitado** | `OL` | Pode criar ocorrências básicas; sem acesso a dados sensíveis |
| **Visualização** | `V` | Apenas leitura; sem criação ou edição |
| **Visualização Parcial** | `VP` | Leitura de subset de dados (ex: dados básicos do acolhido, sem prontuário psicológico) |
| **Visualização Simples** | `VS` | Apenas dashboard básico e listagens públicas da unidade |
| **Bloqueado** | `—` | Sem acesso; item oculto no menu |

---

### 4. Matriz de Permissões por Funcionalidade

#### 4.1 Gestão da Rede e Abrigos

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Ver Painel da Rede | C | — | — | — | — | — | — |
| Cadastrar/Editar Abrigo | C | — | — | — | — | — | — |
| Configurar Vagas Qualificadas | C | — | — | — | — | — | — |
| Ver Painel do Abrigo | C | C | C | C | C | VS | VS |
| Ver Equipe do Abrigo | C | C | V | V | V | V | V |

#### 4.2 Acolhidos e Prontuário

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Cadastrar Acolhido | — | C | C | C | — | — | — |
| Ver Listagem de Acolhidos | C | C | C | C | VP | VP | VP |
| Ver Ficha Básica do Acolhido | C | C | C | C | VP | VP | VP |
| Ver Prontuário Completo | C | C | T | T | VP | VP | — |
| Criar Documentos Técnicos | — | T | T | T | — | — | — |
| Criar Errata | — | T* | T* | T* | — | — | — |
| Anexar Documentos Digitalizados | C | C | C | C | C | — | — |
| Criar PIA | — | V | C | — | — | — | — |
| Ver PIA | C | C | C | C | V | V | — |

> *`T*` = apenas o autor original do documento pode criar errata.

#### 4.3 Medicações

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Cadastrar Medicamento (catálogo) | — | C | — | — | C | — | — |
| Prescrever Medicamento | — | C | C | C | — | — | — |
| Ver Prescrições Ativas | C | C | C | C | V | — | — |
| Gerar Separação Semanal | — | V | — | — | C | — | — |
| Confirmar Separação | — | V | — | — | C | — | — |
| Ver Relatório de Separação | C | C | C | C | C | — | — |

#### 4.4 Agendamentos

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Cadastrar Agendamento | — | C | C | C | C | — | — |
| Ver Agendamentos | C | C | C | C | C | V | V |
| Criar Relatório de Retorno | — | C | T | T | — | — | — |
| Cancelar/Reagendar | — | C | C | C | C | — | — |

#### 4.5 Documentos Instrumentais

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Criar Modelo de Documento | — | V | — | — | C | — | — |
| Preencher/Imprimir Documento | — | C | — | — | C | — | — |
| Ver Documentos Emitidos | C | C | V | V | C | — | — |

#### 4.6 Movimentação de Pessoas

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Iniciar Transferência de Acolhido | — | C | C | C | — | — | — |
| Autorizar Saída (origem) | — | C | — | — | — | — | — |
| Autorizar Chegada (destino) | — | C | — | — | — | — | — |
| Iniciar Desligamento | — | C | C | C | — | — | — |
| Autorizar Desligamento | — | C | — | — | — | — | — |
| Cadastrar Funcionário | — | C | — | — | — | — | — |
| Transferir Funcionário | — | C | — | — | — | — | — |
| Desligar Funcionário | — | C | — | — | — | — | — |

#### 4.7 Ocorrências e Cotidiano

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Registrar Ocorrência | — | C | V | V | V | O | OL |
| Ver Ocorrências | C | C | C | C | C | O | OL |
| Registrar Visita Familiar | — | C | V | V | O | O | — |
| Registrar Atividade/Oficina | — | C | V | V | V | O | — |

#### 4.8 Relatórios e Estatísticas

| Funcionalidade | Coord. | Gerente | Social | Psicól. | Administ. | Educador | Cuidador |
|----------------|--------|---------|--------|---------|-----------|----------|----------|
| Relatórios da Rede | C | — | — | — | — | — | — |
| Relatórios do Abrigo | C | C | — | — | — | — | — |
| Exportar PDF/Excel | C | C | — | — | — | — | — |

---

### 5. Regras de Isolamento e Visibilidade

#### R1 — Isolamento por Abrigo
- Todo dado operacional (acolhido, funcionário, documento, agendamento, ocorrência) pertence a uma `unidade_id`.
- Usuários com perfil diferente de `COORDENACAO` só visualizam dados da sua unidade atual.
- Exceção: prontuário do acolhido é visível em qualquer unidade da rede onde ele estiver ou estiver sido transferido.

#### R2 — Prontuário Unificado
- O prontuário do acolhido é único na rede, independente de quantas passagens ele tenha.
- Documentos de abrigos anteriores são legíveis, mas não editáveis.
- Ocorrências de abrigos anteriores são visíveis apenas para Coordenação e Gerente do abrigo atual.

#### R3 — Autoria e Responsabilidade
- Todo documento técnico tem `autor_id`, `unidade_id`, `criado_em`, `ip_address`.
- Apenas o autor pode criar errata do seu próprio documento.
- Gerentes podem visualizar todos os documentos da unidade, mas não editam documentos de técnicos.

#### R4 — Dupla Autorização
- Transferência de acolhido: gerente origem + gerente destino.
- Desligamento: técnico cria + gerente autoriza.
- Transferência de funcionário: gerente origem + gerente destino.

#### R5 — Sigilo Técnico
- Documentos de acompanhamento psicológico têm flag `sigiloso`.
- Administrativo, Educador e Cuidador NÃO visualizam conteúdo de documentos sigilosos (nem título detalhado).
- Assistente Social visualiza documentos psicológicos apenas se houver necessidade justificada (registro no log de auditoria).

#### R6 — Preservação de Histórico
- Nenhum registro é excluído fisicamente do banco.
- Toda exclusão é lógica (`status: INATIVO`, `deleted_at`).
- Funcionário desligado perde acesso, mas seu histórico de autoria permanece.

---

### 6. Fluxos de Autorização Específicos

#### 6.1 Transferência de Acolhido
```
Técnico cria documento de transferência
    → Sistema bloqueia edições no prontuário do acolhido
    → Notifica Gerente da unidade de origem
Gerente origem autoriza saída
    → Sistema verifica vaga qualificada no destino
    → Notifica Gerente da unidade de destino
Gerente destino autoriza chegada
    → Sistema encerra passagem na origem
    → Sistema cria nova passagem no destino
    → Acolhido muda para status EM_ACOLHIMENTO no novo abrigo
```

#### 6.2 Transferência de Funcionário
```
Gerente origem inicia transferência
    → Sistema coloca funcionário em status EM_TRANSFERENCIA
    → Notifica Gerente destino e Coordenação
Gerente destino autoriza
    → Sistema atualiza unidade do funcionário
    → Coordenação recebe notificação de conclusão
```
