# Live Up - Real-Time Streaming Event Bridge

**Nota de Engenharia:** Este repositório documenta a arquitetura de integração e o fluxo de dados da aplicação. O código-fonte original é mantido em repositório privado para proteger chaves de integração e a propriedade intelectual da lógica de eventos.

## Visão Geral
O Live Up é uma aplicação desktop desenvolvida para estabelecer uma ponte de comunicação de baixíssima latência entre plataformas de Live Streaming (via consumo de APIs de transmissão em tempo real) e o motor de jogos voxel (Minecraft Bedrock). O sistema traduz interações da audiência (presentes, curtidas, seguidores) em eventos dinâmicos e procedurais dentro do jogo.

## Stack Tecnológica
* **Core/Desktop:** Electron, Node.js
* **Backend (Servidor Local):** Express.js
* **Comunicação em Tempo Real:** WebSockets (ws)
* **Frontend UI:** HTML5, CSS3, JavaScript (Vanilla)
* **Integração de Dados:** API de Webhooks/Streaming de Vídeo

## Arquitetura e Soluções de Engenharia

### 1. Arquitetura de Ponte Local (Local Bridge Server)
Para evitar o lag de rede inerente a chamadas externas contínuas, a aplicação foi empacotada via Electron não apenas como uma interface, mas como um host local duplo:
* Levanta um servidor HTTP (Express) na porta 8080 para renderizar a interface do usuário (Dashboard de controle).
* Simultaneamente, levanta um servidor WebSocket (WS) na porta 3000 para aguardar a conexão direta do cliente do jogo.

### 2. Processamento Orientado a Eventos (Event-Driven Integration)
O sistema atua como um ouvinte assíncrono conectado diretamente ao fluxo de dados da plataforma de streaming.
* O tráfego de rede recebido é interceptado e parseado em tempo real. Eventos complexos (como doações de itens virtuais específicos) são mapeados contra um dicionário lógico de configuração em memória.

### 3. Injeção de Comandos Bidirecional via WebSockets
A comunicação com a engine do jogo não depende de mods ou plugins de terceiros. 
* O servidor WebSocket forja pacotes JSON formatados no padrão nativo da engine (`messagePurpose: "commandRequest"`).
* Esses pacotes são transmitidos instantaneamente para a porta de escuta do jogo, executando manipulação de blocos, spawn de entidades e alterações de *scoreboards* com latência na casa dos milissegundos.

### 4. Gerenciamento de Estado e Hot-Swapping
O frontend permite a troca de "Modos de Jogo" em tempo real. Essa alteração de estado no painel de controle (Express) reescreve dinamicamente as regras de conversão de eventos no backend (Node.js) através de requisições de API internas, alterando o comportamento do jogo sem a necessidade de reiniciar a conexão WebSocket.

## Fluxo de Dados (Pipeline)
1. **Ingestão:** A aplicação conecta-se à API pública de streaming via protocolo de long-polling/WebSockets.
2. **Parsing:** O payload de dados brutos (JSON) da audiência é decodificado.
3. **Mapeamento:** O tipo de evento (Follow, Share, Gift) é validado contra a configuração do modo de jogo atual.
4. **Tradução:** O evento é convertido em uma array de comandos executáveis nativos da engine.
5. **Broadcast:** O payload formatado é transmitido via WebSocket local, onde a engine do jogo o recebe e o processa no próximo *tick* do servidor.
