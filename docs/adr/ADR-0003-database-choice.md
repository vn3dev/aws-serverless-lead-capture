[🇧🇷 Versão em Português](#versao-em-portugues)

# ADR-0003 — Database Strategy (Amazon DynamoDB)

[2026-02-19] - by Vinicius Costa  

## Context

The system needs to persist user-submitted lead data, including:

- Name
- Email
- Submission timestamp
- Expiration timestamp (TTL)

The expected access pattern is simple:

- Write-heavy workload
- Occasional reads for administrative or operational purposes
- No complex relational queries
- No joins required
- No transactional multi-entity consistency required

Non-functional requirements include:

- Automatic horizontal scalability
- High availability
- Low operational overhead
- Serverless compatibility
- Pay-per-use pricing model

## Decision

Use Amazon DynamoDB in PAY_PER_REQUEST billing mode

The table design follows:

- Primary key: `id` (unique identifier / dedupe key)
- TTL attribute: `expiresAt` (90-day retention policy)

The database is accessed directly from AWS Lambda using IAM-based authentication

## Alternatives Considered

### 1. Amazon RDS PostgreSQL / MySQL

Would provide:

- Relational structure
- SQL querying
- Transactional guarantees

Rejected because:

- More expensive for low-volume workloads
- Overkill for simple write-heavy form submissions
- Introduces unnecessary relational complexity

### 2. Storing leads only via SES email

Rejected because:

- Email is not a reliable data storage system
- No structured querying capability
- Poor observability and analytics capability

## Rationale

DynamoDB provides:

- Fully managed, serverless database
- Automatic scaling
- High availability
- Low-latency performance
- Tight integration with Lambda
- Native TTL support for data retention

The workload pattern (simple writes, no joins, minimal querying) aligns naturally with a NoSQL key-value model

Using PAY_PER_REQUEST ensures cost efficiency under variable traffic

## Security Considerations

- IAM least-privilege access (dynamodb:PutItem restricted to table ARN)
- No public database endpoint exposure
- Encryption at rest enabled by default (KMS)
- Data retention controlled via TTL (90 days)

## Consequences

### Positive

- No database server management
- Automatic scaling
- Cost-efficient for sporadic traffic
- Strong integration with serverless architecture
- Built-in TTL simplifies data retention strategy

### Negative

- Limited querying flexibility
- No relational joins
- Schema-less design requires disciplined validation
- Data modeling must anticipate access patterns upfront

## Future Improvements

- Redesign table with composite key (PK/SK) for more flexible querying
- Add GSI for administrative queries (e.g., email-based lookup)
- Implement per-email rate limiting using partition key strategy
- Encrypt specific sensitive attributes with customer-managed KMS
- Add backup and restore strategy documentation

---

# 🇧🇷 Versão em Português

# ADR-0003 — Estratégia de Banco de Dados (Amazon DynamoDB)

[19-02-2026] - por Vinicius Costa  

## Contexto

O sistema precisa persistir dados de leads submetidos pelos usuários, incluindo:

- Nome
- Email
- Timestamp da submissão
- Timestamp de expiração (TTL)

O padrão de acesso esperado é simples:

- Carga predominantemente de escrita
- Leituras ocasionais para fins administrativos ou operacionais
- Ausência de consultas relacionais complexas
- Sem necessidade de joins
- Sem necessidade de transações envolvendo múltiplas entidades

Os requisitos não funcionais incluem:

- Escalabilidade horizontal automática
- Alta disponibilidade
- Baixo overhead operacional
- Compatibilidade com arquitetura serverless
- Modelo de custo pay-per-use

## Decisão

Utilizar o Amazon DynamoDB no modo de cobrança PAY_PER_REQUEST

O design inicial da tabela segue:

- Chave primária: `id` (identificador único / chave de deduplicação)
- Atributo TTL: `expiresAt` (política de retenção de 90 dias)

O banco é acessado diretamente pelo AWS Lambda utilizando autenticação baseada em IAM

## Alternativas Consideradas

### 1. Amazon RDS: PostgreSQL / MySQL

Ofereceria:

- Estrutura relacional
- Consultas SQL
- Garantias transacionais

Rejeitado porque:

- Custo superior para workloads de baixo volume
- Complexidade relacional desnecessária para submissões simples
- Não há necessidade de joins ou modelagem relacional

### 2. Armazenar leads apenas via e-mail (SES)

Rejeitado porque:

- E-mail não é sistema confiável de armazenamento estruturado
- Não permite consultas estruturadas
- Observabilidade e análise de dados limitadas

## Justificativa

O DynamoDB oferece:

- Banco de dados totalmente gerenciado e serverless
- Escalabilidade automática
- Alta disponibilidade
- Baixa latência
- Integração nativa com Lambda
- Suporte nativo a TTL para retenção automática

O padrão de workload de escritas simples, sem joins, consultas mínimas se alinha ao modelo NoSQL key-value

O uso de PAY_PER_REQUEST garante eficiência de custo em cenários de tráfego variável

## Considerações de Segurança

- Acesso IAM seguindo princípio do menor privilégio (dynamodb:PutItem restrito ao ARN da tabela)
- Nenhum endpoint público exposto
- Criptografia em repouso habilitada por padrão (KMS)
- Retenção de dados controlada via TTL (90 dias)

## Consequências

### Positivas

- Ausência de gerenciamento de servidor
- Escalabilidade automática
- Eficiência de custo para tráfego esporádico
- Forte alinhamento com arquitetura serverless
- TTL simplifica estratégia de retenção de dados

### Negativas

- Flexibilidade limitada de consulta
- Ausência de joins relacionais
- Modelo schema-less exige validação rigorosa na aplicação
- Modelagem deve antecipar padrões de acesso

## Melhorias Futuras

- Redesenhar a tabela com chave composta (PK/SK) para consultas mais flexíveis
- Adicionar GSI para consultas administrativas (ex: busca por email)
- Implementar limitação por email via modelagem de partição
- Criptografar atributos sensíveis com KMS gerenciado pelo cliente
- Documentar estratégia de backup e recuperação