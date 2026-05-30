# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 5 — Fluxos e Processos Detalhados (v2.0)

---

### 1. Introdução

Descrição passo a passo dos processos principais, incluindo regras de negócio, estados do sistema e notificações.

---

### 2. Estados do Acolhido (Máquina de Estados)

```
PROVISORIO
    → [cadastro completo + escuta inicial salva] → EM_ACOLHIMENTO

EM_ACOLHIMENTO
    → [início transferência] → EM_TRANSFERENCIA
    → [saída temporária registrada] → SAIDA_TEMPORARIA
    → [desligamento autorizado] → DESLIGADO

EM_TRANSFERENCIA
    → [transferência concluída] → EM_ACOLHIMENTO (novo abrigo)
    → [transferência cancelada] → EM_ACOLHIMENTO (abrigo origem)

SAIDA_TEMPORARIA
    → [retorno confirmado] → EM_ACOLHIMENTO
    → [não retorna no prazo + desligamento] → DESLIGADO

DESLIGADO (final)
```

---

### 3. Fluxo 1 — Entrada de Novo Acolhido

**Atores:** Técnico (AS/Psicólogo) ou Gerente

```
1. Acolhido chega ao abrigo (rua, encaminhamento, abordagem social)
2. Técnico acessa "Novo Acolhido"
3. Sistema verifica cadastro prévio:
   a. Busca por nome + nome da mãe (fuzzy)
   b. Se encontrado → oferecer "Nova Passagem" ou "Continuar Cadastro"
   c. Se não encontrado → criar novo cadastro
4. Cadastro inicial:
   - Nome, data nascimento ou idade aproximada, gênero
   - Sistema calcula faixaEtaria automaticamente
   - Verifica vaga qualificada no abrigo
   - Se SEM VAGA → bloqueia cadastro, alerta gerente e coordenação
5. Criação da Passagem:
   - unidadeId = abrigo atual
   - dataEntrada = now()
   - status = EM_ANDAMENTO
   - motivoEntrada = selecionado
6. Escuta Inicial:
   - Documento técnico obrigatório, tipo = ESCUTA_INICIAL
   - Criado no mesmo dia da entrada
   - Após salvar: status = ATIVO, bloqueado
7. Anexos (se houver documentos físicos):
   - Upload de RG, CPF, certidão (se tiver)
   - Se não tiver: marcar como "pendente" na ficha
8. Acolhido entra na listagem ativa do abrigo
   - status = EM_ACOLHIMENTO
```

**Regras de Negócio:**
- R1.1: Vaga qualificada obrigatória. Abrigo não pode ultrapassar `Vaga.quantidade` para a faixa/gênero.
- R1.2: Escuta inicial deve ser criada em até 24h da entrada.
- R1.3: Se acolhido provisorio permanecer > 7 dias sem escuta inicial, sistema alerta gerente e coordenação.

---

### 4. Fluxo 2 — Entrada por Transferência

**Atores:** Sistema, Gerente destino

```
1. Transferência concluída no abrigo de origem (ver Fluxo 5)
2. Sistema notifica Gerente do abrigo destino
3. Gerente destino visualiza:
   - Prontuário completo do acolhido (acessível imediatamente)
   - Dados da passagem anterior
4. Gerente destino autoriza chegada:
   - Verifica vaga qualificada
   - Confere documentos pendentes
5. Sistema:
   - Encerra passagem na origem (dataSaida, status = ENCERRADA)
   - Cria nova passagem no destino (dataEntrada, status = EM_ANDAMENTO)
   - Atualiza acolhido.unidadeAtualId = destino
   - Atualiza acolhido.status = EM_ACOLHIMENTO
6. Técnico do destino pode criar Escuta Inicial de Recepção (opcional)
```

**Regras de Negócio:**
- R2.1: Prontuário é acessível ao gerente destino ANTES da autorização, para avaliação.
- R2.2: Se gerente destino recusar, transferência volta para `RECUSADA` e acolhido permanece na origem.

---

### 5. Fluxo 3 — Acompanhamento Durante a Estadia

#### 5.1 Documentos Técnicos
```
1. Técnico acessa Prontuário do acolhido
2. Clica em "Novo Documento"
3. Seleciona tipo (Acompanhamento Social, Psicológico, Clínico, Receituário)
4. Preenche conteúdo no editor
5. Salva como RASCUNHO (pode editar depois)
6. Quando finalizado, clica em "Salvar e Bloquear"
   - Sistema registra: autor, data/hora, IP
   - Status muda para ATIVO
   - Documento fica imutável
7. Se detectar erro:
   - Autor original clica "Criar Errata"
   - Sistema cria novo documento com tipo = ERRATA
   - Vincula ao documentoPaiId
   - Preenche motivoErrata e conteúdo corrigido
   - Status do original muda para CORRIGIDO
```

**Regras de Negócio:**
- R3.1: Apenas autor original pode criar errata.
- R3.2: Documentos sigilosos (psicológicos) têm flag `sigiloso = true`.
- R3.3: Documento sigiloso só é legível por: autor, gerente da mesma unidade, coordenação, e psicólogos da mesma unidade.

#### 5.2 Medicações
```
1. Técnico (ou médico externo via técnico) acessa "Prescrever Medicamento"
2. Seleciona medicamento do catálogo (ou cadastra novo se não existir)
3. Define esquema posológico: M + T + N (ex: 1 + 0 + 2)
4. Define tipo de uso: CONTINUO ou TRATAMENTO_ESPECIFICO
5. Se tratamento específico: informa dataFim
6. Salva prescrição
7. Administrativo acessa "Medicações → Separação Semanal"
8. Seleciona período (geralmente próximos 7 dias)
9. Sistema calcula para cada acolhido + prescrição ativa:
   - total = (esquemaManha + esquemaTarde + esquemaNoite) × dias_no_período
   - Se prescrição começar no meio do período: calcular apenas dias válidos
10. Administrativo confirma separação
    - Sistema registra data/hora da confirmação
    - Se estoque ativo: dá baixa automática
```

**Regras de Negócio:**
- R3.4: Prescrição só pode ser editada/excluída se não houver separação semanal confirmada vinculada.
- R3.5: Esquema posológico deve ser inteiro ≥ 0.
- R3.6: Medicamento controlado tem alerta visual laranja no relatório.

#### 5.3 Agendamentos
```
1. Administrativo acessa "Agendamentos → Novo"
2. Seleciona acolhido, tipo, data, horário, local, profissional externo
3. Salva (status = AGENDADO)
4. No login do administrativo no dia anterior:
   - Sistema exibe banner/alerta: "Amanhã há X agendamentos"
   - Lista detalhada com preparativos sugeridos
5. Acolhido vai à consulta
6. Técnico acessa o agendamento e clica "Registrar Retorno"
7. Sistema cria Documento técnico vinculado ao agendamento
   - tipo = ACOMPANHAMENTO_CLINICO (ou outro adequado)
   - agendamento.status = REALIZADO
```

#### 5.4 Ocorrências (Educador/Cuidador)
```
1. Educador/Cuidador acessa "Ocorrências → Nova"
2. Seleciona acolhido(s) envolvido(s)
3. Informa tipo, gravidade, descrição, medidas tomadas
4. Anexa foto se necessário
5. Salva
6. Se gravidade = GRAVE:
   - Notificação automática ao gerente (badge no dashboard)
   - E-mail interno (se configurado futuramente)
```

---

### 6. Fluxo 4 — Transferência de Acolhido

```
1. Técnico cria Documento técnico tipo = TRANSFERENCIA
   - Justificativa, abrigo destino sugerido
2. Sistema cria registro em TransferenciaAcolhido
   - status = PENDENTE_ORIGEM
   - acolhido.status = EM_TRANSFERENCIA
   - Bloqueia novos documentos no prontuário (exceto erratas)
3. Notifica Gerente da unidade de origem
4. Gerente origem autoriza saída:
   - status = PENDENTE_DESTINO
   - Verifica se há vaga qualificada no destino
5. Notifica Gerente da unidade destino
6. Gerente destino:
   a. AUTORIZA CHEGADA:
      - status = CONCLUIDA
      - Executa Fluxo 2 (Entrada por Transferência)
   b. RECUSA:
      - status = RECUSADA
      - acolhido.status = EM_ACOLHIMENTO (retorna à origem)
      - Técnico é notificado para providenciar outro destino
```

**Regras de Negócio:**
- R4.1: Dupla autorização obrigatória.
- R4.2: Durante transferência, prontuário é legível mas não editável (exceto errata).
- R4.3: Se recusada, motivo da recusa é obrigatório.

---

### 7. Fluxo 5 — Desligamento de Acolhido

```
1. Técnico cria Documento técnico tipo = DESLIGAMENTO
   - Motivo, parecer técnico, encaminhamentos futuros
2. Sistema cria registro de desligamento vinculado à passagem ativa
3. Técnico submete para aprovação do Gerente
4. Gerente revisa e:
   a. AUTORIZA:
      - Passagem.status = ENCERRADA
      - Passagem.dataSaida = now()
      - Acolhido.status = DESLIGADO
      - Acolhido.unidadeAtualId = null
      - Acolhido sai da listagem ativa
   b. RECUSA:
      - Documento volta para RASCUNHO
      - Técnico deve ajustar e submeter novamente
```

**Regras de Negócio:**
- R5.1: Desligamento só pode ser autorizado por gerente da unidade atual.
- R5.2: Após desligamento, prontuário permanece acessível para consulta em toda a rede.
- R5.3: Motivos de desligamento padronizados: reintegração familiar, adoção, maioridade, óbito, evasão, transferência para outra rede.

---

### 8. Fluxo 6 — Cadastro e Movimentação de Funcionário

#### 6.1 Cadastro
```
1. Gerente acessa "Equipe → Novo Colaborador"
2. Preenche: nome, e-mail, matrícula, perfil, cargo
3. Sistema gera senha temporária
4. Funcionário vinculado à unidade do gerente
5. Funcionário faz primeiro login e redefine senha
```

#### 6.2 Transferência
```
1. Gerente origem inicia transferência
   - Seleciona funcionário e unidade destino
2. Sistema cria TransferenciaFuncionario
   - status = PENDENTE_DESTINO
   - Funcionário.status = EM_TRANSFERENCIA
3. Notifica Gerente destino e Coordenação
4. Gerente destino autoriza:
   - Atualiza funcionário.unidadeId = destino
   - Funcionário.status = ATIVO
   - Coordenação notificada da conclusão
```

#### 6.3 Desligamento
```
1. Gerente acessa "Equipe → Desligar"
2. Informa motivo e data
3. Sistema:
   - Funcionário.status = DESLIGADO
   - Acesso revogado imediatamente
   - Documentos autoriais preservados
```

---

### 9. Fluxo 7 — Ocorrência Grave e Escalonamento

```
1. Educador/Cuidador registra ocorrência com gravidade = GRAVE
2. Sistema:
   - Cria notificação para Gerente (badge no dashboard)
   - Marca ocorrência como "não lida" pelo gerente
3. Gerente acessa ocorrência:
   - Pode adicionar providências administrativas
   - Marca como "lida/encaminhada"
4. Se necessário, Gerente cita ocorrência em documento técnico
```

---

### 10. Resumo dos Fluxos

| Fluxo | Quem inicia | Quem autoriza | Estados afetados | Notifica |
|-------|-------------|---------------|------------------|----------|
| Entrada novo acolhido | Técnico/Gerente | — | PROVISORIO → EM_ACOLHIMENTO | Admin (preparativos) |
| Entrada por transferência | Sistema | Gerente destino | EM_TRANSFERENCIA → EM_ACOLHIMENTO | Gerente destino |
| Acompanhamento | Técnico/Admin | — | — | — |
| Transferência acolhido | Técnico | Gerente origem + destino | EM_ACOLHIMENTO → EM_TRANSFERENCIA → EM_ACOLHIMENTO | Gerentes |
| Desligamento | Técnico | Gerente | EM_ACOLHIMENTO → DESLIGADO | — |
| Cadastro funcionário | Gerente | — | ATIVO | Funcionário |
| Transferência funcionário | Gerente origem | Gerente destino | ATIVO → EM_TRANSFERENCIA → ATIVO | Coordenação |
| Desligamento funcionário | Gerente | — | DESLIGADO | — |
| Ocorrência grave | Educador/Cuidador | — | — | Gerente |
