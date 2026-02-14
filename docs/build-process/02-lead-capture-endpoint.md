Com o s3, CloudFront e DNS configurados, vou começar a fazer a captura dos dados do cliente. No caso, email e nome.

## Passo 02 - Captura de email e nome:

Criei duas novas identidades no SES usando endereços de e-mail

![SES identities page](../img/23-SES-identities.png)

Prossegui para criar a politica que a role do Lambda vai usar. Para isso, fui em IAM e criei uma nova politica em JSON:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudWatchLogsBasic",
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup"],
      "Resource": "*"
    },
    {
      "Sid": "CloudWatchLogsStreamEvents",
      "Effect": "Allow",
      "Action": ["logs:CreateLogStream","logs:PutLogEvents"],
      "Resource": "*"
    },
    {
      "Sid": "AllowSesSendEmail",
      "Effect": "Allow",
      "Action": ["ses:SendEmail","ses:SendRawEmail"],
      "Resource": "*"
    }
  ]
}
```

Os statements vão permitir que o Lambda crie o log no CloudWatch e use o SES. Depois de criada, atribuí a politica em uma nova role no IAM:

![IAM create role page](../img/24-create-role.png)

![IAM create role policy attribution](../img/25-create-role-2.png)

Com a role configurada, hora de criar a function no Lambda:

![Lambda create function configs](../img/26-create-lambda-function.png)

O app roda em Node.js, então selecionei ela como linguagem no runtime. Ajeitei as permissões para incluir a nova role que criamos. Assim que criei, fui até o code source e coloquei a function:

```
import { SESClient, SendEmailCommand } from "@aws-sdk/client-ses";


const ses = new SESClient({ region: "us-east-1" });


const RECEIVER = "EMAIL-QUE-VAI-RECEBER";
const SENDER = "EMAIL-QUE-VAI-MANDAR";


export const handler = async (event) => {
 console.log("Received event:", event);


 const params = {
   Destination: { ToAddresses: [RECEIVER] },
   Message: {
     Body: {
       Text: {
         Data: `Full Name: ${event.name}
Phone: ${event.phone}
Email: ${event.email}
Message: ${event.message}`,
         Charset: "UTF-8",
       },
     },
     Subject: { Data: `Website Query Form: ${event.name}`, Charset: "UTF-8" },
   },
   Source: SENDER,
 };


 await ses.send(new SendEmailCommand(params));


 return {
   statusCode: 200,
   headers: {
     "Content-Type": "application/json",
     "Access-Control-Allow-Origin": "*",
   },
   body: JSON.stringify({ result: "Success" }),
 };
};
```

![Lambda code block](../img/27-lambda-function-code.png)

## IMPORTANTE: Esse código é apenas para meu lab em um ambiente controlado de aprendizado. O código NÃO tem segurança e nem validação e NÃO deve ser usado em ambientes reais.

Lembrete:
    1. `const ses = new SESClient({ region: "us-east-1" });` - Selecionar a mesma região onde o SES esta configurado
    2.  `const RECEIVER = "EMAIL-QUE-VAI-RECEBER";`
        `const SENDER = "EMAIL-QUE-VAI-MANDAR";`
        Trocar por emails válidos

Dei deploy no codigo e testei criando um evento com dados imaginários:

![Lambda function test event page config](../img/28-lambda-function-test-event.png)

![Lambda function test succeeded response](../img/29-lambda-function-test-succeeded.png)

Se tudo deu certo, vamos receber um email com os dados capturados no teste:

![Reciever email inbox](../img/30-gmail-test.png)