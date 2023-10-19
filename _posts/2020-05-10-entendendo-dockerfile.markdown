---
layout: post
title: Entendendo o Dockerfile
date: '2020-05-10 15:37:59'
tags:
- devops
- iac
- sre
- docker
---

Agora que você já deu os [**Primeiros Passos com Docker e muito mais**](/docker-101/) **.** Vamos começar a cobrir cada uma das áreas do Docker para você poder se aprofundar neste universo.

A primeira área será a de **Docker Images** começando com o **Dockerfile**.

O Dockerfile é um documento de texto que contêm todos os comando sque um usuário pode chamar na linha de comando para criar uma imagem.

## Instruções

Por convenção escrevemos as instruções do **Dockerfile** em letras maiúsculas para facilitar a leitura do mesmo, porém não é necessário, utilizaremos este padrão para diferenciar as instruções dos parâmetros.

### FROM

A instrução `FROM` inicializa um novo estágio de construção de imagem e configura a imagem base para as instruções subsequentes. Todo **Dockerfile** deve começar com uma instrução `FROM`, que faz o download da imagem de um repositório (normalmente o [dockerhub](https://hub.docker.com/))

    FROM alpine

### LABEL

Podemos utilizar a instrução `LABEL` para adicionar labels a imagem de forma a ajudar a organização da imagem, informar o mantenedor da mesma, listar informações de licenciamento ou até mesmo ajudar em uma automação.   
Para cada label adicionada precisamos criar uma nova linha iniciando com `LABEL`

    LABEL dev.caiodelgado.version="1.0.0.23"
    LABEL maintainer="Caio Delgado"
    LABEL licence="GPLv3" 
    LABEL dev.caiodelgado.release-date="2020-05-11"

> Existe uma instrução chamada `MAINTAINER` que servia para informar o mantenedor da imagem, porem esta opção foi depreciada uma vez que a instrução `LABEL` é mais flexivel.

### RUN

A opção `RUN` executa qualquer comando em uma nova camada no da imagem e faz o commit do resultado. Ela possui dois formatos de utilização:

- `RUN <comando>` executa o comando por padrão em um shell, que por padrão é `/bin/sh` no linux ou `cmd /S /C` no windows.
- `RUN ["<executavel>", "<parametro1>", "<parametro2>"]` executa o comando com o executavel listado.

    RUN apt-get update \
        apt-get install curl -y
    RUN ["/bin/bash", "-c", "echo caiodelgado.dev"]

> É importante salientar que a cada instrução `RUN` executada uma nova camada de imagem é criada, como boa prática podemos utilizar a contrabarra `\` para continuar uma única instrução run para a próxima linha, também é uma boa pratica usar operadores shell como `||`, `&&`, etc.. para resumir os comandos run a uma linha única. Em um post futuro falaremos sobre melhores praticas na criação de imagens.

### ADD / COPY

As instruções `ADD` e `COPY` são bastante similares, geralmente é mais utilizada a instrução `COPY` por trabalhar de forma mais transparente que a `ADD`.

A instrução `COPY` basicamente copia arquivos locais para o container, enquanto a instruçao `ADD` possui algumas features, como por exemplo extração de arquivos locais `tar` e suporte a URL externas. O melhor uso para a instrução `ADD` é a extração de um arquivo local para a imagem. Suas sintaxes são `<instrução> <origem> <destino>`

    COPY requirements.txt /tmp/

    ADD file.tar.gz /tmp/
    ADD https://caiodelgado.dev/file.tar.gz /tmp/

### EXPOSE

O `EXPOSE` é responsável para indicar ao Docker quais as portas que o container estará ouvindo por conexões, devemos utilizar as portas tradicionais para as aplicações, como por exemplo em um apache, expor a porta 80, em um mysql expor a porta 3306.

    EXPOSE 80

> A opção `EXPOSE`é diferente da opção `-p` ou `--publish` do comando `docker container run`. O `EXPOSE` só tem funcionalidade para o Dockerfile, uma vez que o container é criado sem publicar a porta, o docker sabe que o container estará ouvindo nas portas o `EXPOSE`

### ENV

Para a declaração de variaveis de ambiente, utilizamos a opção `ENV`. Através dela podemos alterar o PATH ou até mesmo configurar uma versão para uma aplicação propriamente dita.

    ENV PATH /usr/local/bin:$PATH
    ENV VAULT_ADDR http://vault.caiodelgado.example:8200

### CMD

Através da instrução `CMD` podemos indicar para executar um software que esteja contido na imagem somados a quaisquer argumentos.

    CMD ["apachectl", "-D", "FOREGROUND"]

### ENTRYPOINT

A instrução `ENTRYPOINT` é utilizada para configurar o comando principal da imagem, e então utilizamos a instrução `CMD` com as flags padrões.

    ENTRYPOINT ["echo"]
    CMD ["--help"]

> Ao rodar o container com `docker container run <imagem>` o container retornará a ajuda do comando, ao executar `docker contaner run <imagem> oi mundo` o container executará `echo oi mundo`.

### VOLUME

Quando precisamos expor arquivos para o docker, utilizamos a opção `VOLUME`, é altamente recomendado que utilizemos `VOLUME` para qualquer parte da imagem mutável ou servida ao usuário.

    VOLUME /usr/data
    VOLUME /var/www/html

### USER

Se um serviço pode ser executado sem privilégios, podemos utilizar a opção `USER` para trocar o usuário para um usuário "não root". Primeiramente precisamos criar este usuário e grupo através da instrução `RUN` e em seguida podemos modificar para o usuário.

    RUN groupadd -r caiodelgado && useradd --no-log-init -r caiodelgado
    USER caiodelgado

### WORKDIR

A instrução `WORKDIR` configura o diretório de trabalho ara qualquer instrução `RUN`, `CMD`, `ENTRYPOINT`, `COPY` e `ADD` que sejam precedidas pela instrução `WORKDIR`. Sempre utilize caminhos absolutos para esta instrução

    WORKDIR /var/www/html
    ADD webpage.tar.gz .

## Exemplo de um Dockerfile 

Para visualizar um Dockerfile, podemos ir no **[Docker Hub](https://hub.docker.com)** pesquisar pela imagem e na descrição clicar na tag desejada. No caso abaixo, selecionei a tag `current-alpine` da imagem `node`

<figure class="kg-card kg-image-card"><img src="/assets/2020/05/image-32.png" class="kg-image" alt loading="lazy"></figure>

    FROM buildpack-deps:buster
    
    RUN groupadd --gid 1000 node \
      && useradd --uid 1000 --gid node --shell /bin/bash --create-home node
    
    ENV NODE_VERSION 14.2.0
    
    RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
      && case "${dpkgArch##*-}" in \
        amd64) ARCH='x64';; \
        ppc64el) ARCH='ppc64le';; \
        s390x) ARCH='s390x';; \
        arm64) ARCH='arm64';; \
        armhf) ARCH='armv7l';; \
        i386) ARCH='x86';; \
        *) echo "unsupported architecture"; exit 1 ;; \
      esac \
      # gpg keys listed at https://github.com/nodejs/node#release-keys
      && set -ex \
      && for key in \
        94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
        FD3A5288F042B6850C66B31F09FE44734EB7990E \
        71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
        DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
        C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
        B9AE9905FFD7803F25714661B63B535A4C206CA9 \
        77984A986EBC2AA786BC0F66B01FBB92821C587A \
        8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
        4ED778F539E3634C779C87C6D7062848A1AB005C \
        A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
        B9E2F5981AA6E0CD28160D9FF13993A75599653C \
      ; do \
        gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
        gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
        gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
      done \
      && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
      && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
      && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
      && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
      && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
      && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
      && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
      # smoke tests
      && node --version \
      && npm --version
    
    ENV YARN_VERSION 1.22.4
    
    RUN set -ex \
      && for key in \
        6A010C5166006599AA17F08146C2130DFD2497F5 \
      ; do \
        gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
        gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
        gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
      done \
      && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
      && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
      && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
      && mkdir -p /opt \
      && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/ \
      && ln -s /opt/yarn-v$YARN_VERSION/bin/yarn /usr/local/bin/yarn \
      && ln -s /opt/yarn-v$YARN_VERSION/bin/yarnpkg /usr/local/bin/yarnpkg \
      && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
      # smoke test
      && yarn --version
    
    COPY docker-entrypoint.sh /usr/local/bin/
    ENTRYPOINT ["docker-entrypoint.sh"]
    
    CMD ["node"]

## Construindo a Imagem

Para construir a imagem, precisamos criar um diretório com um arquivo chamado `Dockerfile` e todos os arquivos que gostariamos de enviar para a imagem, e em seguida executar o comando `docker image build -t <nome>:<tag> .` dentro da pasta que contem o `Dockerfile`

> Lembrando que o Dockerfile é case sensitive, ou seja, se não estiver com a letra "D" maiuscula e todas as demais minusculas o comando não irá reconhecer o arquivo.

Este post faz parte de uma série de posts sobre **Docker.**

Nos próximos posts iremos entender e aplicar as melhores práticas na criação de imagens bem como criar imagens de múltiplos estágios.

Ficamos por aqui com esse post e nos vemos em uma próxima!

