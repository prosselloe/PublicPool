# Public Pool Dashboard v5 — Memòria de miners
<!DOCTYPE html>
<html lang="ca">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
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
</body>
</html>
