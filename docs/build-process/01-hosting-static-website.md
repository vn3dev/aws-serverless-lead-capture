Com os pre-requisitos cumpridos, comecei a criar a infra do website.

Decidi fazer grande parte das configurações por CLI, para praticar e tornar a reprodução mais ágil em um ambiente corporativo. Tudo que fiz pode ser feito via console de forma visual.

## Passo 1 - Criar e configurar bucket:

`aws s3 ls` - Para ver os buckets ja existentes
`aws s3 mb s3://SEU_BUCKET` - Para criar o bucket. O nome precisa ser globalmente único, talvez seja necessário serializar

Ativei o website estatico e configurei os arquivos de erro e de index:

`aws s3api put-bucket-website --bucket SEU_BUCKET --website-configuration '{"IndexDocument":{"Suffix":"index.html"},"ErrorDocument":{"Key":"error.html"}}'`

Você pode conferir se deu certo via comando:

`aws s3api get-bucket-website --bucket SEU_BUCKET`

Ou, via console nas properties:

![Console static website](../img/01-static-website-console.png)

O s3 já esta ativo, mas ainda não mandei os arquivos e nem terminei as configurações de acesso. Se tentarmos acessar pela URL, receberemos um 403:

![Erro 403 ao acessar a URL](../img/02-403.png)

