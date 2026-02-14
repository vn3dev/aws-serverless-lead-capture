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

