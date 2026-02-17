### IMPORTANTE: Esse projeto foi desenvolvido em um ambiente controlado com fins pedagógicos e de auto aprendizado. Esse projeto não passou por critérios de segurança ou autenticação. Usar isso em um ambiente de produção real pode expor a empresa a vulnerabilidades no sistema e causar consequências financeiras e legais de acordo com a Lei Geral de Proteção de Dados e o Marco Civil da Internet. Esse projeto não deve ser reproduzido em um ambiente profissional sem antes passar por uma validação minuciosa de segurança e boas práticas

## Seção 04 - Implementando melhorias de segurança e boas prática

Comecei fazendo uma pequena alteração em como os logs eram registrados no CloudWatch. Anteriormente, os logs continham bodies com dados como nome e email. Mas para a finalidade de observabilidade isso não é necessário. 

Além disso, no Brasil, país em que eu moro, temos leis rigorosas sobre armazenamento de dados pessoais. Armazenar essas informações em um ambiente de monitoramento dificil de governar pode expor a empresa a vazamentos, multas e outros incidentes de segurança.

Para resolver isso e evitar problemas futuros, os logs agora registram apenas id, método e origem pelo Lambda:

```
const requestId = event?.requestContext?.requestId;
const method = event?.httpMethod;
const origin = event?.headers?.origin || event?.headers?.Origin || "";

console.log("request", JSON.stringify({ requestId, method, origin }));
```

Log antigo:

![CloudWatch old log](../img/58-old-logs.png)

Log novo:

![CloudWatch new log](../img/59-new-logs.png)

Dados anonimizados sem informações consideradas dados pessoais ou identificadores

Aproveitando o embalo da proteção de dados, também decidi fazer uma mudança em como o dono do negócio vai receber o e-mail do SES. Anteriormente, o dono tinha acesso explicito a informações consideradas pessoais: O email do usuário. Nessa nova versão, o dono recebe um email censurado

O dono ainda recebe o ID da submissão daquele usuário e pode consultar isso no banco de dados. O objetivo dessa mudança é minimizar impactos e vazamentos caso o dono do negócio tenha o e-mail invadido. Isso cria uma camada a mais de proteção, onde o invasor precisaria ter o acesso ao banco de dados para ler os dados sensíveis

No Lambda, criei uma function para mascarar o email antes de enviar:

```
function maskEmail(email) {
  if (!email || typeof email !== "string") return "";
  const parts = email.trim().split("@");
  if (parts.length !== 2) return email;

  const [user, domain] = parts;
  if (!user) return `*@${domain}`;
  if (user.length === 1) return `${user}***@${domain}`;

  return `${user[0]}***@${domain}`;
}
```

Nova const para armazenar o email mascarado:

`const emailMasked = maskEmail(email);`

Alterar logo a baixo no email também:

`Email: ${emailMasked}`

Email antigo:

![Old email inbox message](../img/60-old-email.png)

Email novo:

![New email inbox message](../img/61-new-email.png)