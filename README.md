<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Painel de Automação Residencial IoT</title>
  <!-- Biblioteca MQTT JS via CDN -->
  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
  <style>
    :root {
      --bg-color: #e0f2fe;        /* Fundo azul claro */
      --card-bg: #ffffff;         /* Cards brancos */
      --text-main: #0f172a;       /* Texto escuro */
      --text-muted: #64748b;      /* Texto secundário */
      --primary: #0284c7;         /* Azul primário */
      --primary-hover: #0369a1;   /* Hover azul */
      --danger: #ef4444;          /* Vermelho para desligar */
      --danger-hover: #dc2626;
      --led-off: #cbd5e1;         /* Cinza para LED desligado */
      --led-on: #eab308;          /* Amarelo para LED ligado */
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
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
      margin-bottom: 2rem;
    }

    header h1 {
      font-size: 2.2rem;
      font-weight: 700;
      color: #0369a1;
      margin-bottom: 0.5rem;
    }

    header p {
      color: var(--text-muted);
      font-size: 1rem;
    }

    .status-bar {
      background: white;
      padding: 0.6rem 1.2rem;
      border-radius: 20px;
      font-size: 0.9rem;
      font-weight: 600;
      display: flex;
      align-items: center;
      gap: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.05);
      margin-bottom: 2rem;
    }

    .status-dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background-color: var(--danger);
    }

    .status-dot.connected {
      background-color: #22c55e;
    }

    .dashboard-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
      gap: 1.5rem;
      width: 100%;
      max-width: 1000px;
    }

    .card {
      background-color: var(--card-bg);
      border-radius: 16px;
      padding: 1.5rem;
      box-shadow: 0 4px 15px rgba(0, 0, 0, 0.04);
      display: flex;
      flex-direction: column;
      align-items: center;
      text-align: center;
      transition: transform 0.2s ease;
    }

    .card:hover {
      transform: translateY(-3px);
    }

    .card-title {
      font-size: 1.25rem;
      font-weight: 600;
      margin-bottom: 1rem;
      display: flex;
      align-items: center;
      gap: 8px;
    }

    /* Estilo do LED Virtual */
    .virtual-led {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      background-color: var(--led-off);
      margin: 1rem 0;
      box-shadow: inset 0 2px 5px rgba(0,0,0,0.2);
      transition: background-color 0.3s ease, box-shadow 0.3s ease;
    }

    .virtual-led.active {
      background-color: var(--led-on);
      box-shadow: 0 0 20px var(--led-on), inset 0 2px 5px rgba(255,255,255,0.5);
    }

    .btn-group {
      display: flex;
      gap: 0.5rem;
      width: 100%;
      margin-top: auto;
    }

    button {
      flex: 1;
      padding: 0.75rem;
      border: none;
      border-radius: 8px;
      font-weight: 600;
      cursor: pointer;
      transition: background-color 0.2s ease;
    }

    .btn-on {
      background-color: var(--primary);
      color: white;
    }

    .btn-on:hover {
      background-color: var(--primary-hover);
    }

    .btn-off {
      background-color: var(--danger);
      color: white;
    }

    .btn-off:hover {
      background-color: var(--danger-hover);
    }

    /* Controles especiais (PWM e Analog0) */
    .slider-container {
      width: 100%;
      margin: 1rem 0;
    }

    .slider {
      width: 100%;
      height: 8px;
      border-radius: 4px;
      background: #e2e8f0;
      outline: none;
      accent-color: var(--primary);
    }

    .sensor-value {
      font-size: 2.5rem;
      font-weight: 700;
      color: var(--primary);
      margin: 0.5rem 0;
    }

    .sensor-unit {
      font-size: 0.9rem;
      color: var(--text-muted);
    }
  </style>
</head>
<body>

  <header>
    <h1>Casa Inteligente IoT</h1>
    <p>Painel de Controle e Monitoramento em Tempo Real</p>
  </header>

  <div class="status-bar">
    <div id="statusDot" class="status-dot"></div>
    <span id="statusText">Desconectado do Broker MQTT</span>
  </div>

  <main class="dashboard-grid">

    <!-- Card 1: Sala (Pino 8) -->
    <div class="card">
      <div class="card-title">🛋️ Luz da Sala</div>
      <div id="ledSala" class="virtual-led"></div>
      <p style="font-size: 0.85rem; color: var(--text-muted); margin-bottom: 1rem;">Porta Digital 8</p>
      <div class="btn-group">
        <button class="btn-on" onclick="enviarComando('sala', 'ON')">Ligar</button>
        <button class="btn-off" onclick="enviarComando('sala', 'OFF')">Desligar</button>
      </div>
    </div>

    <!-- Card 2: Quarto (Pino 10) + Dimerização PWM -->
    <div class="card">
      <div class="card-title">🛏️ Luz do Quarto</div>
      <div id="ledQuarto" class="virtual-led"></div>
      <p style="font-size: 0.85rem; color: var(--text-muted);">Porta PWM 10 (Brilho)</p>
      
      <div class="slider-container">
        <input type="range" min="0" max="100" value="0" class="slider" id="pwmSlider" oninput="atualizarPWM(this.value)">
        <p style="margin-top: 0.5rem; font-weight: 600;">Intensidade: <span id="pwmPercent">0</span>%</p>
      </div>

      <div class="btn-group">
        <button class="btn-on" onclick="enviarComando('quarto', 'ON')">Ligar</button>
        <button class="btn-off" onclick="enviarComando('quarto', 'OFF')">Desligar</button>
      </div>
    </div>

    <!-- Card 3: Cozinha (Pino 13) -->
    <div class="card">
      <div class="card-title">🍳 Luz da Cozinha</div>
      <div id="ledCozinha" class="virtual-led"></div>
      <p style="font-size: 0.85rem; color: var(--text-muted); margin-bottom: 1rem;">Porta Digital 13</p>
      <div class="btn-group">
        <button class="btn-on" onclick="enviarComando('cozinha', 'ON')">Ligar</button>
        <button class="btn-off" onclick="enviarComando('cozinha', 'OFF')">Desligar</button>
      </div>
    </div>

    <!-- Card 4: Sensor Analógico (Pino A0) -->
    <div class="card">
      <div class="card-title">📊 Sensor Analógico</div>
      <div class="sensor-value" id="analogValue">0</div>
      <span class="sensor-unit">Leitura da Entrada A0 (0 a 1023)</span>
    </div>

  </main>

  <script>
    // 🚨 A REGRA DE OURO: Personalize seu prefixo para evitar linha cruzada com outras bancadas!
    const TOPICO_BASE = "minha_bancada_exclusiva/casa/"; 

    // Conexão WebSocket com o Broker Público da HiveMQ
    const brokerUrl = "wss://broker.hivemq.com:8000/mqtt";
    const clientId = "web_dashboard_" + Math.random().toString(16).substr(2, 8);

    const client = mqtt.connect(brokerUrl, { clientId: clientId });

    const statusDot = document.getElementById("statusDot");
    const statusText = document.getElementById("statusText");

    // Evento de Conexão com o Broker MQTT
    client.on("connect", () => {
      statusDot.classList.add("connected");
      statusText.innerText = "Conectado ao HiveMQ (WebSocket)";
      console.log("Conectado com sucesso!");

      // Inscreve-se nos tópicos de estado e telemetria do Arduino
      client.subscribe(TOPICO_BASE + "+/status");
      client.subscribe(TOPICO_BASE + "analogico");
    });

    client.on("offline", () => {
      statusDot.classList.remove("connected");
      statusText.innerText = "Desconectado do Broker";
    });

    // Envio de comandos ON/OFF
    function enviarComando(comodo, acao) {
      const topico = TOPICO_BASE + comodo + "/set";
      client.publish(topico, acao);
      console.log(`Enviado para ${topico}: ${acao}`);

      // Atualiza o estado do brilho visualmente se o botão "Ligar/Desligar" do quarto for clicado
      if (comodo === 'quarto') {
        const slider = document.getElementById('pwmSlider');
        const percent = document.getElementById('pwmPercent');
        if (acao === 'ON') {
          slider.value = 100;
          percent.innerText = "100";
        } else {
          slider.value = 0;
          percent.innerText = "0";
        }
      }
    }

    // Envio do nível de PWM (0 a 100%)
    function atualizarPWM(valor) {
      document.getElementById('pwmPercent').innerText = valor;
      const topico = TOPICO_BASE + "quarto/pwm";
      client.publish(topico, valor.toString());
    }

    // Recebimento de mensagens da nuvem
    client.on("message", (topic, message) => {
      const msgStr = message.toString();
      console.log(`Mensagem recebida [${topic}]: ${msgStr}`);

      // Atualização dos estados dos LEDs virtuais
      if (topic === TOPICO_BASE + "sala/status") {
        atualizarLEDVirtual("ledSala", msgStr === "ON");
      } else if (topic === TOPICO_BASE + "quarto/status") {
        atualizarLEDVirtual("ledQuarto", msgStr === "ON" || parseInt(msgStr) > 0);
      } else if (topic === TOPICO_BASE + "cozinha/status") {
        atualizarLEDVirtual("ledCozinha", msgStr === "ON");
      } else if (topic === TOPICO_BASE + "analogico") {
        document.getElementById("analogValue").innerText = msgStr;
      }
    });

    function atualizarLEDVirtual(elementId, status) {
      const led = document.getElementById(elementId);
      if (status) {
        led.classList.add("active");
      } else {
        led.classList.remove("active");
      }
    }
  </script>
</body>
</html># MEUSITE
