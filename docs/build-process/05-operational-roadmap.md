[🇧🇷 Versão em Português](#-versão-em-português)

### IMPORTANT: This project was developed in a controlled environment for educational and self-learning purposes. This project did not go through security or authentication criteria. Using this in a real production environment may expose the company to system vulnerabilities and cause financial and legal consequences in accordance with the General Data Protection Law (Lei Geral de Proteção de Dados) and the Brazilian Civil Rights Framework for the Internet (Marco Civil da Internet). This project must not be reproduced in a professional environment without first undergoing a thorough security validation and best practices review.

## Section 05 - Improvement Roadmap

This project was developed as an educational lab. Below are planned improvements to evolve it toward a more production-ready architecture:

### High Priority

- **AWS WAF**  
  Add protection against bots, abuse, and common web attacks, complementing API Gateway throttling.

- **Separation of Responsibilities**  
  Split into:
  - Ingestion Lambda (validation + persistence)
  - Processing Lambda (email delivery)

- **SQS Between Lambdas**  
  Make email delivery asynchronous to improve resilience and handle traffic spikes.

- **Observability**  
  Implement metrics, dashboards, and alarms (5xx errors, throttling, SES failures, DLQ monitoring).

---

### Medium Priority

- **Complete Download Flow via Email**  
  Implement secure download link delivery with SES production access.

- **Infrastructure as Code (Terraform)**  
  Migrate resources to IaC to enable reproducibility and infrastructure versioning.

---

### Low Priority

- **DNS Migration to Route 53**  
  Make the infrastructure fully AWS-native. Limited functional impact.


---

# 🇧🇷 Versão em Português

### IMPORTANTE: Esse projeto foi desenvolvido em um ambiente controlado com fins pedagógicos e de auto aprendizado. Esse projeto não passou por critérios de segurança ou autenticação. Usar isso em um ambiente de produção real pode expor a empresa a vulnerabilidades no sistema e causar consequências financeiras e legais de acordo com a Lei Geral de Proteção de Dados e o Marco Civil da Internet. Esse projeto não deve ser reproduzido em um ambiente profissional sem antes passar por uma validação minuciosa de segurança e boas práticas

## Seção 05 - Roadmap de Melhorias

Este projeto foi desenvolvido como lab educacional. Abaixo estão melhorias planejadas para evoluir em direção a um padrão mais próximo de produção:

### Prioridade Alta

- **AWS WAF**  
  Adicionar proteção contra bots, abuso e ataques comuns, complementando o throttling do API Gateway.

- **Separação de responsabilidades**  
  Dividir em:
  - Lambda de ingestão (validação + persistência)
  - Lambda de processamento (envio de e-mail)

- **SQS entre as Lambdas**  
  Tornar o envio de e-mail assíncrono, aumentando resiliência e absorção de picos de tráfego.

- **Observabilidade**  
  Implementar métricas, dashboards e alarmes (erros 5xx, throttling, falhas no SES, DLQ).

---

### Prioridade Média

- **Fluxo completo de download via e-mail**  
  Implementar envio de link seguro para download com production access no SES.

- **Infraestrutura como Código (Terraform)**  
  Migrar recursos para IaC visando reprodutibilidade e versionamento da infraestrutura.

---

### Prioridade Baixa

- **Migração de DNS para Route 53**  
  Tornar a infraestrutura totalmente nativa AWS. Impacto funcional reduzido.