<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Warming Blanket PID Monitor</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;700;900&family=Rajdhani:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --nextion-bg: #0a0e17;
            --nextion-bg-light: #0f1421;
            --nextion-border: #00d4ff;
            --nextion-text: #ffffff;
            --nextion-text-dim: #6b7280;
            --nextion-accent: #00ff88;
            --nextion-warm: #ff6b35;
            --nextion-patient: #a855f7;
        }
        
        * { margin: 0; padding: 0; box-sizing: border-box; }
        
        body {
            font-family: 'Rajdhani', sans-serif;
            background: linear-gradient(135deg, #050810 0%, #0a1628 50%, #0f1b2e 100%);
            color: var(--nextion-text);
            min-height: 100vh;
            background-attachment: fixed;
            position: relative;
            overflow-x: hidden;
        }
        
        body::before {
            content: '';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-image: 
                radial-gradient(circle at 20% 30%, rgba(0, 212, 255, 0.08) 0%, transparent 40%),
                radial-gradient(circle at 80% 70%, rgba(0, 255, 136, 0.05) 0%, transparent 40%);
            pointer-events: none;
            z-index: 0;
        }
        
        .container { 
            max-width: 1600px; 
            margin: 0 auto; 
            padding: 2rem; 
            position: relative;
            z-index: 1;
        }
        
        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 2rem 0 3rem;
            border-bottom: 2px solid var(--nextion-border);
            margin-bottom: 3rem;
            animation: slideDown 0.6s ease-out;
            position: relative;
        }
        
        header::after {
            content: '';
            position: absolute;
            bottom: -2px;
            left: 0;
            width: 100%;
            height: 2px;
            background: linear-gradient(90deg, transparent, var(--nextion-border), transparent);
            animation: glow 3s ease-in-out infinite;
        }
        
        @keyframes glow {
            0%, 100% { opacity: 0.5; }
            50% { opacity: 1; }
        }
        
        @keyframes slideDown {
            from { opacity: 0; transform: translateY(-20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .logo { display: flex; align-items: center; gap: 1rem; }
        
        .logo-icon {
            width: 56px; 
            height: 56px;
            background: linear-gradient(135deg, var(--nextion-warm), var(--nextion-border));
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: 700;
            font-size: 1.8rem;
            box-shadow: 
                0 0 30px rgba(0, 212, 255, 0.5),
                0 8px 24px rgba(255, 107, 53, 0.3);
            position: relative;
            overflow: hidden;
        }
        
        .logo-icon::before {
            content: '';
            position: absolute;
            width: 200%;
            height: 200%;
            background: linear-gradient(45deg, transparent, rgba(255,255,255,0.1), transparent);
            animation: shine 3s infinite;
        }
        
        @keyframes shine {
            0% { transform: translateX(-100%) translateY(-100%) rotate(45deg); }
            100% { transform: translateX(100%) translateY(100%) rotate(45deg); }
        }
        
        h1 {
            font-family: 'Orbitron', sans-serif;
            font-size: 2rem;
            font-weight: 900;
            background: linear-gradient(135deg, var(--nextion-border) 0%, var(--nextion-accent) 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
            letter-spacing: 3px;
            text-shadow: 0 0 30px rgba(0, 212, 255, 0.5);
        }
        
        .subtitle {
            color: var(--nextion-text-dim);
            margin-top: 0.5rem;
            font-size: 0.95rem;
            letter-spacing: 1px;
        }
        
        .status-indicator {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            padding: 0.85rem 1.75rem;
            background: var(--nextion-bg-light);
            border: 2px solid var(--nextion-border);
            border-radius: 50px;
            font-family: 'Orbitron', sans-serif;
            font-size: 0.9rem;
            box-shadow: 
                0 0 20px rgba(0, 212, 255, 0.2),
                inset 0 0 10px rgba(0, 212, 255, 0.05);
        }
        
        .status-dot {
            width: 12px; 
            height: 12px;
            border-radius: 50%;
            background: #4b5563;
            transition: all 0.3s;
            box-shadow: 0 0 0 3px rgba(75, 85, 99, 0.3);
        }
        
        .status-dot.online {
            background: var(--nextion-accent);
            box-shadow: 
                0 0 12px var(--nextion-accent),
                0 0 0 3px rgba(0, 255, 136, 0.3);
            animation: pulse 2s ease-in-out infinite;
        }
        
        @keyframes pulse {
            0%, 100% { 
                opacity: 1; 
                transform: scale(1);
            }
            50% { 
                opacity: 0.7; 
                transform: scale(1.1);
            }
        }
        
        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 2rem;
            margin-bottom: 3rem;
        }
        
        .card {
            background: var(--nextion-bg);
            border: 2px solid var(--nextion-border);
            border-radius: 16px;
            padding: 2rem;
            position: relative;
            overflow: hidden;
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            animation: fadeInUp 0.6s ease-out backwards;
            box-shadow: 
                0 0 20px rgba(0, 212, 255, 0.1),
                inset 0 0 20px rgba(0, 212, 255, 0.02);
        }
        
        .card::before {
            content: '';
            position: absolute;
            top: 0; left: 0; right: 0;
            height: 3px;
            background: linear-gradient(90deg, 
                var(--nextion-border), 
                var(--nextion-accent), 
                var(--nextion-border));
            opacity: 0;
            transition: opacity 0.4s;
        }
        
        .card::after {
            content: '';
            position: absolute;
            inset: 0;
            background: radial-gradient(circle at var(--mouse-x, 50%) var(--mouse-y, 50%), rgba(0, 212, 255, 0.08), transparent 50%);
            opacity: 0;
            transition: opacity 0.3s;
        }
        
        .card:hover {
            border-color: var(--nextion-accent);
            transform: translateY(-6px);
            box-shadow: 
                0 0 40px rgba(0, 212, 255, 0.3),
                0 12px 40px rgba(0, 255, 136, 0.2),
                inset 0 0 30px rgba(0, 212, 255, 0.05);
        }
        
        .card:hover::before { opacity: 1; }
        .card:hover::after { opacity: 1; }
        
        @keyframes fadeInUp {
            from { opacity: 0; transform: translateY(30px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .card:nth-child(1) { animation-delay: 0.1s; }
        .card:nth-child(2) { animation-delay: 0.2s; }
        .card:nth-child(3) { animation-delay: 0.3s; }
        
        .card-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 1.5rem;
        }
        
        .card-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 0.85rem;
            text-transform: uppercase;
            letter-spacing: 2px;
            color: var(--nextion-border);
            font-weight: 600;
        }
        
        .temperature-display {
            font-family: 'Orbitron', sans-serif;
            font-size: 4rem;
            font-weight: 900;
            line-height: 1;
            margin-bottom: 0.5rem;
            position: relative;
        }
        
        .temp-real { 
            color: var(--nextion-warm);
            text-shadow: 0 0 30px rgba(255, 107, 53, 0.6);
        }
        
        .temp-patient { 
            color: var(--nextion-patient);
            text-shadow: 0 0 30px rgba(168, 85, 247, 0.6);
        }
        
        .temp-setpoint { 
            color: var(--nextion-border);
            text-shadow: 0 0 30px rgba(0, 212, 255, 0.6);
        }
        
        .unit { 
            font-size: 1.8rem; 
            color: var(--nextion-text-dim);
            font-weight: 600;
        }
        
        .sub-info {
            color: var(--nextion-text-dim);
            font-size: 0.9rem;
            margin-top: 0.5rem;
            font-family: 'Rajdhani', sans-serif;
            letter-spacing: 0.5px;
        }
        
        .chart-container {
            grid-column: 1 / -1;
            background: var(--nextion-bg);
            border: 2px solid var(--nextion-border);
            border-radius: 16px;
            padding: 2rem;
            animation: fadeInUp 0.6s ease-out 0.4s backwards;
            box-shadow: 
                0 0 30px rgba(0, 212, 255, 0.15),
                inset 0 0 20px rgba(0, 212, 255, 0.03);
            position: relative;
            overflow: hidden;
        }
        
        .chart-container::before {
            content: '';
            position: absolute;
            top: 0; left: 0; right: 0;
            height: 3px;
            background: linear-gradient(90deg, 
                transparent,
                var(--nextion-border), 
                var(--nextion-accent),
                var(--nextion-border),
                transparent);
        }
        
        .chart-wrapper {
            position: relative;
            height: 400px;
            margin-top: 1.5rem;
        }
        
        .legend {
            display: flex;
            gap: 2rem;
            flex-wrap: wrap;
        }
        
        .legend-item {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            padding: 0.5rem 1rem;
            background: rgba(0, 212, 255, 0.05);
            border-radius: 8px;
            transition: all 0.3s;
        }
        
        .legend-item:hover {
            background: rgba(0, 212, 255, 0.1);
            transform: translateY(-2px);
        }
        
        .legend-color {
            width: 14px;
            height: 14px;
            border-radius: 3px;
            box-shadow: 0 0 8px currentColor;
        }
        
        @media (max-width: 768px) {
            .container { padding: 1rem; }
            header { flex-direction: column; gap: 1rem; align-items: flex-start; }
            .grid { grid-template-columns: 1fr; }
            .temperature-display { font-size: 3rem; }
            .chart-wrapper { height: 300px; }
            h1 { font-size: 1.5rem; }
            .legend { justify-content: center; gap: 1rem; }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <div class="logo">
                <div class="logo-icon">🔥</div>
                <div>
                    <h1>Warming Blanket PID</h1>
                    <p class="subtitle">Real-Time Monitoring System • PT100 MAX31865</p>
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
                    <span class="card-title">Suhu Alat</span>
                    <span style="font-size: 1.8rem;">🌡️</span>
                </div>
                <div class="temperature-display temp-real">
                    <span id="currentTemp">--</span><span class="unit">°C</span>
                </div>
                <div class="sub-info">Sensor PT100 MAX31865</div>
            </div>

            <div class="card">
                <div class="card-header">
                    <span class="card-title">Suhu Pasien</span>
                    <span style="font-size: 1.8rem;">👤</span>
                </div>
                <div class="temperature-display temp-patient">
                    <span id="patientTemp">--</span><span class="unit">°C</span>
                </div>
                <div class="sub-info">Sensor PT100 MAX31865</div>
            </div>

            <div class="card">
                <div class="card-header">
                    <span class="card-title">Setpoint</span>
                    <span style="font-size: 1.8rem;">🎯</span>
                </div>
                <div class="temperature-display temp-setpoint">
                    <span id="setpointDisplay">--</span><span class="unit">°C</span>
                </div>
                <div class="sub-info">Target suhu sistem</div>
            </div>
        </div>

        <div class="chart-container">
            <div class="card-header">
                <span class="card-title">Plotting Graph - Temperature Monitoring</span>
                <div class="legend">
                    <div class="legend-item">
                        <div class="legend-color" style="background: #ff6b35;"></div>
                        <span style="font-size: 0.9rem;">Suhu Alat</span>
                    </div>
                    <div class="legend-item">
                        <div class="legend-color" style="background: #a855f7;"></div>
                        <span style="font-size: 0.9rem;">Suhu Pasien</span>
                    </div>
                    <div class="legend-item">
                        <div class="legend-color" style="background: #00d4ff;"></div>
                        <span style="font-size: 0.9rem;">Setpoint</span>
                    </div>
                </div>
            </div>
            <div class="chart-wrapper">
                <canvas id="tempChart"></canvas>
            </div>
        </div>
    </div>

    <script>
        let ws;
        let chart;
        const maxDataPoints = 50;

        // Mouse tracking for card hover effect
        document.querySelectorAll('.card').forEach(card => {
            card.addEventListener('mousemove', (e) => {
                const rect = card.getBoundingClientRect();
                const x = ((e.clientX - rect.left) / rect.width) * 100;
                const y = ((e.clientY - rect.top) / rect.height) * 100;
                card.style.setProperty('--mouse-x', x + '%');
                card.style.setProperty('--mouse-y', y + '%');
            });
        });

        // Initialize Chart.js
        const ctx = document.getElementById('tempChart').getContext('2d');
        chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [{
                    label: 'Suhu Alat',
                    data: [],
                    borderColor: '#ff6b35',
                    backgroundColor: 'rgba(255, 107, 53, 0.15)',
                    borderWidth: 3,
                    tension: 0.4,
                    fill: true,
                    pointRadius: 0,
                    pointHoverRadius: 6,
                    pointHoverBackgroundColor: '#ff6b35',
                    pointHoverBorderColor: '#fff',
                    pointHoverBorderWidth: 2,
                }, {
                    label: 'Suhu Pasien',
                    data: [],
                    borderColor: '#a855f7',
                    backgroundColor: 'rgba(168, 85, 247, 0.15)',
                    borderWidth: 3,
                    tension: 0.4,
                    fill: true,
                    pointRadius: 0,
                    pointHoverRadius: 6,
                    pointHoverBackgroundColor: '#a855f7',
                    pointHoverBorderColor: '#fff',
                    pointHoverBorderWidth: 2,
                }, {
                    label: 'Setpoint',
                    data: [],
                    borderColor: '#00d4ff',
                    backgroundColor: 'rgba(0, 212, 255, 0.1)',
                    borderWidth: 2,
                    borderDash: [8, 4],
                    tension: 0.4,
                    fill: false,
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
                        backgroundColor: 'rgba(10, 14, 23, 0.95)',
                        titleColor: '#00d4ff',
                        titleFont: { size: 13, weight: 'bold', family: 'Orbitron' },
                        bodyColor: '#ffffff',
                        bodyFont: { size: 12, family: 'Rajdhani' },
                        borderColor: '#00d4ff',
                        borderWidth: 2,
                        padding: 14,
                        displayColors: true,
                        boxPadding: 6,
                        usePointStyle: true,
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
                        grid: { 
                            color: 'rgba(0, 212, 255, 0.1)', 
                            drawBorder: false,
                            lineWidth: 1
                        },
                        border: { display: false },
                        ticks: {
                            color: '#00d4ff',
                            font: { size: 11, family: 'Orbitron' },
                            callback: (value) => value + '°C'
                        }
                    },
                    x: {
                        grid: { 
                            color: 'rgba(0, 212, 255, 0.08)', 
                            drawBorder: false 
                        },
                        border: { display: false },
                        ticks: { 
                            color: '#6b7280', 
                            font: { size: 10, family: 'Rajdhani' },
                            maxTicksLimit: 10 
                        }
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
            
            // Update chart
            const now = new Date();
            const timeLabel = now.toLocaleTimeString('id-ID');
            
            chart.data.labels.push(timeLabel);
            chart.data.datasets[0].data.push(data.currentTemp);
            chart.data.datasets[1].data.push(data.patientTemp);
            chart.data.datasets[2].data.push(data.setpoint);
            
            // Limit data points
            if (chart.data.labels.length > maxDataPoints) {
                chart.data.labels.shift();
                chart.data.datasets[0].data.shift();
                chart.data.datasets[1].data.shift();
                chart.data.datasets[2].data.shift();
            }
            
            chart.update('none');
        }

        // Simulate data for demo
        function simulateData() {
            const data = {
                currentTemp: 35 + Math.random() * 8,
                patientTemp: 36 + Math.random() * 2,
                setpoint: 40
            };
            updateDisplay(data);
        }

        // Auto-connect on page load
        window.addEventListener('load', () => {
            connectWebSocket();
            // Demo simulation
            setInterval(simulateData, 1000);
        });

        document.addEventListener('visibilitychange', () => {
            if (!document.hidden && (!ws || ws.readyState !== WebSocket.OPEN)) {
                connectWebSocket();
            }
        });
    </script>
</body>
</html>
