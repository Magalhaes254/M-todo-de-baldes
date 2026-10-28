<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador do Método dos Baldes - Por Rui Magalhães</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #1a73e8;
            --success: #34a853;
            --warning: #fbbc04;
            --danger: #ea4335;
            --bg: #f8f9fa;
            --card-bg: #ffffff;
            --text: #202124;
            --text-secondary: #5f6368;
            --border: #dadce0;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Inter', sans-serif; }
        body { background-color: var(--bg); color: var(--text); padding: 20px; line-height: 1.5; }
        .container { max-width: 900px; margin: 0 auto; }
        header { text-align: center; margin-bottom: 30px; }
        header h1 { font-size: 24px; color: var(--primary); margin-bottom: 5px; }
        header p { color: var(--text-secondary); font-size: 14px; }
        .author-tag { display: inline-block; background: #e8f0fe; color: var(--primary); padding: 4px 12px; border-radius: 12px; font-size: 12px; font-weight: 600; margin-top: 8px; }
        
        .card { background: var(--card-bg); border: 1px solid var(--border); border-radius: 12px; padding: 24px; margin-bottom: 20px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
        .card h2 { font-size: 18px; margin-bottom: 16px; display: flex; align-items: center; justify-content: space-between; }
        
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media (max-width: 768px) { .grid { grid-template-columns: 1fr; } }

        .input-group { margin-bottom: 16px; }
        .input-group label { display: block; font-size: 13px; font-weight: 500; margin-bottom: 6px; color: var(--text-secondary); }
        .input-group input, .input-group select { width: 100%; padding: 10px 12px; border: 1px solid var(--border); border-radius: 8px; font-size: 15px; background: #fff; }
        .input-group input:focus, .input-group select:focus { outline: none; border-color: var(--primary); box-shadow: 0 0 0 2px rgba(26,115,232,0.2); }

        .slider-group { margin-bottom: 14px; }
        .slider-header { display: flex; justify-content: space-between; font-size: 14px; margin-bottom: 4px; font-weight: 500; }
        input[type="range"] { width: 100%; accent-color: var(--primary); cursor: pointer; }

        .summary-stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-bottom: 20px; }
        .stat-box { background: #f1f3f4; padding: 14px; border-radius: 8px; text-align: center; }
        .stat-box span { display: block; font-size: 12px; color: var(--text-secondary); margin-bottom: 4px; }
        .stat-box strong { font-size: 18px; color: var(--text); }
        .stat-box.surplus strong { color: var(--success); }
        .stat-box.deficit strong { color: var(--danger); }

        .chart-container { position: relative; height: 260px; width: 100%; }

        .ai-box { background: #f0f4f8; border-left: 4px solid var(--primary); padding: 16px; border-radius: 0 8px 8px 0; margin-top: 15px; }
        .ai-box h3 { font-size: 15px; color: var(--primary); margin-bottom: 8px; display: flex; align-items: center; gap: 6px; }
        .ai-box p { font-size: 14px; color: var(--text); margin-bottom: 8px; }
        
        .chat-input-area { display: flex; gap: 8px; margin-top: 12px; }
        .chat-input-area input { flex: 1; padding: 10px; border: 1px solid var(--border); border-radius: 8px; font-size: 14px; }
        .btn { background: var(--primary); color: white; border: none; padding: 10px 16px; border-radius: 8px; font-weight: 500; cursor: pointer; transition: background 0.2s; }
        .btn:hover { background: #1557b0; }
        
        .api-key-bar { background: #fff3cd; border: 1px solid #ffeeba; padding: 10px 16px; border-radius: 8px; margin-bottom: 16px; font-size: 13px; display: flex; gap: 10px; align-items: center; }
        .api-key-bar input { flex: 1; padding: 6px 10px; border: 1px solid var(--border); border-radius: 4px; font-size: 13px; }

        footer { text-align: center; margin-top: 30px; font-size: 13px; color: var(--text-secondary); }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Método dos Baldes (Cascata)</h1>
        <p>Gestão financeira automatizada, simples e dinâmica</p>
        <div class="author-tag">Desenvolvido por Rui Magalhães</div>
    </header>

    <!-- Barra de Configuração da Chave API da IA -->
    <div class="api-key-bar">
        <span>🔑 Chave API Gemini:</span>
        <input type="password" id="apiKeyInput" placeholder="Insere a tua chave API aqui para ativar o chat de IA..." />
        <button class="btn" onclick="saveApiKey()" style="padding: 6px 12px; font-size: 12px;">Guardar</button>
    </div>

    <div class="grid">
        <!-- Coluna de Controlo e Sliders -->
        <div>
            <div class="card">
                <h2>1. Rendimento & Bases</h2>
                <div class="input-group">
                    <label>Rendimento Líquido Mensal (€)</label>
                    <input type="number" id="incomeInput" value="2200" min="0" step="50" oninput="calculateBudget()">
                </div>
                <div class="slider-group">
                    <div class="slider-header">
                        <span>Balde 1: Responsabilidades (Fixas)</span>
                        <span id="b1Val">1400 €</span>
                    </div>
                    <input type="range" id="b1Slider" min="0" max="3000" step="25" value="1400" oninput="updateSlider('b1')">
                </div>
                <div class="slider-group">
                    <div class="slider-header">
                        <span>Balde 2: Preparação Anual</span>
                        <span id="b2Val">100 €</span>
                    </div>
                    <input type="range" id="b2Slider" min="0" max="1000" step="10" value="100" oninput="updateSlider('b2')">
                </div>
            </div>

            <div class="card">
                <h2>2. Distribuição do Excedente</h2>
                <div class="slider-group">
                    <div class="slider-header">
                        <span>Balde 3: Fundo de Emergência</span>
                        <span id="b3Val">250 €</span>
                    </div>
                    <input type="range" id="b3Slider" min="0" max="1500" step="25" value="250" oninput="updateSlider('b3')">
                </div>
                <div class="slider-group">
                    <div class="slider-header">
                        <span>Balde 4: Investimentos / Futuro</span>
                        <span id="b4Val">150 €</span>
                    </div>
                    <input type="range" id="b4Slider" min="0" max="1500" step="25" value="150" oninput="updateSlider('b4')">
                </div>
                <div class="slider-group">
                    <div class="slider-header">
                        <span>Balde 5: Dinheiro Livre</span>
                        <span id="b5Val">300 €</span>
                    </div>
                    <input type="range" id="b5Slider" min="0" max="1500" step="25" value="300" oninput="updateSlider('b5')">
                </div>
            </div>
        </div>

        <!-- Coluna de Gráficos e Fundo de Emergência -->
        <div>
            <div class="card">
                <h2>Resumo do Orçamento</h2>
                <div class="summary-stats">
                    <div class="stat-box">
                        <span>Total Baldes (1-2)</span>
                        <strong id="totalFixed">1.500 €</strong>
                    </div>
                    <div class="stat-box">
                        <span>Excedente Disponível</span>
                        <strong id="totalSurplus">700 €</strong>
                    </div>
                    <div class="stat-box surplus" id="surplusBox">
                        <span>Taxa Poupança</span>
                        <strong id="savingsRate">18.2%</strong>
                    </div>
                </div>
                <div class="chart-container">
                    <canvas id="budgetChart"></canvas>
                </div>
            </div>

            <div class="card">
                <h2>Fundo de Emergência (Balde 3)</h2>
                <div class="input-group">
                    <label>Meta de Cobertura (Meses de Despesas do Balde 1)</label>
                    <select id="emergencyMonths" onchange="calculateEmergency()">
                        <option value="3">3 Meses de segurança</option>
                        <option value="6" selected>6 Meses de segurança (Recomendado)</option>
                        <option value="12">12 Meses de segurança máxima</option>
                    </select>
                </div>
                <p id="emergencyText" style="font-size: 14px; margin-top: 8px; color: var(--text-secondary);">
                    Para cobrir as despesas atuais, a meta do fundo é de <strong>8.400 €</strong>. Ao ritmo atual do Balde 3, demorará cerca de <strong>34 meses (2.8 anos)</strong> a atingir. Quanto mais rápido atingires este valor, mais fôlego e paz mental vais sentir!
                </p>
            </div>
        </div>
    </div>

    <!-- Módulo de Assistente IA & Diagnóstico -->
    <div class="card">
        <h2>Assistente IA de Estratégia Financeira</h2>
        <p style="font-size: 14px; color: var(--text-secondary); margin-bottom: 12px;">
            Obtém recomendações automáticas para renegociar contratos, baixar custos fixos e acelerar os teus objetivos.
        </p>
        <button class="btn" onclick="runAiDiagnosis()">Gerar Diagnóstico Automático</button>
        
        <div id="aiOutput" class="ai-box" style="display:none;">
            <h3>🤖 Diagnóstico do Consultor IA</h3>
            <p id="aiText">A analisar os teus baldes...</p>
        </div>

        <div class="chat-input-area">
            <input type="text" id="chatQuery" placeholder="Faz uma pergunta ao assistente (ex: como posso poupar mais no combustível?)..." onkeypress="if(event.key === 'Enter') sendAiChat()" />
            <button class="btn" onclick="sendAiChat()">Enviar</button>
        </div>
    </div>

    <footer>
        <p>&copy; 2026 Método dos Baldes &bull; Desenvolvido por Rui Magalhães</p>
    </footer>
</div>

<script>
    let chart;
    let apiKey = localStorage.getItem('gemini_api_key') || '';
    if (apiKey) { document.getElementById('apiKeyInput').value = apiKey; }

    function saveApiKey() {
        apiKey = document.getElementById('apiKeyInput').value.trim();
        localStorage.setItem('gemini_api_key', apiKey);
        alert('Chave API guardada com sucesso!');
    }

    function initChart() {
        const ctx = document.getElementById('budgetChart').getContext('2d');
        chart = new Chart(ctx, {
            type: 'doughnut',
            data: {
                labels: ['Balde 1: Fixas', 'Balde 2: Preparação', 'Balde 3: Emergência', 'Balde 4: Futuro', 'Balde 5: Livre'],
                datasets: [{
                    data: [1400, 100, 250, 150, 300],
                    backgroundColor: ['#ea4335', '#fbbc04', '#1a73e8', '#34a853', '#9333ea'],
                    borderWidth: 1
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { position: 'bottom', labels: { boxWidth: 12, font: { size: 12 } } } }
            }
        });
        calculateBudget();
    }

    function updateSlider(source) {
        const income = parseFloat(document.getElementById('incomeInput').value) || 0;
        let b1 = parseFloat(document.getElementById('b1Slider').value);
        let b2 = parseFloat(document.getElementById('b2Slider').value);
        let b3 = parseFloat(document.getElementById('b3Slider').value);
        let b4 = parseFloat(document.getElementById('b4Slider').value);
        let b5 = parseFloat(document.getElementById('b5Slider').value);

        // Ajustes automáticos de máximos e limites
        const fixed = b1 + b2;
        const surplus = income - fixed;

        if (surplus < 0) {
            alert('Atenção: Os Baldes 1 e 2 ultrapassam o rendimento líquido total!');
        }

        calculateBudget();
    }

    function calculateBudget() {
        const income = parseFloat(document.getElementById('incomeInput').value) || 0;
        
        // Atualiza max dos sliders de acordo com o rendimento
        document.getElementById('b1Slider').max = income;
        document.getElementById('b2Slider').max = income;

        let b1 = parseFloat(document.getElementById('b1Slider').value);
        let b2 = parseFloat(document.getElementById('b2Slider').value);
        let b3 = parseFloat(document.getElementById('b3Slider').value);
        let b4 = parseFloat(document.getElementById('b4Slider').value);
        let b5 = parseFloat(document.getElementById('b5Slider').value);

        document.getElementById('b1Val').innerText = b1 + ' €';
        document.getElementById('b2Val').innerText = b2 + ' €';
        document.getElementById('b3Val').innerText = b3 + ' €';
        document.getElementById('b4Val').innerText = b4 + ' €';
        document.getElementById('b5Val').innerText = b5 + ' €';

        const totalFixed = b1 + b2;
        const surplus = income - totalFixed;
        const totalAllocated = b3 + b4 + b5;
        const savingsRate = income > 0 ? ((b3 + b4) / income * 100).toFixed(1) : 0;

        document.getElementById('totalFixed').innerText = totalFixed + ' €';
        document.getElementById('totalSurplus').innerText = surplus + ' €';
        document.getElementById('savingsRate').innerText = savingsRate + ' %';

        // Atualizar Gráfico
        chart.data.datasets[0].data = [b1, b2, b3, b4, b5];
        chart.update();

        calculateEmergency();
    }

    function calculateEmergency() {
        const b1 = parseFloat(document.getElementById('b1Slider').value) || 0;
        const b3 = parseFloat(document.getElementById('b3Slider').value) || 1;
        const months = parseInt(document.getElementById('emergencyMonths').value) || 6;

        const target = b1 * months;
        const monthsNeeded = b3 > 0 ? Math.ceil(target / b3) : 0;
        const yearsNeeded = (monthsNeeded / 12).toFixed(1);

        document.getElementById('emergencyText').innerHTML = 
            `Para cobrir as tuas despesas fixas (${b1} €), a meta de ${months} meses é de <strong>${target.toLocaleString()} €</strong>. ` +
            `Ao ritmo de poupança atual no Balde 3 (${b3} €/mês), demorarás cerca de <strong>${monthsNeeded} meses (${yearsNeeded} anos)</strong> a atingir o objetivo. ` +
            `Quanto mais rápido completares este fundo, maior será a sensação de fôlego e estabilidade financeira!`;
    }

    async function runAiDiagnosis() {
        const outputDiv = document.getElementById('aiOutput');
        const textP = document.getElementById('aiText');
        outputDiv.style.display = 'block';

        if (!apiKey) {
            textP.innerHTML = '⚠️ Por favor, insere a tua Chave API Gemini no campo no topo da página para ativar o assistente.';
            return;
        }

        textP.innerHTML = 'A gerar diagnóstico personalizado com base nos teus baldes...';

        const b1 = document.getElementById('b1Slider').value;
        const income = document.getElementById('incomeInput').value;

        const prompt = `Analisa este orçamento em Portugal (desenvolvido por Rui Magalhães): Rendimento ${income}€, Despesas Fixas (Balde 1) ${b1}€. Dá conselhos práticos e diretos em Português de Portugal sobre taxa de esforço, renegociação de contratos de energia/telecomunicações, créditos e estratégias para baixar despesas fixas.`;

        try {
            const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
            });
            const data = await response.json();
            const reply = data.candidates?.[0]?.content?.parts?.[0]?.text || 'Não foi possível obter resposta.';
            textP.innerHTML = reply.replace(/\n/g, '<br>');
        } catch (err) {
            textP.innerHTML = 'Erro ao contactar a API. Verifica a tua chave API.';
        }
    }

    async function sendAiChat() {
        const query = document.getElementById('chatQuery').value.trim();
        if (!query) return;

        const outputDiv = document.getElementById('aiOutput');
        const textP = document.getElementById('aiText');
        outputDiv.style.display = 'block';

        if (!apiKey) {
            textP.innerHTML = '⚠️ Insere primeiro a tua Chave API Gemini no topo.';
            return;
        }

        textP.innerHTML = 'A pensar...';

        try {
            const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ contents: [{ parts: [{ text: query }] }] })
            });
            const data = await response.json();
            const reply = data.candidates?.[0]?.content?.parts?.[0]?.text || 'Sem resposta.';
            textP.innerHTML = `<strong>Pergunta:</strong> ${query}<br><br>${reply.replace(/\n/g, '<br>')}`;
            document.getElementById('chatQuery').value = '';
        } catch (err) {
            textP.innerHTML = 'Erro na ligação ao chat.';
        }
    }

    window.onload = initChart;
</script>

</body>
</html>
