<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Kalkulator Matriks (HTML/JS)</title>
<style>
  body{font-family:system-ui,Segoe UI,Roboto,Helvetica,Arial;line-height:1.4;padding:18px;background:#f7fafc;color:#111}
  h1{font-size:20px;margin:0 0 12px}
  .container{display:flex;gap:18px;flex-wrap:wrap}
  .matrix-card{background:#fff;border:1px solid #e2e8f0;border-radius:8px;padding:12px;min-width:260px;box-shadow:0 1px 2px rgba(0,0,0,0.03)}
  .controls{display:flex;gap:8px;align-items:center;margin-bottom:8px;flex-wrap:wrap}
  label{font-size:13px;color:#333}
  input[type=number]{width:64px;padding:6px;border:1px solid #cbd5e1;border-radius:6px}
  button{background:#2563eb;color:white;border:none;padding:8px 10px;border-radius:6px;cursor:pointer}
  button.ghost{background:#f1f5f9;color:#111;border:1px solid #e2e8f0}
  table{border-collapse:collapse;margin-top:6px}
  td input{width:56px;padding:6px;border:1px solid #e2e8f0;border-radius:4px;text-align:center}
  .ops{display:flex;gap:8px;flex-direction:column}
  select{padding:8px;border-radius:6px;border:1px solid #cbd5e1}
  .result{background:#fff;border:1px dashed #cbd5e1;padding:12px;border-radius:8px;min-width:260px}
  .note{font-size:13px;color:#555;margin-top:10px}
  .row {display:flex;gap:12px;align-items:center;margin-top:8px}
  .small{font-size:13px;padding:6px 8px}
  pre{white-space:pre-wrap;margin:0;font-family:monospace}
  .muted{color:#666;font-size:13px}
  @media (max-width:720px){ .container{flex-direction:column} }
</style>
</head>
<body>
  <h1>Kalkulator Matriks (HTML/JS)</h1>
  <div class="container">
    <div class="matrix-card" id="cardA">
      <div class="controls">
        <label><strong>Matrix A</strong></label>
        <label class="muted">Baris <input id="rowsA" type="number" min="1" value="2"></label>
        <label class="muted">Kolom <input id="colsA" type="number" min="1" value="2"></label>
        <button class="ghost small" id="resizeA">Ubah Ukuran</button>
        <button class="ghost small" id="fillA">Isi contoh</button>
      </div>
      <div id="matrixA"></div>
    </div>

    <div class="matrix-card" id="cardB">
      <div class="controls">
        <label><strong>Matrix B</strong></label>
        <label class="muted">Baris <input id="rowsB" type="number" min="1" value="2"></label>
        <label class="muted">Kolom <input id="colsB" type="number" min="1" value="2"></label>
        <button class="ghost small" id="resizeB">Ubah Ukuran</button>
        <button class="ghost small" id="fillB">Isi contoh</button>
      </div>
      <div id="matrixB"></div>
    </div>

    <div class="ops matrix-card">
      <label><strong>Operasi</strong></label>
      <select id="operation">
        <option value="add">A + B</option>
        <option value="sub">A - B</option>
        <option value="mul">A × B</option>
        <option value="detA">det(A)</option>
        <option value="detB">det(B)</option>
        <option value="invA">A⁻¹</option>
        <option value="invB">B⁻¹</option>
        <option value="transA">Transpose(A)</option>
        <option value="transB">Transpose(B)</option>
        <option value="rrefA">RREF(A)</option>
        <option value="rrefB">RREF(B)</option>
      </select>
      <div class="row">
        <button id="compute">Hitung</button>
        <button id="clear" class="ghost">Bersihkan</button>
      </div>
      <div class="note">Catatan: operasi determinan/invers hanya untuk matriks bujursangkar. Perkalian membutuhkan kolom A = baris B.</div>
    </div>

    <div class="result matrix-card" id="resultCard">
      <label><strong>Hasil</strong></label>
      <div id="result" style="margin-top:8px"><em class="muted">Belum ada perhitungan.</em></div>
    </div>
  </div>

<script>
/* ---------- util: baca/isi matriks ke DOM ---------- */
function createMatrixTable(containerId, rows, cols, defaultValue=0) {
  const container = document.getElementById(containerId);
  container.innerHTML = '';
  const table = document.createElement('table');
  for (let r=0;r<rows;r++){
    const tr = document.createElement('tr');
    for (let c=0;c<cols;c++){
      const td = document.createElement('td');
      const input = document.createElement('input');
      input.type = 'number'; input.step = 'any';
      input.value = defaultValue;
      input.dataset.r = r; input.dataset.c = c;
      td.appendChild(input);
      tr.appendChild(td);
    }
    table.appendChild(tr);
  }
  container.appendChild(table);
}

function readMatrix(containerId) {
  const container = document.getElementById(containerId);
  const inputs = container.querySelectorAll('input');
  if (inputs.length===0) return [];
  // determine rows/cols
  let maxR=0,maxC=0;
  inputs.forEach(inp => {
    maxR = Math.max(maxR, parseInt(inp.dataset.r));
    maxC = Math.max(maxC, parseInt(inp.dataset.c));
  });
  const rows = maxR+1, cols = maxC+1;
  const M = Array.from({length:rows},()=>Array(cols).fill(0));
  inputs.forEach(inp=>{
    const r = parseInt(inp.dataset.r), c = parseInt(inp.dataset.c);
    const v = parseFloat(inp.value);
    M[r][c] = isFinite(v) ? v : 0;
  });
  return M;
}

function showMatrixAsHTML(M) {
  if (!M || M.length===0) return '<em class="muted">[]</em>';
  const rows = M.length, cols = M[0].length;
  let html = '<table>';
  for (let i=0;i<rows;i++){
    html += '<tr>';
    for (let j=0;j<cols;j++){
      html += '<td style="padding:4px 8px;border:1px solid #eef">'+formatNum(M[i][j])+'</td>';
    }
    html += '</tr>';
  }
  html += '</table>';
  return html;
}

function formatNum(x){
  if (Number.isInteger(x)) return x.toString();
  if (!isFinite(x)) return String(x);
  // round to 8 significant digits but keep as number
  return parseFloat(parseFloat(x).toFixed(8)).toString();
}

/* ---------- linear algebra helpers ---------- */
function zeros(r,c){ return Array.from({length:r},()=>Array(c).fill(0)); }

function add(A,B){
  if (A.length===0 || B.length===0) throw 'Matriks kosong';
  const r=A.length,c=A[0].length;
  if (r!==B.length || c!==B[0].length) throw 'Dimensi tidak cocok untuk penjumlahan';
  const R=zeros(r,c);
  for(let i=0;i<r;i++)for(let j=0;j<c;j++)R[i][j]=A[i][j]+B[i][j];
  return R;
}
function sub(A,B){
  if (A.length===0 || B.length===0) throw 'Matriks kosong';
  const r=A.length,c=A[0].length;
  if (r!==B.length || c!==B[0].length) throw 'Dimensi tidak cocok untuk pengurangan';
  const R=zeros(r,c);
  for(let i=0;i<r;i++)for(let j=0;j<c;j++)R[i][j]=A[i][j]-B[i][j];
  return R;
}
function mul(A,B){
  if (A.length===0 || B.length===0) throw 'Matriks kosong';
  const r=A.length, k=A[0].length;
  const k2=B.length, c=B[0].length;
  if (k !== k2) throw 'Jumlah kolom A harus sama dengan jumlah baris B';
  const R=zeros(r,c);
  for(let i=0;i<r;i++){
    for(let j=0;j<c;j++){
      let s=0;
      for(let t=0;t<k;t++) s += (A[i][t]||0)*(B[t][j]||0);
      R[i][j]=s;
    }
  }
  return R;
}

function transpose(A){
  if (A.length===0) return [];
  const r=A.length,c=A[0].length;
  const R=zeros(c,r);
  for(let i=0;i<r;i++)for(let j=0;j<c;j++)R[j][i]=A[i][j];
  return R;
}

function det(A){
  const n = A.length;
  if (n===0) return 0;
  if (!A.every(row=>row.length===n)) throw 'Det hanya untuk matriks bujursangkar';
  // LU via Gaussian elimination with partial pivoting, track sign
  const M = A.map(row=>row.slice());
  let sign = 1;
  for (let i=0;i<n;i++){
    // pivot
    let piv = i;
    for (let r=i+1;r<n;r++) if (Math.abs(M[r][i]) > Math.abs(M[piv][i])) piv = r;
    if (Math.abs(M[piv][i]) < 1e-12) return 0;
    if (piv !== i){ [M[i],M[piv]] = [M[piv],M[i]]; sign *= -1; }
    for (let r=i+1;r<n;r++){
      const f = M[r][i]/M[i][i];
      for (let c=i;c<n;c++) M[r][c] -= f * M[i][c];
    }
  }
  let prod = sign;
  for (let i=0;i<n;i++) prod *= M[i][i];
  return prod;
}

function inverse(A){
  const n = A.length;
  if (n===0) return [];
  if (!A.every(row=>row.length===n)) throw 'Invers hanya untuk matriks bujursangkar';
  // Gauss-Jordan
  const M = A.map(row=>row.slice());
  const I = zeros(n,n).map((r,i)=>r.map((_,j)=> i===j?1:0));
  for (let i=0;i<n;i++){
    // pivot
    let piv=i;
    for (let r=i+1;r<n;r++) if (Math.abs(M[r][i])>Math.abs(M[piv][i])) piv=r;
    if (Math.abs(M[piv][i]) < 1e-12) throw 'Matriks singular (tidak punya invers)';
    if (piv !== i){ [M[i],M[piv]]=[M[piv],M[i]]; [I[i],I[piv]]=[I[piv],I[i]]; }
    const val = M[i][i];
    for (let c=0;c<n;c++){ M[i][c] /= val; I[i][c] /= val; }
    for (let r=0;r<n;r++){
      if (r===i) continue;
      const f = M[r][i];
      for (let c=0;c<n;c++){ M[r][c] -= f*M[i][c]; I[r][c] -= f*I[i][c]; }
    }
  }
  return I;
}

function rref(A){
  const R = A.map(row=>row.slice());
  const rows = R.length, cols = (R[0]||[]).length;
  let r=0;
  for (let c=0;c<cols && r<rows; c++){
    // find pivot row
    let sel = r;
    for (let i=r;i<rows;i++) if (Math.abs(R[i][c]) > Math.abs(R[sel][c])) sel = i;
    if (Math.abs(R[sel][c]) < 1e-12) continue;
    [R[r],R[sel]]=[R[sel],R[r]];
    // normalize
    const div = R[r][c];
    for (let j=c;j<cols;j++) R[r][j] /= div;
    // eliminate others
    for (let i=0;i<rows;i++){
      if (i===r) continue;
      const f = R[i][c];
      for (let j=c;j<cols;j++) R[i][j] -= f*R[r][j];
    }
    r++;
  }
  // round near-zero
  for (let i=0;i<rows;i++) for (let j=0;j<cols;j++) if (Math.abs(R[i][j])<1e-12) R[i][j]=0;
  return R;
}

/* ---------- UI wiring ---------- */
function init() {
  // initial matrices
  createMatrixTable('matrixA',2,2,0);
  createMatrixTable('matrixB',2,2,0);

  document.getElementById('resizeA').addEventListener('click',()=>{
    const r = Math.max(1, parseInt(document.getElementById('rowsA').value));
    const c = Math.max(1, parseInt(document.getElementById('colsA').value));
    createMatrixTable('matrixA', r, c, 0);
  });
  document.getElementById('resizeB').addEventListener('click',()=>{
    const r = Math.max(1, parseInt(document.getElementById('rowsB').value));
    const c = Math.max(1, parseInt(document.getElementById('colsB').value));
    createMatrixTable('matrixB', r, c, 0);
  });

  document.getElementById('fillA').addEventListener('click',()=>{
    // example
    createMatrixTable('matrixA',2,2,0);
    const inputs = document.querySelectorAll('#matrixA input');
    const vals = [ [1,2],[3,4] ];
    inputs.forEach(inp=> inp.value = vals[parseInt(inp.dataset.r)][parseInt(inp.dataset.c)]);
    document.getElementById('rowsA').value = 2; document.getElementById('colsA').value = 2;
  });
  document.getElementById('fillB').addEventListener('click',()=>{
    createMatrixTable('matrixB',2,2,0);
    const inputs = document.querySelectorAll('#matrixB input');
    const vals = [ [5,6],[7,8] ];
    inputs.forEach(inp=> inp.value = vals[parseInt(inp.dataset.r)][parseInt(inp.dataset.c)]);
    document.getElementById('rowsB').value = 2; document.getElementById('colsB').value = 2;
  });

  document.getElementById('compute').addEventListener('click', ()=>{
    const op = document.getElementById('operation').value;
    const A = readMatrix('matrixA');
    const B = readMatrix('matrixB');
    const resultEl = document.getElementById('result');
    try {
      let R;
      switch(op){
        case 'add': R = add(A,B); resultEl.innerHTML = showMatrixAsHTML(R); break;
        case 'sub': R = sub(A,B); resultEl.innerHTML = showMatrixAsHTML(R); break;
        case 'mul': R = mul(A,B); resultEl.innerHTML = showMatrixAsHTML(R); break;
        case 'detA': { const d = det(A); resultEl.innerHTML = '<pre>det(A) = '+formatNum(d)+'</pre>'; break;}
        case 'detB': { const d = det(B); resultEl.innerHTML = '<pre>det(B) = '+formatNum(d)+'</pre>'; break;}
        case 'invA': { const I = inverse(A); resultEl.innerHTML = showMatrixAsHTML(I); break;}
        case 'invB': { const I = inverse(B); resultEl.innerHTML = showMatrixAsHTML(I); break;}
        case 'transA': R = transpose(A); resultEl.innerHTML = showMatrixAsHTML(R); break;
        case 'transB': R = transpose(B); resultEl.innerHTML = showMatrixAsHTML(R); break;
        case 'rrefA': R = rref(A); resultEl.innerHTML = showMatrixAsHTML(R); break;
        case 'rrefB': R = rref(B); resultEl.innerHTML = showMatrixAsHTML(R); break;
        default: resultEl.innerHTML = '<em class="muted">Operasi tidak dikenali</em>';
      }
    } catch (err) {
      resultEl.innerHTML = '<div style="color:#b91c1c"><strong>Error:</strong> '+String(err)+'</div>';
    }
  });

  document.getElementById('clear').addEventListener('click',()=>{
    createMatrixTable('matrixA',2,2,0);
    createMatrixTable('matrixB',2,2,0);
    document.getElementById('rowsA').value = 2; document.getElementById('colsA').value = 2;
    document.getElementById('rowsB').value = 2; document.getElementById('colsB').value = 2;
    document.getElementById('result').innerHTML = '<em class="muted">Belum ada perhitungan.</em>';
  });
}

init();
</script>
</body>
</html>
