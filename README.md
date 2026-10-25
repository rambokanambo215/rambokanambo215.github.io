<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="theme-color" content="#101827">
  <title>ISMAIL STOPWATCH</title>
  <style>
    :root { color-scheme: dark; --bg:#101827; --panel:#1b2638; --text:#f5f7fb; --muted:#aebbd0; --accent:#69e0b0; }
    * { box-sizing: border-box; }
    body { margin:0; min-height:100vh; display:grid; place-items:center; padding:20px; background:radial-gradient(circle at top,#263b55,var(--bg) 60%); color:var(--text); font-family:system-ui,-apple-system,Segoe UI,Roboto,sans-serif; }
    main { width:min(100%,440px); padding:28px 22px; border:1px solid #34445c; border-radius:26px; background:rgba(27,38,56,.94); box-shadow:0 24px 70px #0005; text-align:center; }
    .eyebrow { color:var(--accent); font-size:.78rem; font-weight:800; letter-spacing:.2em; text-transform:uppercase; }
    h1 { margin:8px 0 28px; font-size:1.7rem; }
    #display { font-variant-numeric:tabular-nums; font-size:clamp(2.8rem,12vw,4.2rem); font-weight:750; letter-spacing:-.06em; white-space:nowrap; }
    .sub { color:var(--muted); font-size:.9rem; margin:8px 0 28px; }
    .controls { display:grid; grid-template-columns:1fr 1fr; gap:12px; }
    button { border:0; border-radius:14px; padding:14px 12px; font:inherit; font-weight:750; cursor:pointer; touch-action:manipulation; }
    button:focus-visible { outline:3px solid white; outline-offset:3px; }
    #start { background:var(--accent); color:#10251e; }
    #lap { background:#33445c; color:var(--text); }
    #reset { grid-column:1/-1; background:transparent; color:var(--muted); border:1px solid #465670; }
    .laps-head { display:flex; justify-content:space-between; align-items:center; margin:28px 0 10px; }
    .laps-head h2 { margin:0; font-size:1rem; }
    #laps { list-style:none; padding:0; margin:0; max-height:210px; overflow:auto; text-align:left; }
    #laps li { display:flex; justify-content:space-between; gap:12px; padding:11px 4px; border-bottom:1px solid #34445c; font-variant-numeric:tabular-nums; }
    #laps li span:first-child { color:var(--muted); }
    .empty { color:var(--muted); text-align:center; padding:14px 0; font-size:.9rem; }
    footer { margin-top:20px; color:var(--muted); font-size:.78rem; }
  </style>
</head>
<body>
  <main>
    <div class="eyebrow">Simple • Free • Offline</div>
    <h1>My Stopwatch</h1>
    <div id="display" role="timer" aria-live="off">00:00.00</div>
    <div class="sub" id="status">Ready when you are</div>
    <div class="controls">
      <button id="start" type="button">Start</button>
      <button id="lap" type="button" disabled>Lap</button>
      <button id="reset" type="button">Reset</button>
    </div>
    <div class="laps-head"><h2>Lap times</h2><span id="lap-count">0 laps</span></div>
    <ol id="laps"><li class="empty" id="empty">Your lap times will appear here.</li></ol>
    <footer>Works offline after you save this file on your device.</footer>
  </main>
  <script>
    const display = document.getElementById('display');
    const startBtn = document.getElementById('start');
    const lapBtn = document.getElementById('lap');
    const resetBtn = document.getElementById('reset');
    const lapsList = document.getElementById('laps');
    const statusText = document.getElementById('status');
    const lapCount = document.getElementById('lap-count');
    let running = false, startedAt = 0, elapsed = 0, frame = null, lastLap = 0, laps = 0;

    function format(ms) {
      const hundredths = Math.floor(ms / 10) % 100;
      const seconds = Math.floor(ms / 1000) % 60;
      const minutes = Math.floor(ms / 60000) % 60;
      const hours = Math.floor(ms / 3600000);
      const two = n => String(n).padStart(2, '0');
      return hours > 0 ? `${two(hours)}:${two(minutes)}:${two(seconds)}.${two(hundredths)}` : `${two(minutes)}:${two(seconds)}.${two(hundredths)}`;
    }
    function currentTime() { return elapsed + (running ? performance.now() - startedAt : 0); }
    function tick() { display.textContent = format(currentTime()); if (running) frame = requestAnimationFrame(tick); }
    startBtn.addEventListener('click', () => {
      if (running) {
        elapsed += performance.now() - startedAt;
        running = false;
        cancelAnimationFrame(frame);
        startBtn.textContent = 'Resume';
        lapBtn.disabled = true;
        statusText.textContent = 'Paused';
        display.textContent = format(elapsed);
      } else {
        startedAt = performance.now();
        running = true;
        startBtn.textContent = 'Pause';
        lapBtn.disabled = false;
        statusText.textContent = 'Stopwatch running';
        tick();
      }
    });
    lapBtn.addEventListener('click', () => {
      if (!running) return;
      const total = currentTime();
      const lapTime = total - lastLap;
      lastLap = total;
      laps++;
      const empty = document.getElementById('empty');
      if (empty) empty.remove();
      const li = document.createElement('li');
      const label = document.createElement('span');
      const split = document.createElement('span');
      label.textContent = `Lap ${laps}`;
      split.textContent = `${format(lapTime)}  ·  ${format(total)}`;
      li.append(label, split);
      lapsList.prepend(li);
      lapCount.textContent = `${laps} ${laps === 1 ? 'lap' : 'laps'}`;
    });
    resetBtn.addEventListener('click', () => {
      running = false;
      cancelAnimationFrame(frame);
      elapsed = 0; startedAt = 0; lastLap = 0; laps = 0;
      display.textContent = '00:00.00';
      startBtn.textContent = 'Start';
      lapBtn.disabled = true;
      statusText.textContent = 'Ready when you are';
      lapCount.textContent = '0 laps';
      lapsList.innerHTML = '<li class="empty" id="empty">Your lap times will appear here.</li>';
    });
  </script>
</body>
</html>
