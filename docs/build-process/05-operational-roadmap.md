### IMPORTANTE: Esse projeto foi desenvolvido em um ambiente controlado com fins pedagógicos e de auto aprendizado. Esse projeto não passou por critérios de segurança ou autenticação. Usar isso em um ambiente de produção real pode expor a empresa a vulnerabilidades no sistema e causar consequências financeiras e legais de acordo com a Lei Geral de Proteção de Dados e o Marco Civil da Internet. Esse projeto não deve ser reproduzido em um ambiente profissional sem antes passar por uma validação minuciosa de segurança e boas práticas

## Seção 05 - Planejando um roadmap de melhorias

RASCUNHO: Planejar melhorias no projeto - lista de prioridades
- WAF - prioridade máxima
- + observabilidade, metricas, dashboards 
- separar lambda em ingestão e processamento
- SQS entre os lambdas
- fluxo completo de download enviando e-mail com link, precisaria de production access no SES 
- IaC com terraform
- route 53 para tornar tudo nativo - menos importante