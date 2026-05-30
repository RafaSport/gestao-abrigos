# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 2 — Levantamento de Requisitos Detalhados (v2.0)

---

### 1. Introdução

Este documento detalha todos os requisitos funcionais (RF) e não-funcionais (RNF) do SGA, com descrição completa, ator responsável e prioridade.

---

### 2. Requisitos Funcionais — Prontuário e Documentos (RF01–RF14)

#### RF01 — Cadastro de Acolhido
- **Descrição:** Permitir o cadastro de novo acolhido no sistema, vinculando-o a uma passagem ativa no abrigo.
- **Ator:** Gerente, Assistente Social ou Psicólogo.
- **Campos obrigatórios:** nome completo, data de nascimento (ou idade aproximada), gênero, abrigo de destino.
- **Campos opcionais:** nome da mãe, nome do pai, CPF (se tiver), RG, naturalidade, foto, observações de entrada.
- **Regras:**
  - Se data de nascimento desconhecida, permitir idade aproximada; sistema calcula faixa etária automaticamente.
  - Verificar se já existe cadastro prévio na rede pelo nome + nome da mãe (busca fuzzy).
  - Vaga qualificada obrigatória: abrigo deve ter vaga disponível para a faixa etária/gênero do acolhido.
- **Prioridade:** Alta

#### RF02 — Escuta Inicial
- **Descrição:** Documento técnico obrigatório criado no dia da entrada. Registra o motivo do acolhimento, condições de chegada, demandas imediatas e encaminhamentos.
- **Ator:** Assistente Social ou Psicólogo.
- **Regras:**
  - Só pode ser criado uma vez por passagem.
  - Após salvar, fica bloqueado para edição; erros geram errata.
- **Prioridade:** Alta

#### RF03 — Documentos de Acompanhamento Técnico
- **Descrição:** Criação de documentos técnicos durante a estadia.
- **Tipos:**
  - Acompanhamento Social
  - Acompanhamento Psicológico
  - Acompanhamento Clínico / Uso de Medicamentos
  - Receituário
- **Ator:** Assistente Social, Psicólogo ou Gerente (dependendo do tipo).
- **Regras:**
  - Editor de texto enriquecido simples (negrito, listas, inserção de tabelas básicas).
  - Vinculação obrigatória ao acolhido e à passagem ativa.
  - Após salvar, documento entra em status `ATIVO` e não pode ser editado.
- **Prioridade:** Alta

#### RF04 — Errata de Documento Técnico
- **Descrição:** Correção de um documento técnico já salvo.
- **Ator:** Apenas o autor original do documento.
- **Regras:**
  - O documento original permanece visível e imutável.
  - A errata é um novo documento vinculado ao original (`documento_pai_id`), com tipo `ERRATA`.
  - Deve conter: motivo da correção, data da errata, e o conteúdo corrigido.
  - O status do original muda para `CORRIGIDO`.
- **Prioridade:** Alta

#### RF05 — Documentos Digitalizados (Anexos)
- **Descrição:** Upload de arquivos escaneados ou fotografados vinculados ao prontuário do acolhido.
- **Ator:** Assistente Administrativo, Gerente, Técnico.
- **Tipos de arquivo permitidos:** PDF, JPG, PNG.
- **Categorias:** Documentos Pessoais (RG, CPF, certidão), Receitas Médicas Físicas, Laudos, Outros.
- **Regras:**
  - Tamanho máximo por arquivo: 10MB.
  - Nome do arquivo deve ser normalizado.
  - Envio registrado no log de auditoria.
- **Prioridade:** Alta

#### RF06 — Consulta ao Prontuário Completo
- **Descrição:** Visualização do histórico completo do acolhido, incluindo passagens anteriores em outros abrigos da rede.
- **Ator:** Todos os perfis técnicos (com restrições de sigilo).
- **Regras:**
  - Coordenação vê tudo de todos os abrigos.
  - Gerente/técnico do abrigo atual vê tudo do acolhido, mesmo de abrigos anteriores.
  - Administrativo vê apenas documentos instrumentais e agendamentos (não vê acompanhamento psicológico detalhado).
  - Educador/Cuidador vê dados básicos e ocorrências.
- **Prioridade:** Alta

#### RF07 — Documentos Instrumentais
- **Descrição:** Emissão de documentos pré-formatados (ofícios, declarações, atestados) preenchidos automaticamente com dados do acolhido.
- **Ator:** Assistente Administrativo (criação do modelo) e Gerente/Técnico (preenchimento).
- **Regras:**
  - Modelos criados pelo administrativo com placeholders (ex: `{{nome_acolhido}}`, `{{data_nascimento}}`).
  - Possibilidade de impressão ou exportação em PDF.
- **Prioridade:** Média

#### RF08 — PIA — Plano Individual de Atendimento
- **Descrição:** Cadastro do PIA do acolhido, com metas, prazos, responsáveis e revisões.
- **Ator:** Assistente Social (principal), Gerente (aprovação).
- **Regras:**
  - Campos padronizados conforme orientações SUAS.
  - Revisão periódica obrigatória (a cada 6 meses ou evento significativo).
- **Prioridade:** Média

#### RF09 — Desligamento de Acolhido
- **Descrição:** Documento técnico de encerramento da passagem ativa.
- **Ator:** Técnico cria; Gerente autoriza.
- **Campos:** motivo (reintegração familiar, adoção, maioridade, óbito, evasão, transferência para outra rede), data, encaminhamentos, parecer técnico.
- **Regras:**
  - Após autorização do gerente, acolhido muda status para `DESLIGADO`.
  - Passagem é encerrada com data de saída.
  - Acolhido sai da listagem ativa do abrigo.
  - Prontuário permanece acessível para consulta.
- **Prioridade:** Alta

#### RF10 — Transferência de Acolhido entre Abrigos
- **Descrição:** Movimentação do acolhido de um abrigo para outro dentro da rede.
- **Ator:** Técnico inicia; Gerente origem autoriza saída; Gerente destino autoriza chegada.
- **Regras:**
  - Só pode ser iniciada se houver vaga qualificada no abrigo destino.
  - Durante o processo, acolhido entra em status `EM_TRANSFERENCIA`.
  - Prontuário completo fica acessível ao gerente destino antes da autorização.
  - Concluída a transferência, passagem antiga é encerrada e nova é iniciada no destino.
- **Prioridade:** Alta

#### RF11 — Busca de Acolhido na Rede
- **Descrição:** Permite localizar um acolhido em qualquer abrigo da rede pelo nome, CPF ou código interno.
- **Ator:** Coordenação, Gerente, Técnico.
- **Regras:** Busca fuzzy para nomes; resultados mostram abrigo atual e status.
- **Prioridade:** Média

#### RF12 — Histórico de Passagens
- **Descrição:** Linha do tempo de todas as passagens do acolhido (abrigo, datas, motivos).
- **Ator:** Todos (com restrição de sigilo).
- **Prioridade:** Média

---

### 3. Requisitos Funcionais — Medicações (RF13–RF18)

#### RF13 — Cadastro de Medicamentos no Catálogo
- **Descrição:** Catálogo de medicamentos disponíveis no abrigo.
- **Ator:** Administrativo.
- **Campos:** nome comercial, princípio ativo, apresentação (mg/ml), forma farmacêutica, controlado (boolean), observações.
- **Prioridade:** Alta

#### RF14 — Prescrição de Medicamento
- **Descrição:** Registro da prescrição médica/técnica para um acolhido.
- **Ator:** Gerente, Assistente Social, Psicólogo (quando prescrito por eles ou por médico externo).
- **Campos:** medicamento (catálogo), esquema posológico manhã/tarde/noite (ex: 1+0+2), tipo de uso (contínuo ou tratamento específico), data início, data fim (se tratamento), observações, médico/psicólogo prescritor.
- **Regras:**
  - Esquema deve ser numérico inteiro ≥ 0.
  - Prescrição ativa pode ser suspensa (não excluída).
  - Prescrição só pode ser editada/excluída se ainda não houver separação semanal vinculada.
- **Prioridade:** Alta

#### RF15 — Relatório de Separação Semanal
- **Descrição:** Geração automática de relatório que informa quanto colocar em cada caixinha por acolhido.
- **Ator:** Administrativo consulta e executa.
- **Entrada:** período (data início e fim, geralmente 7 dias).
- **Saída:** lista por acolhido → medicamento → quantidade total = (M+T+N) × dias.
- **Regras:**
  - Só considera prescrições ativas no período.
  - Se prescrição começar ou terminar no meio do período, calcular proporcionalmente.
  - Administrativo confirma a separação; sistema dá baixa no estoque (se módulo estoque ativo).
- **Prioridade:** Alta

#### RF16 — Controle de Estoque de Medicamentos
- **Descrição:** Entrada e saída de medicamentos no abrigo.
- **Ator:** Administrativo.
- **Campos:** medicamento, lote, validade, quantidade entrada, quantidade saída, motivo.
- **Regras:**
  - Alerta automático quando medicamento estiver próximo do vencimento (30 dias).
  - Alerta quando estoque abaixo do mínimo (configurável).
  - Baixa automática quando confirmada separação semanal.
- **Prioridade:** Média

---

### 4. Requisitos Funcionais — Agendamentos (RF17–RF19)

#### RF17 — Cadastro de Agendamento
- **Descrição:** Registro de compromissos do acolhido.
- **Ator:** Administrativo.
- **Campos:** acolhido, tipo (médico, dentista, psicólogo externo, audiência judicial, retirada de documentos, outro), data, horário, local, profissional/órgão externo, observações, status.
- **Regras:**
  - Alerta automático no login do administrativo no dia anterior ao agendamento.
  - Listagem de "Agendamentos de Amanhã" visível no dashboard.
- **Prioridade:** Alta

#### RF18 — Relatório de Retorno
- **Descrição:** Documento técnico vinculado a um agendamento realizado.
- **Ator:** Técnico (Assistente Social ou Psicólogo).
- **Campos:** resumo do atendimento, encaminhamentos, próximos passos, anexos.
- **Regras:** Vinculação obrigatória ao agendamento; agendamento muda status para `REALIZADO`.
- **Prioridade:** Alta

#### RF19 — Cancelamento e Reagendamento
- **Descrição:** Alteração de agendamentos.
- **Ator:** Administrativo.
- **Regras:** Motivo do cancelamento obrigatório; histórico preservado.
- **Prioridade:** Média

---

### 5. Requisitos Funcionais — Acesso, Perfis e Layout (RF20–RF25)

#### RF20 — Autenticação
- **Descrição:** Login com e-mail e senha.
- **Ator:** Todos.
- **Regras:**
  - Senha criptografada (bcrypt).
  - Sessão com JWT ou session cookie.
  - Logout em todos os dispositivos ao alterar senha.
- **Prioridade:** Alta

#### RF21 — Controle de Acesso por Perfil
- **Descrição:** Menu lateral e funcionalidades habilitadas conforme perfil do usuário.
- **Ator:** Sistema (automático).
- **Regras:** Itens inacessíveis devem estar ocultos (não apenas desabilitados).
- **Prioridade:** Alta

#### RF22 — Redirecionamento Pós-Login
- **Descrição:** Direcionar usuário para a tela correta com base no perfil.
- **Regras:** Coordenação → Painel da Rede; demais → Painel do Abrigo.
- **Prioridade:** Alta

#### RF23 — Gestão de Funcionários
- **Descrição:** Cadastro, edição, transferência e desligamento de colaboradores.
- **Ator:** Gerente (cadastro/desligamento da unidade); Coordenação (visualização rede).
- **Regras:**
  - Funcionário vinculado a um abrigo (exceto coordenação).
  - Transferência exige autorização do gerente destino.
  - Desligamento revoga acesso imediatamente.
- **Prioridade:** Alta

#### RF24 — Cadastro de Abrigos (Coordenação)
- **Descrição:** CRUD de abrigos da rede.
- **Ator:** Coordenação.
- **Campos:** nome, perfil de atendimento (faixa etária + gênero), endereço, capacidade total, vagas por qualificação, telefone, e-mail, status (ativo/inativo).
- **Prioridade:** Alta

#### RF25 — Configuração de Vagas Qualificadas
- **Descrição:** Definição de quantas vagas existem para cada perfil (faixa etária/gênero) dentro do abrigo.
- **Ator:** Coordenação.
- **Regras:** Soma das vagas qualificadas não pode exceder capacidade total.
- **Prioridade:** Alta

---

### 6. Requisitos Funcionais — Painel Gerencial (RF26–RF28)

#### RF26 — Painel do Abrigo
- **Descrição:** Dashboard operacional da unidade.
- **Conteúdo:**
  - Cards: vagas totais, ocupadas, disponíveis (por qualificação)
  - Lista de acolhidos ativos (foto, nome, tempo de acolhimento)
  - Agendamentos de hoje e amanhã
  - Documentos criados recentemente
  - Alertas (medicação vencendo, documento pendente)
- **Ator:** Todos os perfis do abrigo (com granularidade diferente).
- **Prioridade:** Alta

#### RF27 — Painel da Coordenação (Rede)
- **Descrição:** Visão consolidada de todos os abrigos.
- **Conteúdo:**
  - Cards por abrigo (nome, vagas, ocupação, perfil)
  - Busca de acolhido na rede
  - Filtros por perfil de abrigo com vaga disponível
  - Indicadores: taxa de ocupação, tempo médio de permanência, desligamentos no mês
- **Ator:** Coordenação.
- **Prioridade:** Alta

#### RF28 — Relatórios Estatísticos
- **Descrição:** Geração de relatórios para prestação de contas.
- **Tipos:**
  - Censo SUAS (quantitativo por sexo, idade, tempo de permanência)
  - Produtividade técnica (documentos por profissional)
  - Consumo de medicamentos (mensal)
  - Movimentação (entradas, transferências, desligamentos)
- **Formatos:** PDF e Excel.
- **Ator:** Coordenação, Gerente (apenas da própria unidade).
- **Prioridade:** Média

---

### 7. Requisitos Funcionais — Ocorrências e Cotidiano (RF29–RF31)

#### RF29 — Registro de Ocorrências
- **Descrição:** Registro de eventos relevantes no cotidiano do abrigo.
- **Ator:** Educador Social, Cuidador.
- **Campos:** data/hora, acolhido(s) envolvido(s), tipo (incidente, comportamento de risco, fuga, retorno, visita, acidente, outro), descrição, medidas tomadas, gravidade (leve, moderada, grave).
- **Regras:**
  - Notificação automática ao gerente para ocorrências graves.
  - Possibilidade de anexo de fotos.
- **Prioridade:** Média

#### RF30 — Registro de Visitas Familiares
- **Descrição:** Controle de visitas de familiares ao abrigo.
- **Ator:** Educador Social, Cuidador, Administrativo.
- **Campos:** data, hora, visitantes (nome, grau de parentesco, documento), acolhido, observações, avaliação do técnico.
- **Prioridade:** Baixa

#### RF31 — Registro de Atividades e Oficinas
- **Descrição:** Registro de atividades educativas, lazer e oficinas.
- **Ator:** Educador Social.
- **Campos:** data, tipo, descrição, participantes, avaliação.
- **Prioridade:** Baixa

---

### 8. Requisitos Não-Funcionais

#### RNF01 — Usabilidade
- Interface deve ser operável por usuários sem formação técnica.
- Termos devem refletir o vocabulário do SUAS e do dia a dia dos abrigos.
- Feedback imediato para todas as ações (toast, spinners, confirmações).

#### RNF02 — Responsividade
- Layout adaptável para desktops (1366×768 mínimo) e tablets (1024×768).
- Menu lateral colapsável em telas menores.

#### RNF03 — Desempenho
- Tempo de resposta < 2s para consultas comuns.
- Listagens paginadas (20 itens por página).
- Upload de arquivos com barra de progresso.

#### RNF04 — Segurança e LGPD
- Senhas criptografadas (bcrypt, salt ≥ 12).
- Sessão expira após 8h de inatividade.
- Log de auditoria de todas as ações (quem, o quê, quando, de onde).
- Dados em trânsito via TLS 1.3.
- Campos sensíveis (prontuário psicológico) com marcação de sigilo.

#### RNF05 — Rastreabilidade
- Todo documento técnico tem autor, data/hora e IP registrados automaticamente.
- Erratas preservam o original.
- Histórico de alterações em cadastros (audit log).

#### RNF06 — Disponibilidade
- Uptime alvo: 99% (considerando infraestrutura municipal).
- Estratégia de backup diário automático do banco de dados.

#### RNF07 — Compatibilidade
- Navegadores: Chrome, Edge, Firefox (últimas 2 versões).
- Sistema operacional: Windows 10+, Linux.

---

### 9. Matriz de Prioridade

| Prioridade | Requisitos | Justificativa |
|------------|------------|---------------|
| **Alta** | RF01–RF07, RF09, RF10, RF13–RF15, RF17, RF18, RF20–RF28 | Funcionalidades core; sem elas o sistema não opera |
| **Média** | RF08, RF11, RF12, RF16, RF19, RF29 | Importantes para gestão completa; podem ser refinadas em iteração |
| **Baixa** | RF30, RF31 | Melhorias de UX e registro cotidiano; não bloqueiam o MVP |
