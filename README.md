<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Vibhor Joshi — Neural Space</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>🧠</text></svg>">
<style>
  /* Reset + base */
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html,body { height:100%; font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; background:#04040a; color:#dfe9ff; -webkit-font-smoothing:antialiased;}
  a { color: inherit; text-decoration: none; }
  .stage { position:relative; min-height:100vh; overflow:hidden; display:flex; align-items:center; justify-content:center; }

  /* Starfield canvas */
  #starfield { position:absolute; inset:0; z-index:0; background: radial-gradient(ellipse at top, rgba(6,7,20,0.6), rgba(2,2,8,0.9) 60%); }

  /* Neon Grid */
  .grid {
    position:absolute; inset:0; z-index:1; pointer-events:none;
    background-image:
      linear-gradient(180deg, rgba(76, 49, 255, 0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(76, 49, 255, 0.02) 1px, transparent 1px);
    background-size: 120px 120px, 120px 120px;
    mix-blend-mode: screen;
    filter: blur(10px) saturate(120%);
    opacity:0.6;
    transform: translateY(10px) scale(1.08);
  }

  /* Main Card Container */
  .card {
    position:relative; z-index:5;
    width:min(1100px, 96%);
    background: linear-gradient(180deg, rgba(10,8,20,0.6), rgba(8,10,20,0.38));
    border: 1px solid rgba(120,110,255,0.14);
    box-shadow: 0 10px 40px rgba(10,6,30,0.7), 0 0 50px rgba(80,50,255,0.06) inset;
    border-radius:18px;
    padding:28px;
    backdrop-filter: blur(6px) saturate(140%);
    overflow:hidden;
  }

  /* Header */
  .hero {
    display:flex; gap:20px; align-items:center;
  }
  .logo {
    width:120px; height:120px; flex: 0 0 120px;
    border-radius:14px;
    display:flex; align-items:center; justify-content:center;
    background: linear-gradient(135deg,#2b1b6d, #5643ff);
    box-shadow: 0 8px 30px rgba(86,67,255,0.25);
    border: 1px solid rgba(255,255,255,0.04);
  }
  .logo h1{font-size:48px; letter-spacing:1px; margin-left:3px; color: #fff;}
  .hero-right { flex:1; }
  .title { font-size:22px; color:#e9ecff; font-weight:700; letter-spacing:0.6px; display:flex; align-items:center; gap:10px; }
  .subtitle { color: #bfc8ff; margin-top:6px; font-size:14px; }
  .badges { margin-top:10px; display:flex; gap:8px; flex-wrap:wrap; }

  /* Enter UI */
  .enter-wrap { margin-top:20px; display:flex; gap:16px; align-items:center; }
  .enter-btn {
    position:relative;
    background: linear-gradient(90deg,#7C5CFF,#53E5FF 70%);
    color:#03021a;
    font-weight:700;
    padding:12px 20px;
    border-radius:12px;
    border:none;
    cursor:pointer;
    box-shadow: 0 6px 24px rgba(83,229,255,0.12), 0 2px 6px rgba(124,92,255,0.12);
    transition:transform .18s ease, box-shadow .18s ease;
    overflow:visible;
  }
  .enter-btn:hover { transform:translateY(-3px) scale(1.02); }
  .sparkle { position:absolute; inset:0; pointer-events:none; mix-blend-mode:screen; }
  .countdown {
    color:#b8c1ff; font-weight:600; font-size:14px; background:rgba(255,255,255,0.02); padding:8px 12px; border-radius:10px; border: 1px solid rgba(120,110,255,0.06);
  }

  /* Animated chase layer */
  .chase-layer { position:absolute; inset:0; z-index:3; pointer-events:none; overflow:visible; }
  .bird, .fox {
    position:absolute; top:8vh; width:110px; height:70px; transform-origin:center;
    filter: drop-shadow(0 10px 18px rgba(0,0,0,0.6));
  }
  .fox { top:60vh; width:120px; height:80px; transform-origin:center; }

  /* Scene content hidden until enter */
  .scene { margin-top:22px; display:grid; grid-template-columns: 1fr 360px; gap:20px; align-items:start; opacity:0; transform:translateY(12px) scale(0.995); transition: all 0.9s cubic-bezier(.2,.9,.2,1); }
  .scene.visible { opacity:1; transform:none; }

  /* Left column (main content) */
  .panel { background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:12px; padding:18px; border:1px solid rgba(90,60,255,0.06); box-shadow: 0 8px 30px rgba(20,10,40,0.45); }
  .section-title { font-weight:800; color:#e9edff; font-size:16px; display:flex; align-items:center; gap:10px; }
  .grid-cards { display:grid; gap:14px; grid-template-columns: repeat(2,1fr); margin-top:12px; }
  .card-small { padding:12px; border-radius:10px; background: linear-gradient(180deg, rgba(255,255,255,0.015), rgba(10,8,20,0.02)); border: 1px solid rgba(100,80,255,0.05); }
  .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", monospace; font-size:13px; color:#c9d1ff; }

  /* Right column (sidebar) */
  .sidebar { position:sticky; top:28px; height:max-content; align-self:start; }
  .skill { margin-top:8px; font-size:13px; color:#cfe0ff; }

  /* Footer Neon line */
  .neon-footer { margin-top:18px; border-top:1px dashed rgba(120,90,255,0.06); padding-top:14px; display:flex; justify-content:space-between; align-items:center; font-size:13px; color:#aeb9ff; }

  /* small screens */
  @media (max-width:900px) {
    .scene { grid-template-columns: 1fr; }
    .hero { flex-direction:column; align-items:flex-start; gap:12px; }
    .logo { width:84px; height:84px; flex:0 0 84px; }
  }

  /* sparkle animation */
  @keyframes sparkle {
    0% { opacity:0; transform: translateY(0) scale(.8) rotate(-6deg); }
    50% { opacity:1; transform: translateY(-6px) scale(1.05) rotate(0deg); }
    100% { opacity:0; transform: translateY(-12px) scale(.8) rotate(6deg); }
  }

  /* bird wing flap */
  @keyframes flap { 0%{transform:rotate(-6deg)} 50%{transform:rotate(12deg)} 100%{transform:rotate(-6deg)} }

  /* fox tail wag */
  @keyframes wag { 0%{transform: rotate(0deg)} 50%{transform: rotate(12deg)} 100%{transform: rotate(0deg)} }

  /* subtle glow */
  .glow { text-shadow: 0 6px 30px rgba(83,229,255,0.08), 0 2px 8px rgba(124,92,255,0.06); }

  /* tiny chip */
  .chip { display:inline-block; padding:6px 10px; border-radius:999px; font-weight:700; background:linear-gradient(90deg, rgba(255,255,255,0.03), rgba(255,255,255,0.01)); border:1px solid rgba(120,110,255,0.06); }
</style>
</head>
<body>
  <div class="stage">
    <canvas id="starfield"></canvas>
    <div class="grid" aria-hidden="true"></div>

    <!-- Chase layer: bird and fox will animate across the scene -->
    <div class="chase-layer" aria-hidden="true">
      <!-- Bird SVG (simple stylized) -->
      <svg class="bird" id="bird" viewBox="0 0 200 100" xmlns="http://www.w3.org/2000/svg" aria-hidden="true" style="left:-180px;">
        <defs>
          <linearGradient id="bgrad" x1="0" x2="1"><stop offset="0" stop-color="#76f0ff"/><stop offset="1" stop-color="#7C5CFF"/></linearGradient>
        </defs>
        <g transform="scale(1)">
          <path d="M10 60 C40 20, 80 20, 110 60 C130 85, 150 90, 180 65" fill="none" stroke="url(#bgrad)" stroke-width="6" stroke-linecap="round"/>
          <g id="wing" transform="translate(70,30)">
            <path d="M0 0 C24 -20, 48 -20, 68 0 C48 -4, 24 -4, 0 0" fill="#bff8ff" opacity="0.95"/>
          </g>
        </g>
      </svg>

      <!-- Fox SVG -->
      <svg class="fox" id="fox" viewBox="0 0 220 120" xmlns="http://www.w3.org/2000/svg" aria-hidden="true" style="left:120%; transform:translateX(0);">
        <defs>
          <linearGradient id="fgrad" x1="0" x2="1"><stop offset="0" stop-color="#ffb86b"/><stop offset="1" stop-color="#ff6b6b"/></linearGradient>
        </defs>
        <g transform="scale(1)">
          <path d="M18 70 C28 30, 64 18, 110 26 C150 34, 182 60, 200 86" fill="none" stroke="url(#fgrad)" stroke-width="8" stroke-linecap="round"/>
          <ellipse cx="140" cy="46" rx="22" ry="14" fill="#ffb86b" opacity="0.92"/>
          <g id="tail" transform="translate(190,70)">
            <path d="M0 0 C20 -6, 34 -20, 40 -34 C30 -30, 10 -12, 0 0" fill="#ffb86b" opacity="0.95"/>
          </g>
        </g>
      </svg>
    </div>

    <div class="card" role="main" aria-labelledby="profileTitle">
      <div class="hero">
        <div class="logo">
          <h1 style="font-size:36px; color:#fff; margin:0">VJ</h1>
        </div>
        <div class="hero-right">
          <div class="title glow" id="profileTitle">Vibhor Joshi — <span style="color:#bff8ff;">Neural Systems & GeoAI</span></div>
          <div class="subtitle">🎯 Machine Learning Enthusiast | 🌍 GeoAI Innovator | 🤖 Interactive Systems Builder</div>

          <div class="badges" style="margin-top:10px;">
            <img alt="views" src="https://komarev.com/ghpvc/?username=vibhorjoshi&style=for-the-badge&color=4A4A8C&label=NEURAL+CONNECTIONS" style="height:28px;border-radius:8px;">
            <span class="chip">Flask • Django • PyTorch</span>
            <span class="chip">GeoAI • OpenCV • CLIP</span>
          </div>

          <div class="enter-wrap">
            <button class="enter-btn" id="enterBtn" aria-haspopup="true" aria-expanded="false">
              ✨ Enter My Space
              <svg class="sparkle" width="100%" height="100%" viewBox="0 0 100 40" preserveAspectRatio="none" style="opacity:0.5;">
                <g fill="none" stroke="rgba(255,255,255,0.8)" stroke-width="1">
                  <path d="M4 30 C22 8, 78 8, 96 30" stroke-linecap="round" style="filter:blur(0.2px)"></path>
                </g>
              </svg>
            </button>
            <div class="countdown" id="count">Ready</div>
          </div>
        </div>
      </div>

      <!-- Hidden scene, revealed after enter -->
      <div class="scene" id="scene">
        <!-- left -->
        <div class="panel">
          <div class="section-title">⚡ Technology Arsenal</div>
          <div class="grid-cards">
            <div class="card-small">
              <div class="mono">Python • PyTorch • TensorFlow</div>
              <div style="margin-top:8px; color:#cbe6ff; font-weight:700;">Computer Vision & Medical Imaging</div>
              <div style="margin-top:6px; font-size:13px; color:#b6c6ff;">Tumor detection, segmentation, high accuracy pipelines.</div>
            </div>
            <div class="card-small">
              <div class="mono">GDAL • GeoPandas • CUDA</div>
              <div style="margin-top:8px; color:#cbe6ff; font-weight:700;">GeoAI & Spatial Systems</div>
              <div style="margin-top:6px; font-size:13px; color:#b6c6ff;">Building footprint extraction, GPU-accelerated workflows.</div>
            </div>
            <div class="card-small">
              <div class="mono">MediaPipe • OpenCV</div>
              <div style="margin-top:8px; color:#cbe6ff; font-weight:700;">Gesture Recognition</div>
              <div style="margin-top:6px; font-size:13px; color:#b6c6ff;">Real-time pose estimation and low-latency control systems.</div>
            </div>
            <div class="card-small">
              <div class="mono">TypeScript • CLIP • FAISS</div>
              <div style="margin-top:8px; color:#cbe6ff; font-weight:700;">AI File Cleaner</div>
              <div style="margin-top:6px; font-size:13px; color:#b6c6ff;">Semantic deduplication, high-performance desktop tooling.</div>
            </div>
          </div>

          <div style="margin-top:16px;" class="section-title">💼 Projects & Highlights</div>
          <div style="margin-top:12px; display:flex; gap:12px; flex-direction:column;">
            <div class="card-small">
              <strong>Fusing-Brains-and-Boundaries</strong><br><small>GPU-accelerated GeoAI framework • building footprint extraction</small>
            </div>
            <div class="card-small">
              <strong>AI File Cleaner</strong><br><small>Next.js + Electron • CLIP / MiniLM deduplication</small>
            </div>
            <div class="card-small">
              <strong>Brain Tumor Detection</strong><br><small>CNN-based MRI classifier • Vercel deployment</small>
            </div>
          </div>

          <div style="margin-top:18px;" class="neon-footer">
            <div>💭 <em>“Innovation begins where boundaries dissolve.”</em></div>
            <div style="display:flex; gap:10px;">
              <a href="https://github.com/vibhorjoshi" target="_blank" rel="noopener" aria-label="GitHub"><img src="https://img.shields.io/badge/GitHub-Profile-181717?style=for-the-badge&logo=github" alt="GitHub" style="height:30px; border-radius:8px;"></a>
              <a href="mailto:vibhorjoshi40@gmail.com" aria-label="Email"><img src="https://img.shields.io/badge/Email-Contact-FF6B6B?style=for-the-badge&logo=gmail" alt="Email" style="height:30px; border-radius:8px;"></a>
            </div>
          </div>
        </div>

        <!-- right sidebar -->
        <aside class="sidebar">
          <div class="panel">
            <div class="section-title">📌 Snapshot</div>
            <div style="margin-top:10px;">
              <div class="skill"><strong>Role:</strong> Machine Learning Enthusiast</div>
              <div class="skill"><strong>Location:</strong> India 🇮🇳</div>
              <div class="skill"><strong>Model Accuracy:</strong> 90%+</div>
              <div class="skill"><strong>Processing:</strong> ~100ms text dedup</div>
              <div style="margin-top:12px;" class="section-title">🧭 Missions</div>
              <div class="skill">Scale GeoAI • Build healthcare ML • Gesture HCI</div>
            </div>
          </div>

          <div style="height:14px;"></div>

          <div class="panel">
            <div class="section-title">⚡ Quick Links</div>
            <div style="margin-top:10px; display:flex; flex-direction:column; gap:8px;">
              <a href="https://github.com/vibhorjoshi/Fusing-Brains-and-Boundaries" target="_blank" rel="noopener" class="card-small">Fusing-Brains-and-Boundaries →</a>
              <a href="https://github.com/vibhorjoshi/ai-file-cleaner" target="_blank" rel="noopener" class="card-small">AI File Cleaner →</a>
              <a href="https://github.com/vibhorjoshi" target="_blank" rel="noopener" class="card-small">GitHub Profile →</a>
            </div>
          </div>
        </aside>
      </div>
    </div>
  </div>

<script>
/* ---------- Starfield ---------- */
const canvas = document.getElementById('starfield');
const ctx = canvas.getContext('2d');
let w = canvas.width = innerWidth;
let h = canvas.height = innerHeight;
window.addEventListener('resize', () => { w = canvas.width = innerWidth; h = canvas.height = innerHeight; initStars(); });

let stars = [];
const STAR_COUNT = Math.round((window.innerWidth * window.innerHeight) / 6500);
function initStars() {
  stars = [];
  for(let i=0;i<STAR_COUNT;i++){
    stars.push({
      x: Math.random()*w,
      y: Math.random()*h,
      r: Math.random()*1.4 + 0.2,
      vx: (Math.random()*0.3 - 0.15),
      vy: (Math.random()*0.05 + 0.02),
      alpha: Math.random()*0.6 + 0.15,
      flick: Math.random()*1.2
    });
  }
}
initStars();

function drawStars(t){
  ctx.clearRect(0,0,w,h);
  // faint nebula
  const g = ctx.createRadialGradient(w*0.2,h*0.1,20, w*0.6,h*0.7, Math.max(w,h));
  g.addColorStop(0,'rgba(120,60,255,0.06)');
  g.addColorStop(1,'rgba(2,3,8,0)');
  ctx.fillStyle = g;
  ctx.fillRect(0,0,w,h);
  // draw
  for(let s of stars){
    s.x += s.vx; s.y += s.vy + Math.sin(t/1000 + s.flick)*0.12;
    if(s.x < -20) s.x = w+20;
    if(s.y > h+20) s.y = -20;
    ctx.globalAlpha = s.alpha * (0.6 + 0.4*Math.sin(t/300 + s.flick));
    ctx.beginPath();
    ctx.fillStyle = 'white';
    ctx.arc(s.x, s.y, s.r, 0, Math.PI*2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;
  requestAnimationFrame(drawStars);
}
requestAnimationFrame(drawStars);

/* ---------- Enter Button + Countdown + Reveal ---------- */
const enterBtn = document.getElementById('enterBtn');
const countEl = document.getElementById('count');
const scene = document.getElementById('scene');
let engaged = false;

function runCountdown() {
  const steps = [3,2,1];
  let i = 0;
  countEl.textContent = 'Entering in 3...';
  const id = setInterval(()=> {
    if(i < steps.length) {
      countEl.textContent = 'Entering in ' + steps[i] + '...';
      i++;
    } else {
      clearInterval(id);
      countEl.textContent = 'Welcome 👋';
      revealScene();
    }
  }, 700);
}

function revealScene(){
  scene.classList.add('visible');
  enterBtn.setAttribute('aria-expanded','true');
  startChase(); // begin bird/fox animation
}

/* Button sparkle (visual) */
enterBtn.addEventListener('click', (e)=>{
  if(engaged) return;
  engaged = true;
  runCountdown();
  // small pulse
  enterBtn.animate([{transform:'scale(1)'},{transform:'scale(0.98)'},{transform:'scale(1)'}], {duration:400, easing:'ease'});
});

/* ---------- Bird + Fox chase animation ---------- */
const bird = document.getElementById('bird');
const fox = document.getElementById('fox');

function startChase() {
  // bird flies left-to-right then loops
  animateBird();
  animateFox();
}

function animateBird() {
  // path across top -> loop
  const dur = 9000;
  bird.style.top = '6vh';
  bird.animate([
    { transform: 'translateX(-220px) rotate(-6deg) scale(0.9)' , left: '-180px'},
    { transform: 'translateX(110vw) rotate(10deg) scale(1.05)', left: '110vw'}
  ], { duration: dur, iterations: Infinity, easing: 'cubic-bezier(.22,.9,.15,1)'});
  // wing flapping
  setInterval(()=> {
    const wing = bird.querySelector('#wing');
    if(wing) wing.animate([{transform:'translate(70px,30px) rotate(-6deg)'},{transform:'translate(70px,30px) rotate(12deg)'},{transform:'translate(70px,30px) rotate(-6deg)'}], {duration:240, iterations:6});
  }, 800);
}

function animateFox(){
  fox.style.top = '62vh';
  const pathDur = 16000;
  fox.animate([
    { transform: 'translateX(0) rotate(0deg)', left: '120%' },
    { transform: 'translateX(-130vw) rotate(-6deg)', left: '-140%' }
  ], { duration: pathDur, iterations: Infinity, easing: 'linear' });

  // tail wag intermittently
  setInterval(()=> {
    const tail = fox.querySelector('#tail');
    if(tail) tail.animate([{transform:'translate(190px,70px) rotate(0deg)'},{transform:'translate(190px,70px) rotate(12deg)'},{transform:'translate(190px,70px) rotate(0deg)'}], {duration:400, iterations:4});
  }, 1200);
}

/* ---------- subtle interactive: hovering 'Enter' causes birds to twinkle ---------- */
enterBtn.addEventListener('mouseover', ()=> {
  for(let s of stars.slice(0,10)) s.alpha = Math.min(1, s.alpha + 0.35);
});
enterBtn.addEventListener('mouseout', ()=> {
  for(let s of stars.slice(0,10)) s.alpha = Math.max(0.12, s.alpha - 0.15);
});

/* Accessibility: Enter via keyboard */
enterBtn.addEventListener('keydown', (e)=> {
  if(e.key === 'Enter' || e.key === ' ') enterBtn.click();
});

/* small initial shimmer on load */
window.addEventListener('load', ()=> {
  countEl.textContent = 'Click → Enter My Space';
  // tiny header shimmer
  document.querySelectorAll('.glow').forEach(el=>{
    el.animate([{ opacity:0.95 }, { opacity:1 }, { opacity:0.95 }], { duration:3000, iterations:1 });
  });
});
</script>
</body>
</html>

