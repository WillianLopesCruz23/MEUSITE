<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Painel de Automação IoT - Casa Inteligente</title>
  <!-- Biblioteca MQTT.js via CDN -->
  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
  <style>
    :root {
      --bg-color: #e0f2fe;        /* Fundo azul claro minimalista */
      --card-bg: #ffffff;         /* Fundo branco dos cards */
      --text-main: #0f172a;       /* Texto escuro */
      --text-muted: #64748b;      /* Texto de apoio */
      --primary: #0284c7;         /* Azul primário */
      --primary-hover: #0369a1;   /* Hover azul */
      --danger: #ef4444;          /* Vermelho para desligar */
      --danger-hover: #dc2626;
      --success: #22c55e;         /* Verde conectado */
      --led-off: #cbd5e1;         /* LED apagado */
      --led-on: #facc15;          /* LED aceso com brilho */
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

    /* Regra de Ouro: Identificador Único */
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

    .dashboard-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(270px, 1fr));
      gap: 1.5rem;
      width: 100%;
      max-width: 1000px;
    }

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
    }

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
  </style>
</head>
<body>

  <header>
    <h1>Casa Inteligente IoT</h1>
    <p>Painel de Controle MQTT via Nuvem HiveMQ</p>
  </header>

  <!-- Regra de Ouro: Identificador Único de Tópico -->
  <div class="config-box">
    <h3>🚨 Regra de Ouro: Evite Linhas Cruzadas</h3>
    <div class="config-inputs">
      <input type="text" id="topicPrefix" value="bancada_grupo1/casa/" placeholder="Ex: bancada_do_joao/casa/">
      <button class="btn-on" style="flex: 0 0 120px;" onclick="atualizarTopicos()">Aplicar Tópico</button>
    </div>
  </div>

  <div class="status-bar">
    <div id="statusDot" class="status-dot"></div>
    <span id="statusText">Conectando ao HiveMQ Cloud...</span>
  </div>

  <main class="dashboard-grid">

    <!-- Card 1: LED 1 - Sala (Pino Digital 8) -->
    <div class="card">
      <span class="pin-badge">PIN 8</span>
      <div class="card-title">🛋️ Luz da Sala</div>
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
      <p style="font-size: 0.8rem; color: var(--text-muted);">LED 2 (Brilho PWM)</p>
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

    <!-- Card 3: LED 3 - Cozinha (Pino Digital 13) -->
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
      <p style="font-size: 0.75rem; color: var(--text-muted);">Atualização dinâmica via MQTT</p>
    </div>

  </main>

  <script>
    let TOPICO_BASE = document.getElementById("topicPrefix").value;
    const brokerUrl = "wss://broker.hivemq.com:8000/mqtt";
    const clientId = "web_dashboard_" + Math.random().toString(16).substr(2, 6);

    const client = mqtt.connect(brokerUrl, { clientId: clientId });

    const statusDot = document.getElementById("statusDot");
    const statusText = document.getElementById("statusText");

    client.on("connect", () => {
      statusDot.classList.add("connected");
      statusText.innerText = "Conectado ao HiveMQ Broker (WebSocket)";
      inscreverTopicos();
    });

    client.on("offline", () => {
      statusDot.classList.remove("connected");
      statusText.innerText = "Desconectado do Broker";
    });

    function inscreverTopicos() {
      client.subscribe(TOPICO_BASE + "sala/status");
      client.subscribe(TOPICO_BASE + "quarto/status");
      client.subscribe(TOPICO_BASE + "cozinha/status");
      client.subscribe(TOPICO_BASE + "analogico");
    }

    function atualizarTopicos() {
      TOPICO_BASE = document.getElementById("topicPrefix").value;
      if(!TOPICO_BASE.endsWith('/')) TOPICO_BASE += '/';
      inscreverTopicos();
    }

    function enviarComando(comodo, acao) {
      const topico = TOPICO_BASE + comodo + "/set";
      client.publish(topico, acao);

      if (comodo === 'quarto') {
        const val = (acao === 'ON') ? 100 : 0;
        document.getElementById('pwmSlider').value = val;
        document.getElementById('pwmPercent').innerText = val;
      }
    }

    function enviarPWM(valor) {
      document.getElementById('pwmPercent').innerText = valor;
      const topico = TOPICO_BASE + "quarto/pwm";
      client.publish(topico, valor.toString());
    }

    client.on("message", (topic, message) => {
      const msgStr = message.toString();

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
