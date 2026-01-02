<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>BankHack · Intrusion Login</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }

    :root {
      --term-bg: #020403;
      --term-fg: #00ff66;
      --term-fg-dim: #00b050;
      --accent: #00e6ff;
      --accent-soft: #0094ff;
      --error: #ff2a4f;
      --warn: #ffe066;
    }

    body {
      background:
        radial-gradient(circle at top, #061c10 0, #000 50%, #000 100%),
        radial-gradient(circle at bottom, #020510 0, #000 60%);
      color: var(--term-fg);
      font-family: "Courier New", monospace;
      font-size: 16px;              /* bigger base font */
      min-height: 100vh;
      overflow: hidden;
      display: flex;
      align-items: center;
      justify-content: center;
      perspective: 1300px;
      cursor: crosshair;
    }

    .power-vignette {
      position: fixed;
      inset: 0;
      background: radial-gradient(circle at center, #000 0, #000 40%, transparent 75%);
      opacity: 0.85;
      pointer-events: none;
      z-index: 50;
      animation: power-on 1.4s ease-out forwards;
    }

    .deck-wrapper {
      width: min(1200px, 100vw);
      height: min(700px, 100vh);   /* slightly taller */
      position: relative;
      transform-style: preserve-3d;
      transition: transform 200ms ease-out, box-shadow 200ms ease-out;
      box-shadow:
        0 40px 80px rgba(0, 0, 0, 0.9),
        0 0 60px rgba(0, 255, 120, 0.2);
      border-radius: 14px;
      overflow: hidden;
    }

    .deck-wrapper.glow {
      box-shadow:
        0 50px 130px rgba(0, 0, 0, 0.95),
        0 0 110px rgba(0, 255, 160, 0.7);
    }

    .deck {
      position: relative;
      inset: 0;
      width: 100%;
      height: 100%;
      background: #000;
      display: flex;
      transform-style: preserve-3d;
    }

    .crt-layer::before,
    .crt-layer::after {
      content: "";
      position: absolute;
      inset: 0;
      pointer-events: none;
      z-index: 8;
    }

    .crt-layer::before {
      background-image: linear-gradient(
        to bottom,
        rgba(0, 0, 0, 0) 0,
        rgba(0, 0, 0, 0.4) 2px
      );
      background-size: 100% 3px;
      mix-blend-mode: multiply;
      opacity: 0.6;              /* softer */
      animation: scanlines-move 9s linear infinite;
    }

    .crt-layer::after {
      background:
        radial-gradient(circle at 10% 0, #0f05 0, transparent 55%),
        radial-gradient(circle at 90% 100%, #0ff5 0, transparent 55%);
      opacity: 0.08;
      animation: crt-flicker 4s infinite alternate;
    }

    .global-glitch {
      pointer-events: none;
      position: fixed;
      inset: 0;
      z-index: 15;
      background:
        repeating-linear-gradient(
          0deg,
          rgba(0, 0, 0, 0.4) 0px,
          rgba(0, 0, 0, 0.4) 2px,
          transparent 2px,
          transparent 4px
        );
      mix-blend-mode: screen;
      opacity: 0;
    }

    .global-glitch.active {
      animation: global-glitch 420ms steps(2, end);
    }

    .particle-layer {
      pointer-events: none;
      position: absolute;
      inset: 0;
      overflow: hidden;
      z-index: 1;
    }

    .particle {
      position: absolute;
      width: 3px;
      height: 3px;
      border-radius: 50%;
      background: #00ff88;
      box-shadow: 0 0 10px #00ff88;
      opacity: 0.22;
      animation: particle-drift 12s linear infinite;
    }

    #matrix {
      position: absolute;
      inset: 0;
      z-index: 0;
      opacity: 0.28;
      pointer-events: none;
      filter: blur(0.4px);
    }

    #terminal-wrap {
      position: relative;
      z-index: 2;
      flex: 3;
      display: flex;
      flex-direction: column;
      border-right: 1px solid #003311;
      background:
        radial-gradient(circle at top, #031308 0, #000 65%),
        repeating-linear-gradient(
          to bottom,
          rgba(0, 0, 0, 0.4) 0,
          rgba(0, 0, 0, 0.4) 1px,
          rgba(0, 0, 0, 0.25) 2px
        );
      box-shadow:
        inset 0 0 35px #00ff6620,
        inset 0 0 80px #00ff6612;
    }

    #terminal-header {
      padding: 8px 16px;
      border-bottom: 1px solid #003311;
      display: flex;
      align-items: center;
      justify-content: space-between;
      background: linear-gradient(to bottom, #001408, #000);
      text-transform: uppercase;
      letter-spacing: 0.14em;
      font-size: 12px;
      color: var(--term-fg-dim);
      position: relative;
      z-index: 3;
    }

    .header-left {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .header-led {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: #14ff72;
      box-shadow: 0 0 10px #14ff72;
      animation: led-pulse 1.8s infinite ease-in-out;
    }

    #terminal-header span.badge {
      padding: 2px 8px;
      border-radius: 2px;
      border: 1px solid #005522;
      background: rgba(0, 40, 18, 0.7);
      color: var(--accent);
      font-size: 11px;
    }

    #terminal {
      flex: 1;
      padding: 18px;
      white-space: pre-wrap;
      word-wrap: break-word;
      overflow-y: auto;
      color: var(--term-fg);
      text-shadow:
        0 0 2px #00ff6633,
        1px 0 1px rgba(0, 255, 0, 0.4),
        -1px 0 1px rgba(0, 255, 255, 0.3);
      position: relative;
      z-index: 2;
      font-size: 15px;     /* larger terminal text */
      line-height: 1.4;
    }

    .status-line { color: var(--term-fg-dim); }
    .status-line.ok { color: var(--term-fg); }
    .status-line.warn {
      color: var(--warn);
      text-shadow: 0 0 3px #ffe06655;
    }
    .status-line.err {
      color: var(--error);
      text-shadow: 0 0 6px #ff2a4fbb;
    }

    .flash-overlay {
      position: absolute;
      inset: 0;
      background: radial-gradient(circle at center, #0f05 0, transparent 60%);
      opacity: 0;
      pointer-events: none;
      z-index: 5;
    }

    .flash-overlay.active {
      animation: flash 260ms ease-out;
    }

    #hud {
      position: relative;
      z-index: 2;
      flex: 1.2;
      display: flex;
      flex-direction: column;
      background: radial-gradient(circle at 0 0, #01100a 0, #000 70%);
      color: var(--term-fg-dim);
      padding: 12px;
      font-size: 12px;
      border-left: 1px solid #00220f;
    }

    .logo-wrap {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-bottom: 6px;
    }

    .glitch {
      position: relative;
      font-size: 20px;
      letter-spacing: 0.22em;
      text-transform: uppercase;
      color: #00ff88;
      text-shadow:
        0 0 4px #00ff88,
        0 0 18px #00ff88;
      animation: glitch-skew 3s infinite alternate;
    }

    .glitch::before,
    .glitch::after {
      content: attr(data-text);
      position: absolute;
      left: 0;
      top: 0;
      width: 100%;
      overflow: hidden;
    }

    .glitch::before {
      left: 1px;
      text-shadow: -2px 0 #ff0066;
      animation: glitch-anim-1 2.2s infinite linear alternate-reverse;
    }

    .glitch::after {
      left: -1px;
      text-shadow: 2px 0 #00e6ff;
      animation: glitch-anim-2 1.7s infinite linear alternate-reverse;
    }

    .hud-section {
      border: 1px solid #003818;
      margin-bottom: 8px;
      padding: 6px;
      box-shadow: inset 0 0 10px #00ff6611;
      background: linear-gradient(
        135deg,
        rgba(0, 30, 10, 0.9),
        rgba(0, 0, 0, 0.9)
      );
      position: relative;
    }

    .hud-section::before {
      content: "";
      position: absolute;
      inset: 0;
      border: 1px solid #00ff6614;
      pointer-events: none;
    }

    .hud-title {
      color: var(--accent-soft);
      margin-bottom: 4px;
      text-transform: uppercase;
      letter-spacing: 0.12em;
      font-size: 10px;
    }

    .hud-kv {
      display: flex;
      justify-content: space-between;
      margin-bottom: 1px;
    }

    .hud-bar {
      position: relative;
      width: 100%;
      height: 6px;
      background: #001109;
      border: 1px solid #00401c;
      overflow: hidden;
      margin-top: 3px;
    }

    .hud-bar-fill {
      position: absolute;
      inset: 0;
      width: 50%;
      background: linear-gradient(
        90deg,
        #00ff88 0,
        #00ff55 40%,
        #ffe066 100%
      );
      box-shadow: 0 0 10px #00ff6633;
      transform-origin: left;
    }

    .hud-dots {
      display: flex;
      gap: 3px;
      margin-top: 4px;
    }

    .hud-dot {
      width: 4px;
      height: 4px;
      border-radius: 50%;
      background: #014d27;
      box-shadow: 0 0 4px #014d27;
    }

    .hud-dot.active {
      background: #00ff66;
      box-shadow: 0 0 6px #00ff66;
    }

    .hud-footer {
      margin-top: auto;
      font-size: 10px;
      color: #006633;
      letter-spacing: 0.16em;
      text-transform: uppercase;
      text-align: center;
      padding-top: 4px;
      border-top: 1px solid #00230f;
    }

    #terminal::-webkit-scrollbar { width: 6px; }
    #terminal::-webkit-scrollbar-track { background: #000; }
    #terminal::-webkit-scrollbar-thumb {
      background: #004d26;
      border-radius: 3px;
    }
    #terminal::-webkit-scrollbar-thumb:hover {
      background: #00aa55;
    }

    /* LOGIN OVERLAY */
    #login-overlay {
      position: fixed;
      inset: 0;
      background: radial-gradient(circle at center, rgba(0,0,0,0.95) 0, rgba(0,0,0,0.98) 60%);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 100;
    }

    .login-box {
      border: 1px solid #00ff66;
      padding: 18px 22px;
      width: 320px;
      max-width: 92vw;
      background: #020604;
      box-shadow:
        0 0 25px #00ff6622,
        0 0 70px #00ff6633;
      text-shadow: 0 0 4px #00ff6644;
      font-size: 13px;
    }

    .login-title {
      text-transform: uppercase;
      letter-spacing: 0.16em;
      margin-bottom: 8px;
      color: #00ff88;
      font-size: 13px;
    }

    .login-sub {
      font-size: 12px;
      color: var(--term-fg-dim);
      margin-bottom: 10px;
    }

    .login-field {
      display: flex;
      flex-direction: column;
      margin-bottom: 8px;
      font-size: 12px;
    }

    .login-field label {
      margin-bottom: 3px;
      color: #00e6ff;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      font-size: 11px;
    }

    .login-field input {
      background: #000;
      border: 1px solid #006633;
      color: var(--term-fg);
      padding: 5px 7px;
      font-family: "Courier New", monospace;
      font-size: 13px;
      outline: none;
    }

    .login-field input:focus {
      border-color: #00ff66;
      box-shadow: 0 0 6px #00ff6633;
    }

    .login-hint {
      font-size: 10px;
      color: #007744;
      margin-bottom: 10px;
    }

    .login-error {
      font-size: 11px;
      color: var(--error);
      min-height: 14px;
      margin-bottom: 6px;
    }

    .login-button {
      width: 100%;
      padding: 7px 8px;
      margin-top: 4px;
      border: 1px solid #00ff66;
      background: linear-gradient(to right, #003318, #008844);
      color: #eafff2;
      font-size: 11px;
      text-transform: uppercase;
      letter-spacing: 0.16em;
      cursor: pointer;
    }

    .login-button:hover {
      background: linear-gradient(to right, #00aa55, #00ff88);
    }

    .login-meta {
      margin-top: 8px;
      font-size: 10px;
      color: #005522;
    }

    @keyframes crt-flicker { 0% { opacity: 0.04; } 100% { opacity: 0.1; } }
    @keyframes scanlines-move { 0% { transform: translateY(0); } 100% { transform: translateY(-3px); } }
    @keyframes flash { 0% { opacity: 0.85; } 100% { opacity: 0; } }
    @keyframes led-pulse {
      0%, 100% { opacity: 0.6; box-shadow: 0 0 6px #14ff72; }
      50% { opacity: 1; box-shadow: 0 0 13px #14ff72; }
    }
    @keyframes glitch-skew {
      0% { transform: skew(0deg); }
      50% { transform: skew(0.4deg); }
      100% { transform: skew(-0.4deg); }
    }
    @keyframes glitch-anim-1 {
      0% { clip-path: inset(0 0 70% 0); transform: translate(-1px, -1px); }
      20% { clip-path: inset(10% 0 40% 0); transform: translate(-2px, 1px); }
      40% { clip-path: inset(40% 0 20% 0); transform: translate(1px, -1px); }
      60% { clip-path: inset(80% 0 5% 0); transform: translate(0, 1px); }
      80% { clip-path: inset(10% 0 60% 0); transform: translate(2px, 0); }
      100% { clip-path: inset(0 0 70% 0); transform: translate(-1px, 0); }
    }
    @keyframes glitch-anim-2 {
      0% { clip-path: inset(80% 0 0 0); transform: translate(1px, 0); }
      20% { clip-path: inset(50% 0 20% 0); transform: translate(0, 1px); }
      40% { clip-path: inset(10% 0 40% 0); transform: translate(-1px, 0); }
      60% { clip-path: inset(40% 0 40% 0); transform: translate(0, -1px); }
      80% { clip-path: inset(70% 0 10% 0); transform: translate(-2px, 1px); }
      100% { clip-path: inset(80% 0 0 0); transform: translate(1px, 0); }
    }
    @keyframes global-glitch {
      0% { opacity: 0.9; transform: translate(0, 0); }
      40% { opacity: 0.6; transform: translate(-2px, 1px); }
      80% { opacity: 0.9; transform: translate(3px, -1px); }
      100% { opacity: 0; transform: translate(0, 0); }
    }
    @keyframes particle-drift {
      0% { transform: translate3d(0, 0, 0); opacity: 0.12; }
      50% { transform: translate3d(40px, -40px, 0); opacity: 0.4; }
      100% { transform: translate3d(-20px, -80px, 0); opacity: 0; }
    }
    @keyframes power-on {
      0% { opacity: 1; transform: scaleY(0.02); transform-origin: center; }
      40% { opacity: 1; transform: scaleY(1); }
      100% { opacity: 0; transform: scaleY(1); }
    }

    @media (max-width: 900px) {
      body { font-size: 14px; }
      .deck-wrapper {
        width: 100vw;
        height: 100vh;
        border-radius: 0;
      }
      #hud {
        order: -1;
        flex-direction: row;
        flex-wrap: wrap;
        gap: 6px;
        padding: 8px;
      }
      .hud-section {
        flex: 1 1 48%;
      }
    }
  </style>
</head>
<body>
<div class="power-vignette"></div>
<div class="global-glitch" id="global-glitch"></div>

<!-- LOGIN OVERLAY -->
<div id="login-overlay">
  <div class="login-box">
    <div class="login-title">SECURE LOGIN - BANKHACK</div>
    <div class="login-sub">Enter operator credentials to arm intrusion core.</div>

    <div class="login-field">
      <label for="login-name">your name</label>
      <input id="login-name" type="text" autocomplete="off" placeholder="Operator">
    </div>

    <div class="login-field">
      <label for="login-user">account id</label>
      <input id="login-user" type="text" autocomplete="off" placeholder="ID">
    </div>
    <div class="login-field">
      <label for="login-pass">access key</label>
      <input id="login-pass" type="password" autocomplete="off" placeholder="KEY">
    </div>

    <div class="login-hint">
      BANKHACK is made to entertain people, for fun! <strong>Rabbit</strong> is making this for a fun game! <strong>Enjoy!</strong>
    </div>
    <div class="login-error" id="login-error"></div>
    <div class="login-meta">ID: test1234 · KEY: test3333</div>
    <button class="login-button" id="login-btn">LOGIN</button>

    <button class="login-button" id="howto-btn" style="margin-top:8px;background:linear-gradient(to right,#003318,#444);">
      HOW TO PLAY
    </button>
  </div>

  <!-- HOW TO PLAY OVERLAY -->
  <div id="howto-overlay" style="
       position: fixed;
       inset: 0;
       display: none;
       align-items: center;
       justify-content: center;
       background: radial-gradient(circle at center, rgba(0,0,0,0.95) 0, rgba(0,0,0,0.98) 60%);
       z-index: 150;">
    <div class="login-box" style="max-width:420px;max-height:80vh;overflow-y:auto;">
      <div class="login-title">HOW TO PLAY - BANKHACK</div>
      <div class="login-error">
        BankHack is a fake hacking simulator made for fun. Nothing here touches real banks.
      </div>

      <pre style="font-size:11px; line-height:1.3; white-space:pre-wrap;">
1) LOG IN
   • ID: test1234
   • KEY: test3333
   • NAME: You Choose!.

2) ARM THE CONSOLE
   • After login, the screen fades into the main terminal.
   • Press any key in the terminal area and start spamming your keyboard.
   • Each key feeds the intrusion script and types more "Python" code.

3) WATCH THE BREACH
   • The console will:
     - Scan fake bank servers and open ports.
     - "Decrypt" the vault.
     - Dump sample accounts and balances.
     - Exfiltrate $5,000,000 to a mule wallet.
   • HUD updates:
     - Accounts: 1247/1247 when the vault is unlocked.
     - Balance: shows $5,000,000.00.
     - Stolen: tracks how much you "steal" with commands.

4) COMMAND MODE
   • When you see: BREACH COMPLETE - laundering through crypto mixer
     the console prints:
       [READY] commands: transfer 1000 | dump | launder | exit
   • Now type commands at the prompt line:
       BankHack> transfer 1000
       BankHack> dump
       BankHack> launder
       BankHack> exit

   COMMANDS:
   - transfer X
       Moves X dollars from the vault to the stolen total.
   - dump
       Shows exposed rich accounts (fake numbers).
   - launder
       Prints a message about cleaning the stolen money.
   - exit
       Leaves command mode. You can re-run the breach by spamming keys again.

REMEMBER: This is a visual toy only. Use it to look cool, not to do real hacking.
      </pre>

      <button class="login-button" id="howto-close" style="margin-top:8px;">
        CLOSE
      </button>
    </div>
  </div>
</div>

<div class="deck-wrapper" id="deck-wrapper">
  <div class="deck crt-layer" id="screen">
    <canvas id="matrix"></canvas>
    <div class="particle-layer" id="particle-layer"></div>

    <div id="terminal-wrap">
      <div id="terminal-header">
        <div class="header-left">
          <span class="header-led"></span>
          <span>BANKHACK · INTRUSION CORE</span>
        </div>
        <span class="badge">IDLE · LOCKED</span>
      </div>
      <div id="terminal"></div>
      <div class="flash-overlay" id="flash"></div>
    </div>

    <aside id="hud">
      <div class="logo-wrap">
        <div class="glitch" data-text="BankHack">BankHack</div>
      </div>

      <div class="hud-section">
        <div class="hud-title">BREACH STATUS</div>
        <div class="hud-kv"><span>target</span><span>secure-bank.gov</span></div>
        <div class="hud-kv"><span>accounts</span><span id="hud-accounts">0/∞</span></div>
        <div class="hud-kv"><span>vault</span><span id="hud-vault">LOCKED</span></div>
        <div class="hud-bar">
          <div class="hud-bar-fill" id="hud-load"></div>
        </div>
      </div>

      <div class="hud-section">
        <div class="hud-title">FUNDS TELEMETRY</div>
        <div class="hud-kv"><span>cluster</span><span>bank://vault-alpha</span></div>
        <div class="hud-kv"><span>shard</span><span>RBT-BANK13</span></div>
        <div class="hud-kv"><span>balance</span><span id="hud-balance">$0.00</span></div>
        <div class="hud-dots" id="hud-dots">
          <span class="hud-dot active"></span>
          <span class="hud-dot active"></span>
          <span class="hud-dot"></span>
          <span class="hud-dot"></span>
        </div>
      </div>

      <div class="hud-section">
        <div class="hud-title">EXTRACTION</div>
        <div class="hud-kv"><span>vector</span><span id="hud-vector">SQLi · ID59353</span></div>
        <div class="hud-kv"><span>signature</span><span>R-0xCASHGRAB</span></div>
        <div class="hud-kv"><span>stolen</span><span id="hud-stolen">$0</span></div>
      </div>

      <div class="hud-footer">
        LOGIN TO ARM · THEN SPAM KEYS
      </div>
    </aside>
  </div>
</div>

<script>
  // MATRIX
  const matrixCanvas = document.getElementById('matrix');
  const mtx = matrixCanvas.getContext('2d');
  const matrixChars = '01█▓░#/*=+<>'.split('');
  let drops = [];
  let fontSize = 16;

  function resizeMatrix() {
    matrixCanvas.width = matrixCanvas.clientWidth;
    matrixCanvas.height = matrixCanvas.clientHeight;
    const columns = Math.floor(matrixCanvas.width / fontSize);
    drops = Array(columns).fill(0);
  }

  window.addEventListener('resize', resizeMatrix);
  resizeMatrix();

  function drawMatrix() {
    mtx.fillStyle = 'rgba(0, 0, 0, 0.08)';
    mtx.fillRect(0, 0, matrixCanvas.width, matrixCanvas.height);

    mtx.fillStyle = '#0f5';
    mtx.font = fontSize + 'px Courier New';

    for (let i = 0; i < drops.length; i++) {
      const char = matrixChars[Math.floor(Math.random() * matrixChars.length)];
      const x = i * fontSize;
      const y = drops[i] * fontSize;

      mtx.fillText(char, x, y);

      if (y > matrixCanvas.height && Math.random() > 0.97) {
        drops[i] = 0;
      } else {
        drops[i]++;
      }
    }
    requestAnimationFrame(drawMatrix);
  }
  drawMatrix();

  // PARTICLES
  const particleLayer = document.getElementById('particle-layer');
  function spawnParticles(count) {
    for (let i = 0; i < count; i++) {
      const p = document.createElement('div');
      p.className = 'particle';
      const x = Math.random() * 100;
      const y = 60 + Math.random() * 40;
      p.style.left = x + '%';
      p.style.top = y + '%';
      p.style.animationDelay = (Math.random() * 12).toFixed(2) + 's';
      particleLayer.appendChild(p);
    }
  }
  spawnParticles(18); // slightly fewer for less lag

  // TERMINAL + HUD
  const terminal = document.getElementById('terminal');
  const flash    = document.getElementById('flash');
  const globalGlitch = document.getElementById('global-glitch');

  const hudHosts     = document.getElementById('hud-accounts');
  const hudAuthBadge = document.querySelector('#terminal-header .badge');
  const hudLoadEl    = document.getElementById('hud-load');
  const hudDots      = document.getElementById('hud-dots').children;
  const hudVault     = document.getElementById('hud-vault');
  const hudBalance   = document.getElementById('hud-balance');
  const hudStolen    = document.getElementById('hud-stolen');
  const hudVector    = document.getElementById('hud-vector');

  // Command line
  const commandLine = document.createElement('div');
  commandLine.style.marginTop = '6px';
  const promptSpan = document.createElement('span');
  promptSpan.textContent = 'BankHack> ';
  const commandInput = document.createElement('span');
  commandInput.contentEditable = 'true';
  commandInput.spellcheck = false;
  commandInput.style.outline = 'none';
  commandInput.style.whiteSpace = 'pre';
  commandLine.appendChild(promptSpan);
  commandLine.appendChild(commandInput);
  terminal.appendChild(commandLine);

  // LOGIN + HOWTO
  const loginOverlay = document.getElementById('login-overlay');
  const loginName    = document.getElementById('login-name');
  const loginUser    = document.getElementById('login-user');
  const loginPass    = document.getElementById('login-pass');
  const loginBtn     = document.getElementById('login-btn');
  const loginError   = document.getElementById('login-error');

  const howtoOverlay = document.getElementById('howto-overlay');
  const howtoBtn     = document.getElementById('howto-btn');
  const howtoClose   = document.getElementById('howto-close');

  const VALID_USER = 'test1234';
  const VALID_PASS = 'test3333';
  let isLoggedIn = false;
  let operatorName = 'operator';

  const scriptLines = [
    "import time, sqlite3, random",
    "from cryptography.fernet import Fernet",
    "",
    "TARGET_DB = 'secure_bank_vault.db'",
    "VAULT_KEY = b'BANKHACK_KEY_2026=='",
    "",
    "def log(msg, level='info'):",
    "    ts = time.strftime('%H:%M:%S')",
    "    print(f'[{ts}] [{level.upper()}] {msg}')",
    "",
    "def bypass_firewall():",
    "    log('probing firewall TCP/443,3389,1433...')",
    "    time.sleep(0.1)",
    "    log('FIREWALL: DOWN - SQLi vector confirmed')",
    "    return True",
    "",
    "def decrypt_vault(key):",
    "    log(f'decrypting {TARGET_DB} with {key[:12]}...')",
    "    log('VAULT: UNLOCKED - 1,247 accounts exposed')",
    "    return sqlite3.connect(TARGET_DB)",
    "",
    "def dump_accounts(db):",
    "    cursor = db.cursor()",
    "    accounts = cursor.execute('SELECT id, balance FROM accounts').fetchall()",
    "    log(f'DUMP: {len(accounts)} high-value targets')",
    "    for acc in accounts[:5]:",
    "        log(f'  ACC#{acc[0]} => ${acc[1]:,.2f}')",
    "",
    "def exfiltrate_funds(db, amount):",
    "    log(f'EXFIL: transferring ${amount:,.0f} to mule acct 0xMULE_WALLET')",
    "    log('TX: CONFIRMED - blockchain anchor reached.')",
    "",
    "def main():",
    "    log('BankHack Breach Framework v2.0 BOOT')",
    "    if bypass_firewall():",
    "        db = decrypt_vault(VAULT_KEY)",
    "        dump_accounts(db)",
    "        exfiltrate_funds(db, 5_000_000)",
    "        log('BREACH COMPLETE - laundering through crypto mixer')",
    "    else:",
    "        log('ABORT: intrusion detected', 'error')",
    "",
    "if __name__ == '__main__':",
    "    main()",
    "",
    "# --- BREACH SUMMARY ---",
    "# accounts_breached => 1247",
    "# funds_exfiltrated => $5,000,000",
    "# traces_cleaned    => CONFIRMED",
    "# re-arm for next hit? spam keys...",
  ];

  const preBootBanner = [
    "BANKHACK · OPERATOR LOGIN ACCEPTED",
    "----------------------------------",
    "mounting vault snapshot...",
    "arming breach playbook...",
    "spinning up intrusion core...",
    ""
  ];

  const gibberishLines = [
    "010101 CORE::DECRYPT << 7E 9A F1 C3 88 42 10 3E",
    ">>> Σλ ƒ(x)=∫HACK e^{-iπt} · 0xDEAD · 0xBEEF dt",
    "entropy::pulse ΔΣ=+0.0329 :: channel desync",
    "CORE-TRACE::sample[42] => 7f ff 0d 13 37 c0 de",
    "shim://inject · lattice[Ω13] · phase=π/2 + jitter"
  ];

  let lineIndex   = 0;
  let charIndex   = 0;
  let typingTimer = null;
  let idleTimer   = null;
  const idleDelay = 400;
  let hostsTotal = 1247;
  let hostsScanned = 0;
  let commandMode = false;
  let currentBalance = 5000000;
  let stolenTotal = 0;

  function injectBanner() {
    const ts = new Date().toTimeString().slice(0, 8);
    const stamped = preBootBanner.map((l, i) =>
      i < 2 ? `[[ ${l} ]]` : `[${ts}] ${l}`
    );
    scriptLines.unshift(...stamped, "");
  }

  function setBadge(text) {
    hudAuthBadge.textContent = text;
  }

  // HOW TO overlay
  if (howtoBtn && howtoOverlay && howtoClose) {
    howtoOverlay.style.display = 'none';

    howtoBtn.addEventListener('click', () => {
      howtoOverlay.style.display = 'flex';
    });

    howtoClose.addEventListener('click', () => {
      howtoOverlay.style.display = 'none';
    });

    document.addEventListener('keydown', (e) => {
      if (howtoOverlay.style.display === 'flex' && e.key === 'Escape') {
        howtoOverlay.style.display = 'none';
      }
    });
  }

  // LOGIN
  function attemptLogin() {
    const u = loginUser.value.trim();
    const p = loginPass.value.trim();
    const nameInput = loginName ? loginName.value.trim() : "";

    if (u === VALID_USER && p === VALID_PASS) {
      operatorName = nameInput || 'operator';
      if (hudVector) {
        hudVector.textContent = `SQLi · ${operatorName.toUpperCase()}`;
      }

      loginError.textContent = "";
      isLoggedIn = true;
      setBadge("LIVE · ARMED");
      loginOverlay.style.transition = "opacity 0.4s ease-out";
      loginOverlay.style.opacity = "0";
      setTimeout(() => {
        loginOverlay.style.display = "none";
      }, 400);
      injectBanner();
      addStatusLine(`[AUTH]  ${operatorName} authenticated. intrusion core ready.`, "ok");
      addStatusLine("[HINT]  press any key to execute breach playbook...", "warn");
      commandInput.focus();
    } else {
      loginError.textContent = "AUTH FAIL :: invalid credentials";
      flashPulse(true);
    }
  }

  loginBtn.addEventListener('click', attemptLogin);
  loginUser.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') attemptLogin();
  });
  loginPass.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') attemptLogin();
  });

  document.addEventListener('keydown', (e) => {
    if (!isLoggedIn) return;

    if (commandMode && e.target === commandInput && e.key === 'Enter') {
      e.preventDefault();
      handleCommand(commandInput.textContent.trim());
      commandInput.textContent = '';
      return;
    }

    if (e.key === 'Enter' && commandMode) {
      commandInput.focus();
      return;
    }

    if (!commandMode) {
      if (e.key.length > 1 && e.key !== ' ' && e.key !== 'Enter') return;
      resetIdleTimer();
      if (!typingTimer) typeChar();
    }
  });

  // TYPING
  function typeChar() {
    if (!idleTimer) {
      typingTimer = null;
      return;
    }

    if (lineIndex >= scriptLines.length) {
      typingTimer = null;
      triggerEndEffects();
      return;
    }

    const line = scriptLines[lineIndex];

    if (charIndex < line.length) {
      insertBeforeCursor(line[charIndex]);
      charIndex++;
      scrollTerminal();
      randomHudJitter();
      maybeDecryptionBurst();
      const base = 11;
      const jitter = Math.random() * 35;
      typingTimer = setTimeout(typeChar, base + jitter);
    } else {
      insertBeforeCursor('\n');
      handleLogicalEvents(line.trim());
      lineIndex++;
      charIndex = 0;
      scrollTerminal();
      typingTimer = setTimeout(typeChar, 95);
    }
  }

  function insertBeforeCursor(text) {
    const node = document.createTextNode(text);
    terminal.insertBefore(node, commandLine);
  }

  function scrollTerminal() {
    terminal.scrollTop = terminal.scrollHeight;
  }

  function resetIdleTimer() {
    if (idleTimer) clearTimeout(idleTimer);
    idleTimer = setTimeout(() => {
      idleTimer = null;
      if (typingTimer) {
        clearTimeout(typingTimer);
        typingTimer = null;
      }
    }, idleDelay);
  }

  function handleLogicalEvents(line) {
    if (line.includes('VAULT: UNLOCKED')) {
      hostsScanned = hostsTotal;
      hudHosts.textContent = `${hostsScanned}/${hostsTotal}`;
      setVaultState('UNLOCKED');
    }
    if (line.includes('DUMP:')) {
      hudHosts.textContent = `${hostsScanned}/${hostsTotal}`;
      updateHudBalance();
    }
    if (line.includes('EXFIL:')) {
      stolenTotal += 5000000;
      updateHudStolen();
      setVaultState('BREACHED');
    }
    if (line.includes('BREACH COMPLETE')) {
      commandMode = true;
      addStatusLine("\n[READY] commands: transfer 1000 | dump | launder | exit", "ok");
      addStatusLine("[HINT]  type commands at BankHack> and press ENTER", "warn");
      commandInput.focus();
    }
  }

  function triggerEndEffects() {
    flashPulse(true);
    maybeGlobalGlitch();
    let pulses = 0;
    const maxPulses = 3;
    const pulseInterval = setInterval(() => {
      pulses++;
      addStatusLine("[AUTH]   ACCESS OVERRIDE REQUEST => DENIED", "err");
      if (pulses >= maxPulses) {
        clearInterval(pulseInterval);
        addStatusLine("[READY]  console re-armed. spam keys to replay breach.", "ok");
      }
    }, 550);
  }

  function addStatusLine(text, type) {
    const span = document.createElement('span');
    span.textContent = text + "\n";
    span.className = "status-line " + (type || "");
    terminal.insertBefore(span, commandLine);
    scrollTerminal();
  }

  function flashPulse(hard) {
    flash.classList.remove('active');
    void flash.offsetWidth;
    flash.style.background = hard
      ? 'radial-gradient(circle at center, #f00a 0, transparent 60%)'
      : 'radial-gradient(circle at center, #0f05 0, transparent 60%)';
    flash.classList.add('active');
  }

  function maybeGlobalGlitch() {
    if (Math.random() < 0.45) {
      globalGlitch.classList.remove('active');
      void globalGlitch.offsetWidth;
      globalGlitch.classList.add('active');
    }
  }

  function maybeDecryptionBurst() {
    if (Math.random() < 0.005) {
      const line = gibberishLines[Math.floor(Math.random() * gibberishLines.length)];
      addStatusLine(line, "warn");
    }
  }

  function setVaultState(state) {
    hudVault.textContent = state;
    hudVault.style.color = state === 'BREACHED'
      ? '#00ff88'
      : (state === 'UNLOCKED' ? '#ffe066' : '#ff4f4f');
  }

  function randomHudJitter() {
    const load = 0.4 + Math.random() * 0.4;
    hudLoadEl.style.transform = `scaleX(${load.toFixed(2)})`;

    for (let i = 0; i < hudDots.length; i++) {
      const active = Math.random() > 0.3;
      hudDots[i].classList.toggle('active', active);
    }
  }

  function updateHudBalance() {
    hudBalance.textContent = `$${currentBalance.toLocaleString()}.00`;
  }

  function updateHudStolen() {
    hudStolen.textContent = `$${stolenTotal.toLocaleString()}`;
  }

  function handleCommand(input) {
    if (!input) return;
    const cmd = input.toLowerCase();
    addStatusLine(`BankHack> ${input}`, "warn");

    if (cmd.startsWith('transfer ')) {
      const amt = parseFloat(cmd.split(' ')[1]);
      if (amt > 0 && currentBalance >= amt) {
        stolenTotal += amt;
        currentBalance -= amt;
        addStatusLine(`[TX] $${amt.toLocaleString()} -> mule wallet CONFIRMED`, "ok");
        updateHudBalance();
        updateHudStolen();
      } else {
        addStatusLine("[ERR] insufficient funds or invalid amount", "err");
      }
    } else if (cmd === 'dump') {
      addStatusLine("[DUMP] ACC#1247 => $2,450,000 | ACC#99 => $15M exposed", "ok");
    } else if (cmd === 'launder') {
      addStatusLine("[LAUNDER] routing through crypto mixers... CLEAN", "ok");
    } else if (cmd === 'exit') {
      commandMode = false;
      addStatusLine("[SHUTDOWN] vault session terminated", "warn");
    } else {
      addStatusLine("[ERR] unknown command - try 'transfer 1000'", "err");
    }
  }

  // 3D TILT
  const deckWrapper = document.getElementById('deck-wrapper');

  function handleTilt(e) {
    const rect = deckWrapper.getBoundingClientRect();
    const x = (e.clientX - rect.left) / rect.width;
    const y = (e.clientY - rect.top) / rect.height;
    const maxRotate = 9;
    const rotateY = (x - 0.5) * maxRotate * 2;
    const rotateX = (0.5 - y) * maxRotate * 2;
    deckWrapper.style.transform =
      `rotateX(${rotateX}deg) rotateY(${rotateY}deg)`;
    deckWrapper.classList.add('glow');
  }

  function resetTilt() {
    deckWrapper.style.transform = 'rotateX(0deg) rotateY(0deg)';
    deckWrapper.classList.remove('glow');
  }

  window.addEventListener('mousemove', handleTilt);
  window.addEventListener('mouseleave', resetTilt);
</script>

</body>
</html>

