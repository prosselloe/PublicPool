# PublicPool
<!DOCTYPE html>
<html lang="ca">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Public Pool Dashboard v5 — Memòria de miners</title>
<style>
:root{--bg:#0b1220;--card:#121c2e;--card2:#0f1727;--text:#e8eef8;--muted:#9aa9bd;--line:#26354d;--accent:#5eead4;--good:#4ade80;--warn:#fbbf24;--bad:#fb7185}
*{box-sizing:border-box}
body{margin:0;background:linear-gradient(135deg,#08101d,#111b2d);color:var(--text);font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif}
.wrap{max-width:1250px;margin:auto;padding:24px}
h1{margin:0 0 6px;font-size:28px}
h2{margin:0 0 12px;font-size:18px}
.sub{color:var(--muted);margin-bottom:20px}
.controls,.grid{display:grid;gap:14px}
.controls{grid-template-columns:1.5fr 2fr auto auto auto}
input,button{font:inherit;border-radius:10px;border:1px solid var(--line);padding:11px 13px}
input{background:#0b1424;color:var(--text);width:100%}
button{background:#18263c;color:var(--text);cursor:pointer}
button:hover{border-color:#3e587a}
.card{background:rgba(18,28,46,.94);border:1px solid var(--line);border-radius:15px;padding:17px;box-shadow:0 8px 28px #0004}
.grid{grid-template-columns:repeat(4,1fr);margin:16px 0}
.kpi{font-size:25px;font-weight:700;margin-top:5px}
.label{color:var(--muted);font-size:13px}
.status{display:inline-flex;align-items:center;gap:7px;padding:6px 10px;border-radius:999px;background:#0b1424;color:var(--muted);font-size:13px}
.dot{width:9px;height:9px;border-radius:50%;background:var(--muted)}
.good .dot{background:var(--good)} .warn .dot{background:var(--warn)} .bad .dot{background:var(--bad)}
.toolbar{display:flex;gap:10px;align-items:center;flex-wrap:wrap;margin-bottom:12px}
table{width:100%;border-collapse:collapse}
th,td{text-align:left;padding:10px;border-bottom:1px solid var(--line);font-size:14px}
th{color:var(--muted);font-weight:600}
.state{font-weight:700}
.state.online{color:var(--good)} .state.recent{color:var(--warn)} .state.offline{color:var(--bad)} .state.unknown{color:#cbd5e1}
pre{white-space:pre-wrap;word-break:break-word;max-height:330px;overflow:auto;background:#08101c;border:1px solid var(--line);padding:12px;border-radius:10px;color:#cbd5e1}
.small{font-size:12px;color:var(--muted)}
.warnbox{border-left:4px solid var(--warn);background:#241c0b;padding:12px;border-radius:8px;color:#f7e5b2}
.okbox{border-left:4px solid var(--good);background:#0b2116;padding:12px;border-radius:8px;color:#c9f7d7}
.errbox{border-left:4px solid var(--bad);background:#260e15;padding:12px;border-radius:8px;color:#ffd1d8}
@media(max-width:850px){.grid{grid-template-columns:repeat(2,1fr)}.controls{grid-template-columns:1fr 1fr}.controls input{grid-column:1/-1}}
@media(max-width:520px){.grid{grid-template-columns:1fr}.controls{grid-template-columns:1fr}.wrap{padding:13px}th:nth-child(3),td:nth-child(3){display:none}}

    #historyChart{background:rgba(255,255,255,.015);border-radius:10px;cursor:crosshair}
    .history-tooltip{position:fixed;display:none;pointer-events:none;z-index:9999;background:rgba(10,14,20,.96);border:1px solid #4a5568;border-radius:8px;padding:8px 10px;font-size:.82rem;line-height:1.35;box-shadow:0 4px 18px rgba(0,0,0,.35)}
  </style>
</head>
<body>
<div class="wrap">
  <h1>⛏️ Public Pool Dashboard</h1>
  <div class="sub">Memòria local dels miners + dades actuals de Public Pool</div>

  <h2>Pool API / Wallet / Controls</h2>
  <div class="card">
    <div class="controls">
      <input id="api" value="https://public-pool.io:40557/api" aria-label="API base">
      <input id="wallet" value="3NH6hkTmy7WcAL37LaUCbyZX1cDM4jXVzq" placeholder="Adreça Bitcoin">
      <button onclick="loadAll()">🔄 Actualitzar</button>
      <button onclick="clearMemory()">🗑️ Esborrar memòria</button>
      <button onclick="exportMemory()">💾 Exportar miners</button>
    </div>
    <div style="margin-top:12px" id="connection"><span class="status"><span class="dot"></span>Esperant...</span></div>
  </div>

  <div>&nbsp;</div>
  <h2>Network / Pool / Wallet Accounting</h2>
  <div class="grid">
    <div class="card"><div class="label">Network Hashrate</div><div class="kpi" id="networkHash">—</div>
      <div class="note">network.networkhashps</div></div>
    <div class="card"><div class="label">Network Difficulty</div><div class="kpi" id="networkDifficulty">—</div>
      <div class="note">network.difficulty</div></div>
    <div class="card"><div class="label">Block Height</div><div class="kpi" id="blockHeight">—</div>
      <div class="note">network.blocks</div></div>
    <div class="card"><div class="label">Current Block Weight</div><div class="kpi" id="blockWeight">—</div>
      <div class="note">network.currentblockweight · WU</div></div>
    <div class="card"><div class="label">Pool Hashrate</div><div class="kpi" id="poolHash">—</div>
      <div class="note">pool.totalHashRate</div></div>
    <div class="card"><div class="label">Pool Miners</div><div class="kpi" id="poolMiners">—</div>
      <div class="note">pool.totalMiners</div></div>
    <div class="card"><div class="label">Accepted Shares</div><div class="kpi" id="acceptedShares">—</div>
      <div class="note">accounting.totalAcceptedShares</div></div>
    <div class="card"><div class="label">Solo Work</div><div class="kpi" id="soloWork">—</div>
      <div class="note">accounting.totalCreditedDifficulty</div></div>
  </div>

  <h2>Client / Wallet</h2>
  <div class="grid">
    <!--
    <div class="card"><div class="label">Pool Hashrate</div><div class="kpi" id="hashrate">—</div></div>
    -->
    <div class="card"><div class="label">Best Difficulty històric</div><div class="kpi" id="difficulty">—</div></div>
    <div class="card"><div class="label">Workers visibles ara</div><div class="kpi" id="visible">—</div></div>
    <div class="card"><div class="label">Miners coneguts</div><div class="kpi" id="known">—</div></div>
    <div class="card"><div class="label">Darrera actualització</div><div class="kpi" id="updated">—</div></div>
  </div>

  <div id="interpretation" class="card" style="margin-bottom:16px"></div>

  <div class="card" style="margin-bottom:16px">
    <h2>Miners</h2>
    <div class="toolbar">
      <span class="small">La memòria es desa al navegador (localStorage). Uptime = temps des de <code>startTime</code> del worker.</span>
      <button onclick="setFilter('all')">Tots</button>
      <button onclick="setFilter('visible')">Visibles</button>
      <button onclick="setFilter('recent')">Recents</button>
      <button onclick="setFilter('offline')">No visibles</button>
    </div>
    <div style="overflow:auto">
      <table>
        <thead><tr><th>Worker</th><th>Estat</th><th style="text-align: right;">Hashrate actual</th>
               <th style="text-align: right;">Best difficulty actual</th><th>Uptime</th><th>Darrera aparició</th></tr></thead>
        <tbody id="workers"><tr><td colspan="6">Encara no hi ha dades.</td></tr></tbody>
      </table>
    </div>
  </div>

  <div class="card" style="margin-bottom:16px">
    <h2>Evolució històrica</h2>
    <div class="toolbar">
      <label>Element <select id="historyTarget"><option value="__wallet__">Cartera</option></select></label>
      <label>Magnitud <select id="historyMetric"><option value="hashrate">Hashrate (mitjana 10 min)</option><option value="bestDifficulty">Millor dificultat</option></select></label>
      <label>Període <select id="historyPeriod"><option value="3600000">1 hora</option><option value="21600000">6 hores</option><option value="86400000">24 hores</option><option value="604800000">7 dies</option><option value="2592000000">30 dies</option></select></label>
      <button id="clearHistory" type="button">🗑 Esborrar historial</button>
    </div>
    <div id="historyInfo" class="small">Recollint dades… · API cada 15 s, històric guardat com a màxim 1 mostra/minut durant 30 dies.</div>
    <div style="position:relative;width:100%;height:360px;margin-top:10px"><canvas id="historyChart" style="width:100%;height:100%;display:block"></canvas></div>
  </div>

  <div class="grid" style="grid-template-columns:repeat(3,1fr)">
    <div class="card"><h2>Pool</h2><pre id="pool">—</pre></div>
    <div class="card"><h2>Network</h2><pre id="network">—</pre></div>
    <div class="card"><h2>Client / Wallet</h2><pre id="client">—</pre></div>
  </div>

  <div class="card">
    <h2>ℹ️ Com interpretar-ho</h2>
    <div class="small">
      Aquest dashboard no inventa workers. Només afegeix a la memòria local els workers que
      Public Pool retorna realment. Si un worker deixa d'aparèixer a <code>/client/{wallet}</code>,
      es conserva com a miner conegut i passa a “No visible” segons el temps transcorregut.
      Això permet distingir entre “ha desaparegut de la resposta API” i “mai no l'hem vist”.
    </div>
  </div>
</div>

<script>
const API=document.getElementById('api').value.trim();
const KEY='publicPoolDashboardV5';
let filter='all';
let memory=loadMemory();

/*
 * Normalització SI (base 1000):
 * B, KB, MB, GB, TB, PB, EB...
 *
 * Per hashrate s'afegeix H/s:
 * H/s, kH/s, MH/s, GH/s, TH/s, PH/s, EH/s...
 *
 * IMPORTANT:
 * currentblockweight és Bitcoin Weight Units (WU), no bytes.
 * Per això es mostra com kWU, MWU, GWU... i no com KB/MB.
 */
function formatSI(value, unit="", decimals=2){
  const n=Number(value);
  if(!Number.isFinite(n)) return "—";
  const units=["","K","M","G","T","P","E","Z","Y"];
  let x=Math.abs(n), i=0;
  while(x>=1000 && i<units.length-1){x/=1000;i++;}
  const sign=n<0?"-":"";
  const d=x>=100?2:x>=10?2:decimals;
  return sign+x.toLocaleString("ca-ES",{minimumFractionDigits:0,maximumFractionDigits:d})+" "+units[i]+unit;
}
function formatBytes(value){ return formatSI(value,"B"); }
function formatHashrate(value){ return formatSI(value,"H/s"); }
function formatDifficulty(value){ return formatSI(value,""); }
function formatWeight(value){ return formatSI(value,"WU"); }
function formatInteger(value){
  const n=Number(value);
  return Number.isFinite(n)?n.toLocaleString("ca-ES"):String(value??"—");
}

function formatDate(ts){
  if(!ts)return '—';
  return new Date(ts).toLocaleString('ca-ES',{dateStyle:'short',timeStyle:'medium'});
}


function formatUptime(startTime, endTime=Date.now()){
  if(!startTime) return '—';
  const start = new Date(startTime).getTime();
  const end = endTime instanceof Date ? endTime.getTime() : Number(endTime);
  if(!Number.isFinite(start) || !Number.isFinite(end) || end < start) return '—';

  let seconds = Math.floor((end - start) / 1000);
  const days = Math.floor(seconds / 86400); seconds %= 86400;
  const hours = Math.floor(seconds / 3600); seconds %= 3600;
  const minutes = Math.floor(seconds / 60); seconds %= 60;

  const parts = [];
  if(days) parts.push(`${days} d`);
  if(hours || days) parts.push(`${hours} h`);
  if(minutes || hours || days) parts.push(`${minutes} min`);
  parts.push(`${seconds} s`);

  return parts.join(' ');
}

function loadMemory(){
  try{return JSON.parse(localStorage.getItem(KEY)||'{}')}catch(e){return {}}
}

function saveMemory(){localStorage.setItem(KEY,JSON.stringify(memory))}

function esc(v){
  return String(v??'').replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
}

function first(o, keys, fallback='—'){
  for(const k of keys){if(o && o[k]!==undefined && o[k]!==null) return o[k]}
  return fallback;
}

function num(v){return typeof v==='number'?v:(Number(v)||0)}

function workerName(w,i){
  return first(w,['name','worker','workerName','id','address'],`worker-${i+1}`);
}

function extractWorkers(data){
  if(Array.isArray(data))return data;
  for(const k of ['workers','miners','clients','devices']){
    if(Array.isArray(data?.[k]))return data[k];
  }
  if(data?.workers && typeof data.workers==='object') return Object.entries(data.workers).map(([name,v])=>({name,...(v||{})}));
  return [];
}

function classify(lastSeen, visibleNow){
  if(visibleNow)return ['online','🟢 ONLINE'];
  const age=Date.now()-lastSeen;
  if(age<120*60*1000)return ['recent','🟠 RECENT'];
  if(age<1440*60*1000)return ['offline','🔴 NO VISIBLE'];
  return ['unknown','⚪ NO VISIBLE > 24 h'];
}

function render(){
  const now=Date.now();
  const rows=[];
  for(const [name,w] of Object.entries(memory)){
    const visibleNow=!!w.visibleNow;
    const [cls,label]=classify(w.lastSeen,visibleNow);
    if(filter==='visible'&&!visibleNow)continue;
    if(filter==='recent'&&cls!=='recent')continue;
    if(filter==='offline'&&(visibleNow||cls==='recent'))continue;
    rows.push({name,w,cls,label});
  }
  rows.sort((a,b)=>b.w.lastSeen-a.w.lastSeen);
  document.getElementById('known').textContent=Object.keys(memory).length;
  document.getElementById('workers').innerHTML=rows.length?rows.map(r=>`
    <tr>
      <td><strong>${esc(r.name)}</strong></td>
      <td class="state ${r.cls}">${r.label}</td>
      <td style="text-align: right;">${esc(formatHashrate(r.w.hashrate))}</td>
      <td style="text-align: right;">${esc(formatDifficulty(r.w.bestDiff??'—'))}</td>
      <td>${esc(formatUptime(r.w.startTime, r.w.visibleNow ? Date.now() : r.w.lastSeen))}</td>
      <td>${esc(formatDate(r.w.lastSeen))}</td>
    </tr>`).join(''):`<tr><td colspan="6">Cap miner coincideix amb aquest filtre.</td></tr>`;
}

function setFilter(f){filter=f;render()}

function clearMemory(){
  if(confirm('Vols esborrar la memòria local de miners?')){
    memory={};saveMemory();render();
  }
}

function exportMemory(){
  const blob=new Blob([JSON.stringify(memory,null,2)],{type:'application/json'});
  const a=document.createElement('a');a.href=URL.createObjectURL(blob);
  a.download='public-pool-miners-v5.json';a.click();URL.revokeObjectURL(a.href);
}
/* ===== Històric de la gràfica ===== */
const HISTORY_KEY='publicPoolDashboardV5History';
const HISTORY_MAX_MS=30*24*60*60*1000;
const HISTORY_SAMPLE_MS=60*1000; // 15 s de polling, però 1 mostra/minut al localStorage.
const historyState={samples:[],lastSavedBucket:0};
function loadHistory(){try{const x=JSON.parse(localStorage.getItem(HISTORY_KEY)||'null');if(x&&Array.isArray(x.samples))historyState.samples=x.samples.filter(s=>s&&Number.isFinite(Number(s.t)));pruneHistory(Date.now(),false)}catch(e){console.warn('Històric:',e)}}
function saveHistory(){try{localStorage.setItem(HISTORY_KEY,JSON.stringify({version:1,samples:historyState.samples}))}catch(e){console.warn('No s’ha pogut guardar l’històric:',e)}}
function pruneHistory(now=Date.now(),save=true){const cut=now-HISTORY_MAX_MS;historyState.samples=historyState.samples.filter(s=>Number(s.t)>=cut);if(save)saveHistory()}
function recordHistory(clientData){
  const now=Date.now(),bucket=Math.floor(now/HISTORY_SAMPLE_MS)*HISTORY_SAMPLE_MS;
  const wallet={hashrate:Number(clientData?.accounting?.hashRateLast10Minutes),bestDifficulty:Number(clientData?.bestDifficulty)};
  const workers={};
  for(const w of (Array.isArray(clientData?.workers)?clientData.workers:[])){const id=String(w.sessionId??w.name??'').trim();if(!id)continue;workers[id]={name:String(w.name??id),hashrate:Number(w.hashRate??w.hashrate),bestDifficulty:Number(w.bestDifficulty??w.bestDiff)}}
  const sample={t:bucket,wallet,workers};const i=historyState.samples.findIndex(s=>Number(s.t)===bucket);
  if(i>=0)historyState.samples[i]=sample;else{historyState.samples.push(sample);historyState.samples.sort((a,b)=>Number(a.t)-Number(b.t))}
  pruneHistory(now,false);if(historyState.lastSavedBucket!==bucket){historyState.lastSavedBucket=bucket;saveHistory()}
}
function historyWorkerOptions(currentWorkers){
  const sel=document.getElementById('historyTarget');if(!sel)return;const prev=sel.value,map=new Map();
  for(const s of historyState.samples)for(const [id,w] of Object.entries(s.workers||{}))if(!map.has(id))map.set(id,w.name||id);
  for(const w of (currentWorkers||[])){const id=String(w.sessionId??w.name??'').trim();if(id&&!map.has(id))map.set(id,String(w.name??id))}
  const have=new Set([...sel.options].map(o=>o.value));for(const [id,name] of map){if(have.has(id))continue;const o=document.createElement('option');o.value=id;o.textContent=name;sel.appendChild(o)}
  if([...sel.options].some(o=>o.value===prev))sel.value=prev;
}
function chartValue(v,m){if(!Number.isFinite(v))return'—';return m==='hashrate'?formatHashrate(v):formatDifficulty(v)}
function drawHistoryChart(){
  const c=document.getElementById('historyChart'),info=document.getElementById('historyInfo');if(!c||!info)return;
  const target=document.getElementById('historyTarget')?.value||'__wallet__',metric=document.getElementById('historyMetric')?.value||'hashrate',period=Number(document.getElementById('historyPeriod')?.value||3600000),now=Date.now(),cut=now-period;
  const rows=[];for(const s of historyState.samples){const tt=Number(s.t);if(tt<cut||tt>now)continue;const o=target==='__wallet__'?s.wallet:s.workers?.[target];const v=Number(o?.[metric]);if(Number.isFinite(v))rows.push({t:tt,v})}
  const r=c.getBoundingClientRect(),dpr=devicePixelRatio||1,w=Math.max(320,Math.round(r.width)),h=Math.max(260,Math.round(r.height));c.width=w*dpr;c.height=h*dpr;const x=c.getContext('2d');x.setTransform(dpr,0,0,dpr,0,0);x.clearRect(0,0,w,h);
  const p={l:76,r:18,t:20,b:42},pw=w-p.l-p.r,ph=h-p.t-p.b;x.font='11px sans-serif';x.fillStyle=getComputedStyle(document.body).color;x.strokeStyle='rgba(128,128,128,.25)';
  if(rows.length<2){x.globalAlpha=.7;x.textAlign='center';x.font='14px sans-serif';x.fillText(rows.length?'Calen més mostres per dibuixar l’evolució.':'Encara no hi ha dades històriques.',w/2,h/2);x.globalAlpha=1;info.textContent=rows.length?'1 mostra · '+new Date(rows[0].t).toLocaleString('ca-ES'):'Recollint dades…';c._historyRows=rows;return}
  let lo=Math.min(...rows.map(a=>a.v)),hi=Math.max(...rows.map(a=>a.v));if(lo===hi){const q=Math.max(Math.abs(lo)*.05,1);lo-=q;hi+=q}else{const q=(hi-lo)*.08;lo-=q;hi+=q}
  const X=tt=>p.l+(tt-cut)/period*pw,Y=v=>p.t+(1-(v-lo)/(hi-lo))*ph;
  for(let i=0;i<=4;i++){const yy=p.t+ph*i/4;x.beginPath();x.moveTo(p.l,yy);x.lineTo(w-p.r,yy);x.stroke();x.textAlign='right';x.fillText(chartValue(hi-(hi-lo)*i/4,metric),p.l-7,yy+4)}
  const labels=w<600?4:6;for(let i=0;i<labels;i++){const xx=p.l+pw*i/(labels-1),tt=cut+period*i/(labels-1),d=new Date(tt);x.textAlign='center';x.fillText(period<=86400000?d.toLocaleTimeString('ca-ES',{hour:'2-digit',minute:'2-digit'}):d.toLocaleDateString('ca-ES',{day:'2-digit',month:'2-digit'}),xx,h-15)}
  x.beginPath();rows.forEach((a,i)=>i?x.lineTo(X(a.t),Y(a.v)):x.moveTo(X(a.t),Y(a.v)));x.strokeStyle=getComputedStyle(document.documentElement).getPropertyValue('--accent').trim()||'#58a6ff';x.lineWidth=2;x.stroke();
  if(rows.length<=240){x.fillStyle=x.strokeStyle;for(const a of rows){x.beginPath();x.arc(X(a.t),Y(a.v),2.5,0,Math.PI*2);x.fill()}}
  const vals=rows.map(a=>a.v),last=rows[rows.length-1];info.textContent=`${rows.length} mostres · darrera: ${chartValue(last.v,metric)} · mín.: ${chartValue(Math.min(...vals),metric)} · màx.: ${chartValue(Math.max(...vals),metric)}`;
  c._historyRows=rows;c._historyMeta={cut,period,p,pw,metric};
}
function installHistory(){
  for(const id of ['historyTarget','historyMetric','historyPeriod'])document.getElementById(id)?.addEventListener('change',drawHistoryChart);
  document.getElementById('clearHistory')?.addEventListener('click',()=>{if(confirm('Vols esborrar tot l’històric de la gràfica?')){historyState.samples=[];historyState.lastSavedBucket=0;localStorage.removeItem(HISTORY_KEY);drawHistoryChart()}});
  const c=document.getElementById('historyChart');if(c){const tip=document.createElement('div');tip.className='history-tooltip';document.body.appendChild(tip);c.addEventListener('mousemove',e=>{const rows=c._historyRows,m=c._historyMeta;if(!rows?.length||!m){tip.style.display='none';return}const rr=c.getBoundingClientRect(),px=e.clientX-rr.left,t=m.cut+(px-m.p.l)/m.pw*m.period;let best=rows[0];for(const a of rows)if(Math.abs(a.t-t)<Math.abs(best.t-t))best=a;const point=m.p.l+(best.t-m.cut)/m.period*m.pw;if(Math.abs(point-px)>18){tip.style.display='none';return}tip.innerHTML=`<strong>${new Date(best.t).toLocaleString('ca-ES')}</strong><br>${chartValue(best.v,m.metric)}`;tip.style.display='block';tip.style.left=e.clientX+12+'px';tip.style.top=e.clientY+12+'px'});c.addEventListener('mouseleave',()=>tip.style.display='none')}
  addEventListener('resize',drawHistoryChart)
}
loadHistory();installHistory();

async function get(path){
  const t=performance.now();
  const r=await fetch(API+path,{cache:'no-store'});
  const ms=Math.round(performance.now()-t);
  const text=await r.text();
  let data;try{data=JSON.parse(text)}catch{data={raw:text}}
  if(!r.ok)throw new Error(`HTTP ${r.status} (${ms} ms): ${text.slice(0,300)}`);
  return {data,ms,status:r.status};
}

function renderRaw(id,obj){document.getElementById(id).textContent=JSON.stringify(obj.data,null,2)}
async function loadAll(){
  const wallet=document.getElementById('wallet').value.trim();
  if(!wallet)return;
  const c=document.getElementById('connection');
  c.innerHTML='<span class="status"><span class="dot"></span>Consultant API...</span>';

  try {
    const [client,pool,network]=await Promise.all([
      get('/client/'+encodeURIComponent(wallet)),
      get('/pool'),
      get('/network')
    ]);
    renderRaw('client',client);renderRaw('pool',pool);renderRaw('network',network);

    const ws=extractWorkers(client.data);
    const now=Date.now();
    // Mark all known workers as not currently visible before applying this response.
    Object.values(memory).forEach(w=>w.visibleNow=false);

    ws.forEach((w,i)=>{
      const name=workerName(w,i);
      const old=memory[name]||{};
      memory[name]={
        ...old,
        name,
        hashrate:first(w,['hashrate','hashRate','totalHashrate','totalHashRate'],old.hashrate??'—'),
        bestDiff:first(w,['bestDifficulty','bestDiff','best_diff','bestShareDifficulty'],old.bestDiff??'—'),
        startTime:first(w,['startTime','start','startedAt'],old.startTime??null),
        lastSeen:now,
        visibleNow:true,
        raw:w
      };
    });
    saveMemory();
    recordHistory(client.data);
    historyWorkerOptions(ws);
    drawHistoryChart();

    document.getElementById('poolHash').textContent=formatHashrate(pool.data?.totalHashRate ?? '—');
    document.getElementById('poolMiners').textContent=formatInteger(pool.data?.totalMiners ?? '—');

    document.getElementById('blockHeight').textContent=formatInteger(network.data?.blocks ?? '—');
    document.getElementById('blockWeight').textContent=formatWeight(network.data?.currentblockweight ?? '—');
    document.getElementById('networkDifficulty').textContent=formatDifficulty(network.data?.difficulty ?? '—');
    document.getElementById('networkHash').textContent=formatHashrate(network.data?.networkhashps ?? '—');

    document.getElementById('acceptedShares').textContent=client.data.accounting?.totalAcceptedShares!=null
      ?formatInteger(client.data.accounting.totalAcceptedShares):"—";
    document.getElementById('soloWork').textContent=client.data.accounting?.totalCreditedDifficulty!=null
      ?formatDifficulty(client.data.accounting.totalCreditedDifficulty):"—";

    /*
    const h=first(client.data,['hashrate','hashRate','totalHashrate','totalHashRate'],
              first(pool.data,['hashrate','hashRate','totalHashrate','totalHashRate'],'—'));
    document.getElementById('hashrate').textContent=formatHashrate(h);
    */
    document.getElementById('difficulty').textContent=formatDifficulty(client.data?.bestDifficulty ?? '—');
    document.getElementById('visible').textContent=ws.length;
    document.getElementById('updated').textContent=new Date().toLocaleTimeString('ca-ES');

    if (ws.length===0) {
      document.getElementById('interpretation').innerHTML=
        `<div class="warnbox"><strong>⚠️ API sense workers visibles</strong><br>
        <span class="small">/client/{wallet} ha respost correctament, però aquesta resposta no conté workers.
        Els miners que ja coneixem es conserven a la memòria local i es mostren com a “NO VISIBLE”.
        Això no demostra que estiguin apagats.</span></div>`;
    } else {
      document.getElementById('interpretation').innerHTML=
        `<div class="okbox"><strong>✅ ${ws.length} worker(s) retornats per Public Pool</strong><br>
        <span class="small">Aquests són els miners que l'API ha fet visibles en aquesta consulta.
        Els miners anteriors que no apareixen es mantenen a la memòria.</span></div>`;
    }
    c.innerHTML=`<span class="status good"><span class="dot"></span>API connectada · client ${client.ms} ms · pool ${pool.ms} ms · network ${network.ms} ms</span>`;
    render();
  } catch(e) {
    c.innerHTML=`<span class="status bad"><span class="dot"></span>${esc(e.message)}</span>`;
    document.getElementById('interpretation').innerHTML=
      `<div class="errbox"><strong>❌ Error consultant Public Pool</strong><br><span class="small">${esc(e.message)}</span></div>`;
  }
}

render();
loadAll();
drawHistoryChart();
setInterval(loadAll,15000);
</script>
</body>
</html>
