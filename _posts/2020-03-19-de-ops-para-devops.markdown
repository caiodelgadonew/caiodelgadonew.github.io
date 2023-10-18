---
layout: post
title: De Ops para DevOps
date: '2020-03-19 03:58:53'
tags:
- devops
- cultura
---

> Este texto é um reboot de um post que escrevi em janeiro de 2019 quando estava começando esta aventura, logo postarei um texto mais atualizado.

Um marco importante na vida de um profissional de infraestrutura de &nbsp;TI é quando ele reconhece que as áreas de desenvolvimento e &nbsp;infraestrutura devem andar em conjunto como um só. Porém, após trabalhar &nbsp;12 anos como Analista de Redes e Infraestrutura Microsoft voltado unicamente para Operações (OPS) e Datacenter, o que faria um profissional a buscar desbravar o mundo **Open Source** e a cultura **DevOps**?

A resposta é simples: Mudança!

No último ano procurei buscar algo que pudesse me satisfazer &nbsp;profissionalmente e sair da zona de conforto e foi por indicação de um &nbsp;amigo que comecei a procurar sobre o termo **DevOps** e entender o porque era imprescindível ter esta cultura no sangue e aplicar seus fundamentos no dia a dia.

Neste momento em sua cabeça devem estar surgindo algumas perguntas:

“Mas Caio, você trabalhou 12 anos com Microsoft, já tem &nbsp;muita experiência na área, você não estaria jogando fora todo o &nbsp;conhecimento já adquirido? Não é tarde para esta mudança drástica? &nbsp;Porque Open Source ( **Linux** ) e não proprietário (Microsoft)? ”

Bem, conhecimento adquirido nunca é jogado fora, todos estes anos &nbsp;serviram para conhecer como uma infraestrutura deve se portar, quais são &nbsp;os pontos críticos de uma rede, dos servidores e datacenters, qual a &nbsp;importância de se ter um backup, firewall, redes segmentadas, documentações e diversas outras coisas.

Estar tarde? Se você estiver vivo, nunca é tarde para mudanças!

Porque **Linux** e não Microsoft?

Esta é a pergunta que todos nós fazemos e eu vou dar alguns motivos &nbsp;técnicos e culturais para você entender a resposta… a começar pelo &nbsp;motivo técnico: _HARDWARE_.

Se você abrir a página da Microsoft para ver quanto de Hardware é necessário para a instalação de um Windows Server 2016 você encontrará os seguintes requerimentos mínimos:  
• Processador de 1.4Ghz e 64 bits  
• 2 GB de memória RAM  
• 45 GB de espaço em disco.

Lógico que hoje existe a versão “_core edition_” que não &nbsp;possui a interface gráfica – que consome incríveis 512MB RAM – porém &nbsp;ainda consome cerca de 32GB de disco rígido. Além disso tente &nbsp;“controlar” um Windows Server 2016 com 2GB de RAM, você terá &nbsp;vontade de colocar fogo em seu servidor e jogá-lo do vigésimo andar, &nbsp;fazendo com que sejam necessários ao menos 4GB para se ter uma &nbsp;experiência saudável à mente humana.

Quando falamos de Linux, podemos encontrar o seguinte cenário, onde &nbsp;irei aproveitar para trazer duas distribuições de “famílias” distintas:

Ubuntu Server 18.04 (requerimento mínimo):  
• Processador 300Mhz x86  
• 384 MB de memória RAM  
• 2,5 GB de espaço em disco

CentOS 7 / RHEL 7 (requerimento recomendável):  
• Processador 1Ghz 64 Bits  
• 1 GB de memória RAM  
• 5 GB de espaço em disco

Podemos observar uma enorme economia em hardware, onde isso pode refletir positivamente quando envolvemos custos de nuvem , mas também considerando a economia nos recursos locais. De uma forma &nbsp;generalista podemos dizer que cada hardware pode comportar 1 servidor &nbsp;com WS2016 ou no mínimo 2 servidores Linux like.

Bom, já falamos do HARDWARE e agora falaremos sobre _CULTURA_:

**DevOps** é um termo que surgiu por volta de 2009 na &nbsp;Velocity Conference, onde John Allspaw e Paul Hammond apresentaram uma &nbsp;palestra intitulada de **“10 Deploys por dia, cooperação entre Dev & Ops no Flickr”** onde eles abriram a apresentação com 3 slides em preto com textos simples em branco dizendo uma única frase em cada um:

> “Dev versus Ops”  
> “Não são minhas máquinas, é seu código”  
> “Não é meu código, são suas máquinas!”

Estas frases são bem conhecidas no dia a dia de qualquer profissional de &nbsp;TI – seja ele Dev ou Ops – o que eles mostraram nesta palestra é que &nbsp;Dev e Ops deveriam formar um casal, onde um completa o trabalho do outro &nbsp;para atingir um único objetivo, o da empresa. Com esta abordagem eles &nbsp;(Flickr) chegaram ao ponto de realizar incríveis 10 deploys por dia sem &nbsp;maiores problemas, com as equipes trabalhando juntas. Ao término da &nbsp;apresentação foi dita uma única frase que em 2009 foi marcante:

> _“Na última semana foram feitos 67 deploys de 496 changes por 18 pessoas”_

Convenhamos que em pleno 2019 esta frase ainda impressiona, imaginem a 10 anos atrás.

O meio **Open Source** nasceu e cresceu através de colaboração, onde a própria comunidade de &nbsp;usuários e desenvolvedores fazem com que o sistema seja constantemente &nbsp;atualizado e melhorado. Toda o sistema não está nas mãos de um único &nbsp;desenvolvedor como é o caso dos softwares proprietários e assim, todos trabalham para um bem comum que é o de melhoria.

Junto com DevOps nasceu também a **IaC** (Infra-as-Code) que é a infraestrutura como um código, &nbsp;onde passamos a tratar toda a infraestrutura da empresa como um código – &nbsp;literalmente – onde o mesmo é versionado em um repositório e assim nos &nbsp;tornamos verdadeiros **“Desenvolvedores de Infraestrutura”** e não mais apenas Analistas de Operação de Infraestrutura. Com a &nbsp;codificação e versionamento ganhamos velocidade e escalabilidade, onde &nbsp;tarefas que custavam dias para serem realizados são concluídas em &nbsp;minutos e até mesmo em segundos.

DevOps é tão certo e realidade que hoje a Microsoft esta correndo &nbsp;atrás do prejuízo e criando sua própria solução DevOps que era conhecido &nbsp;como VSTS (_Visual Studio Team Services_) e hoje se chama Azure DevOps.

E como podemos sair da rotina de ser apenas “Ops” e partir para um mundo DevOps?

Aqui vão algumas dicas:

**Aprenda sobre a cultura** – DevOps é um movimento e uma cultura muito antes de ser um “cargo” por isso os aspectos culturais são muito importantes.

**Aprenda uma linguagem de programação** – É altamente &nbsp;recomendado que você saiba programar, afinal, o futuro já está aqui e &nbsp;tudo esta virando programação até mesmo infraestrutura e redes. Python, &nbsp;Go, Nodejs existem várias opções, não necessariamente você precisa &nbsp;aprender a mesma linguagem que sua empresa utiliza. Eu particularmente &nbsp;recomendo **Python** pelo poder e simplicidade.

**Aprenda a gerenciar Servidores, Virtualização, Redes e Segurança** – A principal tarefa de um profissional DevOps é gerenciar servidores, &nbsp;entender seu funcionamento é muito importante, até a parte mais baixa &nbsp;da tecnologia (arquitetura, memória, processador) são conhecimentos &nbsp;necessários, sistemas operacionais e especialmente Linux, se você tem &nbsp;duvida onde começar, escolha uma distribuição como o **Ubuntu**. Entenda como os protocolos básicos de rede como HTTP, DNS e FTP funcionam, para você implementar a segurança você tem que conhecer os protocolos e entender como não deixar brechas para invasores.

**Aprenda a criar scripts** – Scripts podem ser considerados como uma programação, onde o **Shell Script** é um dos recursos mais utilizados, juntamente aos scripts escritos em **Python** que é uma linguagem onde é possível fazer muita coisa escrevendo pouco.

**Aprenda a instalar e configurar middlewares** – Middlewares são programas que fornecem serviços para aplicações de outros programas, como por exemplo, o Apache, Nginx e JBoss/Wildfly. Estes são utilizados em toda a internet para prover desde pequenos blogs wordpress até grandes marketplaces.

**Aprenda como implantar software** – Após aprender a &nbsp;configurar os middlewares, você vai precisar saber como implantar as &nbsp;aplicações em servidores, você pode ter em um servidor com mais de uma &nbsp;aplicação rodando e utilizar o Nginx como proxy reverso para ambas.

**Aprenda controle de versionamento com GIT** – GIT é o sistema de versionamento mais utilizado na industria de TI. Você &nbsp;não precisa ser um Expert mas é interessante saber o básico para &nbsp;entender a tecnologia.

**Aprenda a Automatizar suas tarefas e Gerenciar configurações** – Automatizando tarefas básicas e usando gerencia de configurações com softwares como **Ansible, Chef, Puppet** fará você ganhar tempo para dedicar a tarefas que trarão crescimento &nbsp;pessoal e profissional, no futuro você terá tantos ambientes para &nbsp;gerenciar que será impossivel fazer de forma manual.

**Aprenda Infraestrutura como Código** – IaC é um dos pontos mais fortes para um ambiente DevOps, ter um ambiente altamente confiável e escalável é uma das tarefas mais importantes para a TI.

**Aprenda a monitorar sua Infraestrutura e seu Software** – Imagine um atleta olímpico, ele precisa de constante monitoramento de &nbsp;seus sinais vitais, alimentação, etc… A infraestrutura e o software não &nbsp;são diferentes, o monitorando com ferramentas apropriadas como **Zabbix e Prometheus** você consegue prever e até evitar incidentes que possam causar um impacto negativo na organização.

**Aprenda sobre Orquestração de Containers** – Aprender **Docker e Kubernets** é altamente recomendado, pois com estas ferramentas você consegue criar &nbsp;e “destruir” um ambiente inteiro em pouco tempo, unindo isto a **IaC** você consegue construir um ambiente altamente dinâmico.

**Aprenda a Compartilhar** – O ponto mais importante de uma **cultura DevOps** é que o processo de aprendizagem é contínuo e compartilhar o &nbsp;conhecimento adquirido pode te ajudar ainda mais e ser um dos pontos &nbsp;chaves em seu processo de mudança.

