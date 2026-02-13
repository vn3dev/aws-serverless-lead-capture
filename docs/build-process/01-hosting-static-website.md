Com os pre-requisitos cumpridos, comecei a criar a infra do website.

## Passo 1 - Criar e configurar bucket:

`aws s3 ls` - Para ver os buckets ja existentes

`aws s3 mb s3://MEU_BUCKET` - Para criar o bucket. O nome precisa ser globalmente único, talvez seja necessário serializar

Ativei o website estatico e configurei os arquivos de erro e de index:

`aws s3api put-bucket-website --bucket MEU_BUCKET --website-configuration '{"IndexDocument":{"Suffix":"index.html"},"ErrorDocument":{"Key":"error.html"}}'`

Você pode conferir se deu certo via comando:

`aws s3api get-bucket-website --bucket MEU_BUCKET`

Ou, via console nas properties:

![Console static website](../img/01-static-website-console.png)

O s3 já esta ativo, mas ainda não mandei os arquivos e nem terminei as configurações de acesso. Se tentarmos acessar pela URL, receberemos um 403:

![Erro 403 ao acessar a URL](../img/02-403.png)

Por padrão, o s3 bloqueia o acesso publico. Para desativar o bloqueio, rodei:

`aws s3api put-public-access-block --bucket MEU_BUCKET --public-access-block-configuration '{"BlockPublicAcls":false,"IgnorePublicAcls":false,"BlockPublicPolicy":false,"RestrictPublicBuckets":false}'`

Ainda preciso configurar as politicas do s3. Para isso:

`aws s3api put-bucket-policy --bucket MEU_BUCKET --policy '{"Version":"2012-10-17","Statement":[{"Sid":"PublicReadGetObject","Effect":"Allow","Principal":"*","Action":["s3:GetObject"],"Resource":["arn:aws:s3:::MEU_BUCKET/*"]}]}'`

Agora o bucket esta configurado e com acesso público. Se tentar acessar pela URL o erro muda para 404. Agora precisamos subir os arquivos para o s3 encontrar o index.html e o error.html. 

![Erro 404 ao acessar a URL](../img/03-404.png)

Para isso, rodei:

`aws s3 cp error.html s3://MEU_BUCKET`

Ao acessar a pagina, ja consigo ver o html de erro:

![HTML de erro](../img/04-error-page.png)

Com a mensagem de erro funcionando, vou subir os arquivos do website usando:

`aws s3 sync . s3://MEU_BUCKET` - Troque "." pelo diretório do app, se já não estiver nele

Se tudo deu certo, o site já esta funcionando:

![Website working homepage](../img/05-hosted-website.png)

O website esta usando o protocolo HTTP. Um dos requisitos é utilizar HTTPS. Para isso, vou precisar de um certificado público da AWS

Vou precisar de um domínio. Eu não pude usar o Route 53 pois ele não esta incluso no free tier da AWS. Por isso, registrei um dominio na Hostinger

Com o dominio em mãos, fui até o ACM e fiz um request para conseguir o certificado:

![ACM list certificates pages](../img/06-empty%20certificates.png)

![ACM public certificate](../img/07-request-certificate.png)

Coloquei o dominio da Hostinger. Incluí um com www:

![ACM domain names](../img/08-domain-names.png)

![ACM cnames aws](../img/09-cname.png)

Agora preciso confirmar a posse do dominio. Para isso, preciso criar um CNAME no painel da Hostinger para cada dominio que fiz a request

Coloquei o mesmo nome e o apontamento que a AWS mostrou no console:

![Hostinger CNAME config](../img/10-cname-hostinger.png)

No console AWS, temos que esperar a propagação do DNS para validar. Deve demorar uns 10 minutos:

![ACM pending validation](../img/11-pending%20verification.png)

![ACM issued validation](../img/12-issued-certificate.png)

Com o certificado validado, vamos configurar o CloudFront para poder privar o s3 e direcionar o tráfego. Na pagina do CloudFront, criei uma nova distribuição:

![CloudFront homepage](../img/13-cloudfront-homepage.png)

Selecionei o s3 que criamos e prossegui. Não usei website endpoint, vou deixar o bucket privado assim que o DNS estiver apontando para o CloudFront

![CloudFront configuration page](../img/14-cloudfront-config.png)

Depois de criado, vá em general e edite as settings:

![CloudFront general settings](../img/15-cloudfront-edit-settings-1.png)

![CloudFront general settings2](../img/16-cloudfront-edit-settings-2.png)

Agora precisamos fazer o dominio apontar para o CloudFront. Na hostinger, criei dois novos registros:

![Hostinger registries](../img/17-hostinger-registry.png)

Agora aguardar até o DNS propagar... Podemos checar no console com `nslookup DOMINIO` para ver se o ip ja atualizou

Confirmei que o dominio estava funcionando e prossegui deixando o bucket privado para permitir o acesso apenas pelo CloudFront

No console AWS, fui no meu bucket e em permissions. Bloqueei o acesso público e alterei as politicas para permitir apenas o CloudFront:

![Bucket block public access](../img/18-bucket-block-public.png)

![Bucket policy edit](../img/19-bucket-policy.png)

```Bucket Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::vn3project-ebook/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::293012440829:distribution/E23ZHPMGO5C5SG"
                }
            }
        }
    ]
}
```

Testando para ver se o acesso direto ao bucket retorna 403:

![Bucket web response](../img/20-bucket-403-web.png)

![Bucket response](../img/21-bucket-403.png)

![Bucket index response](../img/22-bucket-403-index.png)