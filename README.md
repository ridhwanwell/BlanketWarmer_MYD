<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Warming Blanket — Monitor</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mqtt/4.3.7/mqtt.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;500;600;700;800&family=Nunito+Sans:wght@400;600;700&display=swap" rel="stylesheet">
<style>
:root{
  --navy-900:#0f1f3d;--navy-800:#162747;--navy-700:#1e3560;--navy-600:#274478;
  --navy-500:#3358a0;--navy-400:#4a72c0;--navy-300:#7a9fd4;--navy-100:#dce8f8;--navy-50:#eef4fc;
  --white:#ffffff;--gray-50:#f8fafd;--gray-100:#f0f4f9;--gray-200:#e2eaf4;
  --gray-300:#c8d6e8;--gray-400:#9aaec8;--gray-500:#6b85a3;
  --teal:#2ab5c0;--teal-lt:#e6f8f9;--coral:#e8624a;--coral-lt:#fdecea;
  --green:#27ae60;--green-lt:#e8f8f0;--amber:#f59e0b;
  --sidebar-w:300px;--header-h:64px;--radius:14px;--radius-sm:8px;
  --shadow-sm:0 1px 4px rgba(15,31,61,.07);--shadow:0 4px 16px rgba(15,31,61,.10);
}
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html,body{height:100%;overflow:hidden;background:var(--gray-100);color:var(--navy-900);
  font-family:'Nunito Sans',sans-serif;font-size:13.5px;-webkit-font-smoothing:antialiased}
#app{display:grid;grid-template-rows:var(--header-h) 1fr;
  grid-template-columns:1fr var(--sidebar-w);grid-template-areas:"hdr hdr" "main side";
  height:100vh;width:100vw}
header{grid-area:hdr;background:var(--white);border-bottom:1.5px solid var(--gray-200);
  display:flex;align-items:center;justify-content:space-between;padding:0 28px;
  box-shadow:var(--shadow-sm);z-index:20}
.brand{display:flex;align-items:center;gap:12px}
.brand-icon{width:38px;height:38px;background:linear-gradient(135deg,var(--navy-700),var(--navy-500));
  border-radius:10px;display:flex;align-items:center;justify-content:center;
  box-shadow:0 4px 12px rgba(51,88,160,.3)}
.brand-icon svg{width:20px;height:20px;stroke:white}
.brand-text h1{font-family:'Nunito',sans-serif;font-size:1.05rem;font-weight:800;color:var(--navy-900)}
.brand-text p{font-size:.68rem;color:var(--gray-500);font-weight:500;margin-top:1px}
.hdr-right{display:flex;align-items:center;gap:10px}
.conn-pill{display:flex;align-items:center;gap:7px;padding:6px 14px;border-radius:100px;
  border:1.5px solid var(--gray-200);background:var(--gray-50);font-size:.7rem;font-weight:600;color:var(--gray-500);transition:all .35s}
.conn-pill.connected{background:var(--green-lt);border-color:rgba(39,174,96,.35);color:var(--green)}
.conn-dot{width:7px;height:7px;border-radius:50%;background:var(--gray-300);transition:all .35s}
.conn-pill.connected .conn-dot{background:var(--green);box-shadow:0 0 0 3px rgba(39,174,96,.2);animation:breathe 2s ease-in-out infinite}
@keyframes breathe{0%,100%{box-shadow:0 0 0 3px rgba(39,174,96,.2)}50%{box-shadow:0 0 0 6px rgba(39,174,96,.08)}}
.run-badge{display:flex;align-items:center;gap:6px;padding:6px 14px;border-radius:100px;
  font-size:.7rem;font-weight:700;letter-spacing:.06em;background:var(--gray-100);
  color:var(--gray-400);border:1.5px solid var(--gray-200);transition:all .35s}
.run-badge.running{background:var(--teal-lt);border-color:rgba(42,181,192,.4);color:var(--teal)}
.run-dot{width:6px;height:6px;border-radius:50%;background:currentColor;opacity:0;transition:opacity .3s}
.run-badge.running .run-dot{opacity:1;animation:pulse-dot 1.4s ease-in-out infinite}
@keyframes pulse-dot{0%,100%{transform:scale(1)}50%{transform:scale(1.5)}}
.btn-icon{width:36px;height:36px;border-radius:var(--radius-sm);background:var(--gray-100);
  border:1.5px solid var(--gray-200);color:var(--gray-500);cursor:pointer;
  display:flex;align-items:center;justify-content:center;transition:all .2s}
.btn-icon:hover{background:var(--navy-50);border-color:var(--navy-300);color:var(--navy-600)}
.btn-icon svg{width:15px;height:15px;fill:none;stroke:currentColor;stroke-width:2;stroke-linecap:round}
main{grid-area:main;display:flex;flex-direction:column;overflow:hidden;background:var(--gray-100)}
.metric-strip{display:grid;grid-template-columns:repeat(4,1fr);gap:14px;padding:16px 20px 0;flex-shrink:0}
.metric-card{background:var(--white);border-radius:var(--radius);padding:16px 18px;box-shadow:var(--shadow-sm);
  border:1.5px solid var(--gray-200);position:relative;overflow:hidden;transition:box-shadow .2s,transform .2s}
.metric-card:hover{box-shadow:var(--shadow);transform:translateY(-1px)}
.metric-card::before{content:'';position:absolute;top:0;left:0;right:0;height:3px;border-radius:var(--radius) var(--radius) 0 0}
.mc-alat::before  {background:linear-gradient(90deg,var(--teal),#5ce0e8)}
.mc-pasien::before{background:linear-gradient(90deg,var(--coral),#f0896f)}
.mc-sp::before    {background:linear-gradient(90deg,var(--navy-500),var(--navy-300))}
.mc-sw::before    {background:linear-gradient(90deg,var(--green),#5dd890)}
.mc-label{font-size:.65rem;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--gray-400);margin-bottom:8px}
.mc-value{font-family:'Nunito',sans-serif;font-size:1.9rem;font-weight:800;line-height:1;letter-spacing:-.02em}
.mc-alat   .mc-value{color:var(--teal)}
.mc-pasien .mc-value{color:var(--coral)}
.mc-sp     .mc-value{color:var(--navy-600)}
.mc-sw     .mc-value{color:var(--green)}
.mc-unit{font-size:.65rem;font-weight:600;color:var(--gray-400);margin-top:5px}
.mc-icon{position:absolute;right:14px;top:50%;transform:translateY(-50%);font-size:2rem;opacity:.07;pointer-events:none}
@keyframes val-flash{0%{opacity:1}35%{opacity:.2}100%{opacity:1}}
.flash-update{animation:val-flash .32s ease}
.chart-container{flex:1;margin:14px 20px 16px;background:var(--white);border-radius:var(--radius);
  border:1.5px solid var(--gray-200);box-shadow:var(--shadow-sm);display:flex;flex-direction:column;overflow:hidden;min-height:0}
.chart-header{display:flex;align-items:center;justify-content:space-between;padding:12px 18px 10px;
  border-bottom:1px solid var(--gray-100);flex-shrink:0;flex-wrap:wrap;gap:8px}
.chart-title{font-family:'Nunito',sans-serif;font-weight:800;font-size:.85rem;color:var(--navy-800)}
.chart-controls{display:flex;align-items:center;gap:8px;flex-wrap:wrap}
.chart-legend{display:flex;gap:14px}
.leg-item{display:flex;align-items:center;gap:7px;font-size:.68rem;font-weight:600;color:var(--gray-500)}
.leg-line{width:22px;height:2.5px;border-radius:3px}
.pan-hint{display:none;align-items:center;gap:5px;padding:4px 12px;border-radius:100px;
  background:var(--navy-50);border:1.5px solid var(--navy-100);font-size:.68rem;font-weight:700;color:var(--navy-500)}
.pan-hint.show{display:flex}
.btn-live{display:none;padding:5px 14px;border-radius:6px;background:var(--teal-lt);
  border:1.5px solid rgba(42,181,192,.4);color:var(--teal);font-family:'Nunito',sans-serif;
  font-size:.7rem;font-weight:700;cursor:pointer;transition:all .2s}
.btn-live:hover{background:rgba(42,181,192,.2)}
.btn-live.show{display:block}
.chart-canvas-wrap{flex:1;padding:12px 18px;position:relative;min-height:0;
  user-select:none;-webkit-user-select:none;touch-action:none}
.chart-canvas-wrap canvas{position:absolute;inset:12px 18px;
  width:calc(100% - 36px) !important;height:calc(100% - 24px) !important}
aside{grid-area:side;background:var(--white);border-left:1.5px solid var(--gray-200);display:flex;flex-direction:column;overflow:hidden}
.side-section{padding:18px;border-bottom:1px solid var(--gray-100);flex-shrink:0}
.side-title{font-family:'Nunito',sans-serif;font-size:.72rem;font-weight:800;letter-spacing:.1em;
  text-transform:uppercase;color:var(--navy-700);margin-bottom:14px;display:flex;align-items:center;gap:7px}
.side-title::before{content:'';width:3px;height:14px;background:var(--navy-400);border-radius:2px}
.cfg-field{margin-bottom:9px}
.cfg-field label{display:block;font-size:.65rem;font-weight:700;letter-spacing:.06em;text-transform:uppercase;color:var(--gray-500);margin-bottom:4px}
.cfg-field input{width:100%;background:var(--gray-50);border:1.5px solid var(--gray-200);border-radius:var(--radius-sm);
  padding:7px 11px;color:var(--navy-800);font-family:'Nunito Sans',sans-serif;font-size:.8rem;font-weight:500;outline:none;transition:all .2s}
.cfg-field input:focus{border-color:var(--navy-400);background:var(--white);box-shadow:0 0 0 3px rgba(74,114,192,.12)}
.btn-connect{width:100%;margin-top:10px;padding:9px;background:linear-gradient(135deg,var(--navy-700),var(--navy-500));
  border:none;border-radius:var(--radius-sm);color:white;font-family:'Nunito',sans-serif;
  font-size:.78rem;font-weight:700;letter-spacing:.05em;cursor:pointer;transition:all .2s;box-shadow:0 4px 12px rgba(30,53,96,.25)}
.btn-connect:hover{transform:translateY(-1px);box-shadow:0 6px 18px rgba(30,53,96,.32)}
.pid-row{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:14px}
.pid-chip{background:var(--navy-50);border:1.5px solid var(--navy-100);border-radius:var(--radius-sm);padding:10px 6px;text-align:center}
.pid-chip-lbl{font-size:.6rem;font-weight:700;letter-spacing:.1em;text-transform:uppercase;color:var(--navy-400);margin-bottom:5px}
.pid-chip-val{font-family:'Nunito',sans-serif;font-size:1.05rem;font-weight:800;color:var(--navy-800)}
.out-row{display:flex;justify-content:space-between;font-size:.65rem;font-weight:700;color:var(--gray-500);letter-spacing:.05em;text-transform:uppercase;margin-bottom:6px}
.out-track{height:6px;background:var(--gray-100);border-radius:10px;overflow:hidden;border:1px solid var(--gray-200)}
.out-fill{height:100%;width:0%;background:linear-gradient(90deg,var(--teal),var(--navy-400));border-radius:10px;transition:width .5s cubic-bezier(.4,0,.2,1);box-shadow:0 0 8px rgba(42,181,192,.4)}
.btn-export{width:100%;padding:9px;margin-bottom:7px;background:var(--teal-lt);border:1.5px solid rgba(42,181,192,.4);
  border-radius:var(--radius-sm);color:#1a9da8;font-family:'Nunito',sans-serif;font-size:.75rem;font-weight:700;cursor:pointer;transition:all .2s}
.btn-export:hover{background:rgba(42,181,192,.15);border-color:var(--teal);transform:translateY(-1px)}
.btn-clear{width:100%;padding:8px;background:var(--gray-50);border:1.5px solid var(--gray-200);border-radius:var(--radius-sm);
  color:var(--gray-500);font-family:'Nunito',sans-serif;font-size:.73rem;font-weight:600;cursor:pointer;transition:all .2s}
.btn-clear:hover{background:var(--coral-lt);border-color:rgba(232,98,74,.35);color:var(--coral)}
.log-wrap{flex:1;padding:16px 18px;display:flex;flex-direction:column;overflow:hidden;min-height:0}
.log-scroll{flex:1;overflow-y:auto;scrollbar-width:thin;scrollbar-color:var(--gray-200) transparent}
.log-scroll::-webkit-scrollbar{width:3px}
.log-scroll::-webkit-scrollbar-thumb{background:var(--gray-200);border-radius:3px}
.log-entry{display:grid;grid-template-columns:56px 1fr;gap:8px;padding:5px 0;border-bottom:1px solid var(--gray-100);
  font-size:.65rem;line-height:1.5;font-weight:500;opacity:0;animation:log-in .28s ease forwards}
@keyframes log-in{from{opacity:0;transform:translateX(6px)}to{opacity:1;transform:translateX(0)}}
.log-t{color:var(--gray-300);font-weight:600}
.log-m{color:var(--gray-500)}
.log-m.ok{color:var(--green)}
.log-m.warn{color:var(--amber)}
.log-m.err{color:var(--coral)}
@keyframes fade-slide-up{from{opacity:0;transform:translateY(12px)}to{opacity:1;transform:translateY(0)}}
header{animation:fade-slide-up .45s .00s ease both}
.metric-strip{animation:fade-slide-up .45s .07s ease both}
.chart-container{animation:fade-slide-up .45s .14s ease both}
aside{animation:fade-slide-up .45s .10s ease both}
</style>
</head>
<body>
<div id="app">
  <header>
    <div class="brand">
      <div class="brand-icon">
        <svg viewBox="0 0 24 24" fill="none" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
          <path d="M12 2a5 5 0 0 1 5 5c0 3.5-5 11-5 11S7 10.5 7 7a5 5 0 0 1 5-5z"/>
          <circle cx="12" cy="7" r="2"/>
        </svg>
      </div>
      <div class="brand-text">
        <h1>Warming Blanket Monitor</h1>
        <p>ESP32 · MQTT Real-time Dashboard</p>
      </div>
    </div>
    <div class="hdr-right">
      <div class="run-badge" id="runBadge"><div class="run-dot"></div><span id="runLabel">Stopped</span></div>
      <div class="conn-pill" id="connPill"><div class="conn-dot"></div><span id="connLabel">Disconnected</span></div>
      <button class="btn-icon" onclick="toggleFS()" title="Fullscreen">
        <svg viewBox="0 0 24 24"><path d="M4 8V4h4M20 8V4h-4M4 16v4h4M20 16v4h-4"/></svg>
      </button>
    </div>
  </header>

  <main>
    <div class="metric-strip">
      <div class="metric-card mc-alat">
        <div class="mc-label">Suhu Alat</div>
        <div class="mc-value" id="suhuAlat">—</div>
        <div class="mc-unit">°C · DS18B20</div>
        <div class="mc-icon">🌡️</div>
      </div>
      <div class="metric-card mc-pasien">
        <div class="mc-label">Suhu Pasien</div>
        <div class="mc-value" id="suhuPasien">—</div>
        <div class="mc-unit">°C · PT100</div>
        <div class="mc-icon">🫀</div>
      </div>
      <div class="metric-card mc-sp">
        <div class="mc-label">Setpoint</div>
        <div class="mc-value" id="setpointVal">—</div>
        <div class="mc-unit">°C · Target Suhu</div>
        <div class="mc-icon">🎯</div>
      </div>
      <div class="metric-card mc-sw">
        <div class="mc-label">Elapsed Time</div>
        <div class="mc-value" id="swDisplay" style="font-size:1.45rem">00:00:00</div>
        <div class="mc-unit">hh · mm · ss</div>
        <div class="mc-icon">⏱️</div>
      </div>
    </div>

    <div class="chart-container">
      <div class="chart-header">
        <span class="chart-title">📈 Temperature Waveform</span>
        <div class="chart-controls">
          <div class="pan-hint" id="panHint">← Tarik grafik untuk geser →</div>
          <button class="btn-live" id="btnLive" onclick="goLive()">▶ Kembali ke Live</button>
          <div class="chart-legend">
            <div class="leg-item"><div class="leg-line" style="background:var(--teal)"></div><span>Suhu Alat</span></div>
            <div class="leg-item"><div class="leg-line" style="background:none;border-top:2.5px dashed var(--navy-400)"></div><span>Setpoint</span></div>
          </div>
        </div>
      </div>
      <div class="chart-canvas-wrap" id="chartWrap">
        <canvas id="myChart"></canvas>
      </div>
    </div>
  </main>

  <aside>
    <div class="side-section">
      <div class="side-title">Connection</div>
      <div class="cfg-field"><label>Broker Host</label><input id="brokerHost" value="broker.hivemq.com"></div>
      <div class="cfg-field"><label>Port (WebSocket)</label><input id="brokerPort" value="8000"></div>
      <div class="cfg-field"><label>Topic Data</label><input id="topicData" value="warmingblanket/data"></div>
      <button class="btn-connect" id="connBtn" onclick="toggleConnect()">Connect</button>
    </div>
    <div class="side-section">
      <div class="side-title">PID Parameters</div>
      <div class="pid-row">
        <div class="pid-chip"><div class="pid-chip-lbl">Kp</div><div class="pid-chip-val" id="kpVal">—</div></div>
        <div class="pid-chip"><div class="pid-chip-lbl">Ki</div><div class="pid-chip-val" id="kiVal">—</div></div>
        <div class="pid-chip"><div class="pid-chip-lbl">Kd</div><div class="pid-chip-val" id="kdVal">—</div></div>
      </div>
      <div class="out-row"><span>PWM Output</span><span id="outNum">0 / 255</span></div>
      <div class="out-track"><div class="out-fill" id="outBar"></div></div>
    </div>
    <div class="side-section">
      <div class="side-title">Export Data</div>
      <button class="btn-export" onclick="exportExcel()">↓ Export ke Excel (.xlsx)</button>
      <button class="btn-clear"  onclick="clearData()">Hapus Grafik</button>
    </div>
    <div class="log-wrap">
      <div class="side-title">System Log</div>
      <div class="log-scroll" id="logScroll"></div>
    </div>
  </aside>
</div>

<script>
/* ═══════════════════════════════════════════════
   CONFIG — ubah PAGE_SIZE sesuai kebutuhan
   PAGE_SIZE = jumlah titik yang tampil sekaligus
═══════════════════════════════════════════════ */
const PAGE_SIZE = 60;

/* ═══ STATE ═══ */
let mqttClient = null, isRunning = false, chartStarted = false;
let swTimer = null, swStart = null, dataLog = [];

// Semua data disimpan di sini (tidak dibatasi)
const ALL = { labels: [], d0: [], d1: [] };

// viewStart = indeks pertama yg ditampilkan; null = mode live
let viewStart = null;
let dragging = false, dragX0 = 0, dragOff0 = 0;

/* ═══ CHART INIT ═══ */
const canvas = document.getElementById('myChart');
const wrap   = document.getElementById('chartWrap');

const chart = new Chart(canvas.getContext('2d'), {
  type: 'line',
  data: {
    labels: [],
    datasets: [
      { label:'Suhu Alat', data:[], borderColor:'#2ab5c0', backgroundColor:'rgba(42,181,192,.07)',
        borderWidth:2, pointRadius:0, pointHoverRadius:5, tension:0.4, fill:true },
      { label:'Setpoint',  data:[], borderColor:'#4a72c0', backgroundColor:'transparent',
        borderWidth:1.8, borderDash:[6,4], pointRadius:0, tension:0, fill:false }
    ]
  },
  options: {
    responsive:false, animation:false,
    interaction:{ mode:'index', intersect:false },
    scales:{
      x:{ grid:{color:'rgba(15,31,61,.05)'},
          ticks:{color:'#9aaec8',font:{family:'Nunito Sans',size:10,weight:'600'},maxTicksLimit:8,maxRotation:0},
          border:{color:'#e2eaf4'} },
      y:{ min:28, max:50, grid:{color:'rgba(15,31,61,.05)'},
          ticks:{color:'#9aaec8',font:{family:'Nunito Sans',size:10,weight:'600'},callback:v=>v+' °C'},
          border:{color:'#e2eaf4'} }
    },
    plugins:{
      legend:{display:false},
      tooltip:{
        backgroundColor:'#fff',borderColor:'#e2eaf4',borderWidth:1.5,
        titleColor:'#1e3560',bodyColor:'#6b85a3',padding:12,
        titleFont:{family:'Nunito',size:11,weight:'800'},
        bodyFont:{family:'Nunito Sans',size:11,weight:'600'},
        callbacks:{ label: c=>`  ${c.dataset.label}: ${c.parsed.y!==null?c.parsed.y.toFixed(2)+' °C':'—'}` }
      }
    }
  }
});

/* ═══ RESIZE ═══ */
function resizeChart(){
  const w = wrap.clientWidth-36, h = wrap.clientHeight-24;
  if(w<=0||h<=0) return;
  canvas.style.width=w+'px'; canvas.style.height=h+'px';
  canvas.width=Math.round(w*devicePixelRatio); canvas.height=Math.round(h*devicePixelRatio);
  chart.resize(); renderView();
}
window.addEventListener('resize', resizeChart);
setTimeout(resizeChart, 120);

/* ═══ RENDER ENGINE ═══
   - Jika total ≤ PAGE_SIZE: tampil semua, tidak ada pan
   - Jika total > PAGE_SIZE: tampil PAGE_SIZE titik, pan aktif
═══════════════════════ */
function total(){ return ALL.labels.length; }
function canPan(){ return total() > PAGE_SIZE; }
function isLive(){ return viewStart === null; }

function effectiveStart(){
  if(!canPan()) return 0;
  if(isLive())  return total() - PAGE_SIZE;
  return Math.max(0, Math.min(viewStart, total()-PAGE_SIZE));
}

function renderView(){
  const n = total();
  if(n === 0){
    chart.data.labels=[]; chart.data.datasets[0].data=[]; chart.data.datasets[1].data=[];
    chart.update('none'); return;
  }
  const s = effectiveStart();
  const e = canPan() ? s + PAGE_SIZE : n;
  chart.data.labels         = ALL.labels.slice(s,e);
  chart.data.datasets[0].data = ALL.d0.slice(s,e);
  chart.data.datasets[1].data = ALL.d1.slice(s,e);
  chart.update('none');
}

function updateUI(){
  const p = canPan();
  document.getElementById('panHint').classList.toggle('show', p);
  wrap.style.cursor = p ? 'grab' : 'default';
  // Tombol "Kembali ke Live" hanya muncul kalau sedang geser (bukan live)
  document.getElementById('btnLive').classList.toggle('show', p && !isLive());
}

function goLive(){
  viewStart = null;
  updateUI(); renderView();
}

/* ═══ DRAG / PAN ═══ */
wrap.addEventListener('mousedown',  startDrag);
wrap.addEventListener('touchstart', startDrag, {passive:true});
window.addEventListener('mousemove',  doDrag);
window.addEventListener('touchmove',  doDrag, {passive:false});
window.addEventListener('mouseup',    endDrag);
window.addEventListener('touchend',   endDrag);

function clientX(e){ return e.touches ? e.touches[0].clientX : e.clientX; }

function startDrag(e){
  if(!canPan()) return;
  dragging=true;
  dragX0 = clientX(e);
  dragOff0 = effectiveStart();
  wrap.style.cursor='grabbing';
}

function doDrag(e){
  if(!dragging) return;
  e.preventDefault && e.preventDefault();
  const dx = clientX(e) - dragX0;
  // berapa piksel per titik
  const pxPerPt = (canvas.clientWidth||wrap.clientWidth-36) / PAGE_SIZE;
  const delta   = Math.round(dx / pxPerPt);
  // geser kiri = lihat masa lalu (offset berkurang), geser kanan = ke masa kini
  const newStart = Math.max(0, Math.min(dragOff0 - delta, total()-PAGE_SIZE));
  viewStart = newStart;
  // Jika sudah di ujung kanan, kembali ke live
  if(newStart >= total()-PAGE_SIZE) viewStart = null;
  updateUI(); renderView();
}

function endDrag(){
  if(!dragging) return;
  dragging=false;
  wrap.style.cursor = canPan()?'grab':'default';
}

/* ═══ MQTT ═══ */
function toggleConnect(){
  if(mqttClient && mqttClient.connected){
    mqttClient.end(); setConn(false);
    log('Terputus dari broker.','warn');
    document.getElementById('connBtn').textContent='Connect'; return;
  }
  connectMQTT();
}

function connectMQTT(){
  const host = document.getElementById('brokerHost').value.trim();
  const port = parseInt(document.getElementById('brokerPort').value)||8000;
  const cid  = 'wb_'+Math.random().toString(36).substr(2,9);
  const url  = `ws://${host}:${port}/mqtt`;
  log('Menghubungkan ke '+url+'…','warn');
  document.getElementById('connBtn').textContent='Menghubungkan…';
  try{
    if(mqttClient) mqttClient.end(true);
    mqttClient = mqtt.connect(url,{clientId:cid,clean:true,connectTimeout:10000,reconnectPeriod:4000});
    mqttClient.on('connect',()=>{
      setConn(true);
      document.getElementById('connBtn').textContent='Disconnect';
      const t=document.getElementById('topicData').value.trim();
      mqttClient.subscribe(t,e=>e?log('Gagal subscribe','err'):log('Subscribe: '+t,'ok'));
      log('Terhubung · '+cid,'ok');
    });
    mqttClient.on('message',(_,m)=>{ try{onData(JSON.parse(m.toString()));}catch(e){log('Parse error: '+e.message,'err');} });
    mqttClient.on('error',    e  =>{ log('Error: '+e.message,'err'); setConn(false); });
    mqttClient.on('close',    ()=>{ setConn(false); document.getElementById('connBtn').textContent='Connect'; });
    mqttClient.on('reconnect',()=>log('Menghubungkan ulang…','warn'));
  }catch(e){ log('Gagal konek: '+e.message,'err'); }
}

function setConn(yes){
  document.getElementById('connPill').classList.toggle('connected',yes);
  document.getElementById('connLabel').textContent=yes?'Terhubung':'Terputus';
}

/* ═══ DATA ═══ */
function onData(d){
  const was=isRunning; isRunning=!!d.isRunning;
  if(!was&&isRunning)  onStart();
  if(was &&!isRunning) onStop();
  if(d.currentTemp!==undefined) setVal('suhuAlat',   d.currentTemp.toFixed(2));
  if(d.patientTemp!==undefined) setVal('suhuPasien', d.patientTemp<-100?'Error':d.patientTemp.toFixed(2));
  if(d.setpoint   !==undefined) setVal('setpointVal',parseFloat(d.setpoint).toFixed(1));
  if(d.kp         !==undefined) setVal('kpVal',      parseFloat(d.kp).toFixed(2));
  if(d.ki         !==undefined) setVal('kiVal',      parseFloat(d.ki).toFixed(3));
  if(d.kd         !==undefined) setVal('kdVal',      parseFloat(d.kd).toFixed(2));
  if(d.output!==undefined){
    const pct=Math.min(100,(d.output/255)*100);
    document.getElementById('outBar').style.width=pct+'%';
    document.getElementById('outNum').textContent=Math.round(d.output)+' / 255';
  }
  const badge=document.getElementById('runBadge');
  if(isRunning){badge.className='run-badge running';document.getElementById('runLabel').textContent='Running';}
  else         {badge.className='run-badge';         document.getElementById('runLabel').textContent='Stopped';}
  if(isRunning&&chartStarted) addPoint(d);
}

function onStart(){
  log('Device START — grafik & stopwatch direset','ok');
  chartStarted=true;
  ALL.labels.length=0; ALL.d0.length=0; ALL.d1.length=0;
  dataLog=[]; viewStart=null;
  updateUI(); renderView();
  swStart=Date.now();
  if(swTimer) clearInterval(swTimer);
  swTimer=setInterval(tickSw,1000);
}

function onStop(){
  log(`Device STOP — ${dataLog.length} data points`,'warn');
  chartStarted=false;
  if(swTimer){clearInterval(swTimer);swTimer=null;}
}

function addPoint(d){
  const now=new Date();
  const lbl=now.toLocaleTimeString('id-ID',{hour:'2-digit',minute:'2-digit',second:'2-digit'});
  ALL.labels.push(lbl);
  ALL.d0.push(parseFloat(d.currentTemp)??null);
  ALL.d1.push(parseFloat(d.setpoint)   ??null);

  // Saat pertama kali data melebihi PAGE_SIZE, beri tahu user
  if(total()===PAGE_SIZE+1) log('Grafik bisa digeser ← → untuk lihat data sebelumnya','ok');

  // Kalau sedang live, ikut scroll ke kanan otomatis
  // Kalau sedang pan, diam di posisi user
  renderView(); updateUI();

  dataLog.push({
    ts:now.toISOString(),time:lbl,
    elapsed:document.getElementById('swDisplay').textContent,
    suhuAlat:d.currentTemp,suhuPasien:d.patientTemp,
    setpoint:d.setpoint,output:d.output,kp:d.kp,ki:d.ki,kd:d.kd,
  });
}

/* ═══ STOPWATCH ═══ */
function tickSw(){
  if(!swStart) return;
  const s  =Math.floor((Date.now()-swStart)/1000);
  const h  =String(Math.floor(s/3600)).padStart(2,'0');
  const m  =String(Math.floor((s%3600)/60)).padStart(2,'0');
  const sec=String(s%60).padStart(2,'0');
  document.getElementById('swDisplay').textContent=`${h}:${m}:${sec}`;
}

/* ═══ HELPERS ═══ */
function setVal(id,v){
  const el=document.getElementById(id);
  if(!el||el.textContent===String(v)) return;
  el.textContent=v;
  el.classList.remove('flash-update'); void el.offsetWidth; el.classList.add('flash-update');
}
function log(msg,type=''){
  const sc=document.getElementById('logScroll');
  const t=new Date().toLocaleTimeString('id-ID',{hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const el=document.createElement('div');
  el.className='log-entry';
  el.innerHTML=`<span class="log-t">${t}</span><span class="log-m ${type}">${msg}</span>`;
  sc.prepend(el);
  while(sc.children.length>80) sc.removeChild(sc.lastChild);
}
function clearData(){
  ALL.labels.length=0; ALL.d0.length=0; ALL.d1.length=0;
  dataLog=[]; viewStart=null; updateUI(); renderView();
  log('Grafik dihapus.','warn');
}
function toggleFS(){
  const el=document.getElementById('app');
  if(!document.fullscreenElement) el.requestFullscreen?.();
  else document.exitFullscreen?.();
}
document.addEventListener('fullscreenchange',()=>setTimeout(resizeChart,200));

/* ═══ EXCEL EXPORT ═══ */
function exportExcel(){
  if(!dataLog.length){log('Belum ada data untuk diekspor.','warn');return;}
  const wb=XLSX.utils.book_new();
  const rows=[
    ['WARMING BLANKET — DATA MONITOR'],
    ['Export:',new Date().toLocaleString('id-ID')],
    ['Total Data:',dataLog.length+' titik'],[],
    ['No','Timestamp','Waktu','Elapsed','Suhu Alat °C','Suhu Pasien °C','Setpoint °C','Output PWM','Kp','Ki','Kd']
  ];
  dataLog.forEach((r,i)=>rows.push([i+1,r.ts,r.time,r.elapsed,r.suhuAlat,r.suhuPasien,r.setpoint,r.output,r.kp,r.ki,r.kd]));
  const ws1=XLSX.utils.aoa_to_sheet(rows);
  ws1['!cols']=[{wch:4},{wch:26},{wch:11},{wch:12},{wch:14},{wch:16},{wch:13},{wch:12},{wch:7},{wch:7},{wch:7}];
  ws1['!merges']=[{s:{r:0,c:0},e:{r:0,c:10}}];
  XLSX.utils.book_append_sheet(wb,ws1,'Data Monitor');
  const temps=dataLog.map(r=>parseFloat(r.suhuAlat)).filter(v=>!isNaN(v));
  const avg=arr=>(arr.reduce((a,b)=>a+b,0)/arr.length).toFixed(2);
  const ws2=XLSX.utils.aoa_to_sheet([
    ['SUMMARY STATISTIK'],[],
    ['Total Data Points',dataLog.length],['Durasi',dataLog.at(-1)?.elapsed??'—'],[],
    ['Suhu Alat Minimum °C',Math.min(...temps).toFixed(2)],
    ['Suhu Alat Maksimum °C',Math.max(...temps).toFixed(2)],
    ['Suhu Alat Rata-rata °C',avg(temps)],[],
    ['Setpoint °C',dataLog.at(-1)?.setpoint??'—'],
    ['Kp',dataLog.at(-1)?.kp??'—'],['Ki',dataLog.at(-1)?.ki??'—'],['Kd',dataLog.at(-1)?.kd??'—'],
  ]);
  ws2['!cols']=[{wch:26},{wch:18}];
  ws2['!merges']=[{s:{r:0,c:0},e:{r:0,c:1}}];
  XLSX.utils.book_append_sheet(wb,ws2,'Summary');
  const fname=`WarmingBlanket_${new Date().toISOString().slice(0,19).replace(/[T:]/g,'-')}.xlsx`;
  XLSX.writeFile(wb,fname);
  log(`Exported: ${fname} (${dataLog.length} baris)`,'ok');
}

/* ═══ INIT ═══ */
log('Sistem siap. Konfigurasi MQTT lalu klik Connect.','');
setTimeout(connectMQTT,500);
</script>
</body>
</html>
