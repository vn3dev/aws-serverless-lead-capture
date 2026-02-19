[🇧🇷 Versão em Português](#versao-em-portugues)

# ADR-0002 — API Layer Architecture (API Gateway + Lambda)

[2026-02-19] - by Vinicius Costa

## Context

The system requires an HTTP API to:

- Receive form submission data
- Persist data into a database
- Send email notifications via SES
- Return structured JSON responses to the frontend

Non-functional requirements include:

- Automatic scalability
- Serverless architecture
- Low operational overhead
- High availability (≥99.9%)
- Mandatory HTTPS enforcement
- Ability to handle traffic spikes (up to 10x expected volume)

The solution must avoid server management and align with cloud-native principles

## Decision

Use Amazon API Gateway (REST API) integrated with AWS Lambda through Lambda Proxy Integration

API Gateway is responsible for:

- Exposing a public HTTPS endpoint
- Managing throttling configuration
- Forwarding requests to Lambda
- Providing metrics and logging integration via CloudWatch

AWS Lambda is responsible for:

- Input validation and sanitization
- Business logic execution
- Persisting data in DynamoDB
- Sending emails via SES
- Returning structured responses

## Alternatives Considered

### 1. Application Load Balancer + Lambda

Possible serverless alternative. However:

- API Gateway provides more robust API management features
- Better integration with throttling, stages, usage metrics, and request handling

Rejected due to reduced API-specific control

## Rationale

API Gateway + Lambda provides:

- Automatic per-request scaling
- Pay-per-use cost model
- Native integration with CloudWatch
- Built-in throttling capabilities
- Tight IAM integration
- Stateless execution model aligned with serverless best practices

This architecture eliminates persistent server management and reduces operational responsibility

## Security Considerations

- HTTPS enforced by default
- API Gateway throttling enabled
- Input validation and sanitization in Lambda
- Payload size limit enforced (413 for oversized requests)
- Restricted CORS configuration
- Least-privilege IAM roles
- Removal of PII from logs

## Consequences

### Positive

- Automatic scalability
- Cost-efficient at low traffic volumes
- No server maintenance
- Strong AWS service integration
- Decoupled architecture

### Negative

- Cold start latency
- Increased architectural complexity compared to a single server
- Additional abuse protection required (WAF recommended)
- Debugging distributed across multiple services

## Future Improvements

- Integration with AWS WAF for bot protection
- IP-based rate limiting
- Custom metrics (e.g., valid lead count)
- Distributed tracing with AWS X-Ray
- Infrastructure as Code implementation

---

# 🇧🇷 Versão em Português

# ADR-0002 — Arquitetura da Camada de API (API Gateway + Lambda)

[19-02-2026] - por Vinicius Costa  

## Contexto

O sistema requer uma API HTTP para:

- Receber dados do formulário
- Persistir os dados no banco
- Enviar notificação por e-mail via SES
- Retornar resposta JSON ao frontend

Os requisitos não funcionais incluem:

- Escalabilidade automatica
- Arquitetura serverless
- Baixo overhead operacional
- Alta disponibilidade (>=99,9%)
- Suporte a HTTPS obrigatório
- Capacidade de lidar com picos de tráfego (até 10x o volume esperado)

A solução deve evitar gerenciamento de servidores e manter alinhamento com princípios cloud-native

## Decisão

Utilizar Amazon API Gateway (REST API) integrado ao AWS Lambda via Lambda Proxy Integration

A API Gateway é responsável por:

- Expor endpoint HTTPS público
- Gerenciar throttling
- Encaminhar requisições para o Lambda
- Integrar-se com CloudWatch para métricas e logs

O AWS Lambda é responsável por:

- Validar e sanitizar entrada
- Aplicar regras de negócio
- Persistir dados no DynamoDB
- Enviar e-mails via SES
- Retornar resposta estruturada

## Alternativas Consideradas

### 1. Application Load Balancer + Lambda

Possível alternativa serverless, porem:
- API Gateway fornece recursos mais robustos para controle de API
- Melhor integração com throttling, stages e métricas detalhadas

Rejeitado por oferecer menos controle específico de API.

## Justificativa

API Gateway + Lambda oferece:

- Escalabilidade automática por requisição
- Modelo pay-per-use
- Integração nativa com CloudWatch
- Configuração de throttling
- Integração direta com IAM
- Modelo stateless alinhado com boas práticas serverless

Essa arquitetura elimina necessidade de servidores persistentes e reduz superfície de manutenção

## Considerações de Segurança

- HTTPS obrigatório
- Throttling configurado no API Gateway
- Validação e sanitização de input no Lambda
- Limite de payload (413 para excesso)
- CORS restrito a domínios específicos
- Princípio do menor privilégio aplicado nas roles IAM
- Logs sem dados pessoais

## Consequências

### Positivas

- Escalabilidade automática
- Baixo custo em baixo volume
- Zero gerenciamento de servidor
- Boa integração com serviços AWS
- Arquitetura desacoplada

### Negativas

- Latência inicial
- Maior complexidade comparado a um servidor simples
- Necessidade de controle adicional contra abuso (WAF)
- Debug distribuído entre múltiplos serviços

## Melhorias Futuras

- Integração com AWS WAF para proteção contra bots
- Implementação de rate limiting baseado em IP
- Métricas customizadas
- Tracing distribuído: AWS X-Ray
- Provisionamento via Infrastructure as Code