<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Sales Dashboard</title>
  <link rel="preconnect" href="https://cdn.jsdelivr.net" crossorigin>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <style>
    :root{
      --bg:#F3FFFD;
      --card:#FFFFFF;
      --ink:#0F172A;
      --muted:#475569;
      --brand:#18C5BD;
      --shadow:0 10px 25px rgba(2,8,23,.08);
      --radius:18px;
    }
    *{box-sizing:border-box}
    html,body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.45 system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,"Helvetica Neue",Arial}
    a{color:var(--brand);text-decoration:underline;text-underline-offset:2px}
    .wrap{max-width:1280px;margin:32px auto;padding:0 20px}
    .head{display:flex;align-items:center;gap:18px;flex-wrap:wrap}
    .title{font-size:32px;font-weight:800}
    .mood{font-weight:700}
    .grid{display:grid;gap:18px}
    .grid.kpis{grid-template-columns:repeat(4,minmax(0,1fr))}
    .grid.charts{grid-template-columns:1fr}
    @media(min-width:1100px){.grid.charts{grid-template-columns:1fr 1fr}}
    @media(max-width:900px){.grid.kpis{grid-template-columns:repeat(2,minmax(0,1fr))}}
    .card{background:var(--card);border-radius:var(--radius);box-shadow:var(--shadow);padding:18px}
    .kpi-title{color:var(--muted);font-size:14px}
    .kpi-value{font-size:32px;font-weight:800;color:var(--brand)}
    .chart-title{font-size:18px;font-weight:800;margin:0 0 8px}
    .footer{opacity:.7;font-size:12px;margin-top:28px}
    .toolbar{display:flex;gap:10px;align-items:center}
    .btn{appearance:none;border:0;background:var(--brand);color:white;padding:10px 14px;border-radius:999px;font-weight:700;box-shadow:var(--shadow);cursor:pointer}
    .btn.alt{background:#e2e8f0;color:#0f172a}
    input[type="file"]{display:none}
    .file-label{cursor:pointer}
    .hint{font-size:12px;color:var(--muted)}
    code{background:#eef2f7;border-radius:6px;padding:2px 6px}
  </style>
</head>
<body>
  <div class="wrap">
    <header class="head">
      <div class="title">Sales Dashboard</div>
      <div class="mood">⭐ Let’s have a KILLER DAY!</div>
      <div class="toolbar">
        <a id="formLink" class="btn alt" href="#" target="_blank">Open sales form</a>
        <label class="btn file-label" for="file">Load CSV</label>
        <input id="file" type="file" accept=".csv" />
      </div>
    </header>

    <section class="grid kpis">
      <div class="card"><div class="kpi-title">Sales this month</div><div class="kpi-value" id="salesMonth">0</div></div>
      <div class="card"><div class="kpi-title">Sales today</div><div class="kpi-value" id="salesToday">0</div></div>
      <div class="card"><div class="kpi-title">Sales last week</div><div class="kpi-value" id="salesLastWeek">0</div></div>
      <div class="card"><div class="kpi-title">Sales yesterday</div><div class="kpi-value" id="salesYesterday">0</div></div>
      <div class="card"><div class="kpi-title">Sales this week</div><div class="kpi-value" id="salesThisWeek">0</div></div>
      <div class="card"><div class="kpi-title">DA budget total (this month)</div><div class="kpi-value" id="daBudgetMonth">0</div></div>
      <div class="card"><div class="kpi-title">DA budget total (last week)</div><div class="kpi-value" id="daBudgetLastWeek">0</div></div>
      <div class="card"><div class="kpi-title">Lead Pool Points (this week)</div><div class="kpi-value" id="lppThisWeek">0</div></div>
    </section>

    <section class="grid charts">
      <div class="card"><div class="chart-title">This Month's Sales by Sales Rep</div><canvas id="repMonth"></canvas></div>
      <div class="card"><div class="chart-title">Last Week's Sales by Sales Rep</div><canvas id="repLastWeek"></canvas></div>
      <div class="card"><div class="chart-title">Yesterday's Sales by Sales Rep</div><canvas id="repYesterday"></canvas></div>
      <div class="card"><div class="chart-title">Today's Sales by Sales Rep</div><canvas id="repToday"></canvas></div>
      <div class="card"><div class="chart-title">This Week's Sales by Sales Rep</div><canvas id="repThisWeek"></canvas></div>
      <div class="card"><div class="chart-title">This Month's Sales by Team</div><canvas id="teamMonth"></canvas></div>
      <div class="card"><div class="chart-title">Last Week's Sales by Team</div><canvas id="teamLastWeek"></canvas></div>
      <div class="card"><div class="chart-title">This Week's Sales by Team</div><canvas id="teamThisWeek"></canvas></div>
      <div class="card"><div class="chart-title">Lead Pool Points — Last Week</div><canvas id="lppLastWeek"></canvas></div>
      <div class="card"><div class="chart-title">Lead Pool Points — This Week</div><canvas id="lppWeek"></canvas></div>
    </section>

    <p class="footer" id="dataInfo">No data loaded yet. Drop a CSV or configure RAW_CSV_URL.</p>
  </div>

<script>
  const RAW_CSV_URL = "";
  const SALES_FORM_URL = "";
  document.getElementById('formLink').href = SALES_FORM_URL || '#';

  function parseCSV(text){
    const rows = [];
    let i=0, field='', row=[], inQuotes=false;
    while(i < text.length){
      const c = text[i];
      if(inQuotes){
        if(c === '"' && text[i+1] === '"'){ field+='"'; i++; }
        else if(c === '"'){ inQuotes=false; }
        else { field += c; }
      }else{
        if(c === '"'){ inQuotes = true; }
        else if(c === ','){ row.push(field); field=''; }
        else if(c === '\n'){ row.push(field); rows.push(row); row=[]; field=''; }
        else if(c !== '\r'){ field += c; }
      }
      i++;
    }
    if(field.length || row.length){ row.push(field); rows.push(row); }
    const headers = rows.shift().map(h=>h.trim());
    return rows.filter(r=>r.length===headers.length).map(r=>{
      const o={}; headers.forEach((h,idx)=>o[h]=r[idx]); return o;
    });
  }

  const startOfDay=d=>new Date(d.getFullYear(),d.getMonth(),d.getDate());
  const addDays=(d,n)=>{const x=new Date(d);x.setDate(x.getDate()+n);return x;};
  const startOfWeek=d=>{const x=startOfDay(d);const day=(x.getDay()+6)%7;x.setDate(x.getDate()-day);return x;};
  const endOfWeek=d=>addDays(startOfWeek(d),7);
  const startOfMonth=d=>new Date(d.getFullYear(),d.getMonth(),1);
  const endOfMonth=d=>new Date(d.getFullYear(),d.getMonth()+1,1);

  function toDate(s){const d=new Date(s);if(!isNaN(d))return d;return new Date();}

  function rowPoints(r){let pts=1;const pkg=(r['Package']||'').toLowerCase();if(pkg.includes('ultimate'))pts+=0.5;const da=parseFloat((r['DA budget']||'0').replace(/[^0-9.\-]/g,''))||0;if(da>0)pts+=2;const promo=(r['Promotion']||'').toLowerCase().trim();if(promo==='bogo')pts+=0.5;else if(promo==='3for1')pts+=0;else pts+=1;return pts;}
  function groupSum(rows,key){const m=new Map();rows.forEach(r=>{const k=(r[key]||'Unknown').trim();m.set(k,(m.get(k)||0)+1);});return[...m.entries()].sort((a,b)=>b[1]-a[1]);}
  function groupPoints(rows,key){const m=new Map();rows.forEach(r=>{const k=(r[key]||'Unknown').trim();m.set(k,(m.get(k)||0)+rowPoints(r));});return[...m.entries()].sort((a,b)=>b[1]-a[1]);}
  function drawBar(id,labels,vals){const c=document.getElementById(id);if(!c)return;if(c._chart)c._chart.destroy();c._chart=new Chart(c,{type:'bar',data:{labels,datasets:[{data:vals}]},options:{plugins:{legend:{display:false}},scales:{y:{beginAtZero:true}}}});}
  function setText(id,v){document.getElementById(id).textContent=Math.round(v*100)/100;}

  function computeAll(rows){
    const now=new Date();
    const t0=startOfDay(now),t1=addDays(t0,1);
    const y0=addDays(t0,-1),y1=t0;
    const w0=startOfWeek(now),w1=endOfWeek(now);
    const m0=startOfMonth(now),m1=endOfMonth(now);

    const data=rows.map(r=>({...r,__date:toDate(r['Timestamp'])}));
    const today=data.filter(r=>r.__date>=t0&&r.__date<t1);
    const yday=data.filter(r=>r.__date>=y0&&r.__date<y1);
    const thisW=data.filter(r=>r.__date>=w0&&r.__date<w1);
    const thisM=data.filter(r=>r.__date>=m0&&r.__date<m1);

    setText('salesMonth',thisM.length);
    setText('salesToday',today.length);
    setText('salesLastWeek',thisW.length);
    setText('salesYesterday',yday.length);
    setText('salesThisWeek',thisW.length);
    setText('daBudgetMonth',thisM.reduce((a,r)=>a+(parseFloat((r['DA budget']||'0').replace(/[^0-9.\-]/g,''))||0),0));
    setText('daBudgetLastWeek',thisW.reduce((a,r)=>a+(parseFloat((r['DA budget']||'0').replace(/[^0-9.\-]/g,''))||0),0));
    setText('lppThisWeek',thisW.reduce((a,r)=>a+rowPoints(r),0));

    const repThisW=groupSum(thisW,'Email Address').map(([k,v])=>[emailToName(k),v]);
    drawBar('repThisWeek',repThisW.map(x=>x[0]),repThisW.map(x=>x[1]));

    const teamThisW=groupSum(thisW,'Team');
    drawBar('teamThisWeek',teamThisW.map(x=>x[0]),teamThisW.map(x=>x[1]));
  }

  function emailToName(e){const l=(e||'').split('@')[0];if(!l)return'Unknown';return l.split('.').map(s=>s.charAt(0).toUpperCase()+s.slice(1)).join(' ');}

  document.getElementById('file').addEventListener('change',async e=>{const f=e.target.files?.[0];if(!f)return;const t=await f.text();computeAll(parseCSV(t));});
</script>
</body>
</html>
# Sales-board
