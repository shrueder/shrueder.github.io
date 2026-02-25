
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Presupuesto">
<meta name="theme-color" content="#0d0f14">
<title>Presupuesto</title>
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@300;400;500&display=swap" rel="stylesheet">

<script>
/* ── Inline donut chart – no external deps ── */
class DonutChart {
  constructor(canvas, data, colors) {
    this.canvas = canvas;
    this.ctx    = canvas.getContext('2d');
    this.data   = data;
    this.colors = colors;
    this._size();
    window.addEventListener('resize', () => { this._size(); this.draw(); });
  }
  _size() {
    const w = this.canvas.parentElement.clientWidth || 180;
    const s = Math.min(w, 220);
    this.canvas.width  = s;
    this.canvas.height = s;
  }
  update(data, colors) { this.data = data; this.colors = colors; this.draw(); }
  draw() {
    const { ctx, canvas, data, colors } = this;
    const W = canvas.width, H = canvas.height;
    ctx.clearRect(0, 0, W, H);
    const total = data.reduce((a, b) => a + b, 0);
    if (!total) return;
    const cx = W / 2, cy = H / 2;
    const r  = Math.min(W, H) / 2 - 3;
    const cr = r * 0.63;
    let start = -Math.PI / 2;
    data.forEach((v, i) => {
      const sweep = (v / total) * Math.PI * 2;
      ctx.beginPath();
      ctx.moveTo(cx + Math.cos(start) * cr, cy + Math.sin(start) * cr);
      ctx.arc(cx, cy, r,  start, start + sweep);
      ctx.arc(cx, cy, cr, start + sweep, start, true);
      ctx.closePath();
      ctx.fillStyle   = colors[i] || '#888';
      ctx.fill();
      ctx.strokeStyle = '#0d0f14';
      ctx.lineWidth   = 2;
      ctx.stroke();
      start += sweep;
    });
  }
}
</script>

<style>
:root {
  --bg:     #0d0f14;
  --sf:     #151820;
  --sf2:    #1c2030;
  --bd:     #2a2f42;
  --yellow: #f0c040;
  --green:  #5be4a4;
  --red:    #f06060;
  --blue:   #6090f0;
  --purple: #b07cf0;
  --text:   #e8eaf0;
  --muted:  #6a7090;
  --comida: #f0a050;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
html, body { height: 100%; }
body {
  font-family: 'DM Mono','Courier New',monospace;
  background: var(--bg); color: var(--text);
  min-height: 100vh; padding: 16px 12px 90px;
}
.wrap { max-width: 860px; margin: 0 auto; }

/* Header */
header {
  display: flex; align-items: baseline; gap: 12px;
  border-bottom: 1px solid var(--bd); padding-bottom: 14px; margin-bottom: 18px;
}
header h1 {
  font-family: 'Syne',system-ui,sans-serif;
  font-size: clamp(20px,5vw,28px); font-weight: 800;
  color: var(--yellow); letter-spacing: -0.5px;
}
header span { font-size: 10px; color: var(--muted); letter-spacing: 2px; text-transform: uppercase; }

/* Cards */
.summary     { display: grid; grid-template-columns: repeat(3,1fr); gap: 10px; margin-bottom: 10px; }
.balance-row { display: grid; grid-template-columns: repeat(2,1fr); gap: 10px; margin-bottom: 20px; }
.card {
  background: var(--sf); border: 1px solid var(--bd);
  border-radius: 12px; padding: 12px 14px; position: relative; overflow: hidden;
}
.card::before { content:''; position:absolute; top:0;left:0;right:0; height:2px; }
.card.ing::before   { background: var(--blue); }
.card.pres::before  { background: var(--yellow); }
.card.sav::before   { background: var(--green); }
.card.gast::before  { background: var(--red); }
.card.queda::before { background: var(--purple); }
.card label { font-size: 9px; letter-spacing: 2px; text-transform: uppercase; color: var(--muted); display: block; margin-bottom: 5px; }
.card .val  { font-family: 'Syne',system-ui,sans-serif; font-size: clamp(15px,4vw,21px); font-weight: 700; }
.card.ing .val   { color: var(--blue); }
.card.pres .val  { color: var(--yellow); }
.card.sav .val   { color: var(--green); }
.card.gast .val  { color: var(--red); }
.card.queda .val { color: var(--purple); }
.card input[type=number] {
  background: transparent; border: none; outline: none;
  font-family: 'Syne',system-ui,sans-serif; font-size: clamp(15px,4vw,21px);
  font-weight: 700; color: var(--blue); width: 100%; -moz-appearance: textfield;
}
.card input[type=number]::-webkit-outer-spin-button,
.card input[type=number]::-webkit-inner-spin-button { -webkit-appearance: none; }

/* Table */
.table-wrap { background: var(--sf); border: 1px solid var(--bd); border-radius: 14px; overflow: hidden; margin-bottom: 20px; }
.t-head {
  background: var(--sf2); padding: 10px 14px;
  font-family: 'Syne',system-ui,sans-serif; font-size: 10px; font-weight: 700;
  letter-spacing: 3px; text-transform: uppercase; color: var(--muted);
  border-bottom: 1px solid var(--bd); display: flex; justify-content: space-between; align-items: center;
}
.add-btn {
  background: var(--yellow); color: #000; border: none; border-radius: 6px;
  padding: 4px 11px; font-family: 'Syne',system-ui,sans-serif;
  font-size: 10px; font-weight: 700; letter-spacing: 1px; cursor: pointer;
}
.table-scroll { overflow-x: auto; -webkit-overflow-scrolling: touch; }
table { width: 100%; border-collapse: collapse; min-width: 460px; }
thead th {
  background: var(--sf2); padding: 8px 11px;
  font-size: 9px; letter-spacing: 1.5px; text-transform: uppercase;
  color: var(--muted); text-align: left; border-bottom: 1px solid var(--bd); font-weight: 400;
}
thead th.r { text-align: right; }
tbody tr { border-bottom: 1px solid var(--bd); transition: background .12s; }
tbody tr:last-child { border-bottom: none; }
tbody tr:hover { background: var(--sf2); }
td { padding: 8px 11px; font-size: 12px; vertical-align: middle; }
td.r { text-align: right; }

.tag {
  display: inline-block; padding: 2px 8px; border-radius: 20px;
  font-size: 9px; font-weight: 500; letter-spacing: 1px; text-transform: uppercase;
}
.COMIDA { background: rgba(240,160,80,.15);  color: var(--comida); }
.DEUDAS { background: rgba(240,96,96,.15);   color: var(--red); }
.OCIO   { background: rgba(91,228,164,.15);  color: var(--green); }
.FIJO   { background: rgba(96,144,240,.15);  color: var(--blue); }
.OTRO   { background: rgba(160,160,176,.15); color: var(--muted); }
.AHORRO { background: rgba(176,124,240,.15); color: var(--purple); }

td input {
  background: var(--sf2); border: 1px solid transparent; border-radius: 5px;
  color: var(--text); font-family: 'DM Mono','Courier New',monospace;
  font-size: 12px; padding: 3px 6px; outline: none; transition: border-color .2s;
  -moz-appearance: textfield;
}
td input::-webkit-outer-spin-button,
td input::-webkit-inner-spin-button { -webkit-appearance: none; }
td input:focus { border-color: var(--yellow); }
td input.ti { width: 88px; }
td input.ni { width: 70px; text-align: right; }
select {
  background: var(--sf2); border: 1px solid transparent; border-radius: 5px;
  color: var(--text); font-family: 'DM Mono','Courier New',monospace;
  font-size: 11px; padding: 3px 6px; outline: none; cursor: pointer;
}
.pct-bar { display: flex; align-items: center; gap: 5px; }
.pct-track { width: 34px; height: 3px; background: var(--bd); border-radius: 3px; overflow: hidden; flex-shrink: 0; }
.pct-fill  { height: 100%; border-radius: 3px; transition: width .3s; }
.pos { color: var(--green); font-weight: 600; }
.neg { color: var(--red);   font-weight: 600; }
.gauto { color: var(--green); font-weight: 600; }
.del-btn { background: transparent; border: none; color: var(--muted); cursor: pointer; font-size: 15px; padding: 2px 4px; border-radius: 4px; transition: color .2s; }
.del-btn:hover { color: var(--red); }

/* Bottom grid */
.bottom-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin-bottom: 20px; align-items: start; }

.totals-card { background: var(--sf); border: 1px solid var(--bd); border-radius: 14px; overflow: hidden; }
.totals-card table { min-width: unset; }
.totals-card td:nth-child(2),
.totals-card td:nth-child(3) { text-align: right; }

.chart-card {
  background: var(--sf); border: 1px solid var(--bd); border-radius: 14px;
  padding: 16px; display: flex; flex-direction: column; align-items: center; gap: 10px;
}
.chart-lbl { font-size: 9px; letter-spacing: 2px; text-transform: uppercase; color: var(--muted); font-family: 'Syne',system-ui,sans-serif; font-weight: 700; align-self: flex-start; }

/* Donut wrapper — key: relative + flex center */
.donut-wrap {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.donut-wrap canvas { display: block; max-width: 100%; }
.donut-center {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  text-align: center; pointer-events: none; white-space: nowrap;
}
.donut-center .big  { font-family: 'Syne',system-ui,sans-serif; font-size: clamp(12px,2.5vw,17px); font-weight: 800; color: var(--text); display: block; }
.donut-center .small{ font-size: 8px; letter-spacing: 1.5px; text-transform: uppercase; color: var(--muted); }

.legend { width: 100%; display: grid; grid-template-columns: 1fr 1fr; gap: 5px; }
.li { display: flex; align-items: center; gap: 5px; }
.ld { width: 7px; height: 7px; border-radius: 50%; flex-shrink: 0; }
.ln { color: var(--muted); flex: 1; font-size: 9px; }
.lp { color: var(--text); font-weight: 500; font-size: 10px; }

/* Savings */
.sav-detail { background: var(--sf); border: 1px solid var(--bd); border-radius: 14px; padding: 16px; margin-bottom: 20px; }
.sav-detail h3 { font-family: 'Syne',system-ui,sans-serif; font-size: 9px; font-weight: 700; letter-spacing: 3px; text-transform: uppercase; color: var(--muted); margin-bottom: 12px; }
.sav-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(130px,1fr)); gap: 8px; }
.si { background: var(--sf2); border-radius: 8px; padding: 10px 12px; border-left: 3px solid var(--green); }
.si.over { border-left-color: var(--red); }
.si .sl { font-size: 9px; color: var(--muted); letter-spacing: 1px; text-transform: uppercase; margin-bottom: 3px; }
.si .sv { font-family: 'Syne',system-ui,sans-serif; font-size: 14px; font-weight: 700; }
.si .ss { font-size: 9px; color: var(--muted); margin-top: 2px; }

/* Install banner */
#install-banner {
  display: none; position: fixed; bottom: 16px; left: 50%; transform: translateX(-50%);
  background: var(--sf2); border: 1px solid var(--yellow); border-radius: 14px;
  padding: 12px 14px; align-items: center; gap: 10px; z-index: 999;
  box-shadow: 0 8px 32px rgba(0,0,0,.6); max-width: 320px; width: calc(100% - 32px);
}
#install-banner.show { display: flex; }
#install-banner .bi { font-size: 22px; }
#install-banner .bt { flex: 1; }
#install-banner .bt strong { display: block; font-family: 'Syne',sans-serif; font-size: 12px; color: var(--text); }
#install-banner .bt span   { font-size: 10px; color: var(--muted); }
.ib-btn { background: var(--yellow); color: #000; border: none; border-radius: 7px; padding: 7px 12px; font-family: 'Syne',sans-serif; font-size: 11px; font-weight: 700; cursor: pointer; white-space: nowrap; }
.id-btn { background: transparent; border: none; color: var(--muted); cursor: pointer; font-size: 18px; padding: 2px; line-height: 1; }

/* Responsive */
@media (max-width: 600px) {
  .bottom-grid { grid-template-columns: 1fr; }
  .pct-track   { display: none; }
}
@media (max-width: 420px) {
  .summary { grid-template-columns: 1fr 1fr; }
  body { padding: 12px 8px 90px; }
}
</style>
</head>
<body>
<div class="wrap">

  <header>
    <h1>PRESUPUESTO</h1>
    <span>Control de Gastos</span>
  </header>

  <div class="summary">
    <div class="card ing">
      <label>Ingresos</label>
      <input type="number" id="ingreso" value="350" step="0.01" oninput="recalc()">
    </div>
    <div class="card pres">
      <label>Presupuesto</label>
      <div class="val" id="pres-total">B/. 0.00</div>
    </div>
    <div class="card sav">
      <label>Ahorros</label>
      <div class="val" id="ahorros-val">B/. 0.00</div>
    </div>
  </div>

  <div class="balance-row">
    <div class="card gast">
      <label>Gastado</label>
      <div class="val" id="gastado-val">B/. 0.00</div>
    </div>
    <div class="card queda">
      <label>Me queda</label>
      <div class="val" id="queda-val">B/. 0.00</div>
    </div>
  </div>

  <div class="table-wrap">
    <div class="t-head">
      <span>Tabla de Gastos</span>
      <div style="display:flex;align-items:center;gap:10px">
        <div style="display:flex;align-items:center;gap:6px;font-size:10px">
          <div style="width:80px;height:5px;background:var(--bd);border-radius:5px;overflow:hidden">
            <div id="pct-total-bar" style="height:100%;border-radius:5px;background:var(--green);transition:width .3s,background .3s"></div>
          </div>
          <span id="pct-total-label" style="color:var(--green);font-weight:700;min-width:38px">0.0%</span>
        </div>
        <button class="add-btn" onclick="addRow()">+ AGREGAR</button>
      </div>
    </div>
    <div class="table-scroll">
      <table>
        <thead>
          <tr>
            <th>Tipo</th>
            <th>Concepto</th>
            <th class="r">Presup.</th>
            <th>% Ingreso</th>
            <th class="r">Gastado</th>
            <th class="r">Diferencia</th>
            <th></th>
          </tr>
        </thead>
        <tbody id="tbody"></tbody>
      </table>
    </div>
  </div>

  <div class="bottom-grid">
    <div class="totals-card">
      <div class="t-head"><span>Totales por Categoría</span></div>
      <div class="table-scroll">
        <table>
          <thead><tr><th>Categoría</th><th class="r">Gastado</th><th class="r">%</th></tr></thead>
          <tbody id="totals-body"></tbody>
        </table>
      </div>
    </div>

    <div class="chart-card">
      <div class="chart-lbl">Distribución del Gasto</div>
      <div class="donut-wrap">
        <canvas id="donut"></canvas>
        <div class="donut-center">
          <span class="big"  id="donut-total">B/. 0</span>
          <span class="small">GASTADO</span>
        </div>
      </div>
      <div class="legend" id="legend"></div>
    </div>
  </div>

  <div class="sav-detail">
    <h3>Detalle de Ahorros</h3>
    <div class="sav-grid" id="sav-grid"></div>
  </div>

</div>

<div id="install-banner">
  <div class="bi">📊</div>
  <div class="bt"><strong>Instalar App</strong><span>Agrega a tu pantalla de inicio</span></div>
  <button class="ib-btn" id="ib-btn">Instalar</button>
  <button class="id-btn" id="id-btn">×</button>
</div>

<script>
const TIPOS  = ['COMIDA','DEUDAS','OCIO','FIJO','OTRO'];
const COLORS = { COMIDA:'#f0a050', DEUDAS:'#f06060', OCIO:'#5be4a4', FIJO:'#6090f0', OTRO:'#a0a0b0', AHORRO:'#b07cf0' };

let rows = [
  { id:1, tipo:'COMIDA', concepto:'SUPER',    pres:200, gast:206.50 },
  { id:2, tipo:'DEUDAS', concepto:'INTERNET', pres:30,  gast:28.00  },
  { id:3, tipo:'DEUDAS', concepto:'BANCO',    pres:40,  gast:38.50  },
  { id:4, tipo:'DEUDAS', concepto:'TITAN',    pres:5,   gast:4.20   },
  { id:5, tipo:'OCIO',   concepto:'SALIDAS',  pres:20,  gast:10.50  },
  { id:6, tipo:'FIJO',   concepto:'PASAJE',   pres:60,  gast:42.00  },
];
let nextId = 7;
let chart  = null;

const fmt    = n => 'B/. ' + (+n).toFixed(2);
const fmtPct = n => (+n).toFixed(1) + '%';
const getIng = () => parseFloat(document.getElementById('ingreso').value) || 1;

function addRow() {
  rows.push({ id: nextId++, tipo:'OTRO', concepto:'', pres:0, gast:0 });
  renderTable(); recalc();
}
function delRow(id) {
  rows = rows.filter(r => r.id !== id);
  renderTable(); recalc();
}

function onField(id, field, val) {
  const row = rows.find(r => r.id === id);
  if (!row) return;
  const i = getIng();

  if (field === 'presFromPct') {
    /* % edited → cap so total pct across all rows <= 100 */
    let requested = parseFloat(val) || 0;
    const usedPct = rows.filter(r => r.id !== id).reduce((s, r) => s + (r.pres / i * 100), 0);
    const available = Math.max(0, 100 - usedPct);
    const pct = Math.min(requested, available);
    /* if capped, update the input visually */
    if (pct !== requested) {
      const pi = document.getElementById('pct-' + id);
      if (pi) pi.value = pct.toFixed(1);
    }
    row.pres = pct / 100 * i;
    row.gast = row.pres;
    const presEl = document.getElementById('pres-' + id);
    if (presEl) presEl.value = row.pres.toFixed(2);
    const ge = document.getElementById('gast-' + id);
    if (ge) ge.textContent = fmt(row.gast);
    const diff = row.pres - row.gast;
    const de = document.getElementById('diff-' + id);
    if (de) { de.textContent = (diff>=0?'+':'') + fmt(diff); de.className = 'r ' + (diff>=0?'pos':'neg'); }
  } else if (field === 'pres') {
    /* pres edited directly → cap so total pct <= 100 */
    const requested = parseFloat(val) || 0;
    const usedPct = rows.filter(r => r.id !== id).reduce((s, r) => s + (r.pres / i * 100), 0);
    const available = Math.max(0, 100 - usedPct);
    const maxPres = available / 100 * i;
    row.pres = Math.min(requested, maxPres);
    if (row.pres !== requested) {
      const presEl = document.getElementById('pres-' + id);
      if (presEl) presEl.value = row.pres.toFixed(2);
    }
    /* sync pct display */
    const pi = document.getElementById('pct-' + id);
    if (pi && document.activeElement !== pi) pi.value = (row.pres / i * 100).toFixed(1);
  } else if (field === 'tipo' || field === 'concepto') {
    row[field] = val;
  } else {
    row[field] = parseFloat(val) || 0;
  }
  recalc();
}

function renderTable() {
  const i  = getIng();
  const tb = document.getElementById('tbody');
  tb.innerHTML = '';
  rows.forEach(row => {
    const pct  = row.pres / i * 100;
    const fw   = Math.min(pct, 100);
    const fc   = COLORS[row.tipo] || COLORS.OTRO;
    const diff = row.pres - row.gast;
    const dc   = diff >= 0 ? 'pos' : 'neg';
    const ds   = diff >= 0 ? '+' : '';
    const tr   = document.createElement('tr');
    tr.innerHTML = `
      <td><select onchange="onField(${row.id},'tipo',this.value)">
        ${TIPOS.map(t=>`<option value="${t}"${t===row.tipo?' selected':''}>${t}</option>`).join('')}
      </select></td>
      <td><input class="ti" type="text" value="${row.concepto}" placeholder="Concepto"
        oninput="onField(${row.id},'concepto',this.value)"></td>
      <td class="r"><input class="ni" id="pres-${row.id}" type="number" value="${row.pres.toFixed(2)}" step="0.01"
        oninput="onField(${row.id},'pres',this.value)"></td>
      <td><div class="pct-bar">
        <input class="ni" id="pct-${row.id}" type="number" value="${pct.toFixed(1)}" step="0.1" style="width:56px"
          oninput="onField(${row.id},'presFromPct',this.value)">
        <span style="font-size:10px;color:var(--muted)">%</span>
        <div class="pct-track"><div class="pct-fill" id="pf-${row.id}" style="width:${fw}%;background:${fc}"></div></div>
      </div></td>
      <td class="r gauto" id="gast-${row.id}">${fmt(row.gast)}</td>
      <td class="r ${dc}" id="diff-${row.id}">${ds}${fmt(diff)}</td>
      <td><button class="del-btn" onclick="delRow(${row.id})">×</button></td>`;
    tb.appendChild(tr);
  });
}

function recalc() {
  const i       = getIng();
  const totPres = rows.reduce((s,r) => s + r.pres, 0);
  const totGast = rows.reduce((s,r) => s + r.gast, 0);
  const meQueda = i - totGast;
  let   overrun = 0;
  rows.forEach(r => { if (r.gast > r.pres) overrun += r.gast - r.pres; });
  const ahorros = meQueda - overrun;

  document.getElementById('pres-total').textContent  = fmt(totPres);
  document.getElementById('gastado-val').textContent = fmt(totGast);
  document.getElementById('queda-val').textContent   = fmt(meQueda);
  document.getElementById('ahorros-val').textContent = fmt(ahorros);
  document.querySelector('.queda .val').style.color  = meQueda >= 0 ? 'var(--purple)' : 'var(--red)';
  document.querySelector('.sav .val').style.color    = ahorros >= 0 ? 'var(--green)'  : 'var(--red)';

  /* sync pct bars */
  rows.forEach(row => {
    const pi = document.getElementById('pct-' + row.id);
    if (pi && document.activeElement !== pi) pi.value = (row.pres / i * 100).toFixed(1);
    const pf = document.getElementById('pf-' + row.id);
    if (pf) { pf.style.width = Math.min(row.pres/i*100,100) + '%'; pf.style.background = COLORS[row.tipo]||COLORS.OTRO; }
  });

  buildTotals(totGast, ahorros);
  buildSavings(meQueda);

  /* pct total bar */
  const totalPct = rows.reduce((s, r) => s + (r.pres / i * 100), 0);
  const bar = document.getElementById('pct-total-bar');
  const lbl = document.getElementById('pct-total-label');
  if (bar && lbl) {
    const capped = Math.min(totalPct, 100);
    bar.style.width = capped + '%';
    const over = totalPct > 100;
    bar.style.background = over ? 'var(--red)' : totalPct > 85 ? 'var(--yellow)' : 'var(--green)';
    lbl.style.color = over ? 'var(--red)' : totalPct > 85 ? 'var(--yellow)' : 'var(--green)';
    lbl.textContent = totalPct.toFixed(1) + '%';
  }
}

function buildTotals(totGast, ahorros) {
  const cats = {};
  rows.forEach(r => { cats[r.tipo] = (cats[r.tipo]||0) + r.gast; });
  if (ahorros > 0) cats['AHORRO'] = ahorros;

  const total = Object.values(cats).reduce((a,b)=>a+b,0) || 1;
  const tb    = document.getElementById('totals-body');
  tb.innerHTML = '';

  const labels=[], data=[], colors=[];

  Object.keys(cats).forEach(k => {
    const v = cats[k], pct = v/total*100, col = COLORS[k]||COLORS.OTRO;
    labels.push(k); data.push(v); colors.push(col+'cc');
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><span class="tag ${k}">${k}</span></td>
      <td style="text-align:right;color:${col};font-weight:600;font-size:12px">${fmt(v)}</td>
      <td style="text-align:right;color:var(--muted);font-size:11px">${fmtPct(pct)}</td>`;
    tb.appendChild(tr);
  });

  buildChart(labels, data, colors, Object.values(cats).reduce((a,b)=>a+b,0));
}

function buildChart(labels, data, colors, total) {
  document.getElementById('donut-total').textContent = 'B/. ' + total.toFixed(0);

  /* Legend */
  const lg = document.getElementById('legend');
  lg.innerHTML = '';
  labels.forEach((l, i) => {
    const col = colors[i].replace('cc','');
    const pct = total > 0 ? (data[i]/total*100).toFixed(1) : '0.0';
    lg.innerHTML += `<div class="li"><div class="ld" style="background:${col}"></div><span class="ln">${l}</span><span class="lp">${pct}%</span></div>`;
  });

  /* Chart */
  if (!chart) {
    chart = new DonutChart(document.getElementById('donut'), data, colors);
  } else {
    chart.update(data, colors);
  }
}

function buildSavings(meQueda) {
  const g = document.getElementById('sav-grid');
  g.innerHTML = '';
  rows.forEach(r => {
    const over = r.gast - r.pres;
    if (over > 0) g.innerHTML += `<div class="si over">
      <div class="sl">${r.tipo} · ${r.concepto||'—'}</div>
      <div class="sv" style="color:var(--red)">+${fmt(over)}</div>
      <div class="ss">Se excedió del presupuesto</div></div>`;
  });
  g.innerHTML += `<div class="si" style="border-left-color:var(--purple)">
    <div class="sl">ME QUEDA</div>
    <div class="sv" style="color:var(--purple)">${fmt(meQueda)}</div>
    <div class="ss">Ingreso − total gastado</div></div>`;
}

/* Init */
renderTable();
recalc();

/* PWA install */
let deferredPrompt = null;
const banner = document.getElementById('install-banner');
window.addEventListener('beforeinstallprompt', e => {
  e.preventDefault(); deferredPrompt = e; banner.classList.add('show');
});
document.getElementById('ib-btn').addEventListener('click', async () => {
  if (!deferredPrompt) return;
  deferredPrompt.prompt();
  await deferredPrompt.userChoice;
  deferredPrompt = null; banner.classList.remove('show');
});
document.getElementById('id-btn').addEventListener('click', () => banner.classList.remove('show'));
window.addEventListener('appinstalled', () => banner.classList.remove('show'));

/* Inline service worker via blob URL */
if ('serviceWorker' in navigator) {
  const sw = `const C='pres-v1';
self.addEventListener('install',e=>{e.waitUntil(caches.open(C).then(c=>c.addAll([location.pathname])).then(()=>self.skipWaiting()));});
self.addEventListener('activate',e=>{self.clients.claim();});
self.addEventListener('fetch',e=>{e.respondWith(caches.match(e.request).then(r=>r||fetch(e.request)));});`;
  const blob = new Blob([sw], {type:'text/javascript'});
  navigator.serviceWorker.register(URL.createObjectURL(blob)).catch(()=>{});
}
</script>
</body>
</html>
