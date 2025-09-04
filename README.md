# baccarat-predictor3[baccarat_CL4_0_fix5_strict_logical.html](https://github.com/user-attachments/files/22140052/baccarat_CL4_0_fix5_strict_logical.html)
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>百家樂 CL4.0（fix5：嚴格分流預測/紀錄）</title>
<style>
  :root{--bg:#0a0f16;--panel:#111826;--text:#e8f0fb;--muted:#9fb3c8;--border:#223248;--card:#0f1622;
  --banker:#e74c3c;--player:#3498db;--tie:#2ecc71;--pos:#7bd389;--neg:#ff6b6b;--warn:#ffd166;--patternRed:#ff6b6b;--patternBlue:#4da3ff}
  *{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);
  font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Noto Sans TC","PingFang TC","Microsoft JhengHei",Arial,sans-serif}
  .container{max-width:1240px;margin:26px auto;padding:0 16px}.title{font-weight:800;font-size:26px}.subtitle{color:var(--muted);margin:6px 0 16px}
  .panel{background:linear-gradient(180deg,var(--panel),#0c1320);border:1px solid var(--border);border-radius:16px;padding:16px;margin-bottom:14px}
  .row{display:flex;gap:12px;flex-wrap:wrap}.col{flex:1;min-width:260px}
  .btn{border:1px solid var(--border);background:var(--card);color:var(--text);padding:10px 14px;border-radius:12px;cursor:pointer;font-weight:800}
  .btn:hover{filter:brightness(1.05)}.btn-danger{background:#2a1212;border-color:#6d1c1c}
  .btn-b{background:color-mix(in oklab,var(--banker) 18%,#000)}.btn-p{background:color-mix(in oklab,var(--player) 18%,#000)}
  .btn-t{background:color-mix(in oklab,var(--tie) 18%,#000);color:#dff7e6}
  .card{background:var(--card);border:1px solid var(--border);border-radius:14px;padding:12px}
  .kpi{font-size:24px;font-weight:800}.kpi-sub{font-size:12px;color:var(--muted)}
  .bar{height:10px;background:#0b1220;border:1px solid var(--border);border-radius:999px;overflow:hidden;margin-top:8px}
  .bar>div{height:100%}.b>div{background:var(--banker)}.p>div{background:var(--player)}.t>div{background:var(--tie)}
  .hist{display:flex;gap:6px;flex-wrap:wrap}.pill{padding:6px 10px;border-radius:999px;font-weight:700;border:1px solid transparent}
  .pill-b{color:var(--banker);border-color:var(--banker)}.pill-p{color:var(--player);border-color:var(--player)}.pill-t{color:var(--tie);border-color:var(--tie)}
  .table{width:100%;border-collapse:collapse}.table th,.table td{border-bottom:1px dashed var(--border);padding:8px 6px;text-align:right;font-family:ui-monospace,Menlo,Consolas,monospace}
  .table th:first-child,.table td:first-child{text-align:left}
  .toast{position:fixed;right:18px;bottom:18px;background:#0f1a28;border:1px solid #244160;color:#cfe6ff;padding:10px 14px;border-radius:10px;opacity:0;transform:translateY(10px);transition:all .2s ease}
  .toast.show{opacity:1;transform:translateY(0)}
  .road{display:grid;grid-template-columns:repeat(40,18px);grid-auto-rows:18px;gap:3px;padding:8px;background:#0b1220;border:1px solid var(--border);border-radius:10px;overflow:auto}
  .cell{width:18px;height:18px;border-radius:50%}.cell-b{background:var(--banker)}.cell-p{background:var(--player)}.cell-t{background:var(--tie)}
  .d-cell{width:14px;height:14px;border-radius:50%;border:2px solid transparent}.fill-red{background:var(--patternRed)}.fill-blue{background:var(--patternBlue)}
  .ring-b{border-color:var(--banker)}.ring-p{border-color:var(--player)}
</style>
</head>
<body>
<div class="container">
  <div class="title">百家樂 CL4.0（fix5：嚴格分流預測/紀錄）</div>
  <div class="subtitle">預測只顯示；只有回報「本局開…」時才寫入紀錄與盈虧，並把結果追加到路單。</div>

  <div class="panel">
    <div class="row">
      <div class="col">
        <label>快速輸入（最新在右）</label>
        <div class="row" style="gap:10px">
          <button class="btn btn-b" onclick="quick('B')">＋莊 (B)</button>
          <button class="btn btn-p" onclick="quick('P')">＋閒 (P)</button>
          <button class="btn btn-t" onclick="quick('T')">＋和 (T)</button>
          <button class="btn btn-danger" onclick="clearAll()">清空</button>
        </div>
        <div class="card" style="margin-top:12px">
          <div style="font-weight:700;margin-bottom:6px;">彩色歷史</div>
          <div id="seq" class="hist">—</div>
        </div>
      </div>
      <div class="col">
        <label>平滑強度 S / 最近 N 局（0=全部）</label>
        <div class="row" style="gap:10px">
          <input id="smooth" class="col" type="number" min="0" step="0.1" value="3">
          <input id="window" class="col" type="number" min="0" step="1" value="0">
        </div>
        <label style="margin-top:10px">策略與風控</label>
        <div><input id="hardLock" type="checkbox"> 硬鎖：每局必出方向</div>
        <div><input id="neverTie" type="checkbox"> 永不建議 Tie</div>
        <label style="margin-top:8px">門檻 θ（未勾硬鎖時生效）</label>
        <input id="theta" type="number" min="0" max="1" step="0.01" value="0.52">
      </div>
      <div class="col">
        <label>批量輸入（空白/逗號/換行分隔；B/P/T 或 莊/閒/和）</label>
        <textarea id="history" rows="7" placeholder="例：B P P T B P"></textarea>
        <div style="margin-top:8px">
          <button class="btn" onclick="syncFromTextarea()">同步</button>
        </div>
      </div>
    </div>
  </div>

  <div class="panel">
    <div class="row">
      <div class="col">
        <div class="row">
          <div class="col">
            <div class="card">
              <div class="kpi" id="kB">--%</div>
              <div class="kpi-sub">下一局機率：莊</div>
              <div class="bar b"><div id="barB" style="width:0%"></div></div>
            </div>
          </div>
          <div class="col">
            <div class="card">
              <div class="kpi" id="kP">--%</div>
              <div class="kpi-sub">下一局機率：閒</div>
              <div class="bar p"><div id="barP" style="width:0%"></div></div>
            </div>
          </div>
          <div class="col">
            <div class="card">
              <div class="kpi" id="kT">--%</div>
              <div class="kpi-sub">下一局機率：和</div>
              <div class="bar t"><div id="barT" style="width:0%"></div></div>
            </div>
          </div>
        </div>
      </div>
      <div class="col">
        <div class="card">
          <div class="kpi" id="rec">—</div>
          <div class="kpi-sub" id="explain">—</div>
          <div style="margin-top:10px">
            <button class="btn" onclick="recalc()">🔄 重新計算預測</button>
          </div>
        </div>
        <div class="row" style="margin-top:10px;gap:10px">
          <div class="col">
            <label>本金</label>
            <input id="bankroll" type="number" min="0" step="100" value="100000">
          </div>
          <div class="col">
            <label>凱利比例（0~1）</label>
            <input id="kellyFrac" type="number" min="0" max="1" step="0.05" value="0.25">
          </div>
        </div>
        <div class="row" style="margin-top:10px;gap:10px">
          <div class="col">
            <label>Banker 佣金</label>
            <input id="commission" type="number" min="0" max="0.2" step="0.01" value="0.05">
          </div>
          <div class="col">
            <label>Tie 賠率（8/9）</label>
            <input id="tiePayout" type="number" min="0" max="50" step="1" value="8">
          </div>
        </div>
        <div class="row" style="margin-top:10px;gap:10px">
          <div class="col"><button class="btn btn-b" onclick="oneClickResult('B')">本局開 莊 (B)</button></div>
          <div class="col"><button class="btn btn-p" onclick="oneClickResult('P')">本局開 閒 (P)</button></div>
          <div class="col"><button class="btn btn-t" onclick="oneClickResult('T')">本局開 和 (T)</button></div>
        </div>
      </div>
      <div class="col">
        <div class="card">
          <table class="table">
            <thead><tr><th>方向</th><th>勝率 p</th><th>EV</th><th>Kelly%</th><th>建議金額</th></tr></thead>
            <tbody id="tbl"></tbody>
          </table>
        </div>
      </div>
    </div>
  </div>

  <div class="panel">
    <div class="row">
      <div class="col">
        <div class="row">
          <div class="col card"><div class="kpi" id="pnl">0</div><div class="kpi-sub">累計盈虧</div></div>
          <div class="col card"><div class="kpi" id="wr">--%</div><div class="kpi-sub">勝率（不含 Push）</div></div>
          <div class="col card"><div class="kpi" id="streakW">0</div><div class="kpi-sub">最大連贏</div></div>
          <div class="col card"><div class="kpi" id="streakL">0</div><div class="kpi-sub">最大連輸</div></div>
        </div>
      </div>
      <div class="col">
        <div class="card" style="max-height:260px;overflow:auto">
          <div style="font-weight:700;margin-bottom:6px;">紀錄（含實際輸贏）</div>
          <table class="table" id="logTable">
            <thead>
              <tr>
                <th>#</th><th>時間</th><th>歷史</th><th>預測方向</th><th>金額</th>
                <th>實際</th><th>單局盈虧</th><th>狀態</th>
                <th>pB</th><th>pP</th><th>pT</th><th>參數</th>
              </tr>
            </thead>
            <tbody id="logBody"></tbody>
          </table>
          <div class="row" style="margin-top:8px;gap:10px;align-items:center">
            <div><button class="btn" onclick="downloadCSV()">下載 CSV</button> <span class="kpi-sub" id="logCount">0 筆</span></div>
          </div>
        </div>
      </div>
      <div class="col"><div class="kpi-sub">合規提醒：研究用途，不保證獲利；請自律控風險。</div></div>
    </div>
  </div>

  <div class="panel">
    <div class="row" style="gap:12px">
      <div class="col"><div class="card"><div class="kpi-sub" style="font-weight:700">大路</div><div id="roadBig" class="road"></div></div></div>
      <div class="col"><div class="card"><div class="kpi-sub" style="font-weight:700">大眼仔</div><div id="roadEye" class="road"></div></div></div>
      <div class="col"><div class="card"><div class="kpi-sub" style="font-weight:700">小路</div><div id="roadSmall" class="road"></div></div></div>
      <div class="col"><div class="card"><div class="kpi-sub" style="font-weight:700">曱甴路</div><div id="roadCock" class="road"></div></div></div>
    </div>
  </div>
</div>

<div id="toast" class="toast">已重新計算預測</div>

<script>
  const ROWS=6, COLS=40;
  const base={B:0.458597,P:0.446247,T:0.095156};
  let logs=[], cumPnL=0, maxWinStreak=0, maxLoseStreak=0, curWinStreak=0, curLoseStreak=0;
  // 持有「上一次預測快照」（不落地、不寫表；僅在回報結果時才使用）
  let pending = null; // {side:'B'|'P'|'T'|'NONE', amt:number, probs:{B,P,T}, params:'...'}

  function showToast(msg){ const t=document.getElementById('toast'); t.textContent=msg; t.classList.add('show'); setTimeout(()=>t.classList.remove('show'),1200); }

  function parseHistory(txt){
    if(!txt) return [];
    const arr = txt.trim().split(/[\s,，、]+/).filter(Boolean);
    return arr.map(t=>{ const u=t.toUpperCase();
      if(u==='B'||t==='莊')return'B'; if(u==='P'||t==='閒'||t==='闲')return'P'; if(u==='T'||t==='和')return'T'; return null;
    }).filter(Boolean);
  }
  function stringifyHistory(arr){ return arr.length? arr.join(' ') : ''; }
  function renderSeq(hist){ const w=document.getElementById('seq'); w.innerHTML=''; if(!hist.length){w.textContent='—';return;}
    for(const x of hist){const s=document.createElement('span'); s.className='pill '+(x==='B'?'pill-b':x==='P'?'pill-p':'pill-t'); s.textContent=(x==='B'?'莊(B)':x==='P'?'閒(P)':'和(T)'); w.appendChild(s);} }
  function getHistory(){ return parseHistory(document.getElementById('history').value); }
  function setHistory(arr){ document.getElementById('history').value=stringifyHistory(arr); renderSeq(arr); renderAllRoads(arr); }
  function quick(tok){ const cur=getHistory(); cur.push(tok); setHistory(cur); }
  function clearAll(){ setHistory([]); setBars({B:0,P:0,T:0}); document.getElementById('rec').textContent='—'; document.getElementById('explain').textContent='—'; document.getElementById('tbl').innerHTML='';
    logs=[]; cumPnL=0; maxWinStreak=maxLoseStreak=curWinStreak=curLoseStreak=0; pending=null; renderLog(); renderSummary(); }
  function syncFromTextarea(){ const cur=getHistory(); setHistory(cur); }

  function windowed(hist,n){ if(!n||n<=0||n>=hist.length) return hist.slice(); return hist.slice(hist.length-n); }
  function buildTrans(hist){ const T={B:{B:0,P:0,T:0},P:{B:0,P:0,T:0},T:{B:0,P:0,T:0}}; for(let i=0;i<hist.length-1;i++) T[hist[i]][hist[i+1]]++; return T; }
  function dirichletBlend(cnt,S){ const aB=base.B*S,aP=base.P*S,aT=base.T*S; const s=cnt.B+cnt.P+cnt.T+aB+aP+aT; return {B:(cnt.B+aB)/s,P:(cnt.P+aP)/s,T:(cnt.T+aT)/s}; }
  function predictCore(hist,S,N){ const h2=windowed(hist,N); if(h2.length===0) return {...base,last:null,n:0}; const last=h2[h2.length-1]; const probs=dirichletBlend(buildTrans(h2)[last],S); return {...probs,last,n:h2.length}; }
  function setBars(p){ k('kB',p.B); k('kP',p.P); k('kT',p.T); w('barB',p.B); w('barP',p.P); w('barT',p.T);
    function k(id,x){document.getElementById(id).textContent=(x*100).toFixed(2)+'%'} function w(id,x){document.getElementById(id).style.width=(x*100).toFixed(1)+'%'} }
  function evAndKelly(pB,pP,pT,comm,tie,bank,frac){ const out={};
    const evb=pB*(1-comm)+(1-pB-pT)*(-1),varb=Math.max(0,pB*(1-comm)*(1-comm)+(1-pB-pT)-evb*evb),kB=varb>1e-12?Math.max(0,Math.min(1,(evb/varb)*frac)):0;
    const evp=pP*1+(1-pP-pT)*(-1),varp=Math.max(0,pP*1+(1-pP-pT)-evp*evp),kP=varp>1e-12?Math.max(0,Math.min(1,(evp/varp)*frac)):0;
    const evt=pT*tie+(1-pT)*(-1),vart=Math.max(0,pT*tie*tie+(1-pT)-evt*evt),kT=vart>1e-12?Math.max(0,Math.min(1,(evt/vart)*frac)):0;
    out.B={p:pB,EV:evb,Var:varb,kelly:kB,amt:bank*kB}; out.P={p:pP,EV:evp,Var:varp,kelly:kP,amt:bank*kP}; out.T={p:pT,EV:evt,Var:vart,kelly:kT,amt:bank*kT}; return out; }
  function decide(pB,pP,pT,hard,neverTie,theta){ let best='B',bestP=pB; if(!neverTie){ if(pP>bestP){best='P';bestP=pP;} if(pT>bestP){best='T';bestP=pT;} } else { if(pP>bestP){best='P';bestP=pP;} }
    if(hard) return {side:best,conf:bestP,why:'硬鎖：取最大機率'}; if(bestP>=theta) return {side:best,conf:bestP,why:'達門檻 θ='+theta}; return {side:'NONE',conf:bestP,why:'未達門檻，觀望'}; }
  function fmtPct(x){return (x*100).toFixed(2)+'%'} function fmtEV(x){return (x>=0?'+':'')+x.toFixed(4)} function fmtAmt(x){return Math.round(x).toLocaleString('zh-Hant-TW')}

  function recalc(){
    const S=parseFloat(document.getElementById('smooth').value)||3, N=parseInt(document.getElementById('window').value||'0');
    const hard=document.getElementById('hardLock').checked, neverTie=document.getElementById('neverTie').checked;
    const theta=parseFloat(document.getElementById('theta').value)||0.52, bank=parseFloat(document.getElementById('bankroll').value)||0;
    const frac=Math.max(0,Math.min(1,parseFloat(document.getElementById('kellyFrac').value)||0)), comm=Math.max(0,parseFloat(document.getElementById('commission').value)||0.05);
    const tie=Math.max(0,parseFloat(document.getElementById('tiePayout').value)||8);
    const hist=getHistory(); const out=predictCore(hist,S,N); setBars(out);
    const stats=evAndKelly(out.B,out.P,out.T,comm,tie,bank,frac);
    const dec=decide(out.B,out.P,out.T,hard,neverTie,theta), kinfo= dec.side==='NONE'? {EV:0,kelly:0,amt:0} : (dec.side==='B'?stats.B:dec.side==='P'?stats.P:stats.T);
    document.getElementById('rec').textContent = (dec.side==='NONE'?'建議：觀望':'建議：'+(dec.side==='B'?'莊 (B)':dec.side==='P'?'閒 (P)':'和 (T)'));
    document.getElementById('explain').innerHTML = `N=${out.n}；pB/pP/pT=<b>${fmtPct(out.B)}</b>/<b>${fmtPct(out.P)}</b>/<b>${fmtPct(out.T)}</b>；理由：${dec.why}；EV=<b>${fmtEV(kinfo.EV)}</b>，Kelly=<b>${fmtPct(kinfo.kelly)}</b>，建議金額≈<b>${fmtAmt(kinfo.amt)}</b>`;
    // 只存成「待用快照」，不紀錄、不結算
    pending = { side: dec.side, amt: Math.round(kinfo.amt), probs:{B:out.B,P:out.P,T:out.T},
      params:`S=${S},N=${N},θ=${theta},佣=${comm},Tie=${tie},Frac=${frac}` };
    showToast('已重新計算預測（未紀錄，等待實際結果）');
  }

  function oneClickResult(actual){
    // 若尚未有待用快照，先算一次
    if (!pending) recalc();
    const hist=getHistory();
    const now=new Date();
    const row={ idx:logs.length+1, time:now.toLocaleString(), hist:hist.join(' '),
      side: pending.side, sideText: pending.side==='NONE'?'觀望':(pending.side==='B'?'莊 (B)':pending.side==='P'?'閒 (P)':'和 (T)'),
      amt: pending.amt||0, actual:actual, pnl:0, status:'',
      pB:pending.probs.B, pP:pending.probs.P, pT:pending.probs.T, params: pending.params };
    // 結算盈虧（若 side=NONE 或 amt=0，視為 NOBET）
    computePnlForRow(row, getComm(), getTie());
    logs.push(row); renderLog(); renderSummary();
    // 把實際結果寫入路單
    const next=hist.slice(); next.push(actual); setHistory(next);
    // 清空 pending，避免重用上一局預測
    pending = null;
  }

  function getComm(){ return Math.max(0,parseFloat(document.getElementById('commission').value)||0.05); }
  function getTie(){ return Math.max(0,parseFloat(document.getElementById('tiePayout').value)||8); }

  function computePnlForRow(row,comm,tie){ const side=row.side, amt=row.amt||0, act=row.actual;
    if(!act || !side || side==='NONE' || amt<=0){ row.status='NOBET'; return; }
    let pnl=0,status=''; if(side==='B'){ if(act==='B'){pnl=amt*(1-comm);status='WIN';} else if(act==='T'){pnl=0;status='PUSH';} else {pnl=-amt;status='LOSS';} }
    else if(side==='P'){ if(act==='P'){pnl=amt;status='WIN';} else if(act==='T'){pnl=0;status='PUSH';} else {pnl=-amt;status='LOSS';} }
    else if(side==='T'){ if(act==='T'){pnl=amt*tie;status='WIN';} else {pnl=-amt;status='LOSS';} }
    row.pnl=pnl; row.status=status; if(status==='WIN'){curWinStreak++;maxWinStreak=Math.max(maxWinStreak,curWinStreak);curLoseStreak=0;}
    else if(status==='LOSS'){curLoseStreak++;maxLoseStreak=Math.max(maxLoseStreak,curLoseStreak);curWinStreak=0;} cumPnL+=pnl; }

  function renderSummary(){ const el=document.getElementById('pnl'); el.textContent=Math.round(cumPnL).toLocaleString('zh-Hant-TW'); el.className='kpi '+(cumPnL>=0?'pos':'neg');
    let w=0,l=0; for(const r of logs){ if(r.status==='WIN') w++; else if(r.status==='LOSS') l++; } const wr=(w+l)>0? (w/(w+l))*100:0; 
    document.getElementById('wr').textContent=wr.toFixed(2)+'%'; document.getElementById('streakW').textContent=maxWinStreak; document.getElementById('streakL').textContent=maxLoseStreak; }

  function renderLog(){ const body=document.getElementById('logBody'); body.innerHTML=logs.map(r=>`<tr>
      <td>${r.idx}</td><td>${r.time}</td><td>${r.hist}</td><td>${r.sideText}</td><td>${Math.round(r.amt).toLocaleString('zh-Hant-TW')}</td>
      <td>${r.actual||'—'}</td><td>${r.status==='NOBET'?'—':(r.pnl>=0? '+'+Math.round(r.pnl):Math.round(r.pnl))}</td><td>${statusTag(r.status)}</td>
      <td>${(r.pB*100).toFixed(2)}%</td><td>${(r.pP*100).toFixed(2)}%</td><td>${(r.pT*100).toFixed(2)}%</td><td>${r.params}</td></tr>`).join('');
    document.getElementById('logCount').textContent=`${logs.length} 筆`; }
  function statusTag(s){ if(s==='WIN')return'<span class="tag" style="color:var(--pos)">WIN</span>'; if(s==='LOSS')return'<span class="tag" style="color:var(--neg)">LOSS</span>';
    if(s==='PUSH')return'<span class="tag" style="color:var(--warn)">PUSH</span>'; if(s==='NOBET')return'<span class="tag">NOBET</span>'; return ''; }
  function downloadCSV(){ if(!logs.length){alert('目前沒有紀錄可匯出');return;} const header=['#','時間','歷史','預測方向','金額','實際','單局盈虧','狀態','pB','pP','pT','參數'];
    const lines=[header.join(',')].concat(logs.map(r=>[ r.idx,r.time,`"${r.hist}"`,r.sideText,r.amt,r.actual||'', r.status==='NOBET'?'':Math.round(r.pnl), r.status, r.pB.toFixed(6),r.pP.toFixed(6),r.pT.toFixed(6), `\"${r.params}\"` ].join(',')));
    const blob=new Blob([lines.join('\\n')],{type:'text/csv;charset=utf-8;'}); const url=URL.createObjectURL(blob);
    const a=document.createElement('a'); a.href=url; a.download='baccarat_CL4_0_log.csv'; document.body.appendChild(a); a.click(); document.body.removeChild(a); URL.revokeObjectURL(url); }

  function buildBigRoad(hist){ const grid=Array.from({length:ROWS},()=>Array(COLS).fill(null)); const ties={}; let col=-1,row=0,last=null; const entries=[];
    for(let i=0;i<hist.length;i++){ const x=hist[i]; if(x==='T'){ if(col>=0&&grid[row][col]){const k=row+','+col; ties[k]=(ties[k]||0)+1;} continue; }
      if(last===null){ col=0; row=0; grid[row][col]=x; entries.push({c:col,r:row,side:x}); last=x; continue; }
      if(x===last){ if(row+1<ROWS && !grid[row+1][col]) row++; else { col++; } }
      else { col++; row=0; last=x; }
      if(col>=COLS) break; grid[row][col]=x; entries.push({c:col,r:row,side:x}); }
    return {grid,ties,entries}; }
  function colHeight(grid,c){ let h=0; for(let r=0;r<ROWS;r++){ if(grid[r][c]) h++; else break; } return h; }
  function exists(grid,c,r){ if(c<0||r<0||c>=COLS||r>=ROWS) return false; return !!grid[r][c]; }
  function buildDerivedFromBig(big,offsetK){ const d=Array.from({length:ROWS},()=>Array(COLS).fill(null)); let dc=-1,dr=0,lastFill=null;
    for(const e of big.entries){ const c=e.c,r=e.r; if(c-offsetK<0) continue; let fill='B';
      if(r===0){ fill=(colHeight(big.grid,c-1)===colHeight(big.grid,c-1-offsetK))?'R':'B'; }
      else { fill=(exists(big.grid,c-1,r)===exists(big.grid,c-1-offsetK,r))?'R':'B'; }
      const changed=(lastFill!==fill); if(dc===-1){dc=0;dr=0;} else if(changed){dc++;dr=0;} else { if(dr+1<ROWS && !d[dr+1][dc]) dr++; else dc++; }
      if(dc>=COLS) break; d[dr][dc]={fill,ring:(e.side==='B'?'B':'P')}; lastFill=fill; }
    return d; }
  function renderAllRoads(hist){ const big=buildBigRoad(hist);
    const rb=document.getElementById('roadBig'); rb.innerHTML=''; for(let r=0;r<ROWS;r++){ for(let c=0;c<COLS;c++){ const v=big.grid[r][c]; const el=document.createElement('div');
      el.className='cell'+(v?(' cell-'+(v==='B'?'b':v==='P'?'p':'t')):''); rb.appendChild(el);} }
    for(const key in big.ties){ const [r,c]=key.split(',').map(Number), idx=r*COLS+c; const node=rb.children[idx]; if(node){ const dot=document.createElement('div');
      dot.style.width='6px'; dot.style.height='6px'; dot.style.borderRadius='50%'; dot.style.background='var(--tie)'; dot.style.margin='6px auto 0'; node.appendChild(dot);} }
    const eye=buildDerivedFromBig(big,1), small=buildDerivedFromBig(big,2), cock=buildDerivedFromBig(big,3);
    function renderD(g,id){ const el=document.getElementById(id); el.innerHTML=''; for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++){ const v=g[r][c]; const d=document.createElement('div'); d.className= v? ('d-cell '+(v.fill==='R'?'fill-red':'fill-blue')+' '+(v.ring==='B'?'ring-b':'ring-p')) : 'd-cell'; el.appendChild(d);} }
    renderD(eye,'roadEye'); renderD(small,'roadSmall'); renderD(cock,'roadCock'); }

  (function init(){ setHistory([]); })();
</script>
</body>
</html>
