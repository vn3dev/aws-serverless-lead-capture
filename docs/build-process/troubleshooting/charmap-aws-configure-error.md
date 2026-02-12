## 'charmap' codec can't encode character '\x96'

![charmap error message](../img/charmap-error.png)

If, when configuring the AWS access keys in Git Bash, you encounter the error:
`'charmap' codec can't encode character '\x96'`
It is a Windows terminal encoding issue (cp1252).

The terminal is probably using an old encoding. In that case, run:

`export PYTHONIOENCODING=utf-8`

Then, try running aws configure again.

--- 
🇧🇷

Se quando configurar as chaves de acesso da aws no git bash encontrar o erro:
`'charmap' codec can't encode character '\x96'`
É um problema de encoding do terminal do Windows (cp1252)

Provavelmente o terminal esta usando encoding antigo. Nesse caso, rode:

`export PYTHONIOENCODING=utf-8`

Depois, tente rodar `aws configure` novamente