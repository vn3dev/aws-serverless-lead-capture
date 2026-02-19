[🇧🇷 Versão em Português](#versao-em-portugues)

# ADR-0001 — Static Hosting Strategy (S3 + CloudFront)

[2026-02-19] - by Vinicius Costa  

## Context

The system requires a highly available, low-maintenance solution to host a static website (HTML, CSS, JS) with HTTPS support and global distribution

Non-functional requirements include:
- Automatic scalability
- No server management
- HTTPS enforcement (TLS 1.2+)
- High availability (≥99.9%)
- Low operational overhead

## Decision

Use Amazon S3 for static asset storage and Amazon CloudFront as a CDN with HTTPS via AWS Certificate Manager (ACM)

The S3 bucket is configured as private, with access restricted exclusively to CloudFront using an origin access control policy. Direct public access to the bucket is blocked

CloudFront handles:
- TLS termination
- Global content caching
- HTTPS enforcement
- DNS integration via custom domain

## Alternatives Considered

### 1. EC2 with Nginx
Would require:
- Instance management
- Security patching
- Auto-scaling configuration
- Load balancer setup

Rejected due to operational overhead and unnecessary infrastructure complexity for static content

### 2. AWS Amplify Hosting
Simplifies deployment but abstracts infrastructure details.
Rejected to maintain full architectural control and demonstrate explicit infrastructure decisions

## Rationale

S3 + CloudFront provides:

- Fully managed infrastructure
- Automatic horizontal scaling
- Native AWS integration
- Lower cost compared to EC2-based solutions
- Fine-grained access control
- Global edge caching for reduced latency

This architecture aligns with serverless design principles and minimizes operational responsibility

## Security Considerations

- S3 public access block enabled
- Bucket policy restricts access to CloudFront service principal
- Direct S3 access returns 403
- HTTPS enforced via ACM certificate
- DNS configured to point exclusively to CloudFront distribution

## Consequences

### Positive

- High availability
- No server maintenance
- Global distribution
- Cost efficiency
- Strong integration with other AWS services

### Negative

- Cache invalidation required for updates
- More complex DNS and certificate validation setup
- Debugging can be less straightforward due to CDN caching

## Future Improvements

- Infrastructure as Code (Terraform or AWS SAM) for reproducible deployment
- WAF integration at CloudFront layer
- Security headers enforcement (HSTS, CSP, etc.)

---

# 🇧🇷 Versão em Português

# ADR-0001 — Estratégia de Hosting Estático (S3 + CloudFront)

[19-02-2026] - por Vinicius Costa  

## Contexto

O sistema requer uma solução altamente disponivel e de baixa manutenção para hospedar um website estático (HTML, CSS e JavaScript) com suporte a HTTPS e distribuição global

Os requisitos não funcionais incluem:
- Escalabilidade automática
- Ausência de gerenciamento de servidores
- Obrigatoriedade de HTTPS (TLS 1.2+)
- Alta disponibilidade (>=99,9%)
- Baixo overhead operacional

## Decisão

Utilizar o Amazon S3 para armazenamento dos arquivos estáticos e o Amazon CloudFront como CDN, com HTTPS provisionado via AWS Certificate Manager (ACM)

O bucket S3 foi configurado como privado, com acesso restrito exclusivamente ao CloudFront por meio de policy baseada no service principal da distribuição. O acesso publico direto ao bucket foi bloqueado.

O CloudFront é responsável por:
- Terminação TLS
- Cache global de conteúdo
- Garantia de acesso via HTTPS
- Integração com domínio customizado

## Alternativas Consideradas

### 1. EC2 com Nginx
Exigiria:
- Gerenciamento de instâncias
- Aplicação de patches de segurança
- Configuração de auto scaling
- Configuração de load balancer

Rejeitado devido ao overhead operacional desnecessário para entrega de conteudo estático

### 2. AWS Amplify Hosting
Simplifica o deploy, mas abstrai decisões de infraestrutura
Rejeitado para manter controle explícito da arquitetura e demonstrar decisões técnicas conscientes

## Justificativa

A combinação S3 + CloudFront oferece:

- Infraestrutura totalmente gerenciada
- Escalabilidade horizontal automática
- Integração nativa com serviços AWS
- Custo inferior em comparação a soluções baseadas em EC2
- Controle granular de acesso
- Distribuição global via edge locations

Essa arquitetura está alinhada com princípios serverless e reduz responsabilidades operacionais

## Considerações de Segurança

- Bloqueio de acesso público ativado no S3
- Policy do bucket restringindo acesso ao service principal do CloudFront
- Acesso direto ao S3 retorna 403
- HTTPS obrigatório via certificado ACM
- DNS apontando exclusivamente para a distribuição do CloudFront

## Consequências

### Positivas

- Alta disponibilidade
- Ausência de manutenção de servidores
- Distribuição global
- Eficiência de custo
- Forte integração com o ecossistema AWS

### Negativas

- Necessidade de invalidação de cache para atualizações imediatas
- Configuração de DNS e validação de certificado adicionam complexidade inicial
- Depuração pode ser impactada pelo cache da CDN

## Melhorias Futuras

- Provisionamento via Infrastructure as Code (Terraform ou AWS SAM)
- Integração com AWS WAF na camada do CloudFront
- Implementação de headers de segurança (HSTS, CSP, X-Content-Type-Options, e outros...)