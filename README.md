<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Sorting Algorithm Visualizer</title>
  <style>
    :root{
      --bg:#0f172a;
      --panel:#0b1220;
      --muted:#94a3b8;
      --accent1:linear-gradient(180deg,#38bdf8,#0ea5e9);
      --accent2:#f97316;
      --btn:#1e293b;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      background:var(--bg);
      color:#f8fafc;
      font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      display:flex;
      flex-direction:column;
      min-height:100vh;
      align-items:center;
      gap:14px;
      padding:28px;
    }
    header{ text-align:center }
    h1{ margin:0 0 4px; font-size:28px }
    p.lead{ margin:0; color:var(--muted); font-size:14px }

    .controls{
      display:flex;
      gap:8px;
      align-items:center;
      margin-top:8px;
      flex-wrap:wrap;
    }
    button, select, input[type="range"]{
      background:var(--btn);
      color:white;
      border:0;
      padding:9px 12px;
      border-radius:8px;
      cursor:pointer;
      font-size:14px;
    }
    button:disabled{ opacity:0.55; cursor:not-allowed }
    #bars{
      width:100%;
      max-width:980px;
      height:380px;
      border-radius:12px;
      background:rgba(255,255,255,0.02);
      display:flex;
      justify-content:center;
      align-items:flex-end;
      gap:6px;
      padding:18px;
      margin-top:14px;
      box-shadow:0 6px 30px rgba(2,6,23,0.6);
    }
    .bar{
      width:28px;
      border-radius:6px 6px 4px 4px;
      background:var(--accent1);
      display:flex;
      align-items:flex-end;
      justify-content:center;
      color:#021124;
      font-weight:700;
      transition:height 220ms ease, transform 120ms ease, background 180ms;
      box-shadow: 0 2px 8px rgba(2,6,23,0.5) inset;
      font-size:12px;
      padding-bottom:6px;
    }
    .bar.active{ background:var(--accent2); transform:translateY(-6px) scaleX(1.03); color:#fff }
    footer{ color:var(--muted); font-size:13px; margin-top:6px }
    @media (max-width:640px){
      .bar{ width:14px; font-size:9px; padding-bottom:4px }
    }
  </style>
</head>
<body>
  <header>
    <h1>Hitler's Sorting Algorithm Visualizer</h1>
    <p class="lead"></p>
  </header>

  <div class="controls" id="controls">
    <button id="randBtn">Randomize</button>

    <label style="display:flex;gap:8px;align-items:center;">
      Algorithm
      <select id="algorithm">
        <option value="bubble">Bubble Sort</option>
        <option value="selection">Selection Sort</option>
        <option value="insertion">Insertion Sort</option>
        <option value="merge">Merge Sort</option>
        <option value="quick">Quick Sort</option>
      </select>
    </label>

    <label style="display:flex;gap:8px;align-items:center;">
      Size
      <input id="sizeRange" type="range" min="6" max="60" value="22" />
    </label>

    <label style="display:flex;gap:8px;align-items:center;">
      Speed
      <input id="speedRange" type="range" min="20" max="500" value="120" />
    </label>

    <button id="startBtn">Start Sorting</button>
  </div>

  <div id="bars" aria-live="polite"></div>

  <footer>Tip: Click <strong>Randomize</strong> then pick an algorithm and press <strong>Start Sorting</strong>.</footer>

<script>
(() => {
  // ------------ state ------------
  let arr = [];
  const barsRoot = document.getElementById('bars');
  const randBtn = document.getElementById('randBtn');
  const startBtn = document.getElementById('startBtn');
  const algSelect = document.getElementById('algorithm');
  const sizeRange = document.getElementById('sizeRange');
  const speedRange = document.getElementById('speedRange');

  let isSorting = false;

  // ---------- utilities ----------
  function sleep(ms){ return new Promise(r => setTimeout(r, ms)); }
  function disableControls(val){
    [randBtn, startBtn, algSelect, sizeRange, speedRange].forEach(el => el.disabled = val);
  }

  function createBars(){
    barsRoot.innerHTML = '';
    const n = Number(sizeRange.value);
    arr = [];
    for(let i=0;i<n;i++){
      const v = Math.floor(Math.random()*90)+10; // 10..99
      arr.push(v);
      const bar = document.createElement('div');
      bar.className = 'bar';
      // height scaled to fit container (max height ~ 300px)
      bar.style.height = `${v * 3}px`;
      bar.textContent = v;
      barsRoot.appendChild(bar);
    }
  }

  function updateBar(index){
    const bars = barsRoot.children;
    const bar = bars[index];
    bar.style.height = `${arr[index] * 3}px`;
    bar.textContent = arr[index];
  }

  function mark(idx){
    const bars = barsRoot.children;
    if (bars[idx]) bars[idx].classList.add('active');
  }
  function unmark(idx){
    const bars = barsRoot.children;
    if (bars[idx]) bars[idx].classList.remove('active');
  }
  function unmarkAll(){
    const bars = barsRoot.children;
    for(let b of bars) b.classList.remove('active');
  }

  // ---------- Sorting algorithm templates (you must implement by hand) ----------
  // All functions use arr[] and updateBar() + mark/unmark and await sleep(speed)

  async function bubbleSort(){
    // Example implementation (hand-written):
    const n = arr.length;
    const speed = Number(speedRange.value);
    for(let i=0;i<n-1;i++){
      for(let j=0;j<n-i-1;j++){
        mark(j); mark(j+1);
        if (arr[j] > arr[j+1]){
          // swap
          const tmp = arr[j]; arr[j] = arr[j+1]; arr[j+1] = tmp;
          updateBar(j); updateBar(j+1);
        }
        await sleep(speed);
        unmark(j); unmark(j+1);
      }
    }
  }

  async function selectionSort(){
    const n = arr.length;
    const speed = Number(speedRange.value);
    for(let i=0;i<n-1;i++){
      let minIdx = i;
      mark(minIdx);
      for(let j=i+1;j<n;j++){
        mark(j);
        if (arr[j] < arr[minIdx]){
          unmark(minIdx);
          minIdx = j;
          mark(minIdx);
        } else {
          // keep j marked briefly for visibility then unmark
          await sleep(Math.max(20, speed/4));
          unmark(j);
        }
      }
      if (minIdx !== i){
        // swap
        const tmp = arr[i]; arr[i] = arr[minIdx]; arr[minIdx] = tmp;
        updateBar(i); updateBar(minIdx);
      }
      await sleep(speed);
      unmark(minIdx); unmark(i);
    }
  }

  async function insertionSort(){
    const n = arr.length;
    const speed = Number(speedRange.value);
    for(let i=1;i<n;i++){
      let key = arr[i];
      let j = i-1;
      mark(i);
      await sleep(Math.max(20, speed/4));
      while(j >= 0 && arr[j] > key){
        mark(j);
        arr[j+1] = arr[j];
        updateBar(j+1);
        unmark(j+1);
        j--;
        await sleep(speed);
      }
      arr[j+1] = key;
      updateBar(j+1);
      unmark(i);
      unmark(j+1);
      await sleep(Math.max(20, speed/3));
    }
  }

  // Merge sort helper functions
  async function mergeSortVisual(left, right){
    if (left >= right) return;
    const mid = Math.floor((left + right) / 2);
    await mergeSortVisual(left, mid);
    await mergeSortVisual(mid+1, right);
    await merge(left, mid, right);
  }

  async function merge(left, mid, right){
    const speed = Number(speedRange.value);
    const temp = [];
    let i = left, j = mid+1;
    while(i <= mid && j <= right){
      mark(i); mark(j);
      if (arr[i] <= arr[j]){
        temp.push(arr[i++]);
      } else {
        temp.push(arr[j++]);
      }
      await sleep(Math.max(20, speed/4));
      unmark(i-1); unmark(j-1);
    }
    while(i <= mid){ temp.push(arr[i++]); }
    while(j <= right){ temp.push(arr[j++]); }
    for(let k=left;k<=right;k++){
      arr[k] = temp[k-left];
      updateBar(k);
      mark(k);
      await sleep(Math.max(20, speed/3));
      unmark(k);
    }
  }

  // Quick sort helpers
  async function quickSortVisual(low, high){
    if (low < high){
      const p = await partition(low, high);
      await quickSortVisual(low, p-1);
      await quickSortVisual(p+1, high);
    }
  }

  async function partition(low, high){
    const speed = Number(speedRange.value);
    const pivot = arr[high];
    mark(high);
    let i = low-1;
    for(let j=low;j<high;j++){
      mark(j);
      if (arr[j] < pivot){
        i++;
        // swap arr[i], arr[j]
        const tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp;
        updateBar(i); updateBar(j);
        await sleep(speed);
      }
      unmark(j);
    }
    // place pivot
    [arr[i+1], arr[high]] = [arr[high], arr[i+1]];
    updateBar(i+1); updateBar(high);
    await sleep(Math.max(30, speed/2));
    unmark(high); unmark(i+1);
    return i+1;
  }

  // ---------- driver ----------
  async function startSorting(){
    if (isSorting) return;
    isSorting = true;
    disableControls(true);
    unmarkAll();
    const algo = algSelect.value;
    try {
      if (algo === 'bubble') await bubbleSort();
      else if (algo === 'selection') await selectionSort();
      else if (algo === 'insertion') await insertionSort();
      else if (algo === 'merge') await mergeSortVisual(0, arr.length - 1);
      else if (algo === 'quick') await quickSortVisual(0, arr.length - 1);
      // final flash of all bars
      for(let i=0;i<arr.length;i++){ mark(i); }
      await sleep(220);
      unmarkAll();
    } catch (e){
      console.error('Sorting failed:', e);
      alert('An error occurred in sorting. See console for details.');
    } finally {
      isSorting = false;
      disableControls(false);
    }
  }

  // ---------- events ----------
  randBtn.addEventListener('click', () => {
    if (isSorting) return;
    createBars();
  });
  startBtn.addEventListener('click', () => {
    startSorting();
  });
  sizeRange.addEventListener('input', () => {
    if (isSorting) return;
    createBars();
  });

  // initial
  createBars();
})();
</script>
</body>
</html>
