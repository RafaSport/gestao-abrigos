# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 4 — Modelagem de Dados Completa (v2.0)

---

### 1. Introdução

Este documento contém o schema Prisma completo, o diagrama entidade-relacionamento textual, as enums, índices e a estratégia de seeds.

---

### 2. Enums Globais

```prisma
enum PerfilUsuario {
  COORDENACAO
  GERENTE
  ASSISTENTE_SOCIAL
  PSICOLOGO
  ADMINISTRATIVO
  EDUCADOR_SOCIAL
  CUIDADOR
}

enum StatusAcolhido {
  PROVISORIO        // chegou hoje, cadastro incompleto
  EM_ACOLHIMENTO    // passagem ativa
  EM_TRANSFERENCIA  // em processo de mudança de abrigo
  SAIDA_TEMPORARIA  // passagem não encerrada, acolhido fora temporariamente
  DESLIGADO         // passagem encerrada
}

enum StatusDocumento {
  RASCUNHO    // pode editar
  ATIVO       // salvo, bloqueado
  CORRIGIDO   // possui errata vinculada
}

enum TipoDocumento {
  ESCUTA_INICIAL
  ACOMPANHAMENTO_SOCIAL
  ACOMPANHAMENTO_PSICOLOGICO
  ACOMPANHAMENTO_CLINICO
  RECEITUARIO
  DESLIGAMENTO
  TRANSFERENCIA
  ERRATA
  PIA
}

enum TipoUsoMedicamento {
  CONTINUO
  TRATAMENTO_ESPECIFICO
}

enum StatusAgendamento {
  AGENDADO
  REALIZADO
  CANCELADO
  NAO_COMPARECEU
}

enum TipoAgendamento {
  MEDICO
  DENTISTA
  PSICOLOGO_EXTERNO
  AUDIENCIA_JUDICIAL
  RETIRADA_DOCUMENTOS
  OUTRO
}

enum StatusPassagem {
  EM_ANDAMENTO
  ENCERRADA
}

enum Genero {
  MASCULINO
  FEMININO
  NAO_BINARIO
  NAO_INFORMADO
}

enum FaixaEtaria {
  CRIANCA_0_12
  ADOLESCENTE_12_18
  ADULTO_18_60
  IDOSO_60_PLUS
  FAMILIA
}

enum TipoOcorrencia {
  INCIDENTE
  COMPORTAMENTO_RISCO
  FUGA
  RETORNO
  VISITA
  ACIDENTE
  OUTRO
}

enum GravidadeOcorrencia {
  LEVE
  MODERADA
  GRAVE
}

enum StatusTransferencia {
  PENDENTE_ORIGEM
  PENDENTE_DESTINO
  AUTORIZADA
  RECUSADA
  CONCLUIDA
}

enum StatusFuncionario {
  ATIVO
  EM_TRANSFERENCIA
  DESLIGADO
}
```

---

### 3. Schema Prisma Completo

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================================
// UNIDADE (Abrigo)
// ============================================================
model Unidade {
  id            String   @id @default(uuid())
  nome          String
  perfilAtendimento FaixaEtaria
  generoAtendimento Genero    // público-alvo: MASCULINO, FEMININO ou ambos (FAMILIA)
  endereco      String?
  telefone      String?
  email         String?
  capacidadeTotal Int
  status        Boolean  @default(true) // true = ativo
  criadoEm      DateTime @default(now())
  atualizadoEm  DateTime @updatedAt

  // Relacionamentos
  funcionarios  Funcionario[]
  acolhidos     Acolhido[]    // acolhidos ATIVOS nesta unidade
  passagens     Passagem[]
  agendamentos  Agendamento[]
  ocorrencias   Ocorrencia[]
  prescricoes   Prescricao[]
  separacoes    SeparacaoSemanal[]
  transferenciasOrigem TransferenciaAcolhido[] @relation("TransferenciaOrigem")
  transferenciasDestino TransferenciaAcolhido[] @relation("TransferenciaDestino")
  vagas         Vaga[]
  estoque       EstoqueMedicamento[]
  documentos    Documento[]

  @@index([status])
  @@index([perfilAtendimento, generoAtendimento])
}

// ============================================================
// VAGA QUALIFICADA
// ============================================================
model Vaga {
  id            String   @id @default(uuid())
  unidadeId     String
  faixaEtaria   FaixaEtaria
  genero        Genero
  quantidade    Int
  ocupadas      Int      @default(0)

  unidade       Unidade  @relation(fields: [unidadeId], references: [id], onDelete: Cascade)

  @@unique([unidadeId, faixaEtaria, genero])
}

// ============================================================
// FUNCIONÁRIO / USUÁRIO
// ============================================================
model Funcionario {
  id            String   @id @default(uuid())
  nome          String
  matricula     String?  @unique
  email         String   @unique
  senhaHash     String   // bcrypt
  perfil        PerfilUsuario
  cargo         String?
  telefone      String?
  unidadeId     String?  // null apenas para COORDENACAO
  status        StatusFuncionario @default(ATIVO)
  ultimoAcesso  DateTime?
  criadoEm      DateTime @default(now())
  atualizadoEm  DateTime @updatedAt

  // Relacionamentos
  unidade       Unidade? @relation(fields: [unidadeId], references: [id])
  documentosAutor Documento[] @relation("DocumentoAutor")
  erratasAutor  Documento[] @relation("ErrataAutor")
  prescricoes   Prescricao[]
  agendamentosCriados Agendamento[]
  separacoes    SeparacaoSemanal[]
  transferenciasFuncionarioOrigem TransferenciaFuncionario[] @relation("FuncionarioTransferenciaOrigem")
  transferenciasFuncionarioDestino TransferenciaFuncionario[] @relation("FuncionarioTransferenciaDestino")
  ocorrencias   Ocorrencia[]
  logsAuditoria LogAuditoria[]
  arquivos      ArquivoDigitalizado[]

  @@index([email])
  @@index([unidadeId, perfil])
}

// ============================================================
// ACOLHIDO
// ============================================================
model Acolhido {
  id                  String   @id @default(uuid())
  nome                String
  nomeSocial          String?
  dataNascimento      DateTime?
  idadeAproximada     Int?     // usado quando dataNascimento é desconhecida
  genero              Genero
  generoNascimento    Genero?  // para documentação oficial
  faixaEtaria         FaixaEtaria // calculado automaticamente
  nomeMae             String?
  nomePai             String?
  cpf                 String?
  rg                  String?
  orgaoEmissor        String?
  naturalidade        String?
  nacionalidade       String   @default("Brasileira")
  fotoUrl             String?
  observacoesEntrada  String?  @db.Text
  status              StatusAcolhido @default(PROVISORIO)
  unidadeAtualId      String?
  criadoEm            DateTime @default(now())
  atualizadoEm        DateTime @updatedAt

  // Relacionamentos
  unidadeAtual        Unidade? @relation(fields: [unidadeAtualId], references: [id])
  passagens           Passagem[]
  documentos          Documento[]
  arquivos            ArquivoDigitalizado[]
  prescricoes         Prescricao[]
  agendamentos        Agendamento[]
  ocorrencias         Ocorrencia[]
  transferenciasOrigem TransferenciaAcolhido[] @relation("AcolhidoTransferenciaOrigem")
  transferenciasDestino TransferenciaAcolhido[] @relation("AcolhidoTransferenciaDestino")

  @@index([nome])
  @@index([cpf])
  @@index([status])
  @@index([unidadeAtualId])
}

// ============================================================
// PASSAGEM (Estadia em um abrigo)
// ============================================================
model Passagem {
  id            String   @id @default(uuid())
  acolhidoId    String
  unidadeId     String
  dataEntrada   DateTime @default(now())
  dataSaida     DateTime?
  motivoEntrada String   // rua, encaminhamento, transferencia, outros
  motivoSaida   String?  // reintegracao, adocao, maioridade, obito, evasao, transferencia
  status        StatusPassagem @default(EM_ANDAMENTO)
  observacoes   String?  @db.Text

  acolhido      Acolhido @relation(fields: [acolhidoId], references: [id])
  unidade       Unidade  @relation(fields: [unidadeId], references: [id])

  @@index([acolhidoId, status])
  @@index([unidadeId, status])
}

// ============================================================
// DOCUMENTO TÉCNICO
// ============================================================
model Documento {
  id              String   @id @default(uuid())
  acolhidoId      String
  tipo            TipoDocumento
  titulo          String
  conteudo        String   @db.Text
  status          StatusDocumento @default(RASCUNHO)
  sigiloso        Boolean  @default(false)

  // Autoria
  autorId         String
  unidadeId       String
  criadoEm        DateTime @default(now())
  atualizadoEm    DateTime @updatedAt
  ipAddress       String?

  // Errata
  documentoPaiId  String?  // se este documento for uma errata
  motivoErrata    String?  @db.Text

  // Relacionamentos
  acolhido        Acolhido @relation(fields: [acolhidoId], references: [id])
  autor           Funcionario @relation("DocumentoAutor", fields: [autorId], references: [id])
  unidade         Unidade  @relation(fields: [unidadeId], references: [id])
  documentoPai    Documento? @relation("DocumentoErrata", fields: [documentoPaiId], references: [id])
  erratas         Documento[] @relation("DocumentoErrata")
  agendamento     Agendamento? @relation(fields: [agendamentoId], references: [id])
  agendamentoId   String?

  @@index([acolhidoId, tipo])
  @@index([autorId])
  @@index([unidadeId])
  @@index([status])
  @@index([criadoEm])
}

// ============================================================
// ARQUIVO DIGITALIZADO
// ============================================================
model ArquivoDigitalizado {
  id            String   @id @default(uuid())
  acolhidoId    String
  tipo          String   // RG, CPF, CERTIDAO, RECEITA, LAUDO, OUTRO
  descricao     String?
  url           String   // URL do storage (Cloudinary/UploadThing)
  tamanhoBytes  Int
  mimeType      String
  enviadoPorId  String
  criadoEm      DateTime @default(now())

  acolhido      Acolhido @relation(fields: [acolhidoId], references: [id])
  enviadoPor    Funcionario @relation(fields: [enviadoPorId], references: [id])

  @@index([acolhidoId, tipo])
}

// ============================================================
// MEDICAMENTO (Catálogo)
// ============================================================
model Medicamento {
  id              String   @id @default(uuid())
  nomeComercial   String
  principioAtivo  String
  apresentacao    String   // ex: "500mg", "2mg/ml"
  formaFarmaceutica String // comprimido, xarope, injetável...
  controlado      Boolean  @default(false)
  observacoes     String?
  ativo           Boolean  @default(true)

  prescricoes     Prescricao[]
  estoques        EstoqueMedicamento[]

  @@index([nomeComercial])
  @@index([principioAtivo])
}

// ============================================================
// PRESCRIÇÃO
// ============================================================
model Prescricao {
  id              String   @id @default(uuid())
  acolhidoId      String
  medicamentoId   String
  prescritorId    String   // quem registrou no sistema
  esquemaManha    Int      @default(0)
  esquemaTarde    Int      @default(0)
  esquemaNoite    Int      @default(0)
  tipoUso         TipoUsoMedicamento
  dataInicio      DateTime @default(now())
  dataFim         DateTime? // null se contínuo
  observacoes     String?
  ativa           Boolean  @default(true)
  criadoEm        DateTime @default(now())
  atualizadoEm    DateTime @updatedAt

  acolhido        Acolhido @relation(fields: [acolhidoId], references: [id])
  medicamento     Medicamento @relation(fields: [medicamentoId], references: [id])
  prescritor      Funcionario @relation(fields: [prescritorId], references: [id])
  itensSeparacao  ItemSeparacao[]
  unidadeId       String?
  unidade         Unidade? @relation(fields: [unidadeId], references: [id])

  @@index([acolhidoId, ativa])
  @@index([medicamentoId])
}

// ============================================================
// ESTOQUE DE MEDICAMENTOS (por unidade)
// ============================================================
model EstoqueMedicamento {
  id            String   @id @default(uuid())
  unidadeId     String
  medicamentoId String
  lote          String?
  validade      DateTime?
  quantidade    Int      @default(0)
  quantidadeMinima Int   @default(10)
  criadoEm      DateTime @default(now())
  atualizadoEm  DateTime @updatedAt

  unidade       Unidade  @relation(fields: [unidadeId], references: [id])
  medicamento   Medicamento @relation(fields: [medicamentoId], references: [id])

  @@unique([unidadeId, medicamentoId, lote])
  @@index([validade])
}

// ============================================================
// SEPARAÇÃO SEMANAL
// ============================================================
model SeparacaoSemanal {
  id            String   @id @default(uuid())
  unidadeId     String
  dataInicio    DateTime
  dataFim       DateTime
  geradoPorId   String
  confirmadoEm  DateTime?
  observacoes   String?
  criadoEm      DateTime @default(now())

  unidade       Unidade  @relation(fields: [unidadeId], references: [id])
  geradoPor     Funcionario @relation(fields: [geradoPorId], references: [id])
  itens         ItemSeparacao[]

  @@index([unidadeId, dataInicio])
}

// ============================================================
// ITEM DE SEPARAÇÃO
// ============================================================
model ItemSeparacao {
  id              String   @id @default(uuid())
  separacaoId     String
  prescricaoId    String
  acolhidoId      String
  medicamentoId   String
  quantidade      Int      // calculado: (M+T+N) * dias
  confirmado      Boolean  @default(false)

  separacao       SeparacaoSemanal @relation(fields: [separacaoId], references: [id], onDelete: Cascade)
  prescricao      Prescricao @relation(fields: [prescricaoId], references: [id])

  @@index([separacaoId])
}

// ============================================================
// AGENDAMENTO
// ============================================================
model Agendamento {
  id              String   @id @default(uuid())
  acolhidoId      String
  unidadeId       String
  tipo            TipoAgendamento
  data            DateTime
  horario         String?  // "14:30"
  local           String
  profissional    String?  // nome do médico/órgão externo
  observacoes     String?
  status          StatusAgendamento @default(AGENDADO)
  criadoPorId     String
  criadoEm        DateTime @default(now())
  atualizadoEm    DateTime @updatedAt

  acolhido        Acolhido @relation(fields: [acolhidoId], references: [id])
  unidade         Unidade  @relation(fields: [unidadeId], references: [id])
  criadoPor       Funcionario @relation(fields: [criadoPorId], references: [id])
  documentoRetorno Documento[]

  @@index([unidadeId, data])
  @@index([acolhidoId, status])
}

// ============================================================
// TRANSFERÊNCIA DE ACOLHIDO
// ============================================================
model TransferenciaAcolhido {
  id                  String   @id @default(uuid())
  acolhidoId          String
  unidadeOrigemId     String
  unidadeDestinoId    String
  documentoTransferenciaId String? // vinculo ao documento técnico
  status              StatusTransferencia @default(PENDENTE_ORIGEM)
  motivo              String
  autorizadoOrigemPorId String?
  autorizadoOrigemEm  DateTime?
  autorizadoDestinoPorId String?
  autorizadoDestinoEm DateTime?
  observacoes         String?
  criadoEm            DateTime @default(now())
  atualizadoEm        DateTime @updatedAt

  acolhido            Acolhido @relation("AcolhidoTransferenciaOrigem", fields: [acolhidoId], references: [id])
  unidadeOrigem       Unidade  @relation("TransferenciaOrigem", fields: [unidadeOrigemId], references: [id])
  unidadeDestino      Unidade  @relation("TransferenciaDestino", fields: [unidadeDestinoId], references: [id])

  @@index([acolhidoId, status])
  @@index([unidadeOrigemId, status])
  @@index([unidadeDestinoId, status])
}

// ============================================================
// TRANSFERÊNCIA DE FUNCIONÁRIO
// ============================================================
model TransferenciaFuncionario {
  id                  String   @id @default(uuid())
  funcionarioId       String
  unidadeOrigemId     String
  unidadeDestinoId    String
  status              StatusTransferencia @default(PENDENTE_DESTINO)
  autorizadoDestinoPorId String?
  autorizadoDestinoEm DateTime?
  observacoes         String?
  criadoEm            DateTime @default(now())

  funcionario         Funcionario @relation("FuncionarioTransferenciaOrigem", fields: [funcionarioId], references: [id])
  unidadeDestino      Funcionario @relation("FuncionarioTransferenciaDestino", fields: [unidadeDestinoId], references: [id])
  // unidadeOrigem não precisa de relação direta pois está no funcionario

  @@index([funcionarioId, status])
}

// ============================================================
// MODELO DE DOCUMENTO INSTRUMENTAL
// ============================================================
model ModeloDocumentoInstrumental {
  id            String   @id @default(uuid())
  nome          String
  conteudo      String   @db.Text // template com placeholders {{nome}}, etc.
  ativo         Boolean  @default(true)
  criadoPorId   String
  criadoEm      DateTime @default(now())
  atualizadoEm  DateTime @updatedAt

  criadoPor     Funcionario @relation(fields: [criadoPorId], references: [id])
}

// ============================================================
// OCORRÊNCIA
// ============================================================
model Ocorrencia {
  id            String   @id @default(uuid())
  acolhidoId    String?
  unidadeId     String
  tipo          TipoOcorrencia
  gravidade     GravidadeOcorrencia
  descricao     String   @db.Text
  medidasTomadas String? @db.Text
  dataHora      DateTime @default(now())
  registradoPorId String
  anexoUrl      String?
  criadoEm      DateTime @default(now())

  acolhido      Acolhido? @relation(fields: [acolhidoId], references: [id])
  unidade       Unidade  @relation(fields: [unidadeId], references: [id])
  registradoPor Funcionario @relation(fields: [registradoPorId], references: [id])

  @@index([unidadeId, dataHora])
  @@index([acolhidoId])
  @@index([tipo, gravidade])
}

// ============================================================
// LOG DE AUDITORIA (LGPD)
// ============================================================
model LogAuditoria {
  id            String   @id @default(uuid())
  funcionarioId String?
  acolhidoId    String?
  acao          String   // ex: "visualizou_prontuario", "criou_documento"
  entidade      String   // nome da tabela afetada
  entidadeId    String?  // ID do registro afetado
  detalhes      String?  @db.Text // JSON com dados relevantes
  ipAddress     String?
  userAgent     String?
  criadoEm      DateTime @default(now())

  funcionario   Funcionario? @relation(fields: [funcionarioId], references: [id])

  @@index([funcionarioId, criadoEm])
  @@index([acao])
  @@index([acolhidoId])
  @@index([criadoEm])
}
```

---

### 4. Diagrama Entidade-Relacionamento (Resumo)

```
[Unidade] 1 ──── N [Funcionario]
[Unidade] 1 ──── N [Acolhido] (unidadeAtual)
[Unidade] 1 ──── N [Passagem]
[Unidade] 1 ──── N [Vaga]
[Unidade] 1 ──── N [EstoqueMedicamento]
[Unidade] 1 ──── N [SeparacaoSemanal]
[Unidade] 1 ──── N [Agendamento]
[Unidade] 1 ──── N [Ocorrencia]
[Unidade] 1 ──── N [Documento]

[Acolhido] 1 ──── N [Passagem]
[Acolhido] 1 ──── N [Documento]
[Acolhido] 1 ──── N [ArquivoDigitalizado]
[Acolhido] 1 ──── N [Prescricao]
[Acolhido] 1 ──── N [Agendamento]
[Acolhido] 1 ──── N [Ocorrencia]
[Acolhido] 1 ──── N [TransferenciaAcolhido]

[Funcionario] 1 ──── N [Documento] (autor)
[Funcionario] 1 ──── N [Prescricao] (prescritor)
[Funcionario] 1 ──── N [Agendamento] (criador)
[Funcionario] 1 ──── N [SeparacaoSemanal] (gerador)
[Funcionario] 1 ──── N [Ocorrencia] (registrador)
[Funcionario] 1 ──── N [LogAuditoria]

[Documento] 1 ──── N [Documento] (erratas, auto-relacionamento)
[Documento] 1 ──── 1 [Agendamento] (relatório de retorno)

[Medicamento] 1 ──── N [Prescricao]
[Medicamento] 1 ──── N [EstoqueMedicamento]

[Prescricao] 1 ──── N [ItemSeparacao]
[SeparacaoSemanal] 1 ──── N [ItemSeparacao]

[TransferenciaAcolhido] N ──── 1 [Acolhido]
[TransferenciaAcolhido] N ──── 1 [Unidade] (origem)
[TransferenciaAcolhido] N ──── 1 [Unidade] (destino)
```

---

### 5. Seeds Iniciais

```typescript
// prisma/seed.ts

const unidades = [
  {
    nome: "Abrigo Masculino Esperança",
    perfilAtendimento: "ADULTO_18_60",
    generoAtendimento: "MASCULINO",
    capacidadeTotal: 40,
    endereco: "Rua da Esperança, 100, Recife-PE"
  },
  {
    nome: "Abrigo Feminino Renascimento",
    perfilAtendimento: "ADULTO_18_60",
    generoAtendimento: "FEMININO",
    capacidadeTotal: 30,
    endereco: "Rua do Renascimento, 200, Recife-PE"
  }
];

const funcionarios = [
  {
    nome: "Coordenador Geral",
    email: "coordenacao@recife.gov.br",
    perfil: "COORDENACAO",
    senha: "@Temp1234" // deve ser hasheada no seed
  },
  {
    nome: "Gerente Esperança",
    email: "gerente.esperanca@recife.gov.br",
    perfil: "GERENTE",
    unidadeId: "<id_esperanca>",
    senha: "@Temp1234"
  },
  {
    nome: "Assistente Social Esperança",
    email: "as.esperanca@recife.gov.br",
    perfil: "ASSISTENTE_SOCIAL",
    unidadeId: "<id_esperanca>",
    senha: "@Temp1234"
  },
  {
    nome: "Psicólogo Esperança",
    email: "psicologo.esperanca@recife.gov.br",
    perfil: "PSICOLOGO",
    unidadeId: "<id_esperanca>",
    senha: "@Temp1234"
  },
  {
    nome: "Administrativo Esperança",
    email: "admin.esperanca@recife.gov.br",
    perfil: "ADMINISTRATIVO",
    unidadeId: "<id_esperanca>",
    senha: "@Temp1234"
  },
  {
    nome: "Educador Esperança",
    email: "educador.esperanca@recife.gov.br",
    perfil: "EDUCADOR_SOCIAL",
    unidadeId: "<id_esperanca>",
    senha: "@Temp1234"
  },
  {
    nome: "Gerente Renascimento",
    email: "gerente.renascimento@recife.gov.br",
    perfil: "GERENTE",
    unidadeId: "<id_renascimento>",
    senha: "@Temp1234"
  }
];
```

---

### 6. Considerações de Performance

- Índices compostos em `[unidadeId, status]` para todas as listagens por abrigo.
- Índice em `[acolhidoId, status]` para prontuário e passagens.
- Índice em `[criadoEm]` decrescente para "documentos recentes" e "últimas ocorrências".
- Particionamento futuro de `LogAuditoria` por mês (se volume crescer).
- Soft delete em todas as tabelas principais via `status` ou `deletedAt`.
