<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Warming Blanket PID Monitor</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
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
        }
        
        .container { 
            max-width: 1600px; 
            margin: 0 auto; 
            padding: 2rem; 
        }
        
        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 2rem 0 3rem;
            border-bottom: 2px solid var(--nextion-border);
            margin-bottom: 3rem;
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
            font-size: 1.8rem;
            box-shadow: 0 0 30px rgba(0, 212, 255, 0.5);
        }
        
        h1 {
            font-family: 'Orbitron', sans-serif;
            font-size: 2rem;
            font-weight: 900;
            background: linear-gradient(135deg, var(--nextion-border), var(--nextion-accent));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
            letter-spacing: 3px;
        }
        
        .subtitle {
            color: var(--nextion-text-dim);
            margin-top: 0.5rem;
            font-size: 0.95rem;
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
        }
        
        .status-dot {
            width: 12px; 
            height: 12px;
            border-radius: 50%;
            background: #4b5563;
        }
        
        .status-dot.online {
            background: var(--nextion-accent);
            box-shadow: 0 0 12px var(--nextion-accent);
            animation: pulse 2s ease-in-out infinite;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
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
            box-shadow: 0 0 20px rgba(0, 212, 255, 0.1);
            transition: all 0.3s;
        }
        
        .card:hover {
            border-color: var(--nextion-accent);
            transform: translateY(-6px);
            box-shadow: 0 0 40px rgba(0, 212, 255, 0.3);
        }
        
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
        }
        
        .chart-container {
            grid-column: 1 / -1;
            background: var(--nextion-bg);
            border: 2px solid var(--nextion-border);
            border-radius: 16px;
            padding: 2rem;
            box-shadow: 0 0 30px rgba(0, 212, 255, 0.15);
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
                    <p class="subtitle">Real-Time MQTT Monitoring System</p>
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
                <div class="sub-info">Sensor DS18B20</div>
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
                        <span style="font-size: 0.9rem;">Suhu Alat (PT100)</span>
                    </div>
                    <div class="legend-item">
                        <div class="legend-color" style="background: #a855f7;"></div>
                        <span style="font-size: 0.9rem;">Suhu Pasien (DS18B20)</span>
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
        let mqttClient;
        let chart;
        const maxDataPoints = 50;

        // ============= MQTT SETTINGS =============
        const mqtt_broker = 'broker.hivemq.com';
        const mqtt_port = 8000;  // WebSocket port
        const mqtt_topic_data = 'warmingblanket/data';
        const mqtt_topic_control = 'warmingblanket/control';

        // Initialize Chart.js
        const ctx = document.getElementById('tempChart').getContext('2d');
        chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [{
                    label: 'Suhu Alat (PT100)',
                    data: [],
                    borderColor: '#ff6b35',
                    backgroundColor: 'rgba(255, 107, 53, 0.15)',
                    borderWidth: 3,
                    tension: 0.4,
                    fill: true,
                    pointRadius: 0,
                    pointHoverRadius: 6
                }, {
                    label: 'Suhu Pasien (DS18B20)',
                    data: [],
                    borderColor: '#a855f7',
                    backgroundColor: 'rgba(168, 85, 247, 0.15)',
                    borderWidth: 3,
                    tension: 0.4,
                    fill: true,
                    pointRadius: 0,
                    pointHoverRadius: 6
                }, {
                    label: 'Setpoint',
                    data: [],
                    borderColor: '#00d4ff',
                    borderWidth: 2,
                    borderDash: [8, 4],
                    tension: 0.4,
                    fill: false,
                    pointRadius: 0
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
                        bodyColor: '#ffffff',
                        borderColor: '#00d4ff',
                        borderWidth: 2,
                        padding: 14,
                        callbacks: {
                            label: function(context) {
                                let label = context.dataset.label || '';
                                if (label) {
                                    label += ': ';
                                }
                                if (context.parsed.y !== null) {
                                    label += context.parsed.y.toFixed(2) + '°C';
                                }
                                return label;
                            }
                        }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: false,
                        min: 20,
                        max: 60,
                        grid: { color: 'rgba(0, 212, 255, 0.1)' },
                        ticks: { 
                            color: '#00d4ff',
                            callback: function(value) {
                                return value + '°C';
                            }
                        }
                    },
                    x: {
                        grid: { color: 'rgba(0, 212, 255, 0.08)' },
                        ticks: { color: '#6b7280', maxTicksLimit: 10 }
                    }
                }
            }
        });

        // ============= MQTT CONNECTION =============
        function connectMQTT() {
            console.log('Connecting to MQTT broker...');
            
            const clientId = 'web_client_' + Math.random().toString(16).substr(2, 8);
            const connectUrl = `ws://${mqtt_broker}:${mqtt_port}/mqtt`;
            
            mqttClient = mqtt.connect(connectUrl, {
                clientId: clientId,
                clean: true,
                reconnectPeriod: 3000
            });

            mqttClient.on('connect', () => {
                console.log('MQTT Connected!');
                document.getElementById('statusDot').classList.add('online');
                document.getElementById('statusText').textContent = 'ONLINE';
                
                mqttClient.subscribe(mqtt_topic_data, (err) => {
                    if (err) {
                        console.error('Subscribe error:', err);
                    } else {
                        console.log('Subscribed to:', mqtt_topic_data);
                    }
                });
            });

            mqttClient.on('message', (topic, message) => {
                try {
                    const data = JSON.parse(message.toString());
                    console.log('Received:', data);
                    updateDisplay(data);
                } catch (error) {
                    console.error('Parse error:', error);
                }
            });

            mqttClient.on('error', (error) => {
                console.error('MQTT Error:', error);
                document.getElementById('statusDot').classList.remove('online');
                document.getElementById('statusText').textContent = 'ERROR';
            });

            mqttClient.on('close', () => {
                console.log('MQTT Disconnected');
                document.getElementById('statusDot').classList.remove('online');
                document.getElementById('statusText').textContent = 'OFFLINE';
            });

            mqttClient.on('reconnect', () => {
                console.log('Reconnecting to MQTT...');
                document.getElementById('statusText').textContent = 'RECONNECTING';
            });
        }

        // ============= UPDATE DISPLAY =============
        function updateDisplay(data) {
            // Update temperature displays dengan 2 desimal
            document.getElementById('currentTemp').textContent = data.currentTemp.toFixed(2);
            document.getElementById('setpointDisplay').textContent = data.setpoint.toFixed(2);
            document.getElementById('patientTemp').textContent = data.patientTemp.toFixed(2);
            
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

        // ============= SEND COMMAND TO ESP32 =============
        function sendCommand(command, value) {
            if (mqttClient && mqttClient.connected) {
                const payload = {
                    command: command,
                    value: value
                };
                mqttClient.publish(mqtt_topic_control, JSON.stringify(payload));
                console.log('Command sent:', payload);
            } else {
                console.error('MQTT not connected!');
            }
        }

        // Auto-connect on page load
        window.addEventListener('load', () => {
            connectMQTT();
        });
    </script>
</body>
</html>
