[🇧🇷 Versão em Português](#versao-em-portugues)

### IMPORTANT: This project was developed in a controlled environment for educational and self-learning purposes. This project did not go through security or authentication criteria. Using this in a real production environment may expose the company to system vulnerabilities and cause financial and legal consequences in accordance with the General Data Protection Law (Lei Geral de Proteção de Dados) and the Brazilian Civil Rights Framework for the Internet (Marco Civil da Internet). This project must not be reproduced in a professional environment without first undergoing a thorough security validation and best practices review.

## In this section:
- I created the ContactMessages table in DynamoDB via AWS CLI
- I realized the Lambda role did not have permission for DynamoDB and added an inline policy with dynamodb:PutItem
- I updated the Lambda to save the message to DynamoDB and send an email via SES, with basic validation and CORS
- I deployed and tested in the console and on my domain: the email arrived and the items appeared in the table

## Section 03 - Configuring DynamoDB

I will start by creating the table. For that, I used the AWS CLI:

`aws dynamodb create-table --table-name ContactMessages --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --billing-mode PAY_PER_REQUEST --region us-east-1`

![Table general information](../img/47-dynamodb-table.png)

When I created the policy for Lambda to use, I did not include permission for DynamoDB. In this case, I needed to edit the existing permission or create a new one. For educational purposes, I decided to create an inline policy in the Lambda role:

![Policy JSON config](../img/48-dynamo-role-policy.png)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", 
       "Action": ["dynamodb:PutItem"], 
       "Resource": "*" }
  ]
}
```

![Policy list](../img/49-dynamo-new-policy.png)

Next, I needed to update the code in Lambda to include DynamoDB:

![Lambda code page](../img/50-lambda-code-edit.png)

```ts
import { SESClient, SendEmailCommand } from "@aws-sdk/client-ses";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";
import { randomUUID } from "crypto";

const REGION = process.env.AWS_REGION || "us-east-1";        // keep SES & DDB in same region
const TABLE_NAME = process.env.TABLE_NAME || "ContactMessages";

const RECEIVER = "EMAIL-TO-RECEIVE";
const SENDER = "EMAIL-TO-SEND";

const ses = new SESClient({ region: REGION });
const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({ region: REGION }));

const EMAIL_RE = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

export const handler = async (event) => {
  console.log("Received event:", JSON.stringify(event));

  try {
    const body =
      typeof event?.body === "string" ? JSON.parse(event.body) :
      (event?.body ?? event ?? {});

    const { name = "", phone = "", email = "", message = "" } = body;

    if (!name || !email) {
      return {
        statusCode: 400,
        headers: corsHeaders(),
        body: JSON.stringify({ error: "name and email are required" })
      };
    }

    const id = randomUUID();
    const ts = new Date().toISOString();

    // 1) Save to DynamoDB
    await ddb.send(new PutCommand({
      TableName: TABLE_NAME,
      Item: { id, name, phone, email, message, createdAt: ts }
    }));

    // 2) Send email via SES (only add Reply-To if valid)
    const params = {
      Destination: { ToAddresses: [RECEIVER] },
      Message: {
        Subject: { Data: `Website Query Form: ${name}`, Charset: "UTF-8" },
        Body: {
          Text: {
            Data:
`Full Name: ${name}
Phone: ${phone}
Email: ${email}
Message: ${message}
ID: ${id}
Time: ${ts}`,
            Charset: "UTF-8"
          }
        }
      },
      Source: SENDER
    };

    if (EMAIL_RE.test(email)) {
      params.ReplyToAddresses = [email];
    }

    await ses.send(new SendEmailCommand(params));

    return {
      statusCode: 200,
      headers: corsHeaders(),
      body: JSON.stringify({ ok: true, id })
    };
  } catch (err) {
    console.error("Error:", err);
    return {
      statusCode: 500,
      headers: corsHeaders(),
      body: JSON.stringify({ error: "Internal error" })
    };
  }
};

function corsHeaders() {
  return {
    "Content-Type": "application/json",
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Methods": "OPTIONS,POST",
    "Access-Control-Allow-Headers":
      "Content-Type,Authorization,X-Api-Key,X-Amz-Date,X-Amz-Security-Token"
  };
}
```

I deployed and ran a quick test on the Lambda page:

![Lambda deploy test](../img/51-lambda-code-testing.png)

Success. I checked the email to ensure everything was correct:

![Lambda code e-mail](../img/52-lambda-email-confirmation.png)

Final test using the domain:

![vn3infra.com form testing](../img/53-domain-test.png)

![vn3infra.com form response](../img/54-domain-test-2.png)

![vn3infra.com network](../img/55-domain-test-network.png)

![vn3infra.com email inbox](../img/56-domain-test-email.png)

Accessing the DynamoDB table, the data is already being stored:

![DynamoDB table items](../img/57-dynamodb-table.png)

With everything working, the architecture is complete.

---

# 🇧🇷 Versão em Português

### IMPORTANTE: Esse projeto foi desenvolvido em um ambiente controlado com fins pedagógicos e de auto aprendizado. Esse projeto não passou por critérios de segurança ou autenticação. Usar isso em um ambiente de produção real pode expor a empresa a vulnerabilidades no sistema e causar consequências financeiras e legais de acordo com a Lei Geral de Proteção de Dados e o Marco Civil da Internet. Esse projeto não deve ser reproduzido em um ambiente profissional sem antes passar por uma validação minuciosa de segurança e boas práticas

## Nessa seção:
- Fiz a tabela ContactMessages no DynamoDB via AWS CLI
- Percebi que a role do Lambda não tinha permissão no DynamoDB e adicionei uma inline policy com dynamodb:PutItem
- Atualizei o Lambda pra salvar a mensagem no DynamoDB e enviar e-mail via SES, com validação básica e CORS
- Dei deploy e testei no console e no meu domínio: e-mail chegou e os itens apareceram na tabela

## Seção 03 - Configurando DynamoDB

Vou começar criando a table. Para isso usei o AWS CLI:

`aws dynamodb create-table --table-name ContactMessages --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --billing-mode PAY_PER_REQUEST --region us-east-1`

![Table general information](../img/47-dynamodb-table.png)

Quando fiz a policy para o Lambda usar, não incluí o uma permissão para o DynamoDB. Nesse caso, vou precisar editar a permissão que ja tenho ou criar uma nova. Para fins educativos, decidi criar uma inline policy na role do Lambda:

![Policy JSON config](../img/48-dynamo-role-policy.png)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", 
       "Action": ["dynamodb:PutItem"], 
       "Resource": "*" }
  ]
}
```

![Policy list](../img/49-dynamo-new-policy.png)

Em seguida, precisei atualizar o codigo no Lambda, para incluir o DynamoDB:

![Lambda code page](../img/50-lambda-code-edit.png)

```ts
import { SESClient, SendEmailCommand } from "@aws-sdk/client-ses";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";
import { randomUUID } from "crypto";

const REGION = process.env.AWS_REGION || "us-east-1";        // keep SES & DDB in same region
const TABLE_NAME = process.env.TABLE_NAME || "ContactMessages";

const RECEIVER = "EMAIL-QUE-VAI-RECEBER";
const SENDER = "EMAIL-QUE-VAI-MANDAR";

const ses = new SESClient({ region: REGION });
const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({ region: REGION }));

const EMAIL_RE = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

export const handler = async (event) => {
  console.log("Received event:", JSON.stringify(event));

  try {
    const body =
      typeof event?.body === "string" ? JSON.parse(event.body) :
      (event?.body ?? event ?? {});

    const { name = "", phone = "", email = "", message = "" } = body;

    if (!name || !email) {
      return {
        statusCode: 400,
        headers: corsHeaders(),
        body: JSON.stringify({ error: "name and email are required" })
      };
    }

    const id = randomUUID();
    const ts = new Date().toISOString();

    // 1) Save to DynamoDB
    await ddb.send(new PutCommand({
      TableName: TABLE_NAME,
      Item: { id, name, phone, email, message, createdAt: ts }
    }));

    // 2) Send email via SES (only add Reply-To if valid)
    const params = {
      Destination: { ToAddresses: [RECEIVER] },
      Message: {
        Subject: { Data: `Website Query Form: ${name}`, Charset: "UTF-8" },
        Body: {
          Text: {
            Data:
`Full Name: ${name}
Phone: ${phone}
Email: ${email}
Message: ${message}
ID: ${id}
Time: ${ts}`,
            Charset: "UTF-8"
          }
        }
      },
      Source: SENDER
    };

    if (EMAIL_RE.test(email)) {
      params.ReplyToAddresses = [email];
    }

    await ses.send(new SendEmailCommand(params));

    return {
      statusCode: 200,
      headers: corsHeaders(),
      body: JSON.stringify({ ok: true, id })
    };
  } catch (err) {
    console.error("Error:", err);
    return {
      statusCode: 500,
      headers: corsHeaders(),
      body: JSON.stringify({ error: "Internal error" })
    };
  }
};

function corsHeaders() {
  return {
    "Content-Type": "application/json",
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Methods": "OPTIONS,POST",
    "Access-Control-Allow-Headers":
      "Content-Type,Authorization,X-Api-Key,X-Amz-Date,X-Amz-Security-Token"
  };
}
```

Dei deploy e fiz um teste rápido na pagina do Lambda:

![Lambda deploy test](../img/51-lambda-code-testing.png)

Sucesso, uma conferida no email para garantir que esta tudo certo:

![Lambda code e-mail](../img/52-lambda-email-confirmation.png)

Teste final usando o dominio:

![vn3infra.com form testing](../img/53-domain-test.png)

![vn3infra.com form response](../img/54-domain-test-2.png)

![vn3infra.com network](../img/55-domain-test-network.png)

![vn3infra.com email inbox](../img/56-domain-test-email.png)

Acessando a table do DynamoDB, os dados ja estão sendo armazenados:

![DynamoDB table items](../img/57-dynamodb-table.png)

Com tudo funcionando, a arquitetura esta completa 