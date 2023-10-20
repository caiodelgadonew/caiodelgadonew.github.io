---
layout: post
title: Vault - Gerenciando Acesso a AWS com Dynamic Secrets
date: '2020-05-17 14:53:40'
tags:
- devops
- iac
- sre
---

O Vault tem diversas funcionalidades e dizer que ele é apenas um "cofre de senhas" não é correto, uma vez que ele tem todo um motor e diversas integrações  
  
Hoje iremos integrar nosso Vault server criado no post **[Vault 101](/posts/vault-101/)** com a AWS utilizada no post [**Terraform 101**](/terraform-101) para que possamos gerenciar o acesso a AWS de forma segura utilizando **Dynamic Secrets**.

> Caso você tenha chegado agora no blog, verifique todos os posts na [**TAG: IAC**](/tags/iac/) , com certeza você encontrará vários conteúdos interessantes por lá.

## Primeiros Passos

Primeiramente iremos ligar e acessar nossa máquina Vault, bem como Destravar o cofre de segredos.

> Caso não tenha a máquina ainda, a recomendação é que siga o post do **[vault-101](/posts/vault-101/).**

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

## Habilitando a Engine de Secrets da AWS

Agora que temos nossa máquina disponível, podemos então sair da máquina server e executar os comandos diretamente em nossa máquina hospedeira através do vault.

Na sua máquina, efetue o login no vault através do script criado no post **[Vault-101](/vault-101)** e verifique se a conexão esta ok.

<!--kg-card-begin: markdown-->

    $ cd ~/vault/
    $ source configurevault.sh
    $ vault status

<!--kg-card-end: markdown-->

Agora que estamos conectados ao servidor do Vault, podemos habilitar a engine de secrets da AWS através do comando

<!--kg-card-begin: markdown-->

    $ vault secrets enable -path=aws aws

<!--kg-card-end: markdown-->

Uma mensagem informará que a engine foi habilitada com sucesso

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-24.png" class="kg-image" alt loading="lazy"><figcaption>$ vault secrets enable -path=aws aws</figcaption></figure>
## Configurando a Engine de Secrets da AWS

Com a engine habilitada, precisamos configura-la, para isto utilizaremos a chave de acesso com `Full Access` criada anteriormente

> Caso você queira criar uma nova chave de acesso, basta seguir os passos no post **[Terraform 101](/terraform-101)**

<!--kg-card-begin: markdown-->

    $ vault write aws/config/root \
        access_key=AKIARR2QTEG7UHP3KLKE \
        secret_key=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 \
        region=us-east-1

<!--kg-card-end: markdown-->

> Altere o `access_key` e o `secret_key` pelos valores da sua conta, **por questões de &nbsp;segurança este usuário não existe mais em minha conta.**

Com os dados de acesso a AWS, precisamos configurar uma role. O Vault sabe como criar os usuários no _IAM_ através da API da AWS, mas não sabe quais permissões, grupos e políticas anexar a este usuário. Iremos criar uma politica para dar acesso full a **EC2** para os usuários criados.

> Para mais informações sobre politicas da AWS verifique a **[Documentação Oficial](https://docs.aws.amazon.com/pt_br/IAM/latest/UserGuide/access_policies.html).**

<!--kg-card-begin: markdown-->

    vault write aws/roles/ec2-full-access \
            credential_type=iam_user \
            policy_document=-<<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "Stmt1426528957000",
          "Effect": "Allow",
          "Action": [
            "ec2:*"
          ],
          "Resource": [
            "*"
          ]
        }
      ]
    }
    EOF

<!--kg-card-end: markdown--><figure class="kg-card kg-image-card"><img src="/assets/2020/05/image-27.png" class="kg-image" alt loading="lazy"></figure>
## Gerando a credencial temporária para acesso a AWS

Com a engine de secrets da AWS configurada no **Vault** com uma role, podemos solicitar que o Vault faça a geração de uma `access_key` e `secret_key` temporária para esta role ao ler o caminho `aws/creds/<nome>` onde `<nome>` corresponde ao nome da role existente, no nosso caso `ec2-full-access`

<!--kg-card-begin: markdown-->

    $ vault read aws/creds/ec2-full-access

<!--kg-card-end: markdown--><figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-29.png" class="kg-image" alt loading="lazy"><figcaption>$ vault read aws/creds/ec2-full-access</figcaption></figure>

Podemos então utilizar a `access_key` e `secret_key` para acessar a AWS. Podemos também verificar diretamente no painel do _IAM_ na AWS o user criado.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-30.png" class="kg-image" alt loading="lazy"><figcaption>AWS IAM Users</figcaption></figure>

Podemos também revogar a credencial gerada através do comando `vault lease revoke`

<!--kg-card-begin: markdown-->

    $ vault lease revoke aws/creds/ec2-full-access/nHBDTgMmFylGhec6KgISzkvB

<!--kg-card-end: markdown--><figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-31.png" class="kg-image" alt loading="lazy"><figcaption>$ vault lease revoke aws/creds/ec2-full-access/nHBDTgMmFylGhec6KgISzkvB</figcaption></figure>
## Bônus: Criando uma politica aws-read-only e aws-full-acccess

Agora que temos nossa política **ec2-full-access** criada, seria interessante ter uma política **ec2-read-only** para limitarmos o usuário a apenas visualizar os dados da EC2 e uma para dar acesso total a todos os recursos da AWS.

Vamos criar nossa politica **ec2-read-only**

    $ vault write aws/roles/ec2-read-only \
            credential_type=iam_user \
            policy_document=-<<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "Stmt1426528957000",
          "Effect": "Allow",
          "Action": [
            "ec2:Get*",
            "ec2:List*",
            "ec2:Generate*"
          ],
          "Resource": [
            "*"
          ]
        }
      ]
    }
    EOF

Utilize o comando `$ vault read aws/creds/ec2-read-only` para gerar as credenciais

Vamos criar agora a política **aws-full-access**

    $ vault write aws/roles/aws-full-access \
            credential_type=iam_user \
            policy_document=-<<EOF
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "Stmt1426528957000",
          "Effect": "Allow",
          "Action": [
            "*"
          ],
          "Resource": [
            "*"
          ]
        }
      ]
    }
    EOF

Utilize o comando `$ vault read aws/creds/aws-full-access` para gerar as credenciais.

Este post é o terceiro da série de posts onde utilizaremos o **Vault.**

Nos próximos posts iremos integrar o **Vault** com o **Terraform** para gerar de forma segura os recursos na **AWS.**

Ficamos por aqui com esse post e nos vemos em uma próxima!

