---
layout: post
title: Containers ou Máquinas Virtuais - Quando usar cada um?
date: '2020-04-20 11:31:36'
tags:
- devops
- sre
- cloud
- docker
---

Uma dúvida que assombra boa parte da comunidade de Tecnologia é quando usar **Containers** ou quando usar **Máquinas Virtuais.**

Para responder esta dúvida primeiramente precisamos entender bem o conceito e como funciona cada ferramenta.

**Máquinas Virtuais** ou **Virtual Machines (VMs)** são processos em que é feito a emulação **virtualização** de um sistema por completo dentro &nbsp;de um computador, sendo ele composto dos componentes básicos como processador, disco, memória, rede dentre outros. Este tipo de abordagem &nbsp;acaba tendo custo elevado em termos de recursos, uma vez que o próprio &nbsp;sistema operacional instalado em uma **VM** irá consumir boa parte da memória RAM e disco.

> **UPDATE:** Um amigo me corrigiu quanto a questão de **emulação** de um sistema completo, virtualização e emulação são de fato coisas diferentes. Fica aqui um bom artigo sobre o assunto. **[Emulation or Virtualization?](https://www.computerworld.com/article/2551154/emulation-or-virtualization-.html)**

**Container** é um processo em área restrita que executa um &nbsp;aplicativo e suas dependências no sistema operacional hospedeiro. Ao &nbsp;contrário da **VM** , os recursos do sistema operacional(como o kernel) são compartilhados com os **containers** os quais utilizam apenas os recursos necessários para seu processo ser executado.

A aplicação dentro de um container é um único processo em execução na &nbsp;máquina, permitindo ao servidor hospedeiro executar vários containers &nbsp;de forma independente. Este modo de trabalho permite remover conflitos &nbsp;entre dependências e simplificar a implantação de toda a instalação e &nbsp;configuração de forma instantânea, comparado com o modelo tradicional de &nbsp;virtualização.

Uma forma de fazer uma comparação simples entre Containers e Máquinas Virtuais é pensar que uma **Máquina Virtual** é como se fosse uma casa com todos seus ambientes, como por exemplo a garagem, o quarto, a sala, a cozinha, etc... e mesmo que um ambiente esteja "desocupado" o mesmo está consumindo o "espaço" que lhe é reservado. O **Container** por sua vez seria um apartamento, onde em um mesmo prédio(servidor) diversas pessoas tem seu ambiente(apartamento) ocupado e, teoricamente, sem interferir na vida do outro, aproveitando melhor assim o espaço utilizado.

Uma vez que para utilizar uma **Máquina Virtual** precisamos de emular todo o hardware da mesma, separando uma parcela de hardware (CPU+RAM+DISCO e as vezes periféricos) e instalar e parametrizar todo o sistema operacional, os **Containers** não necessariamente necessitam deste tipo de separação (ainda que seja possível) muito menos instalação do sistema operacional.

## Como que o container funciona então?

Bem, o funcionamento de um container não é nada de outro mundo, muito menos uma tecnologia extremamente nova, trata-se basicamente da utilização de recursos já conhecidos do ambiente linux como **namespaces** e **cgroups** somados a recursos de rede como **iptables** dentre outros, para compartilhar o **Kernel** do sistema operacional, sendo apenas necessário os binários e bibliotecas da aplicação a ser rodada no container.

Vamos explicar os principais recursos.

### Linux Namespaces 
<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/04/image-91.png" class="kg-image" alt loading="lazy"><figcaption>Linux Namespaces</figcaption></figure>

Os **namespaces** são recursos computacionais que fornecem isolamento para os containers, limitando seu acesso a recursos do sistema e outros namespaces. Isso significa, por exemplo, que um usuário root dentro de um container é diferente de um usuário root na máquina hospedeira.

### Linux Control Groups
<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/assets/2020/04/image-92.png" class="kg-image" alt loading="lazy"><figcaption>Linux Control Groups</figcaption></figure>

Os **cgroups** são utilizados para fazer o isolamento dos recursos físicos da máquina. Em geral os cgroups podem ser utilizados para controlar estes recursos tais como limites e reserva de CPU, limites e reservas de memória RAM, dispositivos, etc...

### Então quer dizer que Container não é coisa nova?

Não, container de fato não é algo novo.

**Containers** se popularizaram com o **[Docker](https://www.docker.com/)** (por volta de 2012) que é uma ferramenta para gerenciamento de containers de maneira simples, porém hoje existem diversas outras ferramentas para isto como por exemplo o **[lxc](https://linuxcontainers.org/), [cri-o](https://cri-o.io/)**, **[rkt](https://coreos.com/rkt/)**, **[podman](https://podman.io/)**, etc...

O que nos faz chegar a um ponto interessante, é possível rodar containers sem o **docker** ou qualquer outra ferramenta de container citada acima. Não vou ensinar como fazer este tipo de trabalho porque já existem diversos blogs com posts do tipo " **Rodando um container docker, sem o docker**", o que não é o foco deste post.

É de fato interessante subir um container sem a utilização do Docker, porém é um processo doloroso, digamos que uma pessoa com conhecimento avançado consegue subir um container na _ **"unha"** _ em cerca de 20 minutos, com o **Docker** fazemos isto em segundos. Então a minha recomendação é, a menos que você queira entender bem por baixo dos panos como tudo acontece, não gaste seu tempo com isso.

> Não quer dizer que não seja interessante entender o processo bruto dos containers, apenas que na minha opnião o seu tempo pode ser melhor aproveitado.

## Tá, e agora, quando uso Máquinas Virtuais e quando uso Containers?

Isso é uma discussão um tanto quanto _"filosófica"_ e o que posso dar a você é simplesmente minha pura opnião citando os prós e contras de se rodar uma aplicação em container.

Mas antes gostaria de frisar um detalhe importante, um **Container** só possui **sempre** unicamente duas funcionalidades:

- Executar a aplicação que foi embarcada
- Ser destruido (morrer)

> Logicamente após a morte de um container se necessário o mesmo deve ser recriado para fornecer a aplicação novamente. Trata-se de um processo comum e recorrente em uma stack de containers.

Bem, sobre as escolhas vamos pegar alguns tópicos:

### Bancos de Dados

Bancos de dados necessitam de uma performance excepcional, principalmente quando o volume de dados cresce exponencialmente, minha opnião é: **só utilize container**  **para desenvolvimento**

### Ambientes de Desenvolvimento 

Sou um grande fã de desenvolvimento em cima de containers, e acredito que isso garante a funcionalidade e igualdade entre ambientes de desenvolvimento em diversas máquinas, sempre acho válido a utilização de containers neste caso, a menos que você necessite de algum acesso especial a um hardware, ai vale estudar o nível de dificuldade para seu trabalho

> Problemas aparecem quando o ambiente de suporte de software não é identico. **Solomon Hykes** _(Criador do Docker)_

### Microsservices

Microsservices são nativamente criados pensando ser embarcados em containers, na minha opnião sempre é valido.

### Monolitos 

A questão de monolitos também abre uma discussão _"filosófica",_ mas para este caso eu defendo que vale ver a skill da equipe que o mantem e onde rodam as outras aplicações, se a maioria for container, porque não subir seu monolito em container?

> É importante atentar-se que nem todos os monolitos vão funcionar bem em containers, o ideal é que aplicações que forem desenvolvidas "container friendly" sejam executadas em containers.

### Serviços Essenciais de Infraestrutura (LDAP, DNS, DHCP)

Não recomendo que estes serviços estejam rodando em container, uma vez que são vitais para o funcionamento de uma infraestrutura e caso um container "morra" pode ser causado um prejuizo enorme.

> Eu sugiro que todo o serviço essencial seja executado em máquinas virtuais ou até mesmo baremetal, isso vale para o próprio **Docker** em si.

### Casos não citados

Sempre acho que vale a pena refletir os ganhos ou prejuizos em se colocar uma aplicação em container, o melhor passo é discutir com a sua equipe e ver se **worth it** (vale a pena).

## Considerações Finais

Muitas pessoas acham que o container veio pra **substituir** máquinas virtuais, o que não é verdade. No atual momento da tecnologia (2020) nós utilizamos as máquinas virtuais para rodar os containers e os serviços essenciais, pode ser que em um futuro próximo ou não este cenário mude, afinal a tecnologia sempre vem nos surpreendendo, mas nos tempos atuais ambos andam em conjunto, cada um servindo bem ao seu propósito.

Hoje ja existem serviços como ECS / EKS da amazon onde não precisamos gerenciar os recursos físicos do container e executamos eles como "serviços" o que não quer dizer que por baixo a amazon não esteja rodando máquinas virtuais ou até mesmo containers para nos fornecer estes serviços

Espero que esta publicação possa ter esclarecido parte das duvidas com relação a **Containers** x **Máquinas Virtuais.** Ficamos por aqui com esse post e nos vemos em uma próxima.

