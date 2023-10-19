---
layout: post
title: Vault - Habilitando, Configurando e utilizando a Interface de Usuário via web.
date: '2020-05-03 14:48:29'
tags:
- devops
- iac
---

Como dito anteriormente, o Vault disponibiliza acesso aos dados através de Interface Gráfica, Linha de Comando ou Chamada HTTP em sua API.  
  
Hoje iremos habilitar a interface gráfica (web) para nosso Vault server criado no post [**Vault 101 - Aplicando segurança na sua Infraestrutura como Código**](/vault-101/) **.**

> Caso você tenha chegado agora no blog, verifique todos os posts na [**TAG: IAC**](/tag/iac/) , com certeza você encontrará vários conteúdos interessantes por lá.

## Primeiros Passos

Primeiramente iremos ligar e acessar nossa máquina Vault, bem como Destravar o cofre de segredos.

> Caso não tenha a máquina ainda, a recomendação é que siga o post do **[vault-101](/vault-101/).**

Abra um novo terminal e ligue a máquina vault

<!--kg-card-begin: markdown-->

    $ cd ~/vault-101
    $ vagrant up
    $ vagrant ssh

<!--kg-card-end: markdown-->
## Destravando o Cofre

Agora vamos destravar o cofre com 3 das 5 keys que foram geradas anteriormente através do comando `vault operator unseal`

> Note que as keys são diferentes para cada instalação do vault, no meu caso utilizarei as mesmas do tutorial.

<!--kg-card-begin: markdown-->

    $ vault operator unseal -address=http://10.10.10.10:8200

<!--kg-card-end: markdown-->

Execute o comando acima 3 vezes e informe três chaves diferentes, no meu caso o comando ficou

<!--kg-card-begin: markdown-->

    $ vault operator unseal -address=http://10.10.10.10:8200
    > F9Yo6vPumyQfuhTlTqcgkJhG3s02oeKJ6hNmkDKzItDE
    
    $ vault operator unseal -address=http://10.10.10.10:8200
    > tuRLZjNOIv2DPO+TD3P3h9Gv1QQSL8rsFhjAe5q4CheH
    
    $ vault operator unseal -address=http://10.10.10.10:8200
    > qA1TZ5J9iE2V3R6y/5DJh0i29MPTnGLsF/hvia2sOVAS

<!--kg-card-end: markdown-->

Ao final do processo deve ser exibida a mensagem informando que o cofre não está mais no estado `sealed` (lacrado)

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image.png" class="kg-image" alt loading="lazy"><figcaption>Vault Sealed: False</figcaption></figure>
## Habilitando a WEB UI

Para habilitar **WEB UI** (Web User Interface) precisamos editar o arquivo de configuração adicionando o campo `ui = true` e então reiniciar o serviço do vault.

<!--kg-card-begin: markdown-->

    $ sudo vim /etc/vault.d/config.hcl

    storage "mysql" {
      username = "vaultuser"
      password = "caiodelgado.dev"
      database = "vault"
    }
    
    listener "tcp" {
     address = "10.10.10.10:8200"
     tls_disable = 1
    }
    
    ui = true

    $ sudo systemctl restart vault

<!--kg-card-end: markdown-->

Uma vez reiniciado o vault podemos efetuar novamente o procedimento de destravar o cofre que fizemos anteriormente, mas desta vez faremos pela interface web.

> Não é necessário efetuar o procedimento de destravar o cofre antes de configurar a interface web, fizemos isto apenas para mostrar a diferença dos dois modos.

Acesse o endereço [http://vault.caiodelgado.example:8200](http://vault.caiodelgado.example:8200) pelo navegador web, será exibida uma tela solicitando uma porção da chave para remover o selo do cofre, digite 3 das 5 chaves para efetuar o desbloqueio do cofre, e acompanhe o progresso.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-2.png" class="kg-image" alt loading="lazy"><figcaption>http://vault.caiodelgado.example:8200</figcaption></figure>

Uma vez desbloqueado o cofre do Vault, será solicitado o login do usuário, podemos utilizar diversos métodos de login como:

- Token
- Username & Password
- LDAP
- Okta
- JWT
- OIDC
- Radius
- Github

Iremos escolher o médoto token e utilizar o token do usuário root, criado no tutorial **[vault-101](/vault-101).**

> Note que o token é diferente para cada instalação/usuário do vault do vault.

Digite o token e clique em **Sign In.**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-3.png" class="kg-image" alt loading="lazy"><figcaption>Sign In</figcaption></figure>
## Conhecendo a WEB UI

Agora que estamos com nossa WEB UI operacional, podemos explorar a mesma, um fato interessante é que o próprio vault mostra no canto direito da tela um pequeno guia para os primeiros passos.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-4.png" class="kg-image" alt loading="lazy"><figcaption>Vault WEB UI</figcaption></figure>

Na barra superior do Vault, temos acesso aos **secrets, métodos de acesso, políticas e ferramentas.**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-6.png" class="kg-image" alt loading="lazy"><figcaption>Barra Superior (parte esquerda)</figcaption></figure>

Também na barra superior mais para a parte direita, temos acesso ao **status do servidor, linha de comando** e **configurações do usuário.**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-7.png" class="kg-image" alt loading="lazy"><figcaption>Barra Superior (parte direita)</figcaption></figure>
## Secrets

No menu de secrets, podemos ter o acesso ao secret e navegar entre o mesmo, também é possível navegar e verificar o secret bem como copiar, deletar, editar e verificar seu valor como json.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-8.png" class="kg-image" alt loading="lazy"><figcaption>vault/secrets/blog/caiodelgado</figcaption></figure>
## Access

No menu de acesso, podemos configurar métodos de autenticação, entidades, grupos e concessões.

Em métodos de autenticação, vamos adicionar um novo método clicando em **Enable new method +**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-9.png" class="kg-image" alt loading="lazy"><figcaption>Access/Enable new method +</figcaption></figure>

Por hora criaremos através do método **Username & Password** , selecione e clique em **Next**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-10.png" class="kg-image" alt loading="lazy"><figcaption>Username &amp; Password.</figcaption></figure>

Clique em **Enable Method.**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-11.png" class="kg-image" alt loading="lazy"><figcaption>Enable Method</figcaption></figure>

Clique agora em **View method.**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-12.png" class="kg-image" alt loading="lazy"><figcaption>View method</figcaption></figure>

Vamos criar o nosso primeiro usuário, para isto, clique em **Create User +**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-13.png" class="kg-image" alt loading="lazy"><figcaption>Create user +</figcaption></figure>

Vamos criar o usuário **user01** com a senha **caiodelgado.dev** , digite os dados e clique em **Save**

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-14.png" class="kg-image" alt loading="lazy"><figcaption>Create user</figcaption></figure>

Podemos testar agora o login através da **cli** ou pela **web ui** deste novo usuário, por hora faremos via **cli** através da nossa máquina host

<!--kg-card-begin: markdown-->

    $ export VAULT_ADDR='http://vault.caiodelgado.example:8200'
    $ vault login -method=userpass username=user01
    > caiodelgado.dev

<!--kg-card-end: markdown--><figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-15.png" class="kg-image" alt loading="lazy"><figcaption>vault login</figcaption></figure>
## Policies

Através do menu **Policies** podemos definir politicas de acesso através de um json.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-16.png" class="kg-image" alt loading="lazy"><figcaption>Vault/Policies</figcaption></figure>

> Podemos verificar a syntax através da**[Documentação Oficial](https://www.vaultproject.io/docs/concepts/policies.html#policy-syntax)**

## Tools

No menu de **Tools** temos diversas ferramentas que podem nos ajudar a criar, gerar dados randomicos ou até mesmo hashes.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-17.png" class="kg-image" alt loading="lazy"><figcaption>Vault/Tools</figcaption></figure>
## Status

Clicando em **Status** podemos ver se nosso cofre está selado ou não, bem como as métricas (caso o cofre esteja destravado)

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-18.png" class="kg-image" alt loading="lazy"><figcaption>Vault/Status</figcaption></figure>
## CLI

No menu de **CLI** podemos executar comandos pela interface **WEB.** O interessante deste menu é que como já estamos logados na **WEB,** não precisamos efetuar novamente o login via **CLI**.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-19.png" class="kg-image" alt loading="lazy"><figcaption>WEB CLI</figcaption></figure>

Os comandos da **web cli** são um pouco diferentes dos comandos da **cli** , para visualizar a ajuda basta digitar `vault help`

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-20.png" class="kg-image" alt loading="lazy"><figcaption><strong>vault help</strong></figcaption></figure>
## User

No menu de **USER** podemos visualizar novamente o guia, copiar o token ou deslogar o usuário atual.

<figure class="kg-card kg-image-card"><img src="/assets/2020/05/image-23.png" class="kg-image" alt loading="lazy"></figure>

Este post é o segundo da série de posts onde utilizaremos o **vault.**

Nos próximos posts iremos integrar o **vault** com o **terraform** e também gerenciar credenciais da **AWS** de forma dinâmica.

Ficamos por aqui com esse post e nos vemos em uma próxima!

