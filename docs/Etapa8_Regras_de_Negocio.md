# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 8 — Regras de Negócio (v1.0)

---

### 1. Introdução

Regras numéricas, validações e workflows que devem ser implementados no código. Estas regras são obrigatórias e não negociáveis.

---

### 2. Regras de Vagas e Capacidade

#### R-VAGA-01 — Vaga Qualificada
- Cada abrigo possui `Vaga` registros por combinação de `faixaEtaria` + `genero`.
- Soma de `Vaga.quantidade` ≤ `Unidade.capacidadeTotal`.
- Um acolhido só pode ser vinculado a um abrigo se existir vaga para sua `faixaEtaria` + `genero` com `ocupadas < quantidade`.

#### R-VAGA-02 — Cálculo de Ocupação
- `ocupadas` é calculada automaticamente: `COUNT(Acolhido WHERE status = EM_ACOLHIMENTO AND unidadeAtualId = abrigo AND faixaEtaria = X AND genero = Y)`.
- Não é um campo editável manualmente.

#### R-VAGA-03 — Emergência
- Se abrigo receber acolhido fora do perfil em emergência, exigir:
  - Motivo documentado no campo `observacoesEntrada`
  - Notificação automática à coordenação
  - Flag `emergencia = true` na passagem

---

### 3. Regras do Acolhido

#### R-ACOL-01 — Cadastro Provisório
- Status inicial: `PROVISORIO`.
- Em até 7 dias, deve ter:
  - Escuta Inicial criada (documento tipo `ESCUTA_INICIAL`)
  - Data de nascimento confirmada ou idade aproximada registrada
- Após 7 dias sem escuta inicial: alerta diário ao gerente.

#### R-ACOL-02 — Faixa Etária Automática
- Se `dataNascimento` existir: calcular idade e definir `faixaEtaria`.
- Se `idadeAproximada` existir: usar ela para definir `faixaEtaria`.
- `faixaEtaria` nunca pode ser nulo.

#### R-ACOL-03 — Busca de Duplicidade
- Antes de criar novo acolhido, buscar por:
  - Nome similar (fuzzy, mínimo 3 caracteres)
  - CPF exato (se informado)
  - Nome da mãe similar (se informado)
- Se encontrado > 70% similaridade: exibir modal "Possível cadastro existente".

#### R-ACOL-04 — Desligamento
- Só pode ser desligado se houver documento tipo `DESLIGAMENTO` com status `ATIVO`.
- Só gerente pode autorizar.
- Após desligamento: `unidadeAtualId = null`, `status = DESLIGADO`.

---

### 4. Regras de Documentos

#### R-DOC-01 — Imutabilidade
- Documento com `status = ATIVO` não pode ser editado ou excluído.
- Qualquer alteração deve ser via `ERRATA`.

#### R-DOC-02 — Autoria da Errata
- Apenas o `autorId` do documento original pode criar errata.
- Errata deve ter `motivoErrata` com mínimo 20 caracteres.

#### R-DOC-03 — Escuta Inicial Única
- Uma passagem (`EM_ANDAMENTO`) só pode ter UM documento tipo `ESCUTA_INICIAL`.
- Tentativa de criar segunda: bloquear com erro.

#### R-DOC-04 — Sigilo
- Documento com `sigiloso = true`:
  - Visível para: autor, gerente da mesma unidade, coordenação, psicólogos da mesma unidade.
  - Invisível para: administrativo, educador, cuidador, AS de outra unidade.
  - Log de auditoria obrigatório quando visualizado.

#### R-DOC-05 — Vinculação a Agendamento
- Documento tipo `ACOMPANHAMENTO_CLINICO` pode ser vinculado a `Agendamento`.
- Se vinculado, `Agendamento.status` deve ser `REALIZADO`.

---

### 5. Regras de Medicações

#### R-MED-01 — Esquema Posológico
- `esquemaManha`, `esquemaTarde`, `esquemaNoite` devem ser inteiros ≥ 0.
- Soma dos três deve ser > 0 (prescrição não pode ser 0+0+0).

#### R-MED-02 — Prescrição Editável
- Prescrição só pode ser editada/excluída se:
  - `ativa = true`
  - NÃO existir `ItemSeparacao` com `separacao.confirmadoEm != null` vinculado a ela.

#### R-MED-03 — Cálculo de Separação
- Fórmula: `quantidade = (esquemaManha + esquemaTarde + esquemaNoite) × dias_no_período`.
- Se prescrição começar no meio do período: `dias = MIN(dataFim, dataFimPeriodo) - MAX(dataInicio, dataInicioPeriodo) + 1`.
- Se prescrição terminar no meio do período: calcular apenas dias válidos.
- Se `tipoUso = CONTINUO` e sem `dataFim`: considerar todo o período.

#### R-MED-04 — Medicamento Controlado
- Se `Medicamento.controlado = true`: exibir alerta visual (badge laranja) no relatório.
- Não altera cálculo, apenas alerta.

#### R-MED-05 — Estoque
- `EstoqueMedicamento.quantidade` nunca pode ser negativo.
- Baixa automática na confirmação da separação: `quantidade -= SUM(itens.quantidade)`.
- Alerta quando `quantidade ≤ quantidadeMinima`.
- Alerta quando `validade ≤ hoje + 30 dias`.

---

### 6. Regras de Agendamentos

#### R-AGE-01 — Alerta no Login
- No login do ADMINISTRATIVO, se houver agendamento para `amanhã`: exibir banner fixo no topo do dashboard.
- Banner deve listar: quantidade, acolhidos, tipos.

#### R-AGE-02 — Relatório de Retorno
- Só pode ser criado se `Agendamento.status = AGENDADO`.
- Após criação do documento vinculado: `Agendamento.status = REALIZADO`.
- Se acolhido não comparecer: ADMINISTRATIVO pode marcar `NAO_COMPARECEU`.

#### R-AGE-03 — Cancelamento
- Agendamento passado não pode ser cancelado.
- Cancelamento exige motivo obrigatório (mínimo 10 caracteres).

---

### 7. Regras de Transferência

#### R-TRA-01 — Dupla Autorização
- Transferência de acolhido exige:
  1. Autorização do gerente da unidade de origem.
  2. Autorização do gerente da unidade de destino.
- Sem as duas, status permanece `PENDENTE`.

#### R-TRA-02 — Vaga no Destino
- Antes de apresentar ao gerente destino, sistema verifica se há vaga qualificada.
- Se não houver: transferência não pode ser concluída; gerente destino só pode recusar.

#### R-TRA-03 — Bloqueio de Edição
- Durante `EM_TRANSFERENCIA`, prontuário do acolhido fica em modo leitura.
- Apenas erratas podem ser criadas (se necessário corrigir erro grave).

#### R-TRA-04 — Funcionário
- Transferência de funcionário exige autorização do gerente destino.
- Coordenação é notificada mas não precisa autorizar.

---

### 8. Regras de Ocorrências

#### R-OCR-01 — Gravidade Grave
- Ocorrência `GRAVE` gera notificação imediata ao gerente.
- Gerente deve marcar como "lida" em até 24h (alerta persistente).

#### R-OCR-02 — Anonimato
- Ocorrência pode ser registrada sem vincular acolhido específico (ex: incidente geral).
- Campo `acolhidoId` é opcional.

---

### 9. Regras de Autenticação

#### R-AUTH-01 — Senha
- Mínimo 6 caracteres.
- Hash com bcrypt, salt rounds = 12.

#### R-AUTH-02 — Sessão
- Expira em 8 horas de inatividade.
- Token JWT assinado com `NEXTAUTH_SECRET`.

#### R-AUTH-03 — Redirecionamento
- COORDENACAO → `/rede`
- Demais → `/abrigo`
- Não autenticado → `/login`
