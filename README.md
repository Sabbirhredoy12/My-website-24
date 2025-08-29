<!DOCTYPE html>
<html lang="bn">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Pro Calculator</title>
  <style>
    :root{
      --bg:#0f172a; /* slate-900 */
      --panel:#111827; /* gray-900 */
      --panel-2:#1f2937; /* gray-800 */
      --text:#e5e7eb; /* gray-200 */
      --sub:#9ca3af; /* gray-400 */
      --accent:#22d3ee; /* cyan-400 */
      --key:#111827;
      --key-2:#0b1220;
      --op:#0ea5e9; /* sky-500 */
      --warn:#ef4444;
      --ok:#10b981;
      --shadow: 0 10px 25px rgba(0,0,0,.35);
      --radius: 20px;
    }
    [data-theme="light"]{
      --bg:#f8fafc; /* slate-50 */
      --panel:#ffffff; /* white */
      --panel-2:#f1f5f9; /* slate-100 */
      --text:#0f172a; /* slate-900 */
      --sub:#475569; /* slate-600 */
      --key:#ffffff;
      --key-2:#e2e8f0;
      --op:#0284c7;
      --warn:#dc2626;
      --ok:#059669;
      --shadow: 0 10px 25px rgba(2,6,23,.12);
    }

    * { box-sizing: border-box; }
    body{
      margin:0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Noto Sans, Arial, "Apple Color Emoji","Segoe UI Emoji";
      background: radial-gradient(1200px 800px at 80% -10%, rgba(34,211,238,.15), transparent 60%),
                  radial-gradient(900px 700px at -10% 110%, rgba(56,189,248,.15), transparent 60%),
                  var(--bg);
      color:var(--text);
      min-height:100svh; display:grid; place-items:center; padding:20px;
    }
    .app{
      width:min(980px, 100%);
      display:grid; gap:16px;
      grid-template-columns: 1.2fr .8fr;
    }
    @media (max-width: 880px){ .app{ grid-template-columns: 1fr; } }

    .calc{
      background:linear-gradient(180deg, var(--panel), var(--panel-2));
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      padding:18px; display:grid; gap:14px;
    }
    .topbar{ display:flex; align-items:center; gap:10px; justify-content:space-between; }
    .toggles{ display:flex; gap:10px; align-items:center; flex-wrap:wrap; }
    .chip{ border:1px solid rgba(148,163,184,.25); color:var(--text); background:transparent; padding:8px 12px; border-radius:999px; cursor:pointer; }
    .chip.active{ background:rgba(34,211,238,.12); border-color:rgba(34,211,238,.55); }
    .theme{
      border:none; background:var(--key-2); color:var(--text); border-radius:12px; padding:8px 12px; cursor:pointer;
    }

    .display{ background:rgba(15,23,42,.35); border-radius:16px; padding:14px; min-height:108px; display:flex; flex-direction:column; justify-content:center; gap:8px; }
    .expr{ font-size:14px; color:var(--sub); min-height:20px; word-break:break-all; }
    .result{ font-variant-numeric: tabular-nums; font-size:36px; line-height:1.15; }
    .warning{ color:var(--warn); font-size:12px; min-height:16px; }

    .keys{ display:grid; grid-template-columns: repeat(6, 1fr); gap:10px; }
    .btn{ border:none; border-radius:14px; padding:14px; font-size:16px; background:var(--key); color:var(--text); cursor:pointer; box-shadow: inset 0 -2px 0 rgba(255,255,255,.03); }
    .btn:hover{ filter:brightness(1.07); }
    .btn:active{ transform: translateY(1px); }
    .btn.op{ background: linear-gradient(180deg, rgba(2,132,199,.15), rgba(2,132,199,.08)); border:1px solid rgba(2,132,199,.35); }
    .btn.eq{ background: linear-gradient(180deg, rgba(16,185,129,.2), rgba(16,185,129,.12)); border:1px solid rgba(16,185,129,.5); font-weight:700; }
    .btn.span-2{ grid-column: span 2; }

    .side{
      display:grid; gap:16px;
    }
    .card{
      background:linear-gradient(180deg, var(--panel), var(--panel-2)); border-radius: var(--radius); box-shadow: var(--shadow); padding:16px;
    }
    .card h3{ margin:0 0 10px; font-size:16px; color:var(--sub) }
    .history{ max-height:420px; overflow:auto; }
    .history-item{ display:flex; justify-content:space-between; align-items:center; gap:10px; padding:10px; border-radius:10px; background:rgba(2,6,23,.18); cursor:pointer; }
    .history-item:hover{ background:rgba(2,6,23,.28); }
    .history-item .h-exp{ font-size:12px; color:var(--sub); }
    .history-item .h-res{ font-size:16px; }

    .mem{ display:flex; gap:8px; flex-wrap:wrap; }
    .muted{ color:var(--sub); font-size:12px; }
    .kbd{ font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; background:rgba(148,163,184,.16); padding:2px 6px; border-radius:6px; }
  </style>
</head>
<body>
  <div class="app" id="app">
    <section class="calc" aria-label="Calculator">
      <div class="topbar">
        <div class="toggles">
          <button class="chip" id="deg">DEG</button>
          <button class="chip" id="rad">RAD</button>
          <span class="muted">| Keyboard: <span class="kbd">0-9</span>, <span class="kbd">+ - * / ^</span>, <span class="kbd">( )</span>, <span class="kbd">Enter</span>, <span class="kbd">Backspace</span>, <span class="kbd">Esc</span></span>
        </div>
        <button class="theme" id="themeToggle" title="Toggle light/dark">🌙 Theme</button>
      </div>

      <div class="display" role="status" aria-live="polite">
        <div class="expr" id="expr"></div>
        <div class="result" id="result">0</div>
        <div class="warning" id="warn"></div>
      </div>

      <div class="keys" id="keys">
        <!-- Row 1 -->
        <button class="btn op" data-action="clear">AC</button>
        <button class="btn op" data-action="del">DEL</button>
        <button class="btn op" data-fn="percent">%</button>
        <button class="btn op" data-fn="abs">abs</button>
        <button class="btn op" data-const="pi">π</button>
        <button class="btn op" data-const="e">e</button>
        <!-- Row 2 -->
        <button class="btn op" data-fn="sin">sin</button>
        <button class="btn op" data-fn="cos">cos</button>
        <button class="btn op" data-fn="tan">tan</button>
        <button class="btn op" data-fn="sqrt">√</button>
        <button class="btn op" data-op="^">x^y</button>
        <button class="btn op" data-fn="inv">1/x</button>
        <!-- Row 3 -->
        <button class="btn" data-char="7">7</button>
        <button class="btn" data-char="8">8</button>
        <button class="btn" data-char="9">9</button>
        <button class="btn op" data-op="/">÷</button>
        <button class="btn op" data-fn="square">x²</button>
        <button class="btn op" data-fn="cube">x³</button>
        <!-- Row 4 -->
        <button class="btn" data-char="4">4</button>
        <button class="btn" data-char="5">5</button>
        <button class="btn" data-char="6">6</button>
        <button class="btn op" data-op="*">×</button>
        <button class="btn op" data-fn="ln">ln</button>
        <button class="btn op" data-fn="log">log</button>
        <!-- Row 5 -->
        <button class="btn" data-char="1">1</button>
        <button class="btn" data-char="2">2</button>
        <button class="btn" data-char="3">3</button>
        <button class="btn op" data-op="-">−</button>
        <button class="btn op" data-fn="fact">n!</button>
        <button class="btn op" data-fn="sign">±</button>
        <!-- Row 6 -->
        <button class="btn span-2" data-char="0">0</button>
        <button class="btn" data-char=".">.</button>
        <button class="btn op" data-op="+">+</button>
        <button class="btn eq" data-action="equals">=</button>
        <!-- Row 7 -->
        <button class="btn op" data-char="(">(</button>
        <button class="btn op" data-char=")">)</button>
        <button class="btn op" data-action="ans">ANS</button>
        <button class="btn op" data-action="mc">MC</button>
        <button class="btn op" data-action="mr">MR</button>
        <button class="btn op" data-action="mplus">M+</button>
        <button class="btn op" data-action="mminus">M−</button>
      </div>
    </section>

    <aside class="side">
      <div class="card">
        <h3>History</h3>
        <div id="history" class="history" aria-label="History"></div>
      </div>
      <div class="card">
        <h3>Tips</h3>
        <ul class="muted">
          <li>Trig মোড <strong>DEG/RAD</strong> টগল করুন।</li>
          <li><span class="kbd">^</span> দিয়ে ঘাত, যেমন <em>2^8</em>.</li>
          <li>কীবোর্ডে <span class="kbd">Enter</span>=<strong>=</strong>, <span class="kbd">Esc</span>=AC, <span class="kbd">Backspace</span>=DEL।</li>
          <li><strong>ANS</strong> শেষ ফলাফল ইনসার্ট করে।</li>
          <li>History আইটেমে ক্লিক করলে আবার ব্যবহার করতে পারবেন।</li>
        </ul>
      </div>
    </aside>
  </div>

<script>
(() => {
  const exprEl = document.getElementById('expr');
  const resultEl = document.getElementById('result');
  const warnEl = document.getElementById('warn');
  const historyEl = document.getElementById('history');
  const degBtn = document.getElementById('deg');
  const radBtn = document.getElementById('rad');
  const themeToggle = document.getElementById('themeToggle');

  let state = {
    expr: '',
    lastAns: 0,
    memory: 0,
    degMode: true,
    theme: localStorage.getItem('calc_theme') || 'dark',
    history: JSON.parse(localStorage.getItem('calc_history')||'[]'),
  };

  document.documentElement.setAttribute('data-theme', state.theme === 'light' ? 'light' : 'dark');

  function setMode(deg){
    state.degMode = deg; updateModeChips();
  }
  function updateModeChips(){
    (state.degMode ? degBtn : radBtn).classList.add('active');
    (state.degMode ? radBtn : degBtn).classList.remove('active');
  }
  updateModeChips();

  function formatNumber(x){
    if (!isFinite(x)) return '∞';
    // 12 significant digits, strip trailing zeros
    let s = Number(x).toPrecision(12);
    // remove trailing zeros in decimal
    if (s.indexOf('e') === -1){
      s = s.replace(/\.0+$/, '').replace(/(\.[0-9]*?)0+$/, '$1');
    }
    return s;
  }

  function show(expr, res, warn=''){
    exprEl.textContent = expr;
    resultEl.textContent = res;
    warnEl.textContent = warn;
  }

  function pushHistory(exp, res){
    state.history.unshift({exp, res});
    if (state.history.length > 50) state.history.pop();
    localStorage.setItem('calc_history', JSON.stringify(state.history));
    renderHistory();
  }

  function renderHistory(){
    historyEl.innerHTML = '';
    state.history.forEach((h, i) =>{
      const item = document.createElement('div');
      item.className = 'history-item';
      item.innerHTML = `<div><div class="h-exp">${escapeHtml(h.exp)}</div><div class="h-res">${escapeHtml(h.res)}</div></div><button class="theme" data-idx="${i}">Use</button>`;
      item.addEventListener('click', (e)=>{
        if (e.target.tagName === 'BUTTON'){
          state.expr = h.res; computePreview();
        } else {
          state.expr = h.exp; computePreview();
        }
      });
      historyEl.appendChild(item);
    });
  }

  function escapeHtml(s){
    return s.replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;','\'':'&#39;'}[c]));
  }

  function insert(s){
    state.expr += s; computePreview();
  }

  function clearAll(){ state.expr=''; show('', '0', ''); }
  function delOne(){ state.expr = state.expr.slice(0, -1); computePreview(); }

  // --- Tokenizer & Parser (Shunting-yard) ---
  const precedence = { '+':2, '-':2, '*':3, '/':3, '^':4 };
  const rightAssoc = { '^':true };

  function isNumericToken(t){ return /^(?:\d*\.?\d+|\d+\.?\d*)(?:e[+-]?\d+)?$/i.test(t); }

  function tokenize(input){
    const tokens=[]; let i=0;
    while(i < input.length){
      const ch = input[i];
      if (ch===' ') { i++; continue; }
      if (/[0-9\.]/.test(ch)){
        let num = ch; i++;
        while(i<input.length && /[0-9\.eE+-]/.test(input[i])){
          // prevent signs unless preceded by e/E
          if ((input[i]=='+'||input[i]=='-') && !(input[i-1]=='e'||input[i-1]=='E')) break;
          num += input[i++];
        }
        tokens.push(num);
        continue;
      }
      if (/[a-zA-Z]/.test(ch)){
        let id = ch; i++;
        while(i<input.length && /[a-zA-Z0-9_]/.test(input[i])) id += input[i++];
        tokens.push(id);
        continue;
      }
      if (/[+\-*/^(),]/.test(ch)) { tokens.push(ch); i++; continue; }
      // Unknown char
      throw new Error('Invalid character: '+ch);
    }
    return tokens;
  }

  function toRPN(tokens){
    const out=[]; const stack=[];
    let prev = null;
    for (let t of tokens){
      if (isNumericToken(t) || t==='pi' || t==='e' || t==='ANS'){
        out.push(t);
      } else if (/^[a-zA-Z][a-zA-Z0-9_]*$/.test(t)){
        // function
        stack.push(t);
      } else if (t===','){
        while(stack.length && stack[stack.length-1] !== '(') out.push(stack.pop());
      } else if (t in precedence){
        // handle unary minus
        if (t==='-' && (prev===null || (prev in precedence) || prev==='(' || prev===',')){
          // treat as unary negation: push 0 then subtract
          out.push('0');
        }
        while(stack.length){
          const top = stack[stack.length-1];
          if (top in precedence && ((rightAssoc[t] ? precedence[t] < precedence[top] : precedence[t] <= precedence[top]))){
            out.push(stack.pop());
          } else break;
        }
        stack.push(t);
      } else if (t==='('){
        stack.push(t);
      } else if (t===')'){
        while(stack.length && stack[stack.length-1] !== '(') out.push(stack.pop());
        if (!stack.length) throw new Error('Mismatched parentheses');
        stack.pop(); // remove '('
        if (stack.length && /^[a-zA-Z]/.test(stack[stack.length-1])){
          out.push(stack.pop()); // function call
        }
      } else {
        throw new Error('Unknown token: '+t);
      }
      prev = t;
    }
    while(stack.length){
      const t = stack.pop();
      if (t==='('||t===')') throw new Error('Mismatched parentheses');
      out.push(t);
    }
    return out;
  }

  function toRad(x){ return state.degMode ? x*Math.PI/180 : x; }
  function fromRad(x){ return state.degMode ? x*180/Math.PI : x; }

  function evalRPN(rpn){
    const st=[];
    for (let t of rpn){
      if (isNumericToken(t)) st.push(parseFloat(t));
      else if (t==='pi') st.push(Math.PI);
      else if (t==='e') st.push(Math.E);
      else if (t==='ANS') st.push(state.lastAns);
      else if (t in precedence){
        if (st.length<2) throw new Error('Bad expression');
        const b = st.pop(); const a = st.pop();
        switch(t){
          case '+': st.push(a+b); break;
          case '-': st.push(a-b); break;
          case '*': st.push(a*b); break;
          case '/': if (b===0) throw new Error('Division by zero'); st.push(a/b); break;
          case '^': st.push(Math.pow(a,b)); break;
        }
      } else {
        // functions
        const fn = t.toLowerCase();
        let x, y;
        switch(fn){
          case 'sin': x = st.pop(); st.push(Math.sin(toRad(x))); break;
          case 'cos': x = st.pop(); st.push(Math.cos(toRad(x))); break;
          case 'tan': x = st.pop(); st.push(Math.tan(toRad(x))); break;
          case 'asin': x = st.pop(); st.push(fromRad(Math.asin(x))); break;
          case 'acos': x = st.pop(); st.push(fromRad(Math.acos(x))); break;
          case 'atan': x = st.pop(); st.push(fromRad(Math.atan(x))); break;
          case 'sqrt': x = st.pop(); if (x<0) throw new Error('Invalid √'); st.push(Math.sqrt(x)); break;
          case 'ln': x = st.pop(); if (x<=0) throw new Error('Invalid ln'); st.push(Math.log(x)); break;
          case 'log': x = st.pop(); if (x<=0) throw new Error('Invalid log'); st.push(Math.log10(x)); break;
          case 'abs': x = st.pop(); st.push(Math.abs(x)); break;
          case 'inv': x = st.pop(); if (x===0) throw new Error('Division by zero'); st.push(1/x); break;
          case 'square': x=st.pop(); st.push(x*x); break;
          case 'cube': x=st.pop(); st.push(x*x*x); break;
          case 'fact': x=st.pop(); if (x<0 || !isFinite(x)) throw new Error('Invalid n!'); st.push(factorial(x)); break;
          case 'percent': x=st.pop(); st.push(x/100); break;
          case 'sign': x=st.pop(); st.push(-x); break;
          default: throw new Error('Unknown function: '+t);
        }
      }
    }
    if (st.length!==1) throw new Error('Bad expression');
    return st[0];
  }

  function factorial(n){
    // support integer n up to 170 (beyond Infinity)
    if (Math.abs(n - Math.round(n)) > 1e-10) throw new Error('n! needs integer');
    n = Math.round(n);
    if (n>170) throw new Error('n! overflow');
    let r=1; for (let i=2;i<=n;i++) r*=i; return r;
  }

  function computePreview(){
    show(state.expr, state.expr? '…':'0', '');
    if (!state.expr) return;
    try{
      const tokens = tokenize(state.expr);
      const rpn = toRPN(tokens);
      const val = evalRPN(rpn);
      show(state.expr, formatNumber(val), '');
      return val;
    } catch(err){
      show(state.expr, '—', err.message);
    }
  }

  function equals(){
    try{
      const tokens = tokenize(state.expr);
      const rpn = toRPN(tokens);
      const val = evalRPN(rpn);
      const res = formatNumber(val);
      pushHistory(state.expr, res);
      state.lastAns = val;
      state.expr = res; // keep result for chaining
      show(state.expr, res, '');
    }catch(err){
      warnEl.textContent = err.message;
    }
  }

  // --- Memory ops ---
  function mc(){ state.memory = 0; toast('Memory cleared'); }
  function mr(){ insert(String(state.memory)); }
  function mplus(){ const pv=computePreview(); if (pv!=null) { state.memory += pv; toast('Added to M'); } }
  function mminus(){ const pv=computePreview(); if (pv!=null) { state.memory -= pv; toast('Subtracted from M'); } }

  function toast(msg){
    warnEl.textContent = msg; setTimeout(()=>{ if (warnEl.textContent===msg) warnEl.textContent=''; }, 1200);
  }

  // --- Wire buttons ---
  document.getElementById('keys').addEventListener('click', (e)=>{
    const b = e.target.closest('button'); if (!b) return;
    const c = b.dataset.char, op=b.dataset.op, fn=b.dataset.fn, act=b.dataset.action, cst=b.dataset.const;
    if (c) insert(c);
    else if (op) insert(op);
    else if (fn) insert(fn+'(');
    else if (cst) insert(cst);
    else if (act){
      if (act==='clear') clearAll();
      else if (act==='del') delOne();
      else if (act==='equals') equals();
      else if (act==='ans') insert('ANS');
      else if (act==='mc') mc();
      else if (act==='mr') mr();
      else if (act==='mplus') mplus();
      else if (act==='mminus') mminus();
    }
  });

  // Close function call with ) if user clicks function name again
  // (Kept simple: user types ')' when needed.)

  // --- Keyboard input ---
  const allowed = new Set(['0','1','2','3','4','5','6','7','8','9','.','+','-','*','/','^','(',')']);
  window.addEventListener('keydown', (e)=>{
    if (e.key==='Enter'){ e.preventDefault(); equals(); return; }
    if (e.key==='Escape'){ e.preventDefault(); clearAll(); return; }
    if (e.key==='Backspace'){ delOne(); return; }
    if (allowed.has(e.key)) insert(e.key);
  });

  // --- Mode & Theme ---
  degBtn.addEventListener('click', ()=> setMode(true));
  radBtn.addEventListener('click', ()=> setMode(false));
  themeToggle.addEventListener('click', ()=>{
    state.theme = (state.theme==='dark'?'light':'dark');
    document.documentElement.setAttribute('data-theme', state.theme);
    localStorage.setItem('calc_theme', state.theme);
  });

  // Init
  renderHistory(); computePreview();
})();
</script>
</body>
</html>
