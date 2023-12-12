# RabbitMQ

## Vantagens

-   É um dos sistemas mais tradicionais de menssageria;
-   Extremamente rápido e confiável
-   Implementa protocolos AMQP, MQTT, STOMP E HTTP, sendo o mais utilizado o AMQP;
-   Desenvolvido em Erlang;
-   Desacomplamento entre serviços;
-   Por padrão, as mensagens são persistidas em memória;
-   Padrão de mercado, muito utilizado e bastante testado.

## Multiplex Connection: como o Rabbit trata as conexões

<p align="center">
    <img src="./media/rabbit-1.png" />
</>

Utilizando o protocolo http, o RabbitMQ abre apenas uma conexão e cria vários canais dentro dela.

**Observação:** Para cada canal aberto, abre uma thread.

## Como funciona

<p align="center">
    <img src="./media/rabbit-2.gif" />
</>

## Exchange

### Direct exchange

<p align="center">
    <img src="./media/rabbit-3.gif" />
</>

<div>
  <img align="left" src="https://imgur.com/k8HFd0F.png" width=35 alt="Profile"/>
  <sub>Made with 💙 by <a href="https://github.com/venzel">Enéas Almeida</a></sub>
</div>
