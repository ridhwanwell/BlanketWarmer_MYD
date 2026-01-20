<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blanket Warmer Monitor</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600;700&family=DM+Sans:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-primary: #0a0e17;
            --bg-secondary: #141824;
            --bg-tertiary: #1a1f2e;
            --accent-warm: #ff6b35;
            --accent-cool: #00d4ff;
            --accent-patient: #a855f7;
            --accent-success: #00ff88;
            --text-primary: #e8eaed;
            --text-secondary: #9ba3af;
            --border-color: #2a3142;
        }
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'DM Sans', sans-serif;
            background: var(--bg-primary);
            color: var(--text-primary);
            min-height: 100vh;
            background-image: 
                radial-gradient(circle at 20% 30%, rgba(255, 107, 53, 0.08) 0%, transparent 50%),
                radial-gradient(circle at 80% 70%, rgba(0, 212, 255, 0.06) 0%, transparent 50%);
        }
        .container { max-width: 1600px; margin: 0 auto; padding: 2rem; }
        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 2rem 0 3rem;
            border-bottom: 1px solid var(--border-color);
            margin-bottom: 3rem;
            animation: slideDown 0.6s ease-out;
        }
        @keyframes slideDown {
            from { opacity: 0; transform: translateY(-20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .logo { display: flex; align-items: center; gap: 1rem; }
        .logo-icon {
            width: 48px; height: 48px;
            background: linear-gradient(135deg, var(--accent-warm), var(--accent-cool));
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 700;
            font-size: 1.5rem;
            box-shadow: 0 8px 24px rgba(255, 107, 53, 0.3);
        }
        h1 {
            font-size: 1.8rem;
            font-weight: 700;
            background: linear-gradient(135deg, var(--accent-warm), var(--accent-cool));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        .subtitle {
            color: var(--text-secondary);
            margin-top: 0.5rem;
            font-size: 0.9rem;
        }
        .status-indicator {
            display: flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.75rem 1.5rem;
            background: var(--bg-secondary);
            border: 1px solid var(--border-color);
            border-radius: 50px;
            font-family: 'JetBrains Mono', monospace;
            font-size: 0.9rem;
        }
        .status-dot {
            width: 10px; height: 10px;
            border-radius: 50%;
            background: #666;
            transition: all 0.3s;
        }
        .status-dot.online {
            background: var(--accent-success);
            box-shadow: 0 0 12px var(--accent-success);
            animation: pulse 2s ease-in-out infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 2rem;
            margin-bottom: 3rem;
        }
        .card {
            background: var(--bg-secondary);
            border: 1px solid var(--border-color);
            border-radius: 16px;
            padding: 2rem;
            position: relative;
            overflow: hidden;
            transition: all 0.3s;
            animation: fadeInUp 0.6s ease-out backwards;
        }
        .card::before {
            content: '';
            position: absolute;
            top: 0; left: 0; right: 0;
            height: 3px;
            background: linear-gradient(90deg, var(--accent-warm), var(--accent-cool));
            opacity: 0;
            transition: opacity 0.3s;
        }
        .card:hover {
            border-color: var(--accent-warm);
            transform: translateY(-4px);
            box-shadow: 0 12px 40px rgba(255, 107, 53, 0.2);
        }
        .card:hover::before { opacity: 1; }
        @keyframes fadeInUp {
            from { opacity: 0; transform: translateY(30px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .card:nth-child(1) { animation-delay: 0.1s; }
        .card:nth-child(2) { animation-delay: 0.2s; }
        .card:nth-child(3) { animation-delay: 0.3s; }
        .card:nth-child(4) { animation-delay: 0.4s; }
        .card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 1.5rem;
        }
        .card-title {
            font-size: 0.9rem;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: var(--text-secondary);
            font-weight: 600;
        }
        .temperature-display {
            font-size: 3.5rem;
            font-weight: 700;
            font-family: 'JetBrains Mono', monospace;
            line-height: 1;
            margin-bottom: 0.5rem;
        }
        .temp-real { color: var(--accent-warm); }
        .temp-patient { color: var(--accent-patient); }
        .temp-setpoint { color: var(--accent-cool); }
        .unit { font-size: 1.5rem; color: var(--text-secondary); }
        .sub-info {
            color: var(--text-secondary);
            font-size: 0.9rem;
            margin-top: 0.5rem;
            font-family: 'JetBrains Mono', monospace;
        }
        .chart-container {
            grid-column: 1 / -1;
            background: var(--bg-secondary);
            border: 1px solid var(--border-color);
            border-radius: 16px;
            padding: 2rem;
            animation: fadeInUp 0.6s ease-out 0.5s backwards;
        }
        .chart-wrapper {
            position: relative;
            height: 400px;
            margin-top: 1.5rem;
        }
        .legend {
            display: flex;
            gap: 1.5rem;
            flex-wrap: wrap;
        }
        .legend-item {
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        .legend-color {
            width: 12px;
            height: 12px;
            border-radius: 2px;
        }
        .control-panel {
            grid-column: 1 / -1;
            background: var(--bg-tertiary);
            border: 1px solid var(--border-color);
            border-radius: 16px;
            padding: 2rem;
            animation: fadeInUp 0.6s ease-out 0.6s backwards;
        }
        .control-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 2rem;
            margin-top: 1.5rem;
        }
        .control-group {
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
        }
        label {
            font-size: 0.85rem;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            color: var(--text-secondary);
            font-weight: 600;
        }
        input[type="number"] {
            width: 100%;
            padding: 1rem;
            background: var(--bg-secondary);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            color: var(--text-primary);
            font-family: 'JetBrains Mono', monospace;
            font-size: 1.1rem;
            transition: all 0.3s;
        }
        input:focus {
            outline: none;
            border-color: var(--accent-warm);
            box-shadow: 0 0 0 3px rgba(255, 107, 53, 0.1);
        }
        button {
            padding: 1rem 2rem;
            background: linear-gradient(135deg, var(--accent-warm), #ff8555);
            border: none;
            border-radius: 8px;
            color: white;
            font-weight: 600;
            font-size: 1rem;
            cursor: pointer;
            transition: all 0.3s;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            box-shadow: 0 4px 16px rgba(255, 107, 53, 0.3);
        }
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 24px rgba(255, 107, 53, 0.4);
        }
        button:active { transform: translateY(0); }
        button.stop {
            background: linear-gradient(135deg, #ef4444, #dc2626);
        }
        .button-group {
            display: flex;
            gap: 1rem;
            margin-top: 2rem;
        }
        .timer-display {
            font-size: 2.5rem;
            font-weight: 700;
            font-family: 'JetBrains Mono', monospace;
            color: var(--accent-success);
        }
        @media (max-width: 768px) {
            .container { padding: 1rem; }
            .header { flex-direction: column; gap: 1rem; align-items: flex-start; }
            .grid { grid-template-columns: 1fr; }
            .temperature-display { font-size: 2.5rem; }
            .chart-wrapper { height: 300px; }
            .button-group { flex-direction: column; }
            .legend { justify-content: center; }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <div class="logo">
                <div class="logo-icon">🔥</div>
                <div>
                    <h1>Blanket Warmer Control</h1>
                    <p class="subtitle">Sistem Monitoring Real-Time PT100 MAX31865</p>
                </div>
            </div>
            <div class="status-indicator">
                <div class="status-dot" id="statusDot"></div>
                <span id="statusText">CONNECTING</span>
            </div>
        </header>

        <div class="grid">
            <div class="card">
                <div class="card-header">
                    <span class="card-title">Suhu PT100 (Real)</span>
                    <span style="font-size: 1.5rem;">🌡️</span>
                </div>
                <div class="temperature-display temp-real">
                    <span id="currentTemp">--</span><span class="unit">°C</span>
                </div>
                <div class="sub-info">Pembacaan sensor MAX31865</div>
            </div>

            <div class="card">
                <div class="card-header">
                    <span class="card-title">Suhu Setpoint</span>
                    <span style="font-size: 1.5rem;">🎯</span>
                </div>
                <div class="temperature-display temp-setpoint">
                    <span id="setpointDisplay">--</span><span class="unit">°C</span>
                </div>
                <div class="sub-info">Target suhu yang diinginkan</div>
            </div>

            <div class="card">
                <div class="card-header">
                    <span class="card-title">Suhu Pasien</span>
                    <span style="font-size: 1.5rem;">👤</span>
                </div>
                <div class="temperature-display temp-patient">
                    <span id="patientTemp">--</span><span class="unit">°C</span>
                </div>
                <div class="sub-info">Suhu pasien (simulasi)</div>
            </div>

            <div class="card">
                <div class="card-header">
                    <span class="card-title">Timer</span>
                    <span style="font-size: 1.5rem;">⏱️</span>
                </div>
                <div class="timer-display" id="timerDisplay">00:00:00</div>
                <div class="sub-info">Waktu tersisa</div>
            </div>
        </div>

        <div class="chart-container">
            <div class="card-header">
                <span class="card-title">Grafik Suhu Real-Time</span>
                <div class="legend">
                    <div class="legend-item">
                        <div class="legend-color" style="background: #ff6b35;"></div>
                        <span style="font-size: 0.85rem; color: var(--text-secondary);">Suhu Real (PT100)</span>
                    </div>
                    <div class="legend-item">
                        <div class="legend-color" style="background: #00d4ff;"></div>
                        <span style="font-size: 0.85rem; color: var(--text-secondary);">Setpoint</span>
                    </div>
                    <div class="legend-item">
                        <div class="legend-color" style="background: #a855f7;"></div>
                        <span style="font-size: 0.85rem; color: var(--text-secondary);">Suhu Pasien</span>
                    </div>
                </div>
            </div>
            <div class="chart-wrapper">
                <canvas id="tempChart"></canvas>
            </div>
        </div>

        <div class="control-panel">
            <div class="card-header">
                <span class="card-title">Panel Kontrol</span>
                <span style="font-size: 1.5rem;">🎛️</span>
            </div>
            <div class="control-grid">
                <div class="control-group">
                    <label for="setpoint">Target Suhu (°C)</label>
                    <input type="number" id="setpoint" value="40" min="25" max="55" step="0.5">
                </div>
                <div class="control-group">
                    <label for="duration">Durasi (Menit)</label>
                    <input type="number" id="duration" value="60" min="1" max="180" step="1">
                </div>
            </div>
            <div class="button-group">
                <button onclick="startSystem()" id="btnStart">▶️ Start System</button>
                <button onclick="stopSystem()" class="stop" id="btnStop">⏹️ Stop System</button>
            </div>
        </div>
    </div>

    <script>
        let ws;
        let chart;
        const maxDataPoints = 50;

        // Initialize Chart.js
        const ctx = document.getElementById('tempChart').getContext('2d');
        chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [{
                    label: 'Suhu Real (PT100)',
                    data: [],
                    borderColor: '#ff6b35',
                    backgroundColor: 'rgba(255, 107, 53, 0.1)',
                    borderWidth: 3,
                    tension: 0.4,
                    fill: true,
                    pointRadius: 0,
                    pointHoverRadius: 6,
                }, {
                    label: 'Setpoint',
                    data: [],
                    borderColor: '#00d4ff',
                    borderWidth: 2,
                    borderDash: [5, 5],
                    tension: 0.4,
                    fill: false,
                    pointRadius: 0,
                }, {
                    label: 'Suhu Pasien',
                    data: [],
                    borderColor: '#a855f7',
                    backgroundColor: 'rgba(168, 85, 247, 0.1)',
                    borderWidth: 2,
                    tension: 0.4,
                    fill: true,
                    pointRadius: 0,
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                interaction: { intersect: false, mode: 'index' },
                plugins: {
                    legend: { display: false },
                    tooltip: {
                        backgroundColor: '#1a1f2e',
                        titleColor: '#e8eaed',
                        bodyColor: '#9ba3af',
                        borderColor: '#2a3142',
                        borderWidth: 1,
                        padding: 12,
                        callbacks: {
                            label: function(context) {
                                return context.dataset.label + ': ' + context.parsed.y.toFixed(1) + '°C';
                            }
                        }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: false,
                        min: 20,
                        max: 60,
                        grid: { color: '#2a3142', drawBorder: false },
                        ticks: {
                            color: '#9ba3af',
                            callback: (value) => value + '°C'
                        }
                    },
                    x: {
                        grid: { color: '#2a3142', drawBorder: false },
                        ticks: { color: '#9ba3af', maxTicksLimit: 10 }
                    }
                }
            }
        });

        // WebSocket Connection
        function connectWebSocket() {
            ws = new WebSocket('ws://' + window.location.hostname + '/ws');
            
            ws.onopen = () => {
                console.log('WebSocket Connected');
                document.getElementById('statusDot').classList.add('online');
                document.getElementById('statusText').textContent = 'ONLINE';
            };
            
            ws.onclose = () => {
                console.log('WebSocket Disconnected');
                document.getElementById('statusDot').classList.remove('online');
                document.getElementById('statusText').textContent = 'OFFLINE';
                setTimeout(connectWebSocket, 3000);
            };
            
            ws.onerror = (error) => {
                console.error('WebSocket Error:', error);
            };
            
            ws.onmessage = (event) => {
                try {
                    const data = JSON.parse(event.data);
                    updateDisplay(data);
                } catch (error) {
                    console.error('Parse error:', error);
                }
            };
        }

        function updateDisplay(data) {
            // Update temperature displays
            document.getElementById('currentTemp').textContent = data.currentTemp.toFixed(1);
            document.getElementById('setpointDisplay').textContent = data.setpoint.toFixed(1);
            document.getElementById('patientTemp').textContent = data.patientTemp.toFixed(1);
            
            // Update timer display
            if (data.timeRemaining > 0) {
                const hours = Math.floor(data.timeRemaining / 3600);
                const minutes = Math.floor((data.timeRemaining % 3600) / 60);
                const seconds = data.timeRemaining % 60;
                document.getElementById('timerDisplay').textContent = 
                    String(hours).padStart(2, '0') + ':' +
                    String(minutes).padStart(2, '0') + ':' +
                    String(seconds).padStart(2, '0');
            } else {
                document.getElementById('timerDisplay').textContent = '00:00:00';
            }
            
            // Update chart
            const now = new Date();
            const timeLabel = now.toLocaleTimeString('id-ID');
            
            chart.data.labels.push(timeLabel);
            chart.data.datasets[0].data.push(data.currentTemp);
            chart.data.datasets[1].data.push(data.setpoint);
            chart.data.datasets[2].data.push(data.patientTemp);
            
            // Limit data points
            if (chart.data.labels.length > maxDataPoints) {
                chart.data.labels.shift();
                chart.data.datasets[0].data.shift();
                chart.data.datasets[1].data.shift();
                chart.data.datasets[2].data.shift();
            }
            
            chart.update('none');
        }

        function startSystem() {
            const setpoint = parseFloat(document.getElementById('setpoint').value);
            const duration = parseInt(document.getElementById('duration').value) * 60; // Convert to seconds
            
            if (isNaN(setpoint) || setpoint < 25 || setpoint > 55) {
                alert('Target suhu harus antara 25-55°C');
                return;
            }
            
            if (isNaN(duration) || duration <= 0) {
                alert('Durasi harus lebih dari 0 menit');
                return;
            }
            
            const command = {
                command: 'start',
                setpoint: setpoint,
                duration: duration
            };
            
            if (ws && ws.readyState === WebSocket.OPEN) {
                ws.send(JSON.stringify(command));
                console.log('Start command sent:', command);
            } else {
                alert('WebSocket tidak terhubung!');
            }
        }

        function stopSystem() {
            if (ws && ws.readyState === WebSocket.OPEN) {
                ws.send(JSON.stringify({ command: 'stop' }));
                console.log('Stop command sent');
            } else {
                alert('WebSocket tidak terhubung!');
            }
        }

        // Auto-connect on page load
        window.addEventListener('load', () => {
            connectWebSocket();
        });

        // Reconnect when page becomes visible
        document.addEventListener('visibilitychange', () => {
            if (!document.hidden && (!ws || ws.readyState !== WebSocket.OPEN)) {
                connectWebSocket();
            }
        });
    </script>
</body>
</html>
