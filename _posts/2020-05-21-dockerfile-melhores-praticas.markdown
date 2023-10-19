---
layout: post
title: Dockerfile - Melhores Práticas
date: '2020-05-21 14:06:54'
tags:
- devops
- iac
- sre
- docker
---

Hoje iremos passar por uma série de _dicas_ para melhorar nosso conhecimento na etapa de **build** da imagem. Já **[entendemos como funciona o Dockerfile](/entendendo-dockerfile/)**, agora o ideal é que estas imagens sejam criadas utilizando as melhores práticas.

Provavelmente não irei cobrir todas as melhores práticas para a criação da nossa imagem, uma vez que a cada dia que passa novas técnicas são criadas e as atuais são melhoradas, mas quero que você saia daqui hoje sabendo mais do que ontem e menos que amanhã, então vamos lá!

### Relembrando o que é o Dockerfile

O **Docker** constrói a imagem automaticamente lendo as instruções de um `Dockerfile`, que é um documento de texto que contém todos os comandos necessários para a criação de uma determinada imagem.

> Caso você não tenha acompanhado os outros posts sobre Docker, sugiro que veja a **[TAG: Docker](/tag/docker/)** no blog e principalmente o post **[Docker 101](/docker-101)** pois iremos utilizar a máquina criada no post para criação das imagens.

## Recomendações gerais 

Quando criamos uma imagem, através do comando `docker image build` , a imagem definida pelo `Dockerfile` deve gerar containers que são tão efêmeros quanto possível, isso quer dizer que o container deve poder ser parado e/ou destruido a qualquer momento, e reconstruido ou substituido com o mínimo de configuração ou atualização.

Uma boa metodologia para conseguir chegar neste ponto é a do _ **Twelve-factor App** ,_ ou _Aplicação de Doze-fatores,_ e em sua seção de **[processos](https://12factor.net/pt_br/processes)** podemos verificar algumas das motivações para subir containers maneira **stateless** (não armazenam estado).

## Entendendo o contexto de build

Quando executamos o comando `docker image build` , o diretório no qual apontamos (muitas das vezes como `.` para referir o diretório atual) é chamado de `build context`. Por padrão o Docker espera que o `Dockerfile` esteja localizado nesta pasta, mas também podemos especificar uma nova localização através da flag `-f`. Independentemente de onde o `Dockerfile` esteja, todo o conteúdo dos diretórios recursivamente e arquivos é enviado para o Docker daemon como `build context`.

> Por isto não devemos, por exemplo, criar um `Dockerfile` diretamente em nossa home &nbsp;`~/Dockerfile` , uma vez que todo o conteúdo, inclusive o cache de navegadores web e todas aplicações `~/.cache` será enviado para o Docker daemon, muitas das vezes falhando a build ou até mesmo fazendo com que ela demore muito tempo.

A partir de agora tilizaremos nossa máquina **docker01** criada no post **[Docker-101](/docker-101)**. Estou assumindo que você já subiu a máquina e conectou na mesma via `ssh` . Caso você não possua a máquina pode voltar no post do blog para cria-la ou fazer na sua máquina atual com o Docker instalado.

Aqui daremos dois exemplos, um com a criação de uma imagem com o Dockerfile no mesmo diretório do contexto, e outro em diretórios diferentes.

Primeiramente vamos criar um diretório para &nbsp;guardar nosso dockerfile e criar um arquivo com um conteúdo estático

    $ mkdir -p /vagrant/dockerfiles/exemplo1
    $ cd /vagrant/dockerfiles/exemplo1
    $ echo "Dockerfile Melhores Práticas" > blog

Vamos também criar nosso `Dockerfile` para manusear o arquivo e criar a imagem

    $ vim Dockerfile
    
    FROM busybox
    COPY blog /
    RUN cat /blog
    
    $ docker image build -t exemplo:v1 . 

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-35.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image build -t exemplo:v1 . </figcaption></figure>

Agora vamos criar novos diretórios e mover o arquivo `blog` para um diretório diferente do `dockerfile` para construir uma segunda imagem.

    $ mkdir -p dockerfiles context
    $ mv Dockerfile dockerfiles
    $ mv blog context
    $ docker image build --no-cache -t exemplo:v2 -f dockerfiles/Dockerfile context

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-36.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image build --no-cache -t exemplo:v2 -f dockerfiles/Dockerfile context</figcaption></figure>

As duas imagens tem o mesmo tamanho e o mesmo conteúdo, porém note que as imagens tem o `ID` diferente porque criamos a imagem sem utilizar o cache `--no-cache`, ou seja, criamos uma imagem totalmente nova.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-37.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image ls </figcaption></figure>

Caso sejam incluidos arquivos que não são necesssários para a construção da imagem o `build context` se tornará maior e consequentemente uma imagem maior. Isso pode aumentar o tempo de construção, envio e download da imagem e do `container runtime`. Para ver o tamanho do build context basta verificar a mensagem exibida quando executar o build do seu `Dockerfile`.

    Sending build context to Docker daemon 318.6MB

Para fins de teste vamos copiar todo o conteúdo do diretório `/var/log` para o context e construir a imagem.

    $ sudo cp -r /var/log/ /vagrant/dockerfiles/exemplo1/context/
    $ docker image build --no-cache -t exemplo:v3 -f dockerfiles/Dockerfile context

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-38.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image build --no-cache -t exemplo:v3 -f dockerfiles/Dockerfile context</figcaption></figure>

Veja que o context que anteriormente era de apenas 2.6KB desta vez foi de 43MB, o que resulta em um tempo de build maior porém sua imagem continua do mesmo tamanho das outras já que o arquivo foi enviado para o context e não foi utilizado.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-39.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image ls</figcaption></figure>

Tratando de poucos MB o tempo de construção pode não ser muito expressivo, porém imagine em uma grande aplicação com diversos arquivos. Utilizando o comando `time` fiz a medição no caso de 2.6KB que gerou a build em `0m0.910s` contra `0m1.368s` do arquivo de 43MB. esse tempo pode ser superior caso a imagem execute diversos comandos em diversas camadas.

## Excluindo arquivos do build

Para excluir arquivos que não são relevantes a build, pdemos criar um arquivo `.dockerignore` contendo os padrões de exclusão similares aos do `.gitignore` possibilitando que ignoremos arquivos no build sem ter que modificar nosso repositório.

> Para a referência do Docker Ignore veja a **[Documentação Oficial](https://docs.docker.com/engine/reference/builder/#dockerignore-file)**

Vamos criar agora um arquivo `.dockerignore` para que o diretório `log` não seja enviado para a build.

    $ vim context/.dockerignore
    
    # Comentario: Ignorando arquivos do diretorio log
    log
    
    $ docker image build --no-cache -t exemplo:v4 -f dockerfiles/Dockerfile context

Veja que o diretório `log` foi ignorado, uma vez que o `build context` ficou em 2.6kB (agora um pouco maior que a primeira por possuir o arquivo `.dockerignore`) ao invés dos 43MB anteriores.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-41.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image build --no-cache -t exemplo:v4 -f dockerfiles/Dockerfile context</figcaption></figure>
## Melhores práticas parte 1

Na parte 1 das melhores práticas é preciso fazer uma menção ao **[Tibor Vass](https://www.docker.com/blog/author/tibor-vass/)** que fez estas explicações no blog do Docker. Uma vez que o conteúdo base é do mesmo e eu estarei apenas adaptando o post e fazendo minhas modificações. Caso queira ver o post do Tibor o [link é este aqui](https://www.docker.com/blog/intro-guide-to-dockerfile-best-practices/).

Vamos criar um diretório para o exemplo a seguir para guardar nosso Dockerfile e fazer o download de uma aplicação exemplo em java que conta o numero de caracteres bem de um texto.

    $ mkdir -p /vagrant/dockerfiles/parte1/app
    $ cd /vagrant/dockerfiles/parte1
    $ git clone git@github.com:caiodelgadonew/blog-java-app.git app

### Dica #1: A ordem importa para o cache

A ordem dos passos de build é importante, se o cache de um primeiro passo é invalidado pela modificação de arquivos ou linhas do Dockerfile, os arquivos subsequentes do build quebrarão. Sempre faça a ordenação dos passos do que sofrerá menos mudanças para o que sofrerá mais mudança.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-53.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM debian:9
    RUN apt-get update
    RUN apt-get install -y openjdk-8-jdk wget ssh vim
    COPY app /app
    ENTRYPOINT ["java", "-jar", "/app/target/app.jar"]
    
    
    $ docker image build --no-cache -t parte1:v1 .

### Dica #2: &nbsp;COPY mais específico para limitar a quebra de cache 

Só copie o necessário. Se possível evite o **COPY.** Quando copiamos arquivos para nossa imagem, tenha certeza que você está sendo bem específico sob o que quer copiar, qualquer mudança no arquivo copiado quebrará o cache. Copiaremos então apenas a aplicação para a imagem, desta maneira as mudanças nos arquivos não afetarão o cache.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-52.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM debian:9
    RUN apt-get update
    RUN apt-get install -y openjdk-8-jdk wget ssh vim
    COPY app/target/app.jar /app/app.jar
    COPY app/samples /samples
    CMD ["java", "-jar", "/app/app.jar"]
    
    $ docker image build --no-cache -t parte1:v2 .

### Dica #3: Identifique as instruções que podem ser agrupadas

Cada instrução `RUN` cria uma unidade de cache e uma nova camada de imagem, agrupar todos os comandos `RUN` em uma única instrução pode melhorar o desempenho e diminuir a quantidade de camadas uma vez que eles se tornarão uma unidade única cacheavel.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-54.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM debian:9
    RUN apt-get update \
            && apt-get install -y \
               openjdk-8-jdk wget \
               ssh vim   
    COPY app/target/app.jar /app/app.jar
    COPY app/samples /samples
    CMD ["java", "-jar", "/app/app.jar"]
    
    
    $ docker image build --no-cache -t parte1:v3 .

### Dica #4: Remova as dependências desnecessárias

Remover as dependencias desnecessárias e não instalar pacotes de debug é uma boa prática, como por exemplo trocar o `jdk` (Java Development Kit) pelo `jre` (Java Runtime Environment) que é um pacote relativamente menor e contem apenas o necessário para execução. Você pode instalar as ferramentas de debug posteriormente caso necessite. O instalador de pacotes `apt` possui uma flag `--no-install-recommends` que garante que dependencias que não são necessárias não sejam instaladas. Caso precise, adicione elas explicitamente.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-55.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM debian:9
    RUN apt-get update \
            && apt-get install -y --no-install-recommends \
               openjdk-8-jre 
    COPY app/target/app.jar /app/app.jar
    COPY app/samples /samples
    CMD ["java", "-jar", "/app/app.jar"]
    
    $ docker image build --no-cache -t parte1:v4 .

Com a **Dica #4** podemos notar uma diminuição consideravel no tamanho de nossa imagem.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-57.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image ls</figcaption></figure>
### Dica #5: Remover o cache do gerenciador de pacotes

O gerenciador de pacotes mantem seu próprio cache, o &nbsp;`apt` por exemplo guarda seu cache no diretório `/var/lib/apt/lists` e `/var/cache/apt/`. Uma das maneiras de lidar com este problema é remover o cache na mesma instrução que o pacote foi instalado. Remover este cache em outra instrução não irá diminuir o tamanho da imagem.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-58.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM debian:9
    RUN apt-get update \
            && apt-get install -y --no-install-recommends \
               openjdk-8-jre \
            && rm -rf /var/lib/apt/lists \
            && rm -rf /var/cache/apt
    COPY app/target/app.jar /app/app.jar
    COPY app/samples /samples
    CMD ["java", "-jar", "/app/app.jar"]
    
    $ docker image build --no-cache -t parte1:v5 .

Agora com a **Dica #5** nossa imagem ficou relativamente menor.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-59.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image ls</figcaption></figure>
### Dica #6: Utilize imagens oficiais quando possível

Imagens oficiais podem ajudar muito e reduzir bastante o tempo preparando a imagem, isto porque os passos de instalação já vem prontos e normalmente com as melhores praticas aplicadas, isto também fará você ganhar tempo caso tenha multiplos projetos, eles compartilham as mesmas camadas e utilizam a mesma imagem base.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-60.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM openjdk
    COPY app/target/app.jar /app/app.jar
    COPY app/samples /samples
    CMD ["java", "-jar", "/app/app.jar"]
    
    $ docker image build --no-cache -t parte1:v6 .

### Dica #7: &nbsp;Utilize Tags mais específicas

Nunca utilize a tag **latest.** Ela pode receber alguma atualização e em um momento de update sua aplicação pode quebrar, dependendo de quanto tempo passou do seu ultimo build. Ao invés disso, utilize tags mais específicas.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-61.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM openjdk:8
    COPY app/target/app.jar /app/app.jar
    COPY app/samples /samples
    CMD ["java", "-jar", "/app/app.jar"]
    
    $ docker image build --no-cache -t parte1:v7 .

### Dica #8: Procure por flavors mínimos

Existem diversos flavors linux que fazem com que nossa imagem se torne cada vez menor, um bom exemplo são as imagens `slim` e `alpine` as quais são as menores encontradas. A imagem `slim` é baseada no Debian, enquanto a `alpine` é baseada em uma distribuição linux muito menor chamada Alpine. A diferença básica entre elas é que o debian utiliza a biblioteca `GNU libc` enquanto o alpine utiliza `musl lbc`, que apesar de muito menor, pode ter problemas de compatibilidade.

    REPOSITORY TAG SIZE
    openjdk 8 510MB
    openjdk 8-jre 265MB
    openjdk 8-jre-slim 184MB
    openjdk 8-jre-alpine	84.9MB

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-62.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

    $ vim Dockerfile
    
    FROM openjdk:8-jre-alpine
    COPY app/target/app.jar /app/app.jar
    COPY app/samples /samples
    CMD ["java", "-jar", "/app/app.jar"]
    
    $ docker image build --no-cache -t parte1:v8 .

Agora temos uma diminuição enorme em nossa imagem pois estamos utilizando uma imagem base bem menor.

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-63.png" class="kg-image" alt loading="lazy"><figcaption>$ docker image ls</figcaption></figure>
## Melhores práticas parte 2

Agora na parte 2 iremos falar sobre `multi-stage builds`, que é um recurso muito poderoso que apareceu a partir do docker 17.05. Multistage builds são uteis para quem quer otimizar Dockerfiles enquanto mantém eles fáceis de ler e manter.

### Antes do Multi-stage build

O maior desafio das imagens é de fato manter as imagens pequenas, vimos nos exemplos anteriores que conseguimos, ao utilizar algumas das melhores práticas, diminuir bastante o tamanho da imagem. Utilizando imagens `slim` ou `alpine` resolvem boa parte dos nossos problemas mas quando precisamos resolver algo mais &nbsp;complexo podemos utilizar elas somadas ao Multistage build.

### Multi-stage build

O multi-stage build faz com que possamos utilizar diversas instruções `FROM` em um Dockerfile, e cada instrução pode utilizar uma imagem diferente, fazemos isto por exemplo para subir uma imagem, dentro desta imagem instalar os pacotes e coletar apenas os arquivos necessários diretamente para a imagem subsequente. Com isso temos uma imagem muito mais enxuta e otimizada.

Por exemplo vamos criar esta imagem em GO pelo processo normal:

    $ git clone https://github.com/alexellis/href-counter.git /vagrant/dockerfiles/parte2
    $ cd /vagrant/dockerfile/parte2
    $ rm Docker*
    $ vim Dockerfile
    
    FROM golang:1.7.3
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html  
    COPY app.go .
    RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
    
    $ docker image build --no-cache -t parte2:v1 .

Esta imagem ficou com o tamanho de `692MB`, porém o que precisamos nela é apenas o diretório `/go/src/github.com/alexellis/href-counter/app`, podemos então utilizar o multistage build para recolher estes arquivos, chamando a primeira imagem de `builder` através do parâmetro `AS <nome>` e depois invocar a imagem em um segundo estágio através do parâmetro `--from=<nome>`.

    $ vim Dockerfile
    
    FROM golang:1.7.3 AS builder
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html
    COPY app.go .
    RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
    
    FROM alpine:latest
    RUN apk --no-cache add ca-certificates
    WORKDIR /root/
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
    CMD ["./app"]
    
    
    
    $ docker image build --no-cache -t parte2:v2 .

<figure class="kg-card kg-image-card"><img src="/assets/2020/05/image-64.png" class="kg-image" alt loading="lazy"></figure>

Agora tivemos uma "pequena" redução de `692MB` para `11.8MB`.

    REPOSITORY TAG SIZE
    parte2 v2 11.8MB
    parte2 v1 692MB

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-65.png" class="kg-image" alt loading="lazy"><figcaption>Multistage x Normal build</figcaption></figure>

Outra coisa interessante é que ao invés de utilizar uma imagem completa podemos puxar um arquivo de uma imagem já criada anteriormente através da flag `--from=<image>:<tag>`, vamos fazer a cópia de um arquivo da imagem da parte1 para a imagem da parte 2, para isto criaremos um diretório chamado parte 3.

    $ cd /vagrant/dockerfiles
    $ cp -r parte2 parte3
    $ cd parte3
    $ vim Dockerfile 
    
    FROM alpine:latest
    WORKDIR /root/
    COPY --from=parte1:v8 /samples/1.txt .
    CMD ["cat", "1.txt"]
    
    $ docker image build --no-cache -t parte3:v1 .

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-67.png" class="kg-image" alt loading="lazy"><figcaption>Dockerfile</figcaption></figure>

Podemos agora executar nosso container para verificar se o arquivo `1.txt` é de fato o &nbsp;arquivo extraido da imagem `parte1:v8` .

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/05/image-68.png" class="kg-image" alt loading="lazy"><figcaption>$ docker container run --rm -it parte3:v1</figcaption></figure>

É sempre bom tentar diminuir as imagens de docker e seguir as melhores práticas, isso faz com que nosso tempo de deploy ou scale da aplicação seja menor, bem como a necessidade de um armazenamento maior e diversos outros fatores.

Este post faz parte de uma série de posts sobre **Docker.**

O código deste post encontra-se no repositório: [https://github.com/caiodelgadonew/blog-dockerfile-melhores-praticas](https://github.com/caiodelgadonew/blog-dockerfile-melhores-praticas)

Ficamos por aqui com esse post e nos vemos em uma próxima!

