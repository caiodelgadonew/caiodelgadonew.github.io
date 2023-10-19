---
layout: post
title: 'Docker & Traefik 2.0: Proxy-Reverso + Let''s Encrypt + HTTP to HTTPS redirect'
date: '2020-03-25 04:51:56'
tags:
- iac
- sre
- cloud
- docker
---

**[Traefik](https://traefik.io)** é um roteador de borda open-source que facilita a publicação de serviços de maneira fácil. Um ponto bem interessante é que ele integra nativamente com a maioria das tecnologias de cluster, tais como **Kubernetes, Docker, Docker Swarm, AWS, Mesos, Marathon** e [**muito mais**](https://docs.traefik.io/providers/overview/) **.**

O Traefik traz bastante ganho com relação aos softwares comumente utilizados como proxy reverso pois o mesmo já nasceu com práticas **Cloud Native,** ou seja, voltado para computação em nuvem.

Para entender o funcionamento do traefik, primeiro precisamos entender componentes como **providers, routers, services, middlewares, certificados e entrypoints.**

**Providers** são componentes existentes da infraestrutura, podendo ser orquestradores, provedores de cloud dentre outros os quais o traefik vai interagir através de sua _API (application programming interface)_ para encontrar informações relevantes sobre o roteamento e como o traefik irá detectar e reagir a estas mudanças.

**Routers** são responsáveis por conectar as requisições de entradas aos **serviços** que irão lidar com os dados recebidos, durante este processo os roteadores poderão utilizar **middlewares** para atualizar a requisição, atuar antes do encaminhamento do request para o serviço e até mesmo controlar acesso através de um dispositivo de autenticação.

**Certificados** são utilizados para criptografia **TLS** _(Transport Layer Security)_ dos dados enviados para o servidor a fim de garantir uma camada de segurança, o traefik possúi integração com o **[Let's Encrypt](https://letsencrypt.org/)** que utilizaremos para gerar o certificado do website.

**Entrypoints** são os pontos de entrada que o traefik estará escutando as requisições de entrada para encaminhar a um determinado router.

* * *

### Vamos a prática

> Em todos os laboratórios deste blog utilizamos distribuições de linux, também utilizamos os comandos como super user, em ambientes de produção lembre-se de tomar as precauções de segurança.

Em nosso exemplo, utilizaremos uma máquina na **GCP** _(Google Cloud Platform)_ rodando o Docker para efetuar a configuração, você pode utilizar uma máquina em qualquer cloud ou até mesmo a máquina local.

_obs.: Para realizar a etapa de configuração do Let's Encrypt é necessário um DNS válido, ou seja, você precisa ter um registro de domínio válido e apontar o mesmo para a máquina que está executando o traefik._

O primeiro passo é acessar a máquina via **SSH** e efetuar a instalação do Docker na mesma através do comando:

    curl -fsSL https://get.docker.com | sudo bash

Lembrando que este comando não é recomendado para ambientes de produção, uma vez que o mesmo instala o docker com a sua configuração padrão, para ambientes de produção utilize as instruções da[**Documentação Oficial**](https://docs.docker.com/install/).

Após a instalação do Docker iremos instalar uma ferramenta que será responsável por executar subir nossa estrutura e parametrizar o nosso ambiente. Instale o **docker-compose** através dos comandos:

    sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    
    sudo chmod +x /usr/local/bin/docker-compose
    
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

Após instalar o docker compose iremos criar um diretório para guardar os arquivos do nosso ambiente.

    sudo su -
    mkdir /root/traefik
    cd /root/traefik

Dentro desta pasta, precisaremos criar um arquivo **docker-compose.yml** que será responsável por subir o ambiente e um arquivo chamado **acme.json** que é onde serão armazenados os certificados do Let's Encrypt.

    touch acme.json
    chmod 600 acme.json

Antes de criar o arquivo **docker-compose.yml** iremos entender primeiramente o que é necessário para configuração do nosso traefik

<!--kg-card-begin: markdown-->

      traefik:
        image: "traefik:v2.0.0"
        command:
          - --entrypoints.web.address=:80
          - --entrypoints.websecure.address=:443
          - --providers.docker
          - --api
          - --certificatesresolvers.leresolver.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory
          - --certificatesresolvers.leresolver.acme.email=your@email.com
          - --certificatesresolvers.leresolver.acme.storage=/acme.json
          - --certificatesresolvers.leresolver.acme.tlschallenge=true
        ports:
          - "80:80"
          - "443:443"
        volumes:
          - "/var/run/docker.sock:/var/run/docker.sock:ro"
          - "./acme.json:/acme.json"

<!--kg-card-end: markdown-->

As opções inseridas no command, realizam a configuração da parametrização básica do traefik na seguinte sequência:

1. Habilita o entrypoint web na porta 80
2. Habilita o entrypoint websecure na porta 443
3. Configura o provider como docker
4. Configura a API
5. Configura o provedor de certificados (servidor, email, storage, challenge)

É importante notar que temos uma linha com a configuração do servidor de staging do Let's Encrypt, utilizamos esta configuração apenas para testes, uma vez que o servidor de produção do Let's Encrypt tem configurado um [**rate limit**](https://letsencrypt.org/docs/rate-limits/), ou seja, caso sejam solicitados mais certificados do que seu limite semanal ele para de fornecer novos certificados. Quando seu ambiente for para produção pode-se remover a linha:

      - --certificatesresolvers.leresolver.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory

O container do **traefik** também receberá algumas **labels** para ativar recursos como:

**Dashboard**  
_(Endereço de acesso, configuração de tls, entrypoints e caso necessário bloqueio por senha)_

<!--kg-card-begin: markdown-->

    - "traefik.http.routers.traefik.rule=Host(`traefik.docker.localhost`)"
    - "traefik.http.routers.traefik.service=api@internal"
    - "traefik.http.routers.traefik.tls.certresolver=leresolver"
    - "traefik.http.routers.traefik.entrypoints=websecure"

<!--kg-card-end: markdown-->

**Redirect Global HTTP -\> HTTPS**  
_(Regras para redirecionamento de todos os hosts de http para https)_

<!--kg-card-begin: markdown-->

    - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
    - "traefik.http.routers.http-catchall.entrypoints=web"
    - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
    - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"

<!--kg-card-end: markdown-->

Aos containers que estarão recebendo tráfego do traefik também precisam de labels para configurar o roteamento, o acesso e o certificado

<!--kg-card-begin: markdown-->

    - "traefik.http.routers.my-app.rule=Host(`appname.docker.localhost`)"
    - "traefik.http.routers.my-app.middlewares=auth"
    - "traefik.http.routers.my-app.entrypoints=websecure"
    - "traefik.http.routers.my-app.tls=true"
    - "traefik.http.routers.my-app.tls.certresolver=leresolver"

<!--kg-card-end: markdown-->

Agora que entendemos as labels e configurações necessárias, iremos criar nosso arquivo **docker-compose.yml**

    vim docker-compose.yml

<!--kg-card-begin: markdown-->

    version: "3.3"
    
    services:
      traefik:
        image: "traefik:v2.0.0"
        command:
          - --entrypoints.web.address=:80
          - --entrypoints.websecure.address=:443
          - --providers.docker
          - --api
          - --log.level=DEBUG
          - --certificatesresolvers.letsencrypt.acme.email=email@provider.example
          - --certificatesresolvers.letsencrypt.acme.storage=/acme.json
          - --certificatesresolvers.letsencrypt.acme.tlschallenge=true
        ports:
          - "80:80"
          - "443:443"
        volumes:
          - "/var/run/docker.sock:/var/run/docker.sock:ro"
          - "./acme.json:/acme.json"
        labels:
          # Dashboard
          - "traefik.http.routers.traefik.rule=Host(`dashboard.caiodelgado.com.br`)"
          - "traefik.http.routers.traefik.service=api@internal"
          - "traefik.http.routers.traefik.tls.certresolver=letsencrypt"
          - "traefik.http.routers.traefik.entrypoints=websecure"
          - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
          - "traefik.http.routers.http-catchall.entrypoints=web"
          - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
          - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
    
      webapp:
        image: containous/whoami:v1.3.0
        labels:
          - "traefik.http.routers.app1.rule=Host(`web.caiodelgado.com.br`)"
          - "traefik.http.routers.app1.entrypoints=websecure"
          - "traefik.http.routers.app1.tls=true"
          - "traefik.http.routers.app1.tls.certresolver=letsencrypt"
    

<!--kg-card-end: markdown-->

_Lembre-se de alterar os campos de e-mail e as rules por container para seu hostname bem como criar as entradas no DNS para apontar diretamente para estes nomes._

<!--kg-card-begin: markdown-->

    certificatesresolvers.letsencrypt.acme.email=you@email.com
    traefik.http.routers.traefik.rule=Host(`dashboard.example.com`)
    traefik.http.routers.app.rule=Host(`app.example.com`)

<!--kg-card-end: markdown-->

Após a criação do arquivo **docker-compose.yml** podemos prosseguir com a criação do ambiente

    docker-compose up -d 

Podemos acessar agora o dashboard através do endereço configurado no parâmetro **traefik.http.router.traefik.rule** caso as entradas tenham sido criadas corretamente no servidor DNS

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/03/traefik1.png" class="kg-image" alt loading="lazy"><figcaption>Traefik Dashboard</figcaption></figure>

Através do dashboard podemos verificar informações sobre os routers, services e middlewares criados no nosso traefik, verifique também que o website já está com o certificado válido do Let's Encrypt.

<figure class="kg-card kg-image-card"><img src="/assets/2020/03/letsencrypt.png" class="kg-image" alt loading="lazy"></figure>

Podemos também verificar em **Routers** clicando em **Explore** os roteadores criados bem como para onde estão encaminhando cada requisição.

<figure class="kg-card kg-image-card"><img src="/assets/2020/03/routers.png" class="kg-image" alt loading="lazy"></figure><figure class="kg-card kg-image-card"><img src="/assets/2020/03/router1.png" class="kg-image" alt loading="lazy"></figure>

E também podemos acessar esta aplicação através do Host criado, verifique também que a aplicação é automaticamente redirecionada de http para https

<figure class="kg-card kg-image-card"><img src="/assets/2020/03/web.png" class="kg-image" alt loading="lazy"></figure>

Caso seja necessário escalar nossa aplicação, o próprio traefik ficará responsável por efetuar o balanceamento de cargas da aplicação.

Execute no terminal o comando para aumentar a quantidade de replicas para 5 da aplicação web.

<!--kg-card-begin: markdown-->

    docker-compose up -d --scale webapp=5

<!--kg-card-end: markdown-->

Atualize a página da aplicação web e verifique que o IP será alterado informando que estamos caindo em containers diferentes.

<figure class="kg-card kg-image-card"><img src="/assets/2020/03/web1.png" class="kg-image" alt loading="lazy"></figure><figure class="kg-card kg-image-card"><img src="/assets/2020/03/web2.png" class="kg-image" alt loading="lazy"></figure><figure class="kg-card kg-image-card"><img src="/assets/2020/03/web3.png" class="kg-image" alt loading="lazy"></figure>

Outra coisa que podemos fazer para aumentar a segurança do nosso dashboard é adicionar um middleware de autenticação através das seguintes labels:

<!--kg-card-begin: markdown-->

    "traefik.http.routers.traefik.middlewares=authtraefik"
    "traefik.http.middlewares.authtraefik.basicauth.users=user:hashpassword"

<!--kg-card-end: markdown-->

A senha é armazenada no formato **user:hashpassword** e para criar o password criptografado podemos instalar o pacote **apache2-utils** e utilizar a ferramenta **htpasswd** para gerar a senha _(também é preciso que dobremos a quantidade de simbolos $ devido a syntax do yaml)_

<!--kg-card-begin: markdown-->

    apt-get update
    apt-get install apache2-utils
     
    echo $(htpasswd -nbB user "password") | sed -e s/\\$/\\$\\$/g

<!--kg-card-end: markdown-->

A saída do comando adicionaremos ao basicauth e ao docker-compose.yml na seção de labels do container traefik. _(no exemplo o usuário está configurado como_ **_admin_** _e a senha como_ **_caiodelgado.dev_** _)_

<!--kg-card-begin: markdown-->

        labels:
        (...)
          - "traefik.http.routers.traefik.middlewares=authtraefik"
          - "traefik.http.middlewares.authtraefik.basicauth.users=admin:$$2y$$05$$tyQaJegMouMyC902e6ZJL.hieuhcezgXXxwydtjmC0DeMKG7ezhtm"
    
    

<!--kg-card-end: markdown-->

Podemos subir novamente nosso compose e verificar na interface web se a senha é solicitada.

<!--kg-card-begin: markdown-->

    docker-compose up -d

<!--kg-card-end: markdown--><figure class="kg-card kg-image-card"><img src="/assets/2020/03/pass.png" class="kg-image" alt loading="lazy"></figure>

Após digitar a senha o painel é exibido com sucesso.

Agora basta adicionar novos serviços e garantir que as labels necessárias estarão configuradas diretamente no compose.

<!--kg-card-begin: markdown-->

    - "traefik.http.routers.my-app.rule=Host(`appname.docker.localhost`)"
    - "traefik.http.routers.my-app.middlewares=auth"
    - "traefik.http.routers.my-app.entrypoints=websecure"
    - "traefik.http.routers.my-app.tls=true"
    - "traefik.http.routers.my-app.tls.certresolver=leresolver"

<!--kg-card-end: markdown-->

Ficamos por aqui com esse post e nos vemos em uma próxima!

