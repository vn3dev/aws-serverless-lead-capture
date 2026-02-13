Com os pre-requisitos cumpridos, comecei a criar a infra do website.

Decidi fazer grande parte das configurações por CLI, para praticar e tornar a reprodução mais ágil em um ambiente corporativo. Tudo que fiz pode ser feito via console de forma visual.

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