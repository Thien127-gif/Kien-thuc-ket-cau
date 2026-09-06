# Công cụ tính dầm liên tục 3 nhịp

<div class="beam-calc">
<style>
  .beam-calc{ --ink:#1c2b33; --paper:#f4f7f8; --panel:#ffffff; --line:#d7e0e4;
    --accent:#0f5c73; --accent2:#c1571f; --mono: 'IBM Plex Mono','Consolas',monospace;
    background:var(--paper); color:var(--ink); font-family:'Inter','Segoe UI',Arial,sans-serif;
    border:1px solid var(--line); border-radius:10px; padding:20px; margin:16px 0;}
  .beam-calc *{box-sizing:border-box;}
  .beam-calc .wrap{display:grid;grid-template-columns:280px 1fr;gap:18px;}
  .beam-calc .panel{background:var(--panel);border:1px solid var(--line);border-radius:8px;padding:16px;}
  .beam-calc .panel h2{font-size:11px;text-transform:uppercase;letter-spacing:.06em;color:var(--accent);
            margin:0 0 12px;border-bottom:1px solid var(--line);padding-bottom:6px;}
  .beam-calc .panel h2.sp{margin-top:16px;}
  .beam-calc .row{display:grid;grid-template-columns:1fr 1fr 1fr;gap:6px;margin-bottom:8px;}
  .beam-calc .field{display:flex;flex-direction:column;gap:3px;}
  .beam-calc .field label{font-size:11px;color:#5b6b73;font-family:var(--mono);}
  .beam-calc .field input{border:1px solid var(--line);border-radius:4px;padding:5px 6px;
               font-family:var(--mono);font-size:12.5px;width:100%;background:#fbfcfc;}
  .beam-calc .field input:focus{outline:2px solid var(--accent);outline-offset:1px;}
  .beam-calc button{margin-top:14px;width:100%;background:var(--accent);color:#fff;border:none;
         border-radius:6px;padding:10px;font-size:13.5px;font-weight:600;cursor:pointer;}
  .beam-calc button:hover{background:#0c4a5e;}
  .beam-calc .note{font-size:11px;color:#7c8991;line-height:1.5;margin-top:12px;}
  .beam-calc .cards{display:grid;grid-template-columns:repeat(4,1fr);gap:10px;margin-bottom:16px;}
  .beam-calc .card{background:var(--panel);border:1px solid var(--line);border-radius:8px;padding:12px 14px;}
  .beam-calc .card .k{font-size:10.5px;color:#5b6b73;font-family:var(--mono);text-transform:uppercase;letter-spacing:.04em;}
  .beam-calc .card .v{font-size:18px;font-weight:700;font-family:var(--mono);margin-top:4px;}
  .beam-calc .chartbox{background:var(--panel);border:1px solid var(--line);border-radius:8px;
            padding:14px;margin-bottom:14px;}
  .beam-calc .chartbox h3{margin:0 0 8px;font-size:12.5px;color:var(--ink);}
  .beam-calc canvas{max-height:220px;}
  .beam-calc .check{font-size:12px;color:#5b6b73;font-family:var(--mono);margin-top:6px;}
  .beam-calc .ok{color:#1c7a4d;}
</style>

  <div class="wrap">
    <div class="panel">
      <h2>Chiều dài nhịp (m)</h2>
      <div class="row">
        <div class="field"><label>L1</label><input id="bc_L1" type="number" value="52.65" step="0.01"></div>
        <div class="field"><label>L2</label><input id="bc_L2" type="number" value="72" step="0.01"></div>
        <div class="field"><label>L3</label><input id="bc_L3" type="number" value="51.35" step="0.01"></div>
      </div>
      <h2 class="sp">Tải phân bố đều (kN/m)</h2>
      <div class="row">
        <div class="field"><label>q1</label><input id="bc_q1" type="number" value="19.4" step="0.1"></div>
        <div class="field"><label>q2</label><input id="bc_q2" type="number" value="19.4" step="0.1"></div>
        <div class="field"><label>q3</label><input id="bc_q3" type="number" value="19.4" step="0.1"></div>
      </div>
      <h2 class="sp">Độ cứng I mỗi nhịp (m⁴)</h2>
      <div class="row">
        <div class="field"><label>I1</label><input id="bc_I1" type="number" value="3.63" step="0.01"></div>
        <div class="field"><label>I2</label><input id="bc_I2" type="number" value="3.63" step="0.01"></div>
        <div class="field"><label>I3</label><input id="bc_I3" type="number" value="3.63" step="0.01"></div>
      </div>
      <button onclick="bc_run()">Tính lại</button>
      <div class="note">I1=I2=I3 → giống định lý 3 mô men cổ điển. Đổi khác nhau để xem hiệu ứng haunch.</div>
    </div>
    <div>
      <div class="cards">
        <div class="card"><div class="k">M tại B</div><div class="v" id="bc_outMB">–</div></div>
        <div class="card"><div class="k">M tại C</div><div class="v" id="bc_outMC">–</div></div>
        <div class="card"><div class="k">R max</div><div class="v" id="bc_outRmax">–</div></div>
        <div class="card"><div class="k">Cân bằng ΣR</div><div class="v" id="bc_outCheck">–</div></div>
      </div>
      <div class="chartbox">
        <h3>Biểu đồ mô men M(x) — âm trên, dương dưới</h3>
        <canvas id="bc_chartM"></canvas>
      </div>
      <div class="chartbox">
        <h3>Biểu đồ lực cắt V(x)</h3>
        <canvas id="bc_chartV"></canvas>
      </div>
      <div class="check" id="bc_reactions"></div>
    </div>
  </div>
</div>

<script>
(function(){
let bc_chartM, bc_chartV;

window.bc_solve = function(){
  const g = id => document.getElementById(id);
  const L1=+g('bc_L1').value, L2=+g('bc_L2').value, L3=+g('bc_L3').value;
  const q1=+g('bc_q1').value, q2=+g('bc_q2').value, q3=+g('bc_q3').value;
  const I1=+g('bc_I1').value, I2=+g('bc_I2').value, I3=+g('bc_I3').value;

  const a11 = 2*(L1/I1 + L2/I2), a12 = L2/I2;
  const a21 = L2/I2,             a22 = 2*(L2/I2 + L3/I3);
  const b1 = -(q1*L1**3/I1 + q2*L2**3/I2)/4;
  const b2 = -(q2*L2**3/I2 + q3*L3**3/I3)/4;
  const det = a11*a22 - a12*a21;
  const MB = (b1*a22 - a12*b2)/det;
  const MC = (a11*b2 - b1*a21)/det;

  const RA = q1*L1/2 + MB/L1;
  const RD = q3*L3/2 + MC/L3;
  const RB = q1*L1/2+q2*L2/2 - MB*(1/L1+1/L2) + MC/L2;
  const RC = q2*L2/2+q3*L3/2 - MC*(1/L2+1/L3) + MB/L2;
  const sumR = RA+RB+RC+RD, target = q1*L1+q2*L2+q3*L3;

  const N=60, xs=[], Ms=[], Vs=[];
  function pushSpan(L,q,Ma,Mb,offset){
    for(let i=0;i<=N;i++){
      const x=L*i/N;
      const M = Ma*(L-x)/L + Mb*x/L + q*x*(L-x)/2;
      const V = (Mb-Ma)/L + q*(L/2 - x);
      xs.push((offset+x).toFixed(2)); Ms.push(M); Vs.push(V);
    }
  }
  pushSpan(L1,q1,0,MB,0);
  pushSpan(L2,q2,MB,MC,L1);
  pushSpan(L3,q3,MC,0,L1+L2);

  g('bc_outMB').textContent = MB.toFixed(1)+' kN·m';
  g('bc_outMC').textContent = MC.toFixed(1)+' kN·m';
  g('bc_outRmax').textContent = Math.max(RA,RB,RC,RD).toFixed(1)+' kN';
  const okDiff = Math.abs(sumR-target) < 0.01*target;
  g('bc_outCheck').innerHTML = (okDiff? '<span class="ok">khớp ✓</span>' : (sumR-target).toFixed(2));
  g('bc_reactions').textContent =
      `RA=${RA.toFixed(1)} kN   RB=${RB.toFixed(1)} kN   RC=${RC.toFixed(1)} kN   RD=${RD.toFixed(1)} kN`;

  const cfgM = {type:'line', data:{labels:xs, datasets:[
      {label:'M(x)', data:Ms.map(v=>-v), borderColor:'#0f5c73',
       backgroundColor:'rgba(15,92,115,0.12)', fill:true, pointRadius:0, borderWidth:1.6, tension:0.15}]},
      options:{animation:false, scales:{y:{title:{display:true,text:'kN·m (đảo trục)'}},
               x:{title:{display:true,text:'X (m)'}, ticks:{maxTicksLimit:12}}}, plugins:{legend:{display:false}}}};
  const cfgV = {type:'line', data:{labels:xs, datasets:[
      {label:'V(x)', data:Vs, borderColor:'#c1571f',
       backgroundColor:'rgba(193,87,31,0.12)', fill:true, pointRadius:0, borderWidth:1.6}]},
      options:{animation:false, scales:{y:{title:{display:true,text:'kN'}},
               x:{title:{display:true,text:'X (m)'}, ticks:{maxTicksLimit:12}}}, plugins:{legend:{display:false}}}};

  if(bc_chartM) bc_chartM.destroy();
  if(bc_chartV) bc_chartV.destroy();
  bc_chartM = new Chart(g('bc_chartM'), cfgM);
  bc_chartV = new Chart(g('bc_chartV'), cfgV);
};

// Chart.js được nạp bằng JS (thay vì <script src> tĩnh) vì cơ chế điều hướng
// SPA của Quartz (micromorph) chèn <script src> mới một cách bất đồng bộ:
// sự kiện "nav" có thể bắn ra và gọi bc_solve() TRƯỚC KHI Chart.js tải xong,
// gây lỗi "Chart is not defined" ở lần điều hướng đầu tiên vào trang.
// bc_run() đảm bảo luôn đợi Chart.js sẵn sàng rồi mới tính + vẽ biểu đồ.
function bc_loadChart(){
  if (window.Chart) return Promise.resolve();
  if (window.__bcChartPromise) return window.__bcChartPromise;
  window.__bcChartPromise = new Promise((resolve, reject) => {
    const s = document.createElement('script');
    s.src = 'https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.umd.min.js';
    s.onload = () => resolve();
    s.onerror = () => reject(new Error('Không tải được Chart.js từ CDN'));
    document.head.appendChild(s);
  });
  return window.__bcChartPromise;
}

window.bc_run = function(){
  bc_loadChart().then(window.bc_solve).catch((err) => console.error(err));
};

document.addEventListener("nav", () => {
  // chỉ chạy khi đang ở đúng trang này (tránh lỗi khi điều hướng sang trang khác)
  if (document.getElementById('bc_L1')) window.bc_run();
});
})();
</script>
