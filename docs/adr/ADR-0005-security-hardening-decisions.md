[🇧🇷 Versão em Português](#versao-em-portugues)

# ADR-0005 — Security Hardening Strategy

[20-02-2026] - by Vinicius Costa 

## Context

The system exposes a public HTTPS endpoint to receive user-submitted data (name, email)

Because the API is publicly accessible and does not require authentication, it is inherently exposed to:

- Spam and bot abuse
- Automated payload flooding
- Malformed or oversized requests
- Credential misuse
- Data leakage risks
- Cost amplification attacks

The architecture must mitigate these risks while maintaining a serverless, low-operational model.

## Threat Model

Primary risks identified:

1. Abuse of public API endpoint
2. Injection or malformed payload attacks
3. Excessive request volume
4. Exposure of personally identifiable information
5. Over-permissive IAM roles
6. Long-term storage of sensitive data

## Decision

Implement layered security controls across multiple components:

- Input validation and sanitization in Lambda
- Payload size limitation (413 for oversized bodies)
- API Gateway throttling (rate + burst control)
- Conditional writes in DynamoDB to prevent duplication
- TTL for automatic data expiration
- CORS restriction to known origins
- Removal of PII from logs
- Email masking in notifications
- IAM least-privilege enforcement
- CloudWatch alarms + SNS notifications
- Budget alerts for cost anomaly detection

Security controls are distributed across layers instead of relying on a single mechanism

## Alternatives Considered

### 1. Full authentication layer using OAuth / Cognito

Rejected because:
- Business requirement does not require user authentication
- Would introduce significant UX friction
- Over-engineering for simple lead capture use case

## Rationale

Security decisions follow the principle of defense-in-depth:

- Prevent abuse: Throttling + conditional writes
- Reduce attack surface: Least privilege IAM
- Minimize sensitive exposure: Log scrubbing + masking
- Limit blast radius: TTL + cost alerts
- Detect anomalies: CloudWatch alarms

The system assumes that abuse attempts are inevitable and designs controls to limit operational and financial impact

## Security Controls Implemented

### Application Layer

- Input sanitization
- Email format validation
- Payload size restriction
- Robust JSON parsing
- Fail-fast configuration validation

### API Layer

- Throttling enabled
- HTTPS enforcement
- Restricted CORS configuration

### Data Layer

- TTL (90 days)
- Conditional writes (anti-duplicate)
- Encryption at rest (KMS)

### IAM

- Resource-scoped policies
- No wildcard permissions
- Role-based access only

### Observability & Cost Control

- CloudWatch error alarms
- SNS notifications
- Budget alerts (50%, 80%, 100% + forecast)

## Consequences

### Positive

- Reduced abuse surface
- Lower risk of uncontrolled cost escalation
- Minimized PII exposure
- Clear observability of failure conditions
- Improved production-readiness

### Negative

- Increased architectural complexity
- More configuration overhead
- Some protections (e.g., CORS) do not stop non-browser abuse
- Additional improvements (WAF, bot protection)

## Future Improvements

- Integrate AWS WAF with rate-based rules
- Implement bot protection (Turnstile / CAPTCHA)
- Add IP-based blocking
- Implement structured audit logging
- Define formal data deletion workflow
- Conduct periodic IAM policy review
- Introduce automated security scanning (IaC validation)

---

# 🇧🇷 Versão em Português

# ADR-0005 — Estratégia de Hardening de Segurança

[20-02-2026] - by Vinicius Costa 

## Contexto

O sistema expõe um endpoint HTTPS público para receber dados enviados pelos usuários (nome, email)

Por se tratar de uma API pública e sem autenticação, o sistema está inerentemente exposto a:

- Spam e abuso por bots
- Envio automatizado de payloads maliciosos
- Requisições malformadas ou excessivamente grandes
- Uso indevido de credenciais
- Risco de vazamento de dados pessoais
- Ataques de amplificação de custo

A arquitetura deve mitigar esses riscos mantendo o modelo serverless e baixo overhead operacional

## Modelo de Ameaças

Principais riscos identificados:

1. Abuso do endpoint público da API
2. Ataques por injeção ou payload malformado
3. Volume excessivo de requisições
4. Exposição de dados pessoais
5. Roles IAM excessivamente permissivas
6. Retenção prolongada de dados sensíveis

## Decisão

Implementar controles de segurança em camadas (defense-in-depth), distribuídos entre os componentes da arquitetura:

- Validação e sanitização de input no Lambda
- Limite de tamanho de payload (413 para excesso)
- Throttling no API Gateway (rate + burst)
- Escrita condicional no DynamoDB para evitar duplicações
- TTL para expiração automática de dados
- Restrição de CORS a domínios conhecidos
- Remoção de PII dos logs
- Mascaramento de e-mail nas notificações
- Aplicação do princípio do menor privilégio nas policies IAM
- Alarmes no CloudWatch + notificações via SNS
- Alertas de orçamento para detecção de anomalias de custo

Os controles foram distribuídos entre camadas para evitar dependência de um único mecanismo de proteção

## Alternativas Consideradas

### 1. Camada completa de autenticação com OAuth / Cognito

Rejeitado porque:

- O requisito de negócio não exige autenticação
- Introduziria fricção desnecessária ao usuário
- Representaria over-engineering para o caso de uso

## Justificativa

As decisões seguem o princípio de defesa em camadas:

- Prevenir abuso: Throttling + escrita condicional
- Reduzir superfície de ataque: IAM restritivo
- Minimizar exposição de dados sensíveis: Remoção de PII dos logs + mascaramento
- Limitar impacto financeiro: TTL + alertas de orçamento
- Detectar anomalias: alarmes no CloudWatch

Parte-se do pressuposto de que tentativas de abuso são inevitáveis, e a arquitetura foi projetada para limitar impacto operacional e financeiro

## Controles Implementados

### Camada de Aplicação

- Sanitização de input
- Validação de formato de e-mail
- Limite de tamanho de payload
- Parsing robusto de JSON
- Validação fail-fast de variáveis de ambiente

### Camada de API

- Throttling habilitado
- HTTPS obrigatório
- Configuração restrita de CORS

### Camada de Dados

- TTL de 90 dias
- Escrita condicional para anti-duplicação
- Criptografia em repouso (KMS)

### IAM

- Policies com escopo restrito a recursos específicos
- Ausência de permissões wildcard desnecessárias
- Acesso baseado exclusivamente em roles

### Observabilidade e Controle de Custos

- Alarmes de erro no CloudWatch
- Notificações via SNS
- Alertas de orçamento (50%, 80%, 100% + previsão)

## Consequências

### Positivas

- Redução da superfície de abuso
- Menor risco de escalada de custo descontrolada
- Minimização da exposição de PII
- Maior visibilidade sobre falhas
- Arquitetura mais próxima de produção

### Negativas

- Aumento da complexidade arquitetural
- Maior overhead de configuração
- Algumas proteções (ex: CORS) não impedem abuso fora do browser
- Proteções adicionais (WAF, bot protection)

## Melhorias Futuras

- Integração com AWS WAF com regras baseadas em taxa
- Implementação de proteção contra bots (Turnstile / CAPTCHA)
- Bloqueio baseado em IP
- Logging estruturado para auditoria
- Definição formal de fluxo de exclusão de dados
- Revisão periódica das policies IAM
- Validação automática de segurança via IaC