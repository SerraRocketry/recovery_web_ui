# MapFlask

Projeto simples para visualização em tempo real de telemetria em um mapa usando Flask + Socket.IO.

## Visão geral

Este subprojeto (pasta `MapFlask`) abre uma aplicação web que recebe dados via porta serial e os transmite para clientes conectados via WebSocket (Flask-SocketIO). Os dados são exibidos em um mapa (Leaflet) e em uma tabela HTML.

O servidor principal está em `app/app.py`. O frontend principal de mapas e tabela está em `app/static/js/app.js`.

## Dependências

- Python 3.8+
- Flask
- Flask-SocketIO
- pyserial (para comunicação serial)

Você pode instalar as dependências rapidamente com:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Como rodar

1. Conecte o dispositivo serial (ou garanta que exista uma porta serial virtual para teste).
2. Entre na pasta `MapFlask`:

```bash
cd MapFlask
```

3. Execute a aplicação:

```bash
python3 app/app.py
```

Ao iniciar, o programa lista as portas seriais disponíveis e pede que você selecione uma. O servidor web abre automaticamente o navegador em `http://localhost:5000` (porta padrão configurada em `app/app.py`).

## Arquivos importantes

- `app/app.py` - servidor Flask + Socket.IO e thread de leitura da porta serial.
- `app/static/js/app.js` - javascript do frontend que processa eventos Socket.IO e atualiza mapa e tabela.
- `app/templates/index.html` - página principal (template HTML com mapa e tabela) — caso não exista, crie-a com elementos esperados (`mapDIV`, `table`, etc.).
- `module.py` - módulo local (importado em `app.py`) que contém funções de comunicação serial (`base_com`, `list_ports`, `com.read_response`, etc.).

## Eventos WebSocket

O backend emite, entre outros, os seguintes eventos:

- `updateRocket` - dados do foguete (ex.: latitude/longitude/altitude/satellites/rssi/pqd/time). O arquivo `app/static/js/app.js` já implementa o listener para esse evento e atualiza o mapa e a tabela.
- `updateSat` - dados do satélite/busca (ex.: latitude/longitude/altitude/temperatura/umidade/pressao/rssi/pqd/time).

IMPORTANTE: atualmente o frontend implementa apenas a rotina para `updateRocket` (ver `app/static/js/app.js`). Será necessário criar a rotina JavaScript e os elementos HTML correspondentes para tratar e exibir os dados do satélite emitidos no evento `updateSat`.

Sugestões mínimas para implementar a rotina do satélite:

- No HTML: criar uma tabela ou área de informações separada (por exemplo `#satTable`) para mostrar latitude, longitude, altura, temperatura, umidade, pressão e RSSI.
- No JS (`app/static/js/app.js`): adicionar um `socket.on('updateSat', function(msg) { ... })` que faça parsing dos campos recebidos e atualize o DOM e o mapa (por exemplo, adicionando markers numa layer separada ou reutilizando `layerGroup`).

## Logs

O `app/app.py` grava os pacotes recebidos em `MapFlask/log.csv` para posterior análise. Cada linha contém os campos brutos recebidos e um timestamp.

## Desenvolvimento e testes

- Para testar sem hardware, pode simular a saída serial criando um pequeno script que emite linhas no mesmo formato esperado e conectando-o em uma porta serial virtual (são várias ferramentas para isso no Linux, ex.: `socat` ou `tty0tty`).

## String esperada

`TEAM_ID,millis,count,hora,data,alt,lat,lon,sat,altp,temp,p,gp,gr,gy,ap,ar,ay,pqd,rssi` ou alguma mensagem específica. No caso do primeiro tipo:

### Explicação das variáveis

- TEAM_ID — identificador da equipe/dispositivo (string).
- millis — timestamp em milissegundos (normalmente desde a inicialização do equipamento).
- count — contador sequencial de pacotes recebidos (inteiro).
- hora — hora da leitura (formato típico HH:MM:SS, string).
- data — data da leitura (formato típico YYYY-MM-DD ou DD/MM/YYYY, string).
- alt — altitude GPS em metros (float).
- lat — latitude em graus decimais (float, ex.: -23.5505).
- lon — longitude em graus decimais (float).
- sat — número de satélites usados no GPS (inteiro).
- altp — altitude por barômetro (altitude barométrica) em metros — pode diferir de `alt` (float).
- temp — temperatura em °C (float).
- umi — umidade relativa em % (float).
- p — pressão atmosférica em atm (float).
- gp — leitura do giroscópio eixo “pitch” (deg/s; float).
- gr — leitura do giroscópio eixo “roll” (deg/s; float).
- gy — leitura do giroscópio eixo “yaw” (deg/s; float).
- ap — leitura do acelerômetro eixo “pitch” (m/s² ou g; float).
- ar — leitura do acelerômetro eixo “roll” (m/s² ou g; float).
- ay — leitura do acelerômetro eixo “yaw” (m/s² ou g; float).
- pqd — indicador de abertura do paraquedas (bool).
- rssi — intensidade do sinal recebido (RSSI).
