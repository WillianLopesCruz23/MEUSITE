<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dashboard IoT - Automação Residencial</title>
  <!-- Biblioteca MQTT.js via CDN -->
  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
  <style>
    :root {
      --bg-color: #e0f2fe;        /* Azul claro minimalista */
      --card-bg: #ffffff;         /* Card branco */
      --text-main: #0f172a;       /* Texto escuro */
      --text-muted: #64748b;      /* Texto secundário */
      --primary: #0284c7;         /* Azul destaque */
      --primary-hover: #0369a1;   /* Hover azul */
      --danger: #ef4444;          /* Vermelho desligar */
      --danger-hover: #dc2626;
      --success: #22c55e;         /* Verde conectado */
      --led-off: #cbd5e1;         /* Cinza LED apagado */
      --led-on: #facc15;          /* Amarelo LED aceso */
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
    }

    body {
      background-color: var(--bg-color);
      color: var(--text-main);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 2rem 1rem;
    }

    /* Cabeçalho */
    header {
      text-align: center;
      margin-bottom: 1.5rem;
    }

    header h1 {
      font-size: 2.2rem;
      color: #0369a1;
      font-weight: 700;
    }

    header p {
      color: var(--text-muted);
      font-size: 1rem;
      margin-top: 0.3rem;
    }

    /* Regra de Ouro / Configuração do Tópico */
    .config-box {
      background: #ffffff;
      border: 2px solid #bae6fd;
      border-radius: 12px;
      padding: 1rem 1.5rem;
      max-width: 900px;
      width: 100%;
      margin-bottom: 1.5rem;
      box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
    }

    .config-box h3 {
      font-size: 0.95rem;
      color: #0369a1;
      margin-bottom: 0.5rem;
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .config-inputs {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
    }

    .config-inputs input {
      flex: 1;
      min-width: 250px;
      padding: 0.6rem 0.8rem;
      border: 1px solid #cbd5e1;
      border-radius: 8px;
      font-size: 0.9rem;
      outline: none;
    }

    .config-inputs input:focus {
      border-color: var(--primary);
    }

    /* Barra de Status */
    .status-bar {
      background: white;
      padding: 0.6rem 1.2rem;
      border-radius: 30px;
      font-size: 0.88rem;
      font-weight: 600;
      display: flex;
      align-items: center;
      gap: 10px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.05);
      margin-bottom: 2rem;
    }

    .status-dot {
      width: 12px;
      height: 12px;
      border-radius: 50%;
      background-color: var(--danger);
      transition: background-color 0.3s ease;
    }

    .status-dot.connected {
      background-color: var(--success);
    }

    /* Grid do Painel Principal */
    .dashboard-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(270px, 1fr));
      gap: 1.5rem;
      width: 100%;
      max-width: 1000px;
    }

    /* Estilo dos Cards */
    .card {
      background-color: var(--card-bg);
      border-radius: 16px;
      padding: 1.5rem;
      box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.04);
      display: flex;
      flex-direction: column;
      align-items: center;
      text-align: center;
      position: relative;
    }

    .pin-badge {
      position: absolute;
      top: 12px;
      right: 12px;
      background: #f1f5f9;
      color: var(--text-muted);
      font-size: 0.75rem;
      font-weight: 700;
      padding: 4px 8px;
      border-radius: 6px;
    }

    .card-title {
      font-size: 1.2rem;
      font-weight: 600;
      margin-top: 0.5rem;
      margin-bottom: 0.2rem;
    }

    /* LED Virtual com Brilho Dinâmico */
    .virtual-led {
      width: 60px;
      height: 60px;
      border-radius: 50%;
      background-color: var(--led-off);
      margin: 1.2rem 0;
      box-shadow: inset 0 2px 5px rgba(0,0,0,0.2);
      transition: all 0.3s ease;
    }

    .virtual-led.active {
      background-color: var(--led-on);
      box-shadow: 0 0 25px var(--led-on), inset 0 2px 4px rgba(255,255,255,0.8);
    }

    /* Botões */
    .btn-group {
      display: flex;
      gap: 8px;
      width: 100%;
      margin-top: auto;
    }

    button {
      flex: 1;
      padding: 0.7rem;
      border: none;
      border-radius: 8px;
      font-weight: 600;
      font-size: 0.9rem;
      cursor: pointer;
      transition: background-color 0.2s ease;
    }

    .btn-on {
      background-color: var(--primary);
      color: white;
    }
    .btn-on:hover { background-color: var(--primary-hover); }

    .btn-off {
      background-color: var(--danger);
      color: white;
    }
    .btn-off:hover { background-color: var(--danger-hover); }

    /* Slider de PWM */
    .slider-container {
      width: 100%;
      margin: 0.8rem 0;
    }

    .slider {
      width: 100%;
      height: 8px;
      border-radius: 4px;
      background: #e2e8f0;
      outline: none;
      accent-color: var(--primary);
      cursor: pointer;
    }

    .pwm-readout {
      font-size: 0.9rem;
      font-weight: 600;
      color: var(--primary);
      margin-top: 4px;
    }

    /* Sensor Analógico A0 */
    .sensor-display {
      margin: 1rem 0;
      width: 100%;
    }

    .sensor-value {
      font-size: 2.8rem;
      font-weight: 800;
      color: var(--primary);
      line-height: 1;
    }

    .sensor-bar-bg {
      width: 100%;
      height: 10px;
      background: #e2e8f0;
      border-radius: 5px;
      margin-top: 1rem;
      overflow: hidden;
    }

    .sensor-bar-fill {
      height: 100%;
      width: 0%;
      background: var(--primary);
      transition: width 0.3s ease;
    }

    /* Console de Logs MQTT */
    .log-container {
      margin-top: 2rem;
      width: 100%;
      max-width: 1000px;
      background: #0f172a;
      color: #38bdf8;
      border-radius: 12px;
      padding: 1rem;
      font-family: monospace;
      font-size: 0.85rem;
      height: 140px;
      overflow-y: auto;
    }

    .log-title {
      color: #94a3b8;
      font-size: 0.75rem;
      text-transform: uppercase;
      margin-bottom: 0.5rem;
      border-bottom: 1px solid #334155;
      padding-bottom: 4px;
    }
  </style>
</head>
<body>

  <header>
    <h1>Casa Inteligente IoT</h1>
    <p>Painel de Controle MQTT via Nuvem HiveMQ</p>
  </header>

  <!-- Seção Regra de Ouro: Prefixo do Tópico Customizado -->
  <div class="config-box">
    <h3>🚨 Regra de Ouro: Identificador Único da Bancada</h3>
    <div class="config-inputs">
      <input type="text" id="topicPrefix" value="bancada_grupo1/casa/" placeholder="Ex: bancada_do_joao/casa/">
      <button class="btn-on" style="flex: 0 0 120px;" onclick="atualizarTopicos()">Aplicar Tópico</button>
    </div>
  </div>

  <!-- Status da Conexão MQTT -->
  <div class="status-bar">
    <div id="statusDot" class="status-dot"></div>
    <span id="statusText">Conectando ao Broker HiveMQ...</span>
  </div>

  <!-- Cards de Dispositivos e Cômodos -->
  <main class="dashboard-grid">

    <!-- Card 1: LED 1 - Sala (Pino 8) -->
    <div class="card">
      <span class="pin-badge">PIN 8</span>
      <div class="card-title">🛋️️ Luz da Sala</div>
      <p style="font-size: 0.8rem; color: var(--text-muted);">LED 1 (Digital)</p>
      <div id="ledSala" class="virtual-led"></div>
      <div class="btn-group">
        <button class="btn-on" onclick="enviarComando('sala', 'ON')">Ligar</button>
        <button class="btn-off" onclick="enviarComando('sala', 'OFF')">Desligar</button>
      </div>
    </div>

    <!-- Card 2: LED 2 - Quarto (Pino PWM 10) -->
    <div class="card">
      <span class="pin-badge">PIN 10 (PWM)</span>
      <div class="card-title">🛏️ Luz do Quarto</div>
      <p style="font-size: 0.8rem; color: var(--text-muted);">LED 2 (Brilho / Dimerizável)</p>
      <div id="ledQuarto" class="virtual-led"></div>
      
      <div class="slider-container">
        <input type="range" min="0" max="100" value="0" class="slider" id="pwmSlider" oninput="enviarPWM(this.value)">
        <div class="pwm-readout">Intensidade: <span id="pwmPercent">0</span>%</div>
      </div>

      <div class="btn-group">
        <button class="btn-on" onclick="enviarComando('quarto', 'ON')">Ligar (100%)</button>
        <button class="btn-off" onclick="enviarComando('quarto', 'OFF')">Desligar (0%)</button>
      </div>
    </div>

    <!-- Card 3: LED 3 - Cozinha (Pino 13) -->
    <div class="card">
      <span class="pin-badge">PIN 13</span>
      <div class="card-title">🍳 Luz da Cozinha</div>
      <p style="font-size: 0.8rem; color: var(--text-muted);">LED 3 (Digital)</p>
      <div id="ledCozinha" class="virtual-led"></div>
      <div class="btn-group">
        <button class="btn-on" onclick="enviarComando('cozinha', 'ON')">Ligar</button>
        <button class="btn-off" onclick="enviarComando('cozinha', 'OFF')">Desligar</button>
      </div>
    </div>

    <!-- Card 4: Entrada Analógica A0 -->
    <div class="card">
      <span class="pin-badge">PIN A0</span>
      <div class="card-title">📊 Leitura Analógica</div>
      <p style="font-size: 0.8rem; color: var(--text-muted);">Sensor do Arduino (ADC 0-1023)</p>
      
      <div class="sensor-display">
        <div class="sensor-value" id="analogValue">0</div>
        <div class="sensor-bar-bg">
          <div class="sensor-bar-fill" id="analogBar"></div>
        </div>
      </div>
      <p style="font-size: 0.75rem; color: var(--text-muted);">Atualizado em tempo real via MQTT</p>
    </div>

  </main>

  <!-- Console de Mensagens / Debugger -->
  <div class="log-container">
    <div class="log-title">Console de Tráfego MQTT (Debug)</div>
    <div id="logOutput"></div>
  </div>

  <script>
    // Variáveis do MQTT
    let TOPICO_BASE = document.getElementById("topicPrefix").value;
    const brokerUrl = "wss://broker.hivemq.com:8000/mqtt"; // WebSocket HiveMQ
    const clientId = "dashboard_web_" + Math.random().toString(16).substr(2, 6);

    const client = mqtt.connect(brokerUrl, { clientId: clientId });

    // Elementos da Interface
    const statusDot = document.getElementById("statusDot");
    const statusText = document.getElementById("statusText");
    const logOutput = document.getElementById("logOutput");

    function log(msg) {
      const hora = new Date().toLocaleTimeString();
      logOutput.innerHTML = `[${hora}] ${msg}<br>` + logOutput.innerHTML;
    }

    // Conexão com Broker
    client.on("connect", () => {
      statusDot.classList.add("connected");
      statusText.innerText = "Conectado ao HiveMQ Cloud (WebSocket)";
      log("Conectado com sucesso ao broker!");
      inscreverTopicos();
    });

    client.on("offline", () => {
      statusDot.classList.remove("connected");
      statusText.innerText = "Desconectado do Broker";
      log("Conexão perdida.");
    });

    function inscreverTopicos() {
      // Subscrição nos canais de status vindos do Arduino/Python
      const topicos = [
        TOPICO_BASE + "sala/status",
        TOPICO_BASE + "quarto/status",
        TOPICO_BASE + "cozinha/status",
        TOPICO_BASE + "analogico"
      ];

      topicos.forEach(t => {
        client.subscribe(t);
        log(`Inscrito no tópico: <b>${t}</b>`);
      });
    }

    function atualizarTopicos() {
      TOPICO_BASE = document.getElementById("topicPrefix").value;
      if(!TOPICO_BASE.endsWith('/')) TOPICO_BASE += '/';
      log(`Tópico base alterado para: <b>${TOPICO_BASE}</b>`);
      inscreverTopicos();
    }

    // Enviar comandos digitais (ON/OFF)
    function enviarComando(comodo, acao) {
      const topico = TOPICO_BASE + comodo + "/set";
      client.publish(topico, acao);
      log(`Publicado em [${topico}]: ${acao}`);

      // Atualização imediata do slider local se for no quarto
      if (comodo === 'quarto') {
        const val = (acao === 'ON') ? 100 : 0;
        document.getElementById('pwmSlider').value = val;
        document.getElementById('pwmPercent').innerText = val;
      }
    }

    // Enviar controle gradual PWM (0% a 100%)
    function enviarPWM(valor) {
      document.getElementById('pwmPercent').innerText = valor;
      const topico = TOPICO_BASE + "quarto/pwm";
      client.publish(topico, valor.toString());
      log(`Publicado PWM em [${topico}]: ${valor}%`);
    }

    // Recebimento de Mensagens
    client.on("message", (topic, message) => {
      const msgStr = message.toString();
      log(`Recebido em [${topic}]: <b>${msgStr}</b>`);

      if (topic === TOPICO_BASE + "sala/status") {
        atualizarLED("ledSala", msgStr === "ON");
      } 
      else if (topic === TOPICO_BASE + "quarto/status") {
        atualizarLED("ledQuarto", msgStr === "ON" || parseInt(msgStr) > 0);
      } 
      else if (topic === TOPICO_BASE + "cozinha/status") {
        atualizarLED("ledCozinha", msgStr === "ON");
      } 
      else if (topic === TOPICO_BASE + "analogico") {
        const valor = parseInt(msgStr);
        document.getElementById("analogValue").innerText = valor;
        
        // Converte escala de 0-1023 para porcentagem (0-100%) da barra
        const porcentagem = Math.min(100, Math.max(0, (valor / 1023) * 100));
        document.getElementById("analogBar").style.width = porcentagem + "%";
      }
    });

    function atualizarLED(id, ligado) {
      const el = document.getElementById(id);
      if (ligado) {
        el.classList.add("active");
      } else {
        el.classList.remove("active");
      }
    }
  </script>
</body>
</html>
