# RabbitMQ

> Este guia foi elaborado por **Enéas Almeida** com o principal objetivo de facilitar os repasses de informações à equipe.

<p align="center">
    <img src="./media/logo.png" width="150px" />
</p>

RabbitMQ é um serviço de mensageria que utiliza o protocolo **AMQP** (Advanced Message Queuing Protocol) para transporte de dados. Esse protocolo nos permite trabalhar com envio de mensagens assíncronas.

## Vantagens

-   É um dos sistemas mais tradicionais de mensageria;
-   Extremamente rápido e confiável;
-   Implementa protocolos AMQP, MQTT, STOMP E HTTP, sendo o mais utilizado o AMQP;
-   Desenvolvido em Erlang;
-   Desacomplamento entre serviços;
-   Por padrão, as mensagens são persistidas em memória;
-   Padrão de mercado, muito utilizado e bastante testado.

## Multiplex connection: como o RabbitMQ trata as conexões

<p align="center">
    <img src="./media/rabbit-1.png" />
</p>

Utilizando o protocolo TCP, o RabbitMQ abre apenas uma conexão e cria vários canais dentro dela. Isso porque, se cada canal criasse uma nova conexão, seria bastante custoso, pois, passaria sempre pelo processo de estabelecimento de conexão do TCP: Three-way Handshake.

**Observação:** Para cada canal aberto, abre uma thread.

### Kafka vs RabbitMQ

| Características | Kafka                                                                                                                                              | RabbitMQ                                                                                                                                |
| :-------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------- |
| Protocolos      | O Kafka usa um protocolo binário via TCP.                                                                                                          | Protocolo avançado de fila de mensagens (AMQP) com suporte via plug-ins: MQTT, STOMP.                                                   |
| Arquitetura     | Kafka é uma plataforma de streaming distribuída, projetada para lidar com grandes volumes de dados em tempo real.                                  | RabbitMQ é um middleware de mensageria de propósito geral baseado em AMQP.                                                              |
| Persistência    | Kafka mantém os dados por um período configurável e permite que várias partes de um sistema leiam e gravem mensagens em tempo real.                | Por padrão as mensagens são persistidas em memória.                                                                                     |
| Casos de Uso    | É adequado para casos de uso que exigem processamento de stream em tempo real, como pipelines de dados, análise de logs, processamento de eventos. | É ideal para aplicativos que exigem fila de mensagens tradicional, comunicação entre sistemas heterogêneos e integração de aplicativos. |

## Exchange

<p align="center">
    <img src="./media/rabbit-2.gif" />
</p>

Atua como um roteador, ou seja, um ponto de entrada para as mensagens que são posteriormente encaminhadas para as filas específicas.

## As 3 formas mais conhecidas de implementar filas com o RabbitMQ

### 1. Workers

Utilizada quando existe uma alta demanda de processamento bloqueante como: processamento de uma fila de vídeos (que utiliza muito I.O), update em massa em uma carga de dados.

### 2. Publish/Subscribe

Utilizado em muitos cenários do nosso dia dia como: push notification, chat, envio de mensagens, principais tipos dessa categoria:

-   **Direct:** Nesse tipo de implementação o roteamento de mensagens para filas é realizado com base em uma chave de roteamento. A mensagem é entregue à fila cuja chave de roteamento está exatamente correspondente à chave de roteamento da
    mensagem.

<p align="center">
    <img src="./media/direct-1.gif" width="500px" />
</p>

| ⚠️ Observações                                                                                       |
| :--------------------------------------------------------------------------------------------------- |
| As mensagens são distribuidas entre os consumidores, utilizando o algoritimo de Round-Robin.         |
| Um vez a mensagem lida por algum consumidor, não é possível ser lida novamente por outro consumidor. |

<p align="center">
    <img src="./media/direct-2.gif" width="500px" />
</p>

🔑 No exemplo acima, foi alterada o binding key de x para y, dessa forma, as mensagens foram roteadas para outra fila.

-   **Topic:** Similar à Direct Exchange, mas permite um casamento mais sofisticado entre a chave de roteamento e a chave de ligação da fila, usando padrões (wildcards) para rotear mensagens com base em um padrão definido.

<p align="center">
    <img src="./media/topic-1.gif" width="500px" />
</p>

No exemplo acima foi utilizado o wildcard **checkout.log**, com isso ele envia tanto para as filas **\*.log** como **checkout.log**.

-   **Fanout:** Envio de mensagens em massa, como um broadcast. Distribui todas as mensagens que recebe para todas as filas que estão vinculadas a exchange. Ignora a chave de roteamento da mensagem.

<p align="center">
    <img src="./media/fanout-1.gif" width="500px" />
</p>

### 3. RPC (Remote Procedure Call)

Esse tipo de implementação é utilizado na comunicação entre aplicações desenvolvidas em diferentes tecnologias. Exemplo: Um app desenvolvido em Node.js precisa fazer requisições em uma API Golang.

## Queues

-   **FIFO:** o RabbitMQ utiliza o FIFO como padrão, a primeira mensagem que entra, é a primeira mensagem que sai, entretanto, existe uma forma de configurar essa propriedade, porém, se torna algo mais complexo e se faz necesário ter alto domínio do caso de uso.

### Principais propriedades:

-   **Durable:** Por padrão se utiliza filas duráveis, se a fila não for durável, ela vai ser removida assim que broker reiniciar;
-   **Auto-delete:** Quando um consumidor desconecta da fila, automaticamente ela é removida.
-   **Expiry:** Um tempo é definido para receber novas mensagens, caso não receba novas mensagens e/ou o consumidor não esteja conectado, a fila é removida;
-   **Message TTL:** Diz a respeito do tempo de vida útil da mensagem, caso ela não seja consumida em um determinado período de tempo, é perdida e não poderá ser consumida;
-   **Overflow:**
    -   **Drop Head:** Quando a fila entope de mensagens e chega uma nova, a mensagem mais antiga é removida para dar vez a nova que chegou;
    -   **Reject Publish:** Quando a fila entope, as novas mensagens vão sendo rejeitadas, dessa forma, fica claro, que existe uma opção para definir a quantidade máxima de mensagens por fila.
-   **Exclusive:** Apenas o canal que criou a fila pode acessar;
-   **Max length/bytes:** Define a quantidade máxima de mensagens ou bytes por fila e associa a essa opção o Drop Head ou Reject Publish;

### Dead letter queues

-   Algumas mensagens não são entregues, seja por problemas de redes ou outros motivos;
-   São encaminhadas para o Exchange específica com uma fila associada, que encaminham as mensagens para uma Dead letter queue;
-   As mensagens nesse estado, podem vir a serem consumidas para possíveis checagens, como se fosse um registro de log;

### Lazy queues

-   As mensagens são armazenadas em disco e caso o broker reinicie, a mensagens não são perdaidas;
-   Muito custosa, pois, exige muito I/O, esse recurso é utilizado geralmente quando são enviadas milhões de mensagens e os consumidores não dão conta de realizar as leituras de forma correta;

## Confiabilidade

-   Como garantir que as mensagens não serão perdidas no meio do caminho?
-   Como garantir que as mensagens foram processadas corretamente pelos consumidores?

O RabbitMQ disponibiliza alguns recursos pensados em resolver as situações questionadas acima:

### Publisher confirm

A **exchange** ao receber uma mensagem, confirma ao produtor que recebeu corretamente a mensagem.

<p align="center">
    <img src="./media/rabbit-ack-nack.gif" />
</p>

### Consumer acknowledgement

O **consumidor** ao receber uma mensagem, retorna uma das 3 confirmações abaixo para fila:

| Tipo             | Como funciona                                                                                 |
| :--------------- | --------------------------------------------------------------------------------------------- |
| **Basic.Ack**    | O consumidor recebe uma mensagem e processa o handler corretamente. Devolve um Ack para fila. |
| **Basic.Reject** | O consumidor recebe uma mensagem porém não processa o handler. Devolve um Reject para fila.   |
| **Basic.Nack**   | É exatamente como o Reject com uma excecção: processa várias mensagens ao mesmo tempo.        |

<p align="center">
    <img src="./media/rabbit-ack-1.gif" />
</p>

### Filas persistidas

As mensagens são persitidas em disco.

<hr />

### Passo a passo

```bash
docker-compose up -d
```

### Passo 1: acessa a dashboard

http://localhost:15672

<p align="center">
    <img src="./media/rabbit-painel-1.png" />
</p>

### Passo 2: adiciona uma fila

<p align="center">
    <img src="./media/rabbit-painel-2.png" />
</p>

<p align="center">
    <img src="./media/rabbit-painel-3.png" />
</p>

### Passo 3: verifica as exchanges defaults

<p align="center">
    <img src="./media/rabbit-painel-4.png" />
</p>

### Passo 4: volta na fila customers criada

<p align="center">
    <img src="./media/rabbit-painel-5.png" />
</p>

### Passo 5: realiza o binding de uma fila com uma exchange

<p align="center">
    <img src="./media/rabbit-painel-6.png" />
</p>

### Passo 6: envia uma mensagem

<p align="center">
    <img src="./media/rabbit-painel-7.png" />
</p>

### Passo 5: consome a fila

<p align="center">
    <img src="./media/rabbit-painel-8.png" />
</p>

<hr />

<div>
  <img align="left" src="https://imgur.com/k8HFd0F.png" width=35 alt="Profile"/>
  <sub>Made with 💙 by <a href="https://github.com/venzel">Enéas Almeida</a></sub>
</div>
