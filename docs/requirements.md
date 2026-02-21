[Versão PT-BR](#versao-em-portugues)

# 1. Business Requirements

### BR-01  
The system must allow users to request and download a free ebook via a web form

### BR-02  
The system must collect and store the following contact details:
- Name
- Email address
- Timestamp of submission

### BR-03  
The system must send an email notification after a successful download request

### BR-04  
The system must persist lead data for future analytics and marketing purposes

### BR-05  
No user authentication is required for ebook download

# 2. Non-Functional Requirements

### NFR-01  
The system must automatically handle traffic spikes up to 10x the expected daily volume without manual intervention

### NFR-02  
The system must scale horizontally without requiring infrastructure reconfiguration

### NFR-03  
The system must rely exclusively on managed AWS services with an SLA of at least 99.9% availability

### NFR-04  
The system must not require manual restarts or server maintenance to remain operational

### NFR-05  
All API endpoints must enforce HTTPS using TLS 1.2 or higher

### NFR-06  
Data must be encrypted at rest using AWS-managed encryption

### NFR-07  
Access to storage resources must follow the principle of least privilege via IAM roles

### NFR-08  
All external access must be via HTTPS

---

# 🇧🇷 Versão em Português

# 1. Regras de Negócio

### RN-01  
O sistema deve permitir que os usuários solicitem e baixem um ebook gratuito por meio de um formulário web

### RN-02  
O sistema deve coletar e armazenar os seguintes dados de contato:
- Nome
- Endereço de email
- Timestamp da submissão

### RN-03  
O sistema deve enviar uma notificação por email após uma solicitação de download bem-sucedida

### RN-04  
O sistema deve persistir os dados dos leads para futuras finalidades de análise e marketing

### RN-05  
Não é necessária autenticação de usuário para o download do ebook

# 2. Requisitos Não Funcionais

### RNF-01  
O sistema deve lidar automaticamente com picos de tráfego de até 10x o volume diário esperado sem intervenção manual

### RNF-02  
O sistema deve escalar horizontalmente sem exigir reconfiguração da infraestrutura

### RNF-03  
O sistema deve depender exclusivamente de serviços gerenciados da AWS com um SLA de pelo menos 99,9% de disponibilidade

### RNF-04  
O sistema não deve exigir reinicializações manuais ou manutenção de servidor para permanecer operacional

### RNF-05  
Todos os endpoints da API devem impor HTTPS utilizando TLS 1.2 ou superior

### RNF-06  
Os dados devem ser criptografados em repouso utilizando criptografia gerenciada pela AWS

### RNF-07  
O acesso aos recursos de armazenamento deve seguir o princípio do menor privilégio por meio de funções IAM

### RNF-08  
Todo acesso externo deve ser via HTTPS