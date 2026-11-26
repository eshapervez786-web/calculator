<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Calculator</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  :root {
    box-sizing: border-box;
    padding-top: env(safe-area-inset-top, 0px);
    padding-bottom: env(safe-area-inset-bottom, 0px);
    color-scheme: light;
    --bg: #e4e8e6;
    --body: #f4f6f4;
    --screen: #1f2a2e;
    --screen-text: #eef3f0;
    --screen-dim: #8fa3a0;
    --key: #ffffff;
    --key-text: #1f2a2e;
    --key-shadow: #c3cbc8;
    --fn: #d3dcd9;
    --op: #2f6f6a;
    --op-text: #ffffff;
    --eq: #e0b43a;
    --eq-text: #1f2a2e;
    --focus: #2f6f6a;
  }
  @media (prefers-color-scheme: dark) {
    :root:not([data-theme="light"]) {
      color-scheme: dark;
      --bg: #12181a;
      --body: #1a2326;
      --screen: #0d1214;
      --key: #27343a;
      --key-text: #eef3f0;
      --key-shadow: #0d1214;
      --fn: #34454c;
      --op: #3f8a83;
      --focus: #e0b43a;
    }
  }
  :root[data-theme="dark"] {
    color-scheme: dark;
    --bg: #12181a;
    --body: #1a2326;
    --screen: #0d1214;
    --key: #27343a;
    --key-text: #eef3f0;
    --key-shadow: #0d1214;
    --fn: #34454c;
    --op: #3f8a83;
    --focus: #e0b43a;
  }
  html { scroll-padding-top: env(safe-area-inset-top, 0px); height: 100%; }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    min-height: 100%;
    display: grid;
    place-items: center;
    padding: 20px;
    background: var(--bg);
    font-family: "DM Mono", ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  }
  .calc {
    width: 100%;
    max-width: 340px;
    background: var(--body);
    border-radius: 22px;
    padding: 16px;
    box-shadow: 0 18px 40px -18px rgba(20, 30, 30, .45);
  }
  .screen {
    background: var(--screen);
    color: var(--screen-text);
    border-radius: 12px;
    padding: 14px 16px 12px;
    margin-bottom: 14px;
    text-align: right;
    min-height: 104px;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    overflow: hidden;
  }
  .history {
    min-height: 1.4em;
    font-size: 14px;
    color: var(--screen-dim);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .value {
    font-size: 44px;
    font-weight: 500;
    line-height: 1.15;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: clip;
  }
  .value.error { font-size: 22px; color: #f0a79a; }
  .keys {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
  }
  button {
    font: inherit;
    font-size: 20px;
    height: 58px;
    border: 0;
    border-radius: 12px;
    background: var(--key);
    color: var(--key-text);
    box-shadow: 0 3px 0 var(--key-shadow);
    cursor: pointer;
    transition: transform .06s ease, box-shadow .06s ease;
    -webkit-tap-highlight-color: transparent;
  }
  button:active { transform: translateY(3px); box-shadow: 0 0 0 var(--key-shadow); }
  button:focus-visible { outline: 3px solid var(--focus); outline-offset: 2px; }
  button.fn { background: var(--fn); }
  button.op { background: var(--op); color: var(--op-text); box-shadow: 0 3px 0 rgba(0, 0, 0, .25); }
  button.op.active { outline: 3px solid var(--eq); outline-offset: 2px; }
  button.eq { background: var(--eq); color: var(--eq-text); box-shadow: 0 3px 0 rgba(0, 0, 0, .25); }
  button.zero { grid-column: span 2; }
  @media (prefers-reduced-motion: reduce) { button { transition: none; } }
</style>
</head>
<body>
  <main class="calc" aria-label="Calculator">
    <div class="screen" role="group" aria-label="Display">
      <div class="history" id="history"></div>
      <div class="value" id="value" aria-live="polite">0</div>
    </div>
    <div class="keys" id="keys">
      <button class="fn" data-action="clear">AC</button>
      <button class="fn" data-action="negate" aria-label="Plus or minus">±</button>
      <button class="fn" data-action="percent" aria-label="Percent">%</button>
      <button class="op" data-op="÷" aria-label="Divide">÷</button>

      <button data-digit="7">7</button>
      <button data-digit="8">8</button>
      <button data-digit="9">9</button>
      <button class="op" data-op="×" aria-label="Multiply">×</button>

      <button data-digit="4">4</button>
      <button data-digit="5">5</button>
      <button data-digit="6">6</button>
      <button class="op" data-op="−" aria-label="Subtract">−</button>

      <button data-digit="1">1</button>
      <button data-digit="2">2</button>
      <button data-digit="3">3</button>
      <button class="op" data-op="+" aria-label="Add">+</button>

      <button class="zero" data-digit="0">0</button>
      <button data-action="decimal" aria-label="Decimal point">.</button>
      <button class="eq" data-action="equals" aria-label="Equals">=</button>
    </div>
  </main>

<script>
(() => {
  const valueEl = document.getElementById('value');
  const historyEl = document.getElementById('history');
  const keysEl = document.getElementById('keys');

  // State
  let current = '0';       // number being typed (string)
  let stored = null;       // left operand (number)
  let operator = null;     // '+', '−', '×', '÷'
  let overwrite = true;    // next digit replaces the display
  let lastOp = null;       // for repeated equals
  let lastOperand = null;
  let error = false;

  const MAX_DIGITS = 12;

  function compute(a, op, b) {
    switch (op) {
      case '+': return a + b;
      case '−': return a - b;
      case '×': return a * b;
      case '÷': return b === 0 ? NaN : a / b;
    }
    return b;
  }

  // Round away floating-point noise (0.1 + 0.2 -> 0.3)
  function clean(n) {
    return parseFloat(n.toPrecision(12));
  }

  function format(str) {
    if (error) return str;
    const neg = str.startsWith('-');
    const body = neg ? str.slice(1) : str;
    const [int, dec] = body.split('.');
    const grouped = int.replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    return (neg ? '-' : '') + grouped + (dec !== undefined ? '.' + dec : '');
  }

  function numToString(n) {
    if (!isFinite(n)) return null;
    let s = String(clean(n));
    if (s.includes('e')) s = n.toExponential(6).replace(/\.?0+e/, 'e');
    return s;
  }

  function showError(msg) {
    error = true;
    current = msg;
    stored = null;
    operator = null;
    overwrite = true;
  }

  function render() {
    valueEl.textContent = format(current);
    valueEl.classList.toggle('error', error);
    // shrink long numbers to fit
    const len = valueEl.textContent.length;
    valueEl.style.fontSize = error ? '' : len > 14 ? '26px' : len > 11 ? '32px' : len > 8 ? '38px' : '';
    historyEl.textContent = stored !== null && operator
      ? format(numToString(stored)) + ' ' + operator
      : '';
    document.querySelectorAll('button.op').forEach(b =>
      b.classList.toggle('active', !!operator && overwrite && b.dataset.op === operator));
  }

  function clearErrorIfNeeded() {
    if (error) { error = false; current = '0'; overwrite = true; }
  }

  function inputDigit(d) {
    clearErrorIfNeeded();
    if (overwrite) { current = d; overwrite = false; }
    else {
      if (current.replace(/[-.]/g, '').length >= MAX_DIGITS) return;
      current = current === '0' ? d : current === '-0' ? '-' + d : current + d;
    }
  }

  function inputDecimal() {
    clearErrorIfNeeded();
    if (overwrite) { current = '0.'; overwrite = false; return; }
    if (!current.includes('.')) current += '.';
  }

  function chooseOperator(op) {
    if (error) return;
    if (operator && !overwrite) {
      const result = compute(stored, operator, parseFloat(current));
      const s = numToString(result);
      if (s === null) { showError('Cannot divide by 0'); return; }
      stored = parseFloat(s);
      current = s;
    } else if (!operator) {
      stored = parseFloat(current);
    }
    operator = op;
    overwrite = true;
    lastOp = null;
  }

  function equals() {
    if (error) return;
    let a, op, b;
    if (operator) {
      a = stored; op = operator; b = parseFloat(current);
      lastOp = op; lastOperand = b;
    } else if (lastOp) {          // repeat last operation
      a = parseFloat(current); op = lastOp; b = lastOperand;
    } else return;
    const result = compute(a, op, b);
    const s = numToString(result);
    if (s === null) { showError('Cannot divide by 0'); return; }
    historyEl.textContent = format(numToString(a)) + ' ' + op + ' ' + format(numToString(b)) + ' =';
    current = s;
    stored = null;
    operator = null;
    overwrite = true;
    valueEl.textContent = format(current);
    valueEl.classList.remove('error');
    // keep the expression in history after render
    const expr = historyEl.textContent;
    render();
    historyEl.textContent = expr;
    return true;
  }

  function clearAll() {
    current = '0'; stored = null; operator = null;
    overwrite = true; lastOp = null; lastOperand = null; error = false;
  }

  function backspace() {
    if (error) { clearAll(); return; }
    if (overwrite) return;
    current = current.length > 1 && !(current.length === 2 && current.startsWith('-'))
      ? current.slice(0, -1) : '0';
  }

  function negate() {
    if (error) return;
    if (current === '0' && overwrite) return;
    current = current.startsWith('-') ? current.slice(1) : '-' + current;
  }

  function percent() {
    if (error) return;
    const n = parseFloat(current);
    // 200 + 10% -> 10% of 200; otherwise divide by 100
    const result = stored !== null && operator && (operator === '+' || operator === '−')
      ? stored * n / 100
      : n / 100;
    current = numToString(result);
    overwrite = true;
  }

  // Click handling
  keysEl.addEventListener('click', e => {
    const btn = e.target.closest('button');
    if (!btn) return;
    if (btn.dataset.digit) inputDigit(btn.dataset.digit);
    else if (btn.dataset.op) chooseOperator(btn.dataset.op);
    else {
      switch (btn.dataset.action) {
        case 'clear': clearAll(); break;
        case 'decimal': inputDecimal(); break;
        case 'negate': negate(); break;
        case 'percent': percent(); break;
        case 'equals': if (equals()) return; break;
      }
    }
    render();
  });

  // Keyboard support
  const opKeys = { '+': '+', '-': '−', '*': '×', 'x': '×', '/': '÷' };
  document.addEventListener('keydown', e => {
    if (e.ctrlKey || e.metaKey || e.altKey) return;
    const k = e.key;
    if (/^[0-9]$/.test(k)) inputDigit(k);
    else if (k === '.' || k === ',') inputDecimal();
    else if (opKeys[k]) { e.preventDefault(); chooseOperator(opKeys[k]); }
    else if (k === 'Enter' || k === '=') {
      e.preventDefault();
      if (equals()) return;
    }
    else if (k === 'Backspace') backspace();
    else if (k === 'Escape' || k === 'Delete') clearAll();
    else if (k === '%') percent();
    else return;
    render();
  });

  render();
})();
</script>
</body>
</html>