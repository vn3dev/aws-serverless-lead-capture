[🇧🇷 Versão em Português](#-versão-em-português)

# ADR-0006 — Data Retention and Compliance Strategy (LGPD Alignment)

[20-02-2026] - by Vinicius Costa 

## Context

The system collects and stores personal data voluntarily submitted by users through a web form, including:

- Name
- Email address
- Submission timestamp

These data elements are classified as personal data under the Brazilian General Data Protection Law (LGPD – Law No. 13.709/2018)

The architecture must ensure:

- Data minimization
- Limited retention period
- Legitimate purpose
- Reduced exposure risk
- Mitigation of legal and financial impact

Although the project is educational in nature, it was designed considering practices applicable to real-world environments

## Decision

Implement automatic data retention control using DynamoDB TTL (Time To Live) with a retention period of 90 days

Attribute used:
- `expiresAt` (Unix timestamp in seconds)

Records are automatically deleted after the configured retention period

Additionally:

- CloudWatch logs do not store PII
- Email notifications use partial masking
- IAM permissions follow the principle of least privilege

## Alternatives Considered

### 1. Indefinite data retention

Rejected because:

- Increases legal risk
- Increases financial exposure in case of breach
- Violates LGPD principles of necessity and data minimization

### 2. Manual periodic deletion

Rejected because:

- Prone to human error
- Not scalable
- Lacks automation

### 3. Shorter retention period

Considered viable; however:

- Could limit basic marketing analysis
- 90 days was considered a balance between utility and risk reduction

## Rationale

Automatic TTL-based deletion provides:

- Reduced long-term exposure risk
- Lower potential breach impact
- Controlled database growth
- Alignment with the principles of necessity and purpose limitation

The strategy assumes that lead data does not need indefinite storage to fulfill its intended business purpose.

## Compliance Considerations

While the project does not implement a full formal compliance framework, it considers:

- Data minimization: Collect only necessary fields
- Purpose limitation: Use exclusively for contact
- Time-bound storage
- Removal of PII from logs
- Encryption at rest and in transit

## Consequences

### Positive

- Reduced legal risk
- Reduced financial exposure
- Automated retention control
- Simplified data governance

### Negative

- Historical data unavailable after 90 days
- Requires clear documentation of retention policy
- TTL does not guarantee immediate deletion

## Future Improvements

- Implement explicit consent capture in the form
- Store accepted privacy policy version
- Create a manual deletion workflow upon user request
- Document formal retention policy in repository
- Evaluate partial anonymization after defined period
- Implement access auditing for sensitive data

---

# 🇧🇷 Versão em Português

# ADR-0006 — Estratégia de Retenção de Dados e Conformidade (LGPD)

[20-02-2026] - by Vinicius Costa 

## Contexto

O sistema coleta e armazena dados pessoais fornecidos voluntariamente pelo usuário por meio de formulário web, incluindo:

- Nome
- Endereço de e-mail
- Timestamp da submissão

Esses dados são classificados como dados pessoais conforme a Lei Geral de Proteção de Dados (Lei nº 13.709/2018 – LGPD)

A arquitetura deve garantir:

- Minimização de dados
- Retenção limitada
- Finalidade legítima
- Redução de risco de exposição
- Controle de impacto financeiro e jurídico

O projeto tem caráter educacional, mas foi projetado considerando boas práticas aplicáveis a ambientes reais

## Decisão

Implementar estratégia de retenção automática de dados utilizando TTL (Time To Live) no DynamoDB, com período definido de 90 dias

Atributo utilizado:
- `expiresAt` (timestamp Unix em segundos)

Os dados são automaticamente excluídos após o período de retenção configurado

Além disso:

- Logs no CloudWatch não armazenam PII
- E-mails enviados ao responsável utilizam mascaramento parcial
- As permissões IAM seguem princípio do menor privilégio

## Alternativas Consideradas

### 1. Retenção indefinida

Rejeitado porque:

- Aumenta risco jurídico
- Aumenta risco financeiro em caso de vazamento
- Viola princípio de necessidade e minimização da LGPD

### 2. Exclusão manual periódica

Rejeitado porque:

- Processo suscetível a erro humano
- Não escalável
- Não automatizado

### 3. Retenção inferior a 90 dias

Considerado viável, porém:

- Poderia comprometer análises básicas de marketing
- 90 dias foi considerado equilíbrio entre utilidade e minimização

## Justificativa

A retenção automática via TTL oferece:

- Redução do risco de exposição prolongada
- Mitigação de impacto em caso de incidente
- Controle de crescimento do banco de dados
- Alinhamento com princípio da necessidade e limitação da finalidade

A estratégia assume que dados de leads não precisam ser armazenados indefinidamente para cumprir a finalidade proposta

## Considerações de Conformidade

Embora o projeto não implemente todos os mecanismos formais de compliance, ele considera:

- Princípio da minimização: Coleta apenas dados necessários
- Princípio da finalidade: Uso exclusivo para contato
- Limitação temporal de armazenamento
- Redução de exposição em logs
- Uso de criptografia em repouso e em trânsito

## Consequências

### Positivas

- Redução do risco jurídico
- Redução do risco financeiro
- Controle automático de retenção
- Simplificação da governança de dados

### Negativas

- Perda de histórico após 90 dias
- Necessidade de documentação clara da política de retenção
- TTL não garante exclusão imediata

## Melhorias Futuras

- Implementar registro explícito de consentimento no formulário
- Armazenar versão da política de privacidade aceita
- Criar fluxo para exclusão manual sob solicitação do titular
- Documentar política formal de retenção no repositório
- Avaliar anonimização parcial após determinado período
- Implementar auditoria de acesso aos dados