<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Smart Home IoT - Painel de Controle Minimalista</title>
  <!-- Fontes e Ícones Phosphor -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap" rel="stylesheet">
  <script src="https://unpkg.com/@phosphor-icons/web"></script>
  <!-- Biblioteca MQTT.js via CDN -->
  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>

  <style>
    :root {
      /* Paleta de Cores Minimalista - Azul Claro & UI Premium */
      --bg-gradient: linear-gradient(135deg, #e0f2fe 0%, #bae6fd 50%, #f0f9ff 100%);
      --glass-bg: rgba(255, 255, 255, 0.75);
      --glass-border: rgba(255, 255, 255, 0.8);
      --card-shadow: 0 20px 40px -15px rgba(2, 132, 199, 0.12);
      
      --text-main: #0c4a6e;
      --text-muted: #64748b;
      --primary: #0284c7;
      --primary-hover: #0369a1;
      
      --danger: #f43f5e;
      --danger-hover: #e11d48;
      --success: #10b981;
      
      --bulb-off: #cbd5e1;
      --bulb-glow: #f59e0b;
      --bulb-shadow: rgba(245, 158, 11, 0.4);
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Plus Jakarta Sans', -apple-system, sans-serif;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      background: var(--bg-gradient);
      background-attachment: fixed;
      color: var(--text-main);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 2.5rem 1rem;
    }

    /* Container Principal */
    .app-container {
      width: 100%;
      max-width: 1080px;
      display: flex;
      flex-direction: column;
      gap: 1.8rem;
    }

    /* Cabeçalho */
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: var(--glass-bg);
      backdrop-filter: blur(16px);
      border: 1px solid var(--glass-border);
      padding: 1.25rem 2rem;
      border-radius: 24px;
      box-shadow: var(--card-shadow);
      flex-wrap: wrap;
      gap: 1rem;
    }

    .brand {
      display: flex;
      align-items: center;
      gap: 12px;
    }

    .brand-icon {
      width: 46px;
      height: 46px;
      background: #0284c7;
      color: white;
      border-radius: 14px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.6rem;
      box-shadow: 0 8px 16px rgba(2, 132, 199, 0.25);
    }

    .brand-text h1 {
      font-size: 1.4rem;
      font-weight: 800;
      color: #0369a1;
      letter-spacing: -0.5px;
    }

    .brand-text p {
      font-size: 0.85rem;
      color: var(--text-muted);
      font-weight: 500;
    }

    /* Badge de Conexão */
    .connection-badge {
      display: flex;
      align-items: center;
      gap: 8px;
      background: rgba(255, 255, 255, 0.8);
      padding: 0.6rem 1.2rem;
      border-radius: 50px;
      font-size: 0.85rem;
      font-weight: 700;
      border: 1px solid rgba(2, 132, 199, 0.15);
    }

    .status-dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background-color: var(--danger);
      box-shadow: 0 0 0 4px rgba(244, 63, 94, 0.2);
      transition: all 0.3s ease;
    }

    .status-dot.connected {
      background-color: var(--success);
      box-shadow: 0 0 0 4px rgba(16, 185, 129, 0.2);
    }

    /* Banner da Regra de Ouro */
    .rule-card {
      background: linear-gradient(135deg, #ffffff 0%, #f0f9ff 100%);
      border: 1px solid #bae6fd;
      border-radius: 20px;
      padding: 1.25rem 1.8rem;
      box-shadow: var(--card-shadow);
      display: flex;
      flex-direction: column;
      gap: 0.8rem;
    }

    .rule-header {
      display: flex;
      align-items: center;
      gap: 8px;
      color: #0369a1;
      font-weight: 700;
      font-size: 1rem;
    }

    .rule-input-group {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
    }

    .rule-input-group input {
      flex: 1;
      min-width: 280px;
      padding: 0.75rem 1rem;
      border: 1px solid #93c5fd;
      border-radius: 12px;
      font-size: 0.95rem;
      font-weight: 600;
      color: #0c4a6e;
      outline: none;
      background: white;
      transition: all 0.2s ease;
    }

    .rule-input-group input:focus {
      border-color: var(--primary);
      box-shadow: 0 0 0 3px rgba(2, 132, 199, 0.15);
    }

    .btn-apply {
      padding: 0.75rem 1.5rem;
      background: var(--primary);
      color: white;
      border: none;
      border-radius: 12px;
      font-weight: 700;
      cursor: pointer;
      transition: all 0.2s ease;
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .btn-apply:hover {
      background: var(--primary-hover);
      transform: translateY(-1px);
    }

    /* Grid dos Cômodos */
    .dashboard-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
      gap: 1.5rem;
    }

    /* Cards Minimalistas */
    .card {
      background: var(--glass-bg);
      backdrop-filter: blur(12px);
      border: 1px solid var(--glass-border);
      border-radius: 24px;
      padding: 1.8rem 1.5rem;
      box-shadow: var(--card-shadow);
      display: flex;
      flex-direction: column;
      align-items: center;
      position: relative;
      transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }

    .card:hover {
      transform: translateY(-4px);
      box-shadow: 0 25px 50px -12px rgba(2, 132, 199, 0.18);
    }

    .pin-badge {
      position: absolute;
      top: 16px;
      right: 16px;
      background: rgba(2, 132, 199, 0.08);
      color: var(--primary);
      font-size: 0.72rem;
      font-weight: 800;
      padding: 4px 10px;
      border-radius: 20px;
      letter-spacing: 0.5px;
    }

    .room-header {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-top: 0.2rem;
      margin-bottom: 0.2rem;
    }

    .room-header i {
      font-size: 1.4rem;
      color: var(--primary);
    }

    .room-title {
      font-size: 1.15rem;
      font-weight: 700;
      color: #0c4a6e;
    }

    .room-subtitle {
      font-size: 0.8rem;
      color: var(--text-muted);
      font-weight: 500;
      margin-bottom: 1rem;
    }

    /* Lâmpada Virtual Gradiente com Efeito Glow Realista */
    .bulb-container {
      position: relative;
      width: 80px;
      height: 80px;
      display: flex;
      align-items: center;
      justify-content: center;
      margin: 0.8rem 0 1.2rem 0;
    }

    .bulb-icon {
      font-size: 3.8rem;
      color: var(--bulb-off);
      transition: all 0.4s ease;
      z-index: 2;
    }

    .bulb-glow-effect {
      position: absolute;
      width: 60px;
      height: 60px;
      border-radius: 50%;
      background: var(--bulb-glow);
      opacity: 0;
      filter: blur(20px);
      transition: all 0.4s ease;
      z-index: 1;
    }

    /* Estado Ativo da Lâmpada */
    .card.active .bulb-icon {
      color: var(--bulb-glow);
      filter: drop-shadow(0 0 10px rgba(245, 158, 11, 0.6));
    }

    .card.active .bulb-glow-effect {
      opacity: 0.8;
      transform: scale(1.3);
    }

    /* Controles de Ação (Botões Ligar / Desligar) */
    .btn-group {
      display: flex;
      gap: 8px;
      width: 100%;
      margin-top: auto;
    }

    .btn-action {
      flex: 1;
      padding: 0.75rem 0.5rem;
      border: none;
      border-radius: 14px;
      font-size: 0.88rem;
      font-weight: 700;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      transition: all 0.2s ease;
    }

    .btn-on {
      background: var(--primary);
      color: white;
      box-shadow: 0 4px 12px rgba(2, 132, 199, 0.25);
    }
    .btn-on:hover { background: var(--primary-hover); transform: scale(1.02); }

    .btn-off {
      background: #f1f5f9;
      color: #64748b;
    }
    .btn-off:hover { background: #e2e8f0; color: var(--danger); }

    /* Slider de Brilho PWM */
    .slider-box {
      width: 100%;
      background: rgba(255, 255, 255, 0.6);
      border: 1px solid rgba(2, 132, 199, 0.1);
      padding: 0.8rem 1rem;
      border-radius: 16px;
      margin-bottom: 1rem;
    }

    .slider-info {
      display: flex;
      justify-content: space-between;
      font-size: 0.8rem;
      font-weight: 700;
      color: var(--text-main);
      margin-bottom: 6px;
    }

    .slider-info span:last-child {
      color: var(--primary);
    }

    .range-slider {
      width: 100%;
      height: 6px;
      border-radius: 3px;
      background: #cbd5e1;
      outline: none;
      accent-color: var(--primary);
      cursor: pointer;
    }

    /* Gauge / Display Analógico A0 */
    .sensor-card-body {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 0.5rem 0;
    }

    .sensor-big-number {
      font-size: 3rem;
      font-weight: 800;
      color: var(--primary);
      line-height: 1;
      letter-spacing: -1px;
    }

    .voltage-readout {
      font-size: 0.85rem;
      font-weight: 700;
      color: var(--text-muted);
      margin-top: 4px;
      margin-bottom: 1rem;
    }

    .progress-bar-bg {
      width: 100%;
      height: 12px;
      background: #e2e8f0;
      border-radius: 10px;
      overflow: hidden;
      padding: 2px;
    }

    .progress-bar-fill {
      height: 100%;
      width: 0%;
      background: linear-gradient(90deg, #38bdf8 0%, #0284c7 100%);
      border-radius: 8px;
      transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }

    /* Console de Terminal Transparente */
    .terminal-section {
      background: rgba(15, 23, 42, 0.92);
      backdrop-filter: blur(16px);
      border-radius: 20px;
      padding: 1.2rem 1.5rem;
      color: #e2e8f0;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
    }

    .terminal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-bottom: 0.8rem;
      border-bottom: 1px solid rgba(255, 255, 255, 0.1);
      margin-bottom: 0.8rem;
    }

    .terminal-title {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 0.85rem;
      font-weight: 700;
      color: #38bdf8;
      text-transform: uppercase;
      letter-spacing: 0.5px;
    }

    .terminal-logs {
      font-family: 'Courier New', Courier, monospace;
      font-size: 0.82rem;
      height: 110px;
      overflow-y: auto;
      display: flex;
      flex-direction: column-reverse;
      gap: 4px;
    }

    .log-item {
      line-height: 1.4;
      color: #94a3b8;
    }

    .log-item b {
      color: #38bdf8;
    }

    .log-item.sent { color: #a7f3d0; }
    .log-item.received { color: #fef08a; }

    /* Responsividade */
    @media (max-width: 600px) {
      body { padding: 1rem; }
      header { text-align: center; justify-content: center; }
      .brand { flex-direction: column; }
    }
  </style>
</head>
<body>

  <div class="app-container">

    <!-- Cabeçalho Principal -->
    <header>
      <div class="brand">
        <div class="brand-icon"><i class="ph-bold ph-house-line"></i></div>
        <div class="brand-text">
          <h1>Automação Residencial</h1>
          <p>Painel de Controle IoT via Nuvem HiveMQ</p>
        </div>
      </div>

      <div class="connection-badge">
        <div id="statusDot" class="status-dot"></div>
        <span id="statusText">Conectando...</span>
      </div>
    </header>

    <!-- Regra de Ouro (Tópico Exclusivo) -->
    <div class="rule-card">
      <div class="rule-header">
        <i class="ph-bold ph-warning-circle" style="font-size: 1.3rem; color: #f59e0b;"></i>
        <span>🚨 Regra de Ouro: Identificador Exclusivo da Bancada</span>
      </div>
      <div class="rule-input-group">
        <input type="text" id="topicPrefix" value="bancada_grupo1/casa/" placeholder="Ex: bancada_do_joao/casa/">
        <button class="btn-apply" onclick="atualizarTopicos()">
          <i class="ph-bold ph-check"></i> Aplicar Tópico
        </button>
      </div>
    </div>

    <!-- Grid do Painel de Controle -->
    <main class="dashboard-grid">

      <!-- Card 1: Luz da Sala (Porta 8) -->
      <div class="card" id="cardSala">
        <span class="pin-badge">PIN 8</span>
        <div class="room-header">
          <i class="ph-bold ph-armchair"></i>
          <span class="room-title">Luz da Sala</span>
        </div>
        <p class="room-subtitle">LED 1 (Saída Digital)</p>

        <div class="bulb-container">
          <div class="bulb-glow-effect"></div>
          <i class="ph-fill ph-lightbulb bulb-icon"></i>
        </div>

        <div class="btn-group">
          <button class="btn-action btn-on" onclick="enviarComando('sala', 'ON')">
            <i class="ph-bold ph-power"></i> Ligar
          </button>
          <button class="btn-action btn-off" onclick="enviarComando('sala', 'OFF')">Desligar</button>
        </div>
      </div>

      <!-- Card 2: Luz do Quarto (Porta 10 PWM) -->
      <div class="card" id="cardQuarto">
        <span class="pin-badge">PIN 10 PWM</span>
        <div class="room-header">
          <i class="ph-bold ph-bed"></i>
          <span class="room-title">Luz do Quarto</span>
        </div>
        <p class="room-subtitle">LED 2 (Brilho Dimerizável)</p>

        <div class="bulb-container">
          <div class="bulb-glow-effect" id="glowQuarto"></div>
          <i class="ph-fill ph-lightbulb bulb-icon" id="iconQuarto"></i>
        </div>

        <div class="slider-box">
          <div class="slider-info">
            <span>Intensidade PWM</span>
            <span id="pwmPercent">0%</span>
          </div>
          <input type="range" min="0" max="100" value="0" class="range-slider" id="pwmSlider" oninput="enviarPWM(this.value)">
        </div>

        <div class="btn-group">
          <button class="btn-action btn-on" onclick="enviarComando('quarto', 'ON')">
            <i class="ph-bold ph-power"></i> 100%
          </button>
          <button class="btn-action btn-off" onclick="enviarComando('quarto', 'OFF')">0%</button>
        </div>
      </div>

      <!-- Card 3: Luz da Cozinha (Porta 13) -->
      <div class="card" id="cardCozinha">
        <span class="pin-badge">PIN 13</span>
        <div class="room-header">
          <i class="ph-bold ph-cooking-pot"></i>
          <span class="room-title">Luz da Cozinha</span>
        </div>
        <p class="room-subtitle">LED 3 (Saída Digital)</p>

        <div class="bulb-container">
          <div class="bulb-glow-effect"></div>
          <i class="ph-fill ph-lightbulb bulb-icon"></i>
        </div>

        <div class="btn-group">
          <button class="btn-action btn-on" onclick="enviarComando('cozinha', 'ON')">
            <i class="ph-bold ph-power"></i> Ligar
          </button>
          <button class="btn-action btn-off" onclick="enviarComando('cozinha', 'OFF')">Desligar</button>
        </div>
      </div>

      <!-- Card 4: Entrada Analógica (Porta A0) -->
      <div class="card">
        <span class="pin-badge">PIN A0</span>
        <div class="room-header">
          <i class="ph-bold ph-gauge"></i>
          <span class="room-title">Entrada A0</span>
        </div>
        <p class="room-subtitle">Leitura do Sensor Analógico</p>

        <div class="sensor-card-body">
          <div class="sensor-big-number" id="analogValue">0</div>
          <div class="voltage-readout" id="voltageValue">0.00 V (0%)</div>

          <div class="progress-bar-bg">
            <div class="progress-bar-fill" id="analogBar"></div>
          </div>
        </div>
      </div>

    </main>

    <!-- Console Terminal MQTT -->
    <div class="terminal-section">
      <div class="terminal-header">
        <div class="terminal-title">
          <i class="ph-bold ph-terminal-window"></i> Console de Tráfego MQTT
        </div>
        <span style="font-size: 0.75rem; color: #64748b;">WebSocket @ broker.hivemq.com</span>
      </div>
      <div class="terminal-logs" id="logOutput">
        <div class="log-item">Iniciando aplicação web...</div>
      </div>
    </div>

  </div>

  <script>
    // Variáveis Globais do MQTT
    let TOPICO_BASE = document.getElementById("topicPrefix").value;
    const brokerUrl = "wss://broker.hivemq.com:8000/mqtt";
    const clientId = "web_dashboard_ui_" + Math.random().toString(16).substr(2, 6);

    const client = mqtt.connect(brokerUrl, { clientId: clientId });

    // Referências do DOM
    const statusDot = document.getElementById("statusDot");
    const statusText = document.getElementById("statusText");
    const logOutput = document.getElementById("logOutput");

    function addLog(msg, type = '') {
      const hora = new Date().toLocaleTimeString();
      const div = document.createElement('div');
      div.className = `log-item ${type}`;
      div.innerHTML = `[${hora}] ${msg}`;
      logOutput.prepend(div);
    }

    // Eventos de Conexão MQTT
    client.on("connect", () => {
      statusDot.classList.add("connected");
      statusText.innerText = "Conectado ao HiveMQ";
      addLog("Conexão estabelecida com sucesso ao Broker MQTT Cloud!", "sent");
      inscreverTopicos();
    });

    client.on("offline", () => {
      statusDot.classList.remove("connected");
      statusText.innerText = "Desconectado";
      addLog("Conexão MQTT perdida.", "received");
    });

    function inscreverTopicos() {
      const topicos = [
        TOPICO_BASE + "sala/status",
        TOPICO_BASE + "quarto/status",
        TOPICO_BASE + "cozinha/status",
        TOPICO_BASE + "analogico"
      ];

      topicos.forEach(t => {
        client.subscribe(t);
        addLog(`Inscrito para escutar o tópico: <b>${t}</b>`);
      });
    }

    function atualizarTopicos() {
      TOPICO_BASE = document.getElementById("topicPrefix").value;
      if(!TOPICO_BASE.endsWith('/')) TOPICO_BASE += '/';
      addLog(`Prefixo alterado para: <b>${TOPICO_BASE}</b>`);
      inscreverTopicos();
    }

    // Publicação de Comandos
    function enviarComando(comodo, acao) {
      const topico = TOPICO_BASE + comodo + "/set";
      client.publish(topico, acao);
      addLog(`Publicado em [${topico}]: <b>${acao}</b>`, "sent");

      if (comodo === 'quarto') {
        const pct = (acao === 'ON') ? 100 : 0;
        document.getElementById('pwmSlider').value = pct;
        atualizarVisualPWM(pct);
      }
    }

    function enviarPWM(valor) {
      atualizarVisualPWM(valor);
      const topico = TOPICO_BASE + "quarto/pwm";
      client.publish(topico, valor.toString());
      addLog(`Publicado PWM em [${topico}]: <b>${valor}%</b>`, "sent");
    }

    function atualizarVisualPWM(valor) {
      document.getElementById('pwmPercent').innerText = valor + "%";
      const opacity = valor / 100;
      const glow = document.getElementById('glowQuarto');
      const icon = document.getElementById('iconQuarto');
      
      glow.style.opacity = opacity * 0.8;
      glow.style.transform = `scale(${1 + (opacity * 0.3)})`;

      if (valor > 0) {
        icon.style.color = 'var(--bulb-glow)';
        document.getElementById('cardQuarto').classList.add('active');
      } else {
        icon.style.color = 'var(--bulb-off)';
        document.getElementById('cardQuarto').classList.remove('active');
      }
    }

    // Recebimento de Mensagens em Tempo Real
    client.on("message", (topic, message) => {
      const msgStr = message.toString();
      addLog(`Recebido [${topic}]: <b>${msgStr}</b>`, "received");

      if (topic === TOPICO_BASE + "sala/status") {
        setCardActive("cardSala", msgStr === "ON");
      } 
      else if (topic === TOPICO_BASE + "quarto/status") {
        const isON = msgStr === "ON" || (parseInt(msgStr) > 0);
        setCardActive("cardQuarto", isON);
      } 
      else if (topic === TOPICO_BASE + "cozinha/status") {
        setCardActive("cardCozinha", msgStr === "ON");
      } 
      else if (topic === TOPICO_BASE +
