[🇧🇷 Versão em Português](#-versão-em-português)

# ADR-0004 — Email Delivery Strategy using Amazon SES

[19-02-2026] - by Vinicius Costa  

## Context

The system must send email notifications after a successful form submission

The email flow includes:

- Notifying the business owner about a new lead
- Optionally enabling reply-to functionality
- Ensuring reliable delivery
- Avoiding high operational complexity

Non-functional requirements include:

- High availability
- Managed infrastructure
- Cost efficiency
- Integration with serverless architecture

The solution must avoid managing SMTP servers or third-party email infrastructure manually

## Decision

Use Amazon Simple Email Service (SES) to send transactional emails directly from AWS Lambda

The Lambda function integrates with SES via the AWS SDK and sends structured notification emails upon successful database writes

Two verified identities are configured in SES:

- Sender identity
- Receiver identity (for sandbox environment)

## Alternatives Considered

### 1. Storing data only in database sending no email

Rejected because:

- Email provides immediate notification
- Improves operational awareness
- Enables rapid lead follow-up

## Rationale

Amazon SES provides:

- Fully managed email infrastructure
- High deliverability rates
- Native AWS IAM integration
- Cost-efficient pay-per-use model
- Direct SDK integration with Lambda
- Low latency within the same AWS region

Using SES aligns with the serverless architecture and avoids managing outbound email infrastructure

## Security Considerations

- IAM least-privilege permissions (ses:SendEmail restricted to verified identities)
- Email masking applied in notification body
- PII removed from logs
- TLS encryption enforced in transit
- No hardcoded credentials (IAM role-based access)

## Consequences

### Positive

- No SMTP server management
- High availability
- Seamless integration with Lambda
- Low operational overhead
- Scalable transactional email delivery

### Negative

- SES sandbox restrictions during development
- Requires domain/email identity verification
- Potential deliverability tuning required in production
- Risk of email-based PII exposure if inbox compromised

## Future Improvements

- Move from sandbox to production SES environment
- Implement bounce and complaint handling (SNS integration)
- Add email delivery monitoring metrics
- Consider double opt-in flow for compliance
- Replace direct PII email notifications with internal dashboard reference (ID-only email)

---

# 🇧🇷 Versão em Português

# ADR-0004 — Estratégia de Envio de E-mail com Amazon SES

[19-02-2026] - por Vinicius Costa

## Contexto

O sistema deve enviar notificações por e-mail após uma submissão bem-sucedida do formulário

O fluxo de e-mail inclui:

- Notificar o responsável pelo negócio sobre um novo lead
- Permitir funcionalidade de reply-to quando aplicável
- Garantir entrega confiável
- Evitar alta complexidade operacional

Os requisitos não funcionais incluem:

- Alta disponibilidade
- Infraestrutura gerenciada
- Eficiência de custo
- Integração com arquitetura serverless

A solução deve evitar gerenciamento manual de servidores SMTP ou infraestrutura externa

## Decisão

Utilizar o Amazon Simple Email Service (SES) para envio de e-mails transacionais diretamente a partir do AWS Lambda

A função Lambda integra-se ao SES por meio do AWS SDK e envia e-mails estruturados após a persistência bem-sucedida no banco de dados

Foram configuradas duas identidades verificadas no SES:

- Identidade de envio (sender)
- Identidade de recebimento (para ambiente sandbox)

## Alternativas Consideradas

### 1. Armazenar dados apenas no banco sem envio de e-mail

Rejeitado porque:

- O e-mail fornece notificação imediata
- Melhora a consciência operacional
- Permite contato rápido com o lead

## Justificativa

O Amazon SES oferece:

- Infraestrutura de e-mail totalmente gerenciada
- Alta taxa de entregabilidade
- Integração nativa com IAM
- Modelo de custo pay-per-use
- Integração direta com Lambda via SDK
- Baixa latência quando utilizado na mesma região AWS

O uso do SES está alinhado com a abordagem serverless do projeto e elimina a necessidade de gerenciar infraestrutura de envio de e-mails

## Considerações de Segurança

- Permissões IAM seguindo princípio do menor privilégio (ses:SendEmail restrito às identidades verificadas)
- Mascaramento de e-mail no corpo da notificação
- Remoção de PII dos logs
- Criptografia TLS em trânsito
- Ausência de credenciais hardcoded (acesso baseado em role IAM)

## Consequências

### Positivas

- Ausência de gerenciamento de servidor SMTP
- Alta disponibilidade
- Integração direta com Lambda
- Baixo overhead operacional
- Escalabilidade automática para e-mails transacionais

### Negativas

- Restrições do ambiente sandbox durante desenvolvimento
- Necessidade de verificação de domínio/identidade
- Pode exigir ajustes de entregabilidade em produção
- Risco de exposição de PII caso a caixa de entrada seja comprometida

## Melhorias Futuras

- Solicitar saída do ambiente sandbox para produção
- Implementar tratamento de bounce e complaint via SNS
- Monitoramento de métricas de entrega
- Implementar fluxo de double opt-in para conformidade
- Substituir envio direto de PII por notificação contendo apenas ID e consulta interna