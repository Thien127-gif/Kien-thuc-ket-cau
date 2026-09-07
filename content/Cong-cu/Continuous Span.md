# Công cụ tính dầm liên tục N nhịp

<!-- ======================================================================
     CONG CU TINH DAM LIEN TUC N NHIP
     Dan nguyen khoi nay vao 1 note Quartz, ben duoi dong tieu de "# ..."
     Neu dung nhieu cong cu tren CUNG 1 trang, phai doi het id (nSpans,
     spanInputs, material, dataTable... va cac ham JS) sang ten khac nhau
     de tranh trung, vi HTML id phai la duy nhat tren 1 trang.
     ====================================================================== -->
<div class="beam-calc">
<style>
  /* -- BIEN MAU / FONT DUNG CHUNG, chi anh huong ben trong .beam-calc -- */
  .beam-calc{
    --ink:#1c2b33; --paper:#f4f7f8; --panel:#ffffff; --line:#d7e0e4;
    --accent:#0f5c73; --accent2:#c1571f; --mono: 'IBM Plex Mono','Consolas',monospace;
    background:var(--paper); color:var(--ink); font-family:'Inter','Segoe UI',Arial,sans-serif;
    border:1px solid var(--line); border-radius:10px; overflow:hidden; margin:16px 0;
  }
  .beam-calc *{box-sizing:border-box;}
  /* -- BO CUC 2 COT: panel nhap lieu ben trai, ket qua ben phai -- */
  .beam-calc .wrap{display:grid;grid-template-columns:340px 1fr;gap:0;}
  .beam-calc .panel{background:var(--panel);border-right:1px solid var(--line);padding:22px;
         max-height:calc(100vh - 84px); overflow-y:auto;}
  .beam-calc .panel h2{font-size:11.5px;text-transform:uppercase;letter-spacing:.06em;color:var(--accent);
            margin:20px 0 10px;border-bottom:1px solid var(--line);padding-bottom:6px;}
  .beam-calc .panel h2:first-child{margin-top:0;}
  .beam-calc .field{display:flex;flex-direction:column;gap:3px;margin-bottom:8px;}
  .beam-calc .field label{font-size:11px;color:#5b6b73;font-family:var(--mono);}
  .beam-calc .field input,.beam-calc .field select{border:1px solid var(--line);border-radius:4px;padding:6px 7px;
               font-family:var(--mono);font-size:12.5px;width:100%;background:#fbfcfc;}
  .beam-calc .row3{display:grid;grid-template-columns:1fr 1fr;gap:8px;}
  /* -- END: khoi CSS bo cuc chung -- */
  /* -- BAT DAU: CSS rieng cho khu vuc nhap chieu dai nhip (sinh dong bang JS) -- */
  .beam-calc #spanInputs{display:flex;flex-direction:column;gap:6px;margin-bottom:6px;}
  .beam-calc #spanInputs .sp{display:grid;grid-template-columns:24px 1fr;gap:6px;align-items:center;}
  .beam-calc #spanInputs .sp span{font-family:var(--mono);font-size:12px;color:#5b6b73;text-align:right;}
  /* -- KET THUC: CSS khu vuc nhap chieu dai nhip -- */
  /* -- BAT DAU: CSS bang du lieu X-I-w-P (dan duoc tu Excel) -- */
  .beam-calc table.dtable{width:100%;border-collapse:collapse;font-family:var(--mono);font-size:11.5px;}
  .beam-calc table.dtable th{background:#eef3f4;padding:4px 3px;font-weight:600;border:1px solid var(--line);}
  .beam-calc table.dtable td{border:1px solid var(--line);padding:0;}
  .beam-calc table.dtable td input{border:none;width:100%;padding:4px 4px;font-family:var(--mono);
                         font-size:11.5px;background:transparent;}
  .beam-calc table.dtable td input:focus{outline:1.5px solid var(--accent);}
  .beam-calc .tblbtns{display:flex;gap:6px;margin-top:6px;}
  .beam-calc .tblbtns button{flex:1;padding:5px;font-size:11px;border:1px solid var(--line);
                  background:#fff;border-radius:4px;cursor:pointer;}
  .beam-calc .hint{font-size:10.5px;color:#7c8991;line-height:1.5;margin-top:6px;}
  /* -- KET THUC: CSS bang du lieu -- */
  .beam-calc .maincta{margin-top:18px;width:100%;background:var(--accent);color:#fff;border:none;
         border-radius:6px;padding:12px;font-size:14px;font-weight:700;cursor:pointer;}
  .beam-calc .maincta:hover{background:#0c4a5e;}
  /* -- BAT DAU: CSS khu vuc ket qua (so do nhip + 2 bieu do + bang tong hop + chu thich) -- */
  .beam-calc main{padding:22px 32px;min-width:0;}
  .beam-calc .block{background:var(--panel);border:1px solid var(--line);border-radius:8px;
         padding:16px;margin-bottom:16px;min-width:0;}
  .beam-calc .block h3{margin:0 0 10px;font-size:13px;}
  .beam-calc canvas.chartc{max-height:230px;}
  /* -- BAT DAU: CSS zoom/cuon cho so do nhip + bieu do (them khi nhieu nhip bi tran khung) -- */
  .beam-calc .zoomrow{display:flex;align-items:center;justify-content:space-between;margin-bottom:10px;}
  .beam-calc .zoomrow h3{margin:0;}
  .beam-calc .zoombtns{display:flex;gap:4px;flex-shrink:0;}
  .beam-calc .zoombtns button,.beam-calc .zoomreset{border:1px solid var(--line);background:#fff;border-radius:4px;
                        cursor:pointer;font-family:var(--mono);color:var(--accent);}
  .beam-calc .zoombtns button{width:26px;height:26px;font-size:14px;line-height:1;}
  .beam-calc .zoomreset{padding:4px 8px;font-size:11px;white-space:nowrap;}
  .beam-calc .svgscroll{overflow-x:auto;border:1px solid var(--line);border-radius:6px;background:#fbfcfc;}
  .beam-calc svg#spanSvg{display:block;height:auto;}
  /* -- KET THUC: CSS zoom/cuon -- */
  .beam-calc table.sumtable{width:100%;border-collapse:collapse;font-size:12.5px;font-family:var(--mono);}
  .beam-calc table.sumtable th{background:#eef3f4;padding:6px 8px;border:1px solid var(--line);text-align:center;}
  .beam-calc table.sumtable td{padding:6px 8px;border:1px solid var(--line);text-align:center;}
  .beam-calc .legend{font-size:12px;color:#3a4a52;line-height:1.85;}
  .beam-calc .legend b{font-family:var(--mono);color:var(--accent);}
  /* -- KET THUC: CSS khu vuc ket qua -- */
</style>
<div class="wrap">
  <div class="panel">
    <!-- BAT DAU: nhap so nhip + chieu dai tung nhip -->
    <h2>Số nhịp &amp; chiều dài (m)</h2>
    <div class="field"><label>Số nhịp</label><input id="nSpans" type="number" min="1" max="15" value="3"></div>
    <div id="spanInputs"></div>
    <!-- KET THUC: nhap so nhip + chieu dai -->
    <!-- BAT DAU: chon vat lieu / mo dun dan hoi E -->
    <h2>Vật liệu — Mô đun đàn hồi E</h2>
    <div class="field">
      <label>Chọn vật liệu</label>
      <select id="material">
        <option value="30e6">Bê tông C20/25 (Ecm≈30 GPa)</option>
        <option value="31e6">Bê tông C25/30 (Ecm≈31 GPa)</option>
        <option value="33e6" selected>Bê tông C30/37 (Ecm≈33 GPa)</option>
        <option value="34e6">Bê tông C35/45 (Ecm≈34 GPa)</option>
        <option value="35e6">Bê tông C40/50 (Ecm≈35 GPa)</option>
        <option value="37e6">Bê tông C50/60 (Ecm≈37 GPa)</option>
        <option value="210e6">Thép kết cấu (E=210 GPa)</option>
        <option value="custom">Tùy chỉnh...</option>
      </select>
    </div>
    <div class="field" id="customEwrap" style="display:none;">
      <label>E tùy chỉnh (kN/m²)</label><input id="customE" type="number" value="33000000">
    </div>
    <!-- KET THUC: chon vat lieu -->
    <!-- BAT DAU: bang du lieu X - I - w - P (nhap tay hoac dan tu Excel) -->
    <h2>Bảng số liệu: X · I · w · P</h2>
    <table class="dtable" id="dataTable">
      <thead><tr><th>X (m)</th><th>I (m⁴)</th><th>w (kN/m)</th><th>P (kN)</th></tr></thead>
      <tbody id="dataBody"></tbody>
    </table>
    <div class="tblbtns">
      <button onclick="addRow()">+ Thêm dòng</button>
      <button onclick="clearTable()">Xoá hết</button>
    </div>
    <div class="hint">Có thể dán trực tiếp từ Excel: bôi đen 1 hoặc nhiều cột (X, hoặc X+I, hoặc cả 4 cột), click vào đúng ô đầu tiên trong bảng rồi Ctrl+V — bảng tự điền theo đúng số dòng dán vào, tự thêm dòng nếu thiếu.<br><br>
    <b>w</b> giữ nguyên giá trị từ dòng đó đến dòng kế tiếp (tải "bật/tắt" theo lý trình — muốn tắt tải, thêm 1 dòng w=0 tại vị trí kết thúc). <b>P</b> chỉ áp dụng đúng tại lý trình của dòng đó (tải tập trung, không kéo dài). <b>I</b> nội suy tuyến tính giữa 2 dòng liền kề.</div>
    <!-- KET THUC: bang du lieu -->
    <button class="maincta" onclick="runCalcSafe()">Tính toán</button>
  </div>
  <!-- KET THUC: toan bo panel nhap lieu ben trai -->
  <!-- BAT DAU: khu vuc ket qua ben phai - dung dung thu tu So do nhip > M(x) > V(x) > bang tong hop > chu thich -->
  <main>
    <!-- KHOI 1: so do nhip (ve tu dong bang ham drawSpanDiagram trong JS).
         Khung tu gian rong theo so nhip (khong bi tran/chong chu nua);
         cuon ngang de xem het, hoac bam nut phong to/thu nho/vua khung. -->
    <div class="block">
      <div class="zoomrow">
        <h3>Sơ đồ nhịp tính</h3>
        <div class="zoombtns">
          <button onclick="zoomSvg(-1)" title="Thu nhỏ">−</button>
          <button onclick="zoomSvg(1)" title="Phóng to">+</button>
          <button onclick="zoomSvg(0)" title="Vừa khung">⤢</button>
        </div>
      </div>
      <div class="svgscroll"><svg id="spanSvg" viewBox="0 0 900 220"></svg></div>
    </div>
    <!-- KHOI 2: bieu do mo men M(x). Cuon chuot de zoom, keo de di chuyen
         (chartjs-plugin-zoom) - huu ich khi nhieu nhip lam truc X qua day. -->
    <div class="block">
      <div class="zoomrow">
        <h3>Biểu đồ mô men M(x) — âm trên, dương dưới</h3>
        <button class="zoomreset" onclick="resetChartZoom()" title="Về mặc định">⤢ Reset zoom</button>
      </div>
      <canvas id="chartM" class="chartc"></canvas>
      <div class="hint">Cuộn chuột (hoặc chụm 2 ngón) để phóng to/thu nhỏ; giữ và kéo để di chuyển.</div>
    </div>
    <!-- KHOI 3: bieu do luc cat V(x) -->
    <div class="block">
      <div class="zoomrow">
        <h3>Biểu đồ lực cắt V(x)</h3>
        <button class="zoomreset" onclick="resetChartZoom()" title="Về mặc định">⤢ Reset zoom</button>
      </div>
      <canvas id="chartV" class="chartc"></canvas>
      <div class="hint">Cuộn chuột (hoặc chụm 2 ngón) để phóng to/thu nhỏ; giữ và kéo để di chuyển.</div>
    </div>
    <!-- KHOI 4: bang tong hop M, R tai tung goi -->
    <div class="block">
      <h3>Bảng tổng hợp kết quả</h3>
      <table class="sumtable" id="sumTable"></table>
    </div>
    <!-- KHOI 5: chu thich ky hieu - anh co the sua/them dong o day khi can -->
    <div class="block">
      <h3>Chú thích ký hiệu</h3>
      <div class="legend">
        <b>A, B, C...</b> — tên các gối theo thứ tự bảng chữ cái, tính từ đầu dầm.<br>
        <b>L</b> — chiều dài nhịp (m). <b>x</b> — lý trình đo từ đầu dầm (m).<br>
        <b>I</b> — mô men quán tính tiết diện (m⁴). <b>E</b> — mô đun đàn hồi vật liệu (kN/m²).<br>
        <b>w</b> — tải trọng phân bố đều (kN/m). <b>P</b> — tải trọng tập trung (kN).<br>
        <b>M</b> — mô men uốn (kN·m), quy ước âm (hogging) vẽ trên đường chuẩn, dương (sagging) vẽ dưới.<br>
        <b>V</b> — lực cắt (kN). <b>R</b> — phản lực gối (kN).
      </div>
    </div>
    <!-- KET THUC: chu thich ky hieu -->
  </main>
  <!-- KET THUC: khu vuc ket qua -->
</div>
<script id="cs-logic">
let chartM, chartV;
/* ==========================================================================
   PHAN 1: GIAO DIEN NHAP LIEU
   - renderSpanInputs: ve lai o nhap chieu dai nhip moi khi doi so nhip
   - addRow/clearTable/setupPaste: quan ly bang du lieu X-I-w-P, cho dan Excel
   ========================================================================== */
function renderSpanInputs(){
  const n = Math.max(1, Math.min(15, +document.getElementById('nSpans').value||3));
  const wrap = document.getElementById('spanInputs');
  const labels = [];
  for(let i=0;i<=n;i++) labels.push(String.fromCharCode(65+i));
  let html='';
  const defaults=[52.65,72,51.35];
  for(let i=0;i<n;i++){
    const v = defaults[i] !== undefined ? defaults[i] : 20;
    html += `<div class="sp"><span>${labels[i]}-${labels[i+1]}</span>
             <input type="number" class="spanLen" data-i="${i}" value="${v}" step="0.01"></div>`;
  }
  wrap.innerHTML = html;
}
document.getElementById('nSpans').addEventListener('input', renderSpanInputs);
document.getElementById('material').addEventListener('change', function(){
  document.getElementById('customEwrap').style.display = this.value==='custom' ? 'block':'none';
});
function addRow(vals){
  const tb = document.getElementById('dataBody');
  const tr = document.createElement('tr');
  const cols = vals || ['','','',''];
  tr.innerHTML = cols.map(v=>`<td><input type="text" value="${v}"></td>`).join('');
  tb.appendChild(tr);
}
function clearTable(){ document.getElementById('dataBody').innerHTML=''; addRow(); }
function setupPaste(){
  document.getElementById('dataBody').addEventListener('paste', function(e){
    const target = e.target;
    if(target.tagName!=='INPUT') return;
    e.preventDefault();
    const text = (e.clipboardData||window.clipboardData).getData('text');
    const lines = text.split(/\r?\n/).filter(l=>l.length>0);
    const startRow = target.closest('tr');
    const startCellIdx = Array.from(startRow.children).indexOf(target.closest('td'));
    let rows = Array.from(document.getElementById('dataBody').children);
    let startRowIdx = rows.indexOf(startRow);
    lines.forEach((line, li)=>{
      const cells = line.split('\t');
      let rIdx = startRowIdx+li;
      rows = Array.from(document.getElementById('dataBody').children);
      if(rIdx >= rows.length){ addRow(); rows = Array.from(document.getElementById('dataBody').children); }
      const rowEl = rows[rIdx];
      cells.forEach((val, ci)=>{
        const cIdx = startCellIdx+ci;
        if(cIdx < rowEl.children.length){
          rowEl.children[cIdx].querySelector('input').value = val.trim();
        }
      });
    });
  });
}
/* ==========================================================================
   PHAN 2: DOC DU LIEU DAU VAO TU GIAO DIEN
   - getE: lay mo dun dan hoi (theo vat lieu chon hoac tuy chinh)
   - getSpans: lay mang chieu dai tung nhip
   - getTable: doc bang X-I-w-P, sap xep theo X tang dan
   ========================================================================== */
function getE(){
  const m = document.getElementById('material').value;
  if(m==='custom') return +document.getElementById('customE').value;
  return +m;
}
function getSpans(){
  return Array.from(document.querySelectorAll('.spanLen')).map(el=>+el.value);
}
function getTable(){
  const rows=[];
  document.querySelectorAll('#dataBody tr').forEach(tr=>{
    const inp = tr.querySelectorAll('input');
    const x=+inp[0].value;
    if(inp[0].value==='' ) return;
    rows.push({x, I:+inp[1].value||0, w:+inp[2].value||0, P:+inp[3].value||0});
  });
  rows.sort((a,b)=>a.x-b.x);
  return rows;
}
/* ==========================================================================
   PHAN 3: DONG CO TINH TOAN (KHONG NEN SUA NEU CHUA HIEU RO CONG THUC)
   Phuong phap luc - dinh ly 3 mo men tong quat, giai he 3 duong cheo
   (tridiagonal) cho (so_nhip - 1) an so la mo men tai cac goi giua.
   Da kiem chung khop 100% voi ban Python doc lap (xem lai hoi thoai truoc).
   ========================================================================== */
function linspace(a,b,n){ const r=[]; for(let i=0;i<=n;i++) r.push(a+(b-a)*i/n); return r; }
function trapz(y,x){ let s=0; for(let i=0;i<x.length-1;i++) s+=(y[i]+y[i+1])/2*(x[i+1]-x[i]); return s; }
// ham chinh: nhan vao chieu dai nhip, E, bang du lieu -> tra ve mo men/luc cat/phan luc
function solveBeam(spans, E, rows, N_per_span){
  const nSpans = spans.length;
  const supportX=[0];
  spans.forEach(L=>supportX.push(supportX[supportX.length-1]+L));
  const bx = rows.map(r=>r.x), bI = rows.map(r=>r.I), bw = rows.map(r=>r.w);
  const pts = rows.filter(r=>r.P).map(r=>({x:r.x,P:r.P}));
  function I_at(x){
    if(x<=bx[0]) return bI[0];
    if(x>=bx[bx.length-1]) return bI[bI.length-1];
    for(let i=0;i<bx.length-1;i++){
      if(bx[i]<=x && x<=bx[i+1]){
        if(bx[i+1]===bx[i]) return bI[i];
        const t=(x-bx[i])/(bx[i+1]-bx[i]);
        return bI[i]+t*(bI[i+1]-bI[i]);
      }
    }
    return bI[bI.length-1];
  }
  function w_at(x){
    if(x<bx[0]) return 0;
    let val=bw[0];
    for(let i=0;i<bx.length;i++){ if(bx[i]<=x) val=bw[i]; else break; }
    return val;
  }
  const spanGrids=[];
  for(let k=0;k<nSpans;k++){
    const xa=supportX[k], xb=supportX[k+1];
    let base = linspace(xa,xb,N_per_span);
    let extra = bx.filter(x=>x>xa && x<xb).concat(pts.filter(p=>p.x>xa&&p.x<xb).map(p=>p.x));
    let all = base.concat(extra).map(v=>Math.round(v*1e7)/1e7);
    all = Array.from(new Set(all)).sort((a,b)=>a-b);
    spanGrids.push(all);
  }
  function spanM0(k){
    const xa=supportX[k], xb=supportX[k+1], L=xb-xa;
    const gx = spanGrids[k];
    const gw = gx.map(w_at);
    const integrand = gx.map((x,i)=>gw[i]*(xb-x));
    let momA = trapz(integrand, gx);
    pts.forEach(p=>{ if(p.x>=xa && p.x<=xb) momA += p.P*(xb-p.x); });
    const R_left = momA/L;
    const V=new Array(gx.length), M=new Array(gx.length);
    for(let i=0;i<gx.length;i++){
      let v = R_left;
      if(i>0) v -= trapz(gw.slice(0,i+1), gx.slice(0,i+1));
      pts.forEach(p=>{ if(p.x>=xa && p.x<=xb && p.x <= gx[i]+1e-9) v -= p.P; });
      V[i]=v;
    }
    M[0]=0;
    for(let i=1;i<gx.length;i++) M[i]=M[i-1]+(V[i-1]+V[i])/2*(gx[i]-gx[i-1]);
    return {gx,M,V,R_left};
  }
  const M0data = spanGrids.map((_,k)=>spanM0(k));
  function mVal(i, spanK, x){
    const xa=supportX[spanK], xb=supportX[spanK+1], L=xb-xa;
    if(spanK===i-1) return (x-xa)/L;
    if(spanK===i) return (xb-x)/L;
    return 0;
  }
  const nInt = nSpans-1;
  let Msup;
  if(nInt===0){ Msup=[]; }
  else{
    const A=Array.from({length:nInt},()=>new Array(nInt).fill(0));
    const b=new Array(nInt).fill(0);
    function integrateSpan(k, fa, fb){
      const gx=spanGrids[k];
      const EIv = gx.map(x=>E*I_at(x));
      const integrand = gx.map((x,idx)=>fa[idx]*fb[idx]/EIv[idx]);
      return trapz(integrand, gx);
    }
    for(let i=1;i<=nInt;i++){
      const touching=[i-1,i];
      let diag=0;
      touching.forEach(sk=>{
        const gx=spanGrids[sk];
        const mv=gx.map(x=>mVal(i,sk,x));
        diag += integrateSpan(sk,mv,mv);
      });
      A[i-1][i-1]=diag;
      if(i-1>=1){
        const sk=i-1;
        const gx=spanGrids[sk];
        const mi=gx.map(x=>mVal(i,sk,x));
        const mim1=gx.map(x=>mVal(i-1,sk,x));
        const val=integrateSpan(sk,mi,mim1);
        A[i-1][i-2]=val; A[i-2][i-1]=val;
      }
      let rhs=0;
      touching.forEach(sk=>{
        const {gx,M} = M0data[sk];
        const mv=gx.map(x=>mVal(i,sk,x));
        const EIv=gx.map(x=>E*I_at(x));
        const integrand=gx.map((x,idx)=>mv[idx]*M[idx]/EIv[idx]);
        rhs += trapz(integrand,gx);
      });
      b[i-1]=-rhs;
    }
    Msup = solveLinear(A,b);
  }
  const Mall=[0,...Msup,0];
  // reconstruct global M(x), V(x)
  const allX=[], allM=[], allV=[];
  for(let k=0;k<nSpans;k++){
    const {gx,M,V} = M0data[k];
    const xa=supportX[k], xb=supportX[k+1], L=xb-xa;
    for(let idx=0;idx<gx.length;idx++){
      let mAdd=0, vAdd=0;
      for(let i=1;i<nSpans;i++){
        if(k===i-1){ mAdd += Mall[i]*(gx[idx]-xa)/L; vAdd += Mall[i]*(1/L); }
        else if(k===i){ mAdd += Mall[i]*(xb-gx[idx])/L; vAdd += Mall[i]*(-1/L); }
      }
      allX.push(gx[idx]); allM.push(M[idx]+mAdd); allV.push(V[idx]+vAdd);
    }
  }
  // reactions
  function Vtotal(k, idx){
    const {V}=M0data[k];
    const xa=supportX[k],xb=supportX[k+1],L=xb-xa;
    let v=V[idx];
    for(let i=1;i<nSpans;i++){
      if(k===i-1) v+=Mall[i]*(1/L);
      else if(k===i) v+=Mall[i]*(-1/L);
    }
    return v;
  }
  const R=[];
  R.push(Vtotal(0,0));
  for(let i=1;i<nSpans;i++){
    const vl=Vtotal(i-1, M0data[i-1].gx.length-1);
    const vr=Vtotal(i,0);
    R.push(vr-vl);
  }
  R.push(-Vtotal(nSpans-1, M0data[nSpans-1].gx.length-1));
  return {supportX, Mall, R, allX, allM, allV};
}
// giai he phuong trinh tuyen tinh bang khu Gauss (dung cho ma tran 3 duong cheo o tren)
function solveLinear(A,b){
  const n=b.length;
  const M=A.map((row,i)=>[...row,b[i]]);
  for(let i=0;i<n;i++){
    let piv=i;
    for(let r=i+1;r<n;r++) if(Math.abs(M[r][i])>Math.abs(M[piv][i])) piv=r;
    [M[i],M[piv]]=[M[piv],M[i]];
    for(let r=0;r<n;r++){
      if(r===i) continue;
      const f=M[r][i]/M[i][i];
      for(let c=i;c<=n;c++) M[r][c]-=f*M[i][c];
    }
  }
  return M.map((row,i)=>row[n]/row[i]);
}
/* ==========================================================================
   PHAN 4: VE SO DO NHIP (SVG) - goi, kich thuoc nhip, vung tai phan bo/tap trung
   Khung SVG tu gian rong theo so nhip (moi nhip ~130px) thay vi nhoi het
   vao 900px co dinh - nhieu nhip thi cuon ngang de xem, tranh chu chong chu.
   ========================================================================== */
let svgZoom = 1, svgBaseW = 900, svgBaseH = 220;
function zoomSvg(dir){
  svgZoom = dir === 0 ? 1 : Math.min(3, Math.max(0.4, svgZoom + dir*0.25));
  const svg = document.getElementById('spanSvg');
  if(svg) svg.style.width = (svgBaseW*svgZoom)+'px';
}
function drawSpanDiagram(spans, supportX, rows){
  const svg = document.getElementById('spanSvg');
  const totalL = supportX[supportX.length-1];
  const mL=40,mR=40, y0=110;
  // Vua khung khi it nhip (giong cu, khong cuon); chi gian rong hon khung
  // khi nhieu nhip can toi thieu ~130px/nhip de nhan/chu khong de len nhau.
  const perSpan = 130;
  const containerW = document.querySelector('.beam-calc .svgscroll')?.clientWidth || 860;
  const W = Math.max(containerW, mL+mR+spans.length*perSpan), H=220;
  svgBaseW = W; svgBaseH = H;
  const scale=(W-mL-mR)/totalL;
  const X = x=> mL+x*scale;
  let parts=[];
  parts.push(`<line x1="${mL}" y1="${y0}" x2="${W-mR}" y2="${y0}" stroke="#1c2b33" stroke-width="3"/>`);
  // supports
  supportX.forEach((sx,i)=>{
    const px=X(sx);
    const label=String.fromCharCode(65+i);
    parts.push(`<polygon points="${px},${y0} ${px-9},${y0+16} ${px+9},${y0+16}" fill="#eef3f4" stroke="#1c2b33" stroke-width="1.2"/>`);
    parts.push(`<line x1="${px-14}" y1="${y0+16}" x2="${px+14}" y2="${y0+16}" stroke="#1c2b33" stroke-width="1"/>`);
    parts.push(`<text x="${px}" y="${y0+34}" text-anchor="middle" font-size="13" font-weight="700">${label}</text>`);
    parts.push(`<line x1="${px}" y1="${y0-70}" x2="${px}" y2="${y0}" stroke="#c7d0d3" stroke-width="0.6" stroke-dasharray="2,2"/>`);
  });
  // span length dimension
  supportX.forEach((sx,i)=>{
    if(i<supportX.length-1){
      const x1=X(sx), x2=X(supportX[i+1]);
      const ym=y0+50;
      parts.push(`<line x1="${x1}" y1="${ym}" x2="${x2}" y2="${ym}" stroke="#5b6b73" stroke-width="0.7"/>`);
      parts.push(`<line x1="${x1}" y1="${ym-4}" x2="${x1}" y2="${ym+4}" stroke="#5b6b73" stroke-width="0.7"/>`);
      parts.push(`<line x1="${x2}" y1="${ym-4}" x2="${x2}" y2="${ym+4}" stroke="#5b6b73" stroke-width="0.7"/>`);
      parts.push(`<text x="${(x1+x2)/2}" y="${ym+16}" text-anchor="middle" font-size="11" fill="#5b6b73">${(supportX[i+1]-sx).toFixed(2)} m</text>`);
    }
  });
  // distributed load zones (from w step function): sample finely, find contiguous w!=0 runs
  const bx=rows.map(r=>r.x), bw=rows.map(r=>r.w);
  function w_at(x){ if(bx.length===0||x<bx[0]) return 0; let v=bw[0]; for(let i=0;i<bx.length;i++){ if(bx[i]<=x) v=bw[i]; else break;} return v; }
  const N=400; let zones=[]; let curStart=null, curVal=0;
  for(let i=0;i<=N;i++){
    const x=totalL*i/N; const v=w_at(x);
    if(v!==0 && curStart===null){ curStart=x; curVal=v; }
    if(v===0 && curStart!==null){ zones.push([curStart,x,curVal]); curStart=null; }
  }
  if(curStart!==null) zones.push([curStart,totalL,curVal]);
  zones.forEach(([xa,xb,val])=>{
    const x1=X(xa), x2=X(xb), ytop=y0-38;
    parts.push(`<line x1="${x1}" y1="${ytop}" x2="${x2}" y2="${ytop}" stroke="#0f5c73" stroke-width="1"/>`);
    for(let xx=x1; xx<=x2; xx+= Math.max(10,(x2-x1)/8)){
      parts.push(`<line x1="${xx}" y1="${ytop}" x2="${xx}" y2="${y0-2}" stroke="#0f5c73" stroke-width="1" marker-end="url(#ar)"/>`);
    }
    parts.push(`<text x="${(x1+x2)/2}" y="${ytop-6}" text-anchor="middle" font-size="10.5" fill="#0f5c73">w=${val} kN/m</text>`);
  });
  // point loads
  rows.filter(r=>r.P).forEach(r=>{
    const px=X(r.x);
    parts.push(`<line x1="${px}" y1="${y0-55}" x2="${px}" y2="${y0-2}" stroke="#c1571f" stroke-width="1.6" marker-end="url(#ar2)"/>`);
    parts.push(`<text x="${px}" y="${y0-58}" text-anchor="middle" font-size="10.5" fill="#c1571f" font-weight="700">P=${r.P}kN</text>`);
  });
  svg.setAttribute('viewBox',`0 0 ${W} ${H}`);
  svg.style.width = (W*svgZoom)+'px';
  svg.innerHTML = `<defs>
    <marker id="ar" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#0f5c73" stroke-width="1.4"/></marker>
    <marker id="ar2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="#c1571f" stroke-width="1.6"/></marker>
    </defs>` + parts.join('');
}
/* ==========================================================================
   PHAN 5: HAM CHINH - noi ket tat ca lai khi bam nut "Tinh toan"
   Goi solveBeam -> ve so do -> ve 2 bieu do Chart.js -> dien bang tong hop
   ========================================================================== */
function runCalc(){
  const spans = getSpans();
  const E = getE();
  const rows = getTable();
  if(rows.length<2 || spans.length<1){ alert('Cần nhập ít nhất 2 dòng dữ liệu và 1 nhịp.'); return; }
  const res = solveBeam(spans, E, rows, 50);
  drawSpanDiagram(spans, res.supportX, rows);
  const labels = res.allX.map(x=>x.toFixed(2));
  if(chartM) chartM.destroy();
  if(chartV) chartV.destroy();
  // zoomPlugin: cuon chuot de phong to, giu+keo de di chuyen (chi truc X,
  // vi truc can quan tam khi nhieu nhip la vi tri doc theo dam).
  const zoomPlugin = {
    pan:{enabled:true, mode:'x'},
    zoom:{wheel:{enabled:true}, pinch:{enabled:true}, drag:{enabled:false}, mode:'x'}
  };
  chartM = new Chart(document.getElementById('chartM'),{type:'line',
    data:{labels, datasets:[{data:res.allM.map(v=>-v), borderColor:'#0f5c73',
      backgroundColor:'rgba(15,92,115,0.12)', fill:true, pointRadius:0, borderWidth:1.5}]},
    options:{animation:false, scales:{y:{title:{display:true,text:'kN·m (đảo trục)'}},
             x:{title:{display:true,text:'X (m)'}, ticks:{maxTicksLimit:14}}},
             plugins:{legend:{display:false}, zoom:zoomPlugin}}});
  chartV = new Chart(document.getElementById('chartV'),{type:'line',
    data:{labels, datasets:[{data:res.allV, borderColor:'#c1571f',
      backgroundColor:'rgba(193,87,31,0.12)', fill:true, pointRadius:0, borderWidth:1.5}]},
    options:{animation:false, scales:{y:{title:{display:true,text:'kN'}},
             x:{title:{display:true,text:'X (m)'}, ticks:{maxTicksLimit:14}}},
             plugins:{legend:{display:false}, zoom:zoomPlugin}}});
  const sumEl = document.getElementById('sumTable');
  let head='<tr><th>Gối</th>'+res.supportX.map((_,i)=>`<th>${String.fromCharCode(65+i)}</th>`).join('')+'</tr>';
  let rowM='<tr><td>M (kN·m)</td>'+res.Mall.map(v=>`<td>${v.toFixed(1)}</td>`).join('')+'</tr>';
  let rowR='<tr><td>R (kN)</td>'+res.R.map(v=>`<td>${v.toFixed(1)}</td>`).join('')+'</tr>';
  const sumR=res.R.reduce((a,b)=>a+b,0);
  sumEl.innerHTML = head+rowM+rowR+
    `<tr><td colspan="${res.supportX.length+1}" style="text-align:left;font-size:11px;color:#5b6b73;">ΣR = ${sumR.toFixed(2)} kN (kiểm tra cân bằng)</td></tr>`;
}
// Chart.js duoc nap bang JS (thay vi <script src> tinh) vi co che SPA cua
// Quartz (micromorph) chen <script src> moi mot cach BAT DONG BO: su kien
// "nav" co the ban ra va goi runCalc() TRUOC KHI Chart.js tai xong, gay loi
// "Chart is not defined" o lan dieu huong dau tien vao trang.
// Nap them chartjs-plugin-zoom (cuon chuot de zoom truc X) sau khi Chart.js
// da san sang, roi dang ky bang Chart.register(). Ca 2 deu nap qua the
// <script> tao dong (khong dung <script src> tinh) vi ly do da giai thich
// o tren (SPA cua Quartz chen script bat dong bo).
function loadChartJs(){
  if (window.Chart && window.__csZoomReady) return Promise.resolve();
  if (window.__csChartPromise) return window.__csChartPromise;
  function loadScript(src){
    return new Promise((res, rej) => {
      const s = document.createElement('script');
      s.src = src;
      s.onload = () => res();
      s.onerror = () => rej(new Error('Khong tai duoc '+src));
      document.head.appendChild(s);
    });
  }
  const chartReady = window.Chart ? Promise.resolve()
    : loadScript('https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.umd.min.js');
  window.__csChartPromise = chartReady
    .then(() => loadScript('https://cdnjs.cloudflare.com/ajax/libs/chartjs-plugin-zoom/2.2.0/chartjs-plugin-zoom.min.js'))
    .then(() => {
      if(!window.__csZoomReady && window.Chart && window.ChartZoom){
        Chart.register(window.ChartZoom);
        window.__csZoomReady = true;
      }
    });
  return window.__csChartPromise;
}
function runCalcSafe(){
  loadChartJs().then(runCalc).catch((err)=>console.error(err));
}
function resetChartZoom(){
  if(chartM && chartM.resetZoom) chartM.resetZoom();
  if(chartV && chartV.resetZoom) chartV.resetZoom();
}
// khoi tao khi mo trang - chay lai moi lan dieu huong vi Quartz dung SPA
function init(){
  renderSpanInputs();
  addRow([0,3.63,19.4,0]);     // dong vi du 1 - anh sua/xoa tuy y
  addRow([176,3.63,19.4,0]);   // dong vi du 2 - anh sua/xoa tuy y
  setupPaste();
  loadChartJs().then(runCalc).catch((err)=>console.error(err));
}
document.addEventListener("nav", () => {
  if (document.getElementById('nSpans')) init(); // chi chay dung trang nay, tranh loi o trang khac
});
</script>
<!-- Luoi an toan: co che SPA (micromorph) cua Quartz doi khi KHONG thuc thi
     lai the <script> ben tren khi dieu huong lan dau vao trang nay (tuy vi
     tri trung ngau nhien voi trang truoc do trong cay DOM). Anh <img> loi
     nay luon chay onerror bat ke cach noi dung duoc chen vao DOM, nen dung
     no de "cuu" bang cach tu chay lai script neu phat hien init() chua ton
     tai. Neu script da chay binh thuong thi day la no-op, khong lam gi ca. -->
<img src="cs-boot-trigger-404.gif" alt="" style="display:none" onerror="if(typeof window.init!=='function'){var sc=document.getElementById('cs-logic');var s=document.createElement('script');s.textContent=sc.textContent;document.body.appendChild(s);window.init&amp;&amp;window.init();}">
</div>
<!-- ======================================================================
     KET THUC cong cu tinh dam lien tuc N nhip
     ====================================================================== -->
