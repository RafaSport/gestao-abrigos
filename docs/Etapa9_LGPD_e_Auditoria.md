# Sistema de Gestão de Abrigos (SGA)

Rede de Acolhimento — Prefeitura do Recife

## Etapa 9 — LGPD e Auditoria (v1.0)

---

### 1. Introdução

Este documento define as medidas de proteção de dados pessoais e o sistema de auditoria do SGA, em conformidade com a Lei Geral de Proteção de Dados (Lei 13.709/2018).

---

### 2. Classificação dos Dados

| Categoria | Dados | Nível de Sensibilidade |
|-----------|-------|------------------------|
| **Básicos** | Nome, idade, gênero, foto | Médio |
| **Identificação** | CPF, RG, certidão, naturalidade | Alto |
| **Filiação** | Nome da mãe, nome do pai | Médio |
| **Saúde** | Prescrições, diagnósticos, receituários | Alto (dados sensíveis LGPD) |
| **Psicológico** | Acompanhamento psicológico, PIA | Alto (sigilo profissional) |
| **Ocorrências** | Incidentes, comportamentos de risco | Alto |
| **Funcionários** | E-mail, senha, matrícula | Médio |

---

### 3. Medidas de Segurança

#### 3.1 Em Repouso
- Banco de dados PostgreSQL na Neon com SSL obrigatório.
- Campos de senha: hash bcrypt (nunca plaintext).
- Campos sensíveis (CPF, RG) podem ser criptografados com AES-256 futuramente (não obrigatório no MVP, mas arquitetura deve permitir).

#### 3.2 Em Trânsito
- TLS 1.3 obrigatório (Vercel fornece automaticamente).
- Cookies de sessão: `secure`, `httpOnly`, `sameSite = lax`.

#### 3.3 Acesso
- Autenticação obrigatória para todas as rotas exceto `/login`.
- Middleware verifica sessão em cada requisição.
- Server Actions verificam `session.user.perfil` antes de executar.

---

### 4. Sistema de Auditoria (LogAuditoria)

#### 4.1 Eventos Obrigatórios
Todo evento abaixo gera registro automático em `LogAuditoria`:

| Evento | Entidade | Detalhes |
|--------|----------|----------|
| Login bem-sucedido | Funcionario | e-mail, IP, userAgent |
| Login falho | — | e-mail tentado, IP |
| Logout | Funcionario | — |
| Visualizou prontuário | Acolhido | acolhidoId, motivo implícito |
| Criou documento | Documento | tipo, acolhidoId |
| Editou rascunho | Documento | documentoId |
| Bloqueou documento | Documento | documentoId, IP |
| Criou errata | Documento | documentoPaiId, motivo |
| Cadastrou acolhido | Acolhido | nome, unidadeId |
| Transferiu acolhido | TransferenciaAcolhido | origem, destino |
| Desligou acolhido | Acolhido | motivo |
| Prescreveu medicamento | Prescricao | medicamento, acolhidoId |
| Gerou separação | SeparacaoSemanal | período, itens |
| Registrou ocorrência | Ocorrencia | tipo, gravidade |
| Visualizou documento sigiloso | Documento | documentoId, justificativa se AS |

#### 4.2 Retenção
- Logs de auditoria: mínimo 5 anos (conforme lei de arquivos públicos).
- Após 5 anos: anonimizar `funcionarioId` e `acolhidoId` (manter apenas ações agregadas).

#### 4.3 Consulta
- Apenas COORDENACAO pode consultar logs de auditoria.
- Interface futura: tela `/auditoria` com filtros por data, funcionário, ação.

---

### 5. Consentimento e Titularidade

- Acolhidos em abrigo são pessoas hipossuficientes (crianças, adolescentes, vulneráveis).
- **Responsável legal:** Prefeitura do Recife, através da Secretaria de Assistência Social.
- **Operador:** equipe técnica dos abrigos (funcionários com perfil adequado).
- O sistema deve registrar o responsável técnico pelo acolhido (gerente da unidade) como "encarregado" dos dados daquele acolhido.

#### 5.1 Direitos do Titular (futuro)
- Implementar rotas para atender requisições LGPD:
  - Confirmação de existência de dados
  - Acesso aos dados (prontuário já cobre)
  - Correção (via errata para documentos; via edição para cadastro)
  - Anonimização (após desligamento + prazo)
  - Portabilidade (exportação do prontuário em PDF)
  - Eliminação (não aplicável a dados de menores em abrigo — obrigação legal de guarda)

---

### 6. Sigilo Profissional

- **Psicólogos:** documentos psicológicos têm sigilo profissional (CFP/CFP).
- **Assistentes Sociais:** documentos sociais têm sigilo técnico (CFESS).
- O sistema NÃO permite que gerentes ou coordenação editem documentos técnicos.
- A visualização de documentos sigilosos por outros profissionais (ex: AS vendo doc psicológico) deve ser justificada e logada.

---

### 7. Anonimização Futura

- Após 5 anos do desligamento do acolhido:
  - `nome` → hash ou "ANON_12345"
  - `cpf`, `rg` → removidos ou cifrados irreversivelmente
  - `fotoUrl` → removido
  - Documentos técnicos mantidos para estatísticas, mas sem identificação direta
- Implementação: job agendado (cron) ou trigger anual.
