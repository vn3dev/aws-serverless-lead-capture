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

Agora sobre CORS, o código permitia qualquer origin chamar a minha API. Isso permite que outros sites usem a API e leiam as respostas. Para corrigir isso, ativei o lambda proxy integrations no API Gateway, passando a responsabilidade do CORS para o Lambda. No Lambda, especifiquei quais origins são válidas e alterei corsHeaders(event) para validar a origin

No Lambda:

```
const ALLOWED_ORIGINS = new Set([
  "https://vn3infra.com",
  "https://www.vn3infra.com",
]);
```

```
function corsHeaders(event) {
  const origin = event?.headers?.origin || event?.headers?.Origin || "";
  const allowOrigin = ALLOWED_ORIGINS.has(origin) ? origin : "null";

  return {
    "Content-Type": "application/json",
    "Access-Control-Allow-Origin": allowOrigin,
    "Vary": "Origin",
    "Access-Control-Allow-Methods": "OPTIONS,POST",
    "Access-Control-Allow-Headers": "Content-Type",
  };
}
```

Ao tentar dar fetch com a minha API em um alguma outra origin:

![Fetching my API from google.com](../img/62-response.png)

Para evitar alguns abusos do input do usuário, defini alguns limites para os dados que o usuario pode inserir:

```
const LIMITS = {
  name: 100,
  email: 254,
  phone: 30,
  message: 1000,
};
```

```
function sanitize(value, maxLen) {
  if (value === undefined || value === null) return "";
  const str = String(value).trim();
  if (!str) return "";
  return str.length > maxLen ? str.slice(0, maxLen) : str;
}
```

```
const name = sanitize(body?.name, LIMITS.name);
const email = sanitize(body?.email, LIMITS.email).toLowerCase();
const phone = sanitize(body?.phone, LIMITS.phone);
const message = sanitize(body?.message, LIMITS.message);
```

```
if (!EMAIL_RE.test(email)) {
  return {
    statusCode: 400,
    headers: corsHeaders(event),
    body: JSON.stringify({ error: "invalid email format" }),
  };
}
```

Mesmo que o usuário coloque mais dos caracteres inseridos, o banco só registra até o max

Para evitar payloads muito grandes e inundar o database. Foi feita uma alteração no Lambda code para retornar 413 caso o payload seja maior que 2000 caracteres:

```
    if (body.length>2000) return {
        statusCode: 413,
        headers: corsHeaders(event),
        body: JSON.stringify({ error: "message too long" }),
    }
```
Também ativei o throttling nas configs do API Gateway e defini o rate limit para 10 e o burst para 20

![Throttling settings](../img/63-enable-throttling.png)

Além disso, configurei variáveis de ambiente no Lambda para armazenar os e-mails e a table do Ddb. Se uma variável estiver vazia, o Lambda não executa e retorna 500. Se todas as variáveis forem validadas, ele retorna 200 e executa o Lambda normalmente.

![Lambda env variables](../img/64-env-var-lambda.png)

No codigo Lambda, fiz as variáveis de ambiente:

```
const REGION = mustEnv("AWS_REGION");
const TABLE_NAME = mustEnv("TABLE_NAME");
const RECEIVER = mustEnv("RECEIVER_EMAIL");
const SENDER = mustEnv("SENDER_EMAIL");
```

Clients só depois de validar:

```
const ses = new SESClient({ region: REGION });
const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({ region: REGION }));
```

Fail-fast. Se env estiver faltando, nem tenta processar request:

```
  const envCheck = validateRuntimeConfig();
  if (!envCheck.ok) {
    console.error("config_error", envCheck);
    return {
      statusCode: 500,
      headers: corsHeaders(event),
      body: JSON.stringify({
        error: "Service misconfigured",
        missing: envCheck.missing, // opcional: pode remover em prod
      }),
    };
  }
```

```
function validateRuntimeConfig() {
  const required = {
    AWS_REGION: REGION,
    TABLE_NAME,
    RECEIVER_EMAIL: RECEIVER,
    SENDER_EMAIL: SENDER,
  };

  const missing = Object.entries(required)
    .filter(([, v]) => !v)
    .map(([k]) => k);

  return { ok: missing.length === 0, missing };
}
```

Reconheço que usar o WAF seria mais facil e eficiente. Mas como estou no free tier e esse é apenas um lab para aprendizado, optei por manter o projeto sem custos