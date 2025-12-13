
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Akash — Exem Portfolio</title>

<!-- Google Font (allowed external) -->
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;600;800&display=swap" rel="stylesheet">

<style>
  :root{
    --bg-1: #05060a;
    --bg-2: #0b0f14;
    --accent1: #6EE7FF; /* glacier cyan */
    --accent2: #D8E9FF; /* soft snow */
    --glass: rgba(255,255,255,0.03);
    --card: rgba(255,255,255,0.02);
    --shadow-strong: 0 20px 50px rgba(0,0,0,0.8);
    --shadow-soft: 0 10px 30px rgba(0,0,0,0.6);
    --text: #e8f0ff;
  }

  *{box-sizing:border-box}
  html,body{height:100%;margin:0;font-family:"Montserrat",system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;}
  body{
    background: linear-gradient(180deg,var(--bg-1),var(--bg-2));
    color:var(--text);
    overflow:hidden;
  }

  /* Canvas container (snow) covers the whole screen */
  #snow-canvas{
    position:fixed; left:0; top:0; width:100%; height:100%; z-index:0;
    pointer-events:none;
  }

  /* Main app layout */
  .app{
    position:relative;
    min-height:100vh;
    z-index:5;
    display:flex;
    align-items:center;
    justify-content:center;
    padding:40px;
  }

  /* Glass panel that holds navigation & content */
  .shell{
    width:min(1200px,95%);
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border-radius:18px;
    padding:28px;
    box-shadow: var(--shadow-strong), inset 0 1px 0 rgba(255,255,255,0.02);
    backdrop-filter: blur(6px) saturate(120%);
    display:grid;
    grid-template-columns: 320px 1fr;
    gap:24px;
    align-items:stretch;
    overflow:hidden;
  }

  /* Left column: profile + nav */
  .left{
    padding:22px;
    border-radius:12px;
    background: linear-gradient(180deg, rgba(0,0,0,0.22), rgba(255,255,255,0.02));
    box-shadow: var(--shadow-soft);
    display:flex;
    flex-direction:column;
    gap:18px;
    min-height:420px;
  }

  /* Big name with animated shadow */
  .brand{
    display:flex;
    align-items:center;
    gap:12px;
  }

  .logo{
    width:64px;height:64px;border-radius:12px;
    background: linear-gradient(135deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    display:flex;align-items:center;justify-content:center;font-weight:800;
    font-size:20px;color:var(--accent2);box-shadow: 0 10px 30px rgba(0,0,0,0.7);
    border:1px solid rgba(255,255,255,0.03);
  }

  .name-wrap{
    display:flex;flex-direction:column;
  }

  h1.name{
    margin:0;font-size:26px;letter-spacing:1px;
    position:relative;
    color:var(--accent2);
    /* thick animated multi-layer shadow for 'Akash' */
    text-shadow:
      0 2px 0 rgba(0,0,0,0.6),
      0 6px 20px rgba(0,0,0,0.9),
      0 0 12px rgba(110,231,255,0.06);
  }

  /* Animated moving shadow under name (pseudo 3D) */
  .name-shadow{
    margin-top:6px;font-size:12px;color:rgba(210,230,255,0.6);
    transform-origin:left center;
    filter: blur(1px);
    animation: floatShadow 3.8s ease-in-out infinite;
  }

  @keyframes floatShadow{
    0%{ transform:translateY(0) rotateX(0) scale(1); opacity:0.95 }
    50%{ transform:translateY(6px) rotateX(6deg) scale(1.02); opacity:1 }
    100%{ transform:translateY(0) rotateX(0) scale(1); opacity:0.95 }
  }

  /* Nav */
  nav{
    display:flex;flex-direction:column;gap:10px;
  }
  .nav-btn{
    display:flex;align-items:center;gap:12px;padding:10px 12px;border-radius:10px;
    color:var(--accent2);text-decoration:none;font-weight:600;
    cursor:pointer;border:1px solid rgba(255,255,255,0.02);
    transition:transform .2s, box-shadow .2s, background .2s;
    box-shadow: 0 8px 30px rgba(0,0,0,0.6);
    background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));
  }
  .nav-btn:hover{ transform:translateX(6px); box-shadow: 0 20px 60px rgba(0,0,0,0.7) }
  .nav-btn.active{
    background: linear-gradient(90deg, rgba(110,231,255,0.06), rgba(216,233,255,0.02));
    border-left:4px solid rgba(110,231,255,0.5);
    transform:translateX(6px);
  }

  /* Right column: pages */
  .content{
    padding:18px;border-radius:12px; background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(0,0,0,0.02));
    min-height:420px; box-shadow:var(--shadow-soft); position:relative; overflow:hidden;
  }

  /* Page transitions */
  .page{
    position:absolute; inset:18px; padding:34px; border-radius:10px; overflow:auto;
    background: linear-gradient(180deg, rgba(9,12,15,0.35), rgba(0,0,0,0.25));
    transition: transform .5s cubic-bezier(.2,.9,.2,1), opacity .45s ease;
    opacity:0; transform: translateY(12px) scale(.995); pointer-events:none;
  }
  .page.active{ opacity:1; transform: translateY(0) scale(1); pointer-events:auto; }

  /* Page header */
  .page h2{ margin-top:0; letter-spacing:0.6px; color:var(--accent1) }

  /* Cards and project tiles */
  .card{
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border-radius:12px;padding:16px;margin-bottom:14px;
    box-shadow: 0 12px 40px rgba(0,0,0,0.65);
    border:1px solid rgba(255,255,255,0.02);
  }

  .projects-grid{ display:grid; grid-template-columns:repeat(auto-fit,minmax(220px,1fr)); gap:14px; margin-top:12px; }

  .project{
    padding:12px;border-radius:10px;
    background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(0,0,0,0.03));
    transition:transform .25s, box-shadow .25s;
    cursor:pointer; box-shadow: 0 10px 30px rgba(0,0,0,0.6);
    border:1px solid rgba(255,255,255,0.02);
  }
  .project:hover{ transform:translateY(-8px); box-shadow: 0 22px 60px rgba(0,0,0,0.75) }

  /* Shadow effect for interactive elements */
  .glow{
    position:relative;
  }
  .glow::after{
    content:""; position:absolute; inset:0; border-radius:inherit;
    box-shadow: 0 30px 80px rgba(110,231,255,0.03), inset 0 2px 10px rgba(255,255,255,0.02);
    pointer-events:none;
  }

  /* Contact form */
  form.contact-form{ display:flex; flex-direction:column; gap:12px; max-width:560px; }
  input, textarea{
    background: rgba(255,255,255,0.02); color:var(--text); border:1px solid rgba(255,255,255,0.03);
    padding:12px;border-radius:10px; outline:none; font-size:14px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.6);
  }
  textarea{ min-height:140px; resize:vertical; }
  .btn{
    padding:12px 16px;border-radius:10px;font-weight:700;cursor:pointer;border:none;
    background: linear-gradient(90deg,var(--accent1),var(--accent2)); color:#00121a;
    box-shadow: 0 14px 45px rgba(110,231,255,0.09);
  }

  /* Footer */
  .meta{ font-size:12px; color:rgba(210,230,255,0.6); }

  /* Responsive */
  @media (max-width:900px){
    .shell{ grid-template-columns:1fr; }
    .left{ order:2 }
    .content{ order:1; min-height:540px }
  }
</style>
</head>
<body>

<canvas id="snow-canvas"></canvas>

<div class="app">
  <div class="shell">
    <div class="left">
      <div class="brand">
        <div class="logo">A</div>
        <div class="name-wrap">
          <h1 class="name">Akash</h1>
          <div class="name-shadow">Shadow • Designer • Developer</div>
        </div>
      </div>

      <nav>
        <div class="nav-btn active" data-page="home">🏠 Home</div>
        <div class="nav-btn" data-page="about">👨‍💻 About</div>
        <div class="nav-btn" data-page="projects">🧩 Projects</div>
        <div class="nav-btn" data-page="contact">✉️ Contact</div>
      </nav>

      <div class="card glow" style="margin-top:auto">
        <h3 style="margin:0 0 8px 0">Quick info</h3>
        <p style="margin:0" class="meta">Glacier-snow themed portfolio — single file. All elements have shadow + animation focus.</p>
      </div>
    </div>

    <div class="content">
      <!-- Home -->
      <section id="home" class="page active">
        <h2>Welcome</h2>
        <div class="card glow">
          <p>Hello! I'm <strong>Akash</strong>, a creative developer who loves premium dark UIs, animations, and heavy shadow design. This portfolio is built as a single file with a glacier-snow animated background and layered shadows for depth.</p>
        </div>

        <div class="card">
          <h3>Skills</h3>
          <div style="display:flex;gap:8px;flex-wrap:wrap;margin-top:8px">
            <span class="pill" style="padding:8px 10px;border-radius:8px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.03);box-shadow:var(--shadow-soft)">HTML</span>
            <span class="pill" style="padding:8px 10px;border-radius:8px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.03);box-shadow:var(--shadow-soft)">CSS / Animation</span>
            <span class="pill" style="padding:8px 10px;border-radius:8px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.03);box-shadow:var(--shadow-soft)">JavaScript</span>
            <span class="pill" style="padding:8px 10px;border-radius:8px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.03);box-shadow:var(--shadow-soft)">UI/UX</span>
          </div>
        </div>
      </section>

      <!-- About -->
      <section id="about" class="page">
        <h2>About</h2>
        <div class="card">
          <p>I design and build interactive interfaces with a focus on depth — layered shadows, nuanced animations and subtle particles (see background). I enjoy crafting premium aesthetics that feel tactile and modern.</p>
        </div>

        <div class="card">
          <h3>Experience</h3>
          <ul style="margin:8px 0 0 18px">
            <li>Frontend developer — single-page magical experiences</li>
            <li>Animated UI specialist — CSS & JS driven</li>
            <li>Designer — dark, neon & glacier themed visuals</li>
          </ul>
        </div>
      </section>

      <!-- Projects -->
      <section id="projects" class="page">
        <h2>Projects</h2>
        <div class="projects-grid">
          <div class="project card">
            <h4>Glacier UI</h4>
            <p>Dark theme UI kit with animated particles and shadow system.</p>
          </div>
          <div class="project card">
            <h4>3D Clock</h4>
            <p>Interactive 3D clock built with CSS transform and JS timing.</p>
          </div>
          <div class="project card">
            <h4>Portfolio Single-File</h4>
            <p>Single HTML/CSS/JS file portfolio with full navigation and animations.</p>
          </div>
          <div class="project card">
            <h4>Experiment: Snow Canvas</h4>
            <p>Canvas-based glacier snow system with parallax and depth.</p>
          </div>
        </div>
      </section>

      <!-- Contact -->
      <section id="contact" class="page">
        <h2>Contact</h2>
        <div class="card">
          <p>Want to collaborate or need help building a premium portfolio? Drop a message.</p>
          <form id="contactForm" class="contact-form" onsubmit="return false;">
            <input id="name" placeholder="Your name" required>
            <input id="email" type="email" placeholder="Email" required>
            <textarea id="message" placeholder="Message"></textarea>
            <div style="display:flex;gap:10px;align-items:center">
              <button id="sendBtn" class="btn" type="button">Send message</button>
              <div id="sentNote" class="meta" style="opacity:0;transition:opacity .3s">Message not sent (demo)</div>
            </div>
          </form>
        </div>
      </section>

    </div>
  </div>
</div>

<script>
/* ====== PAGE NAVIGATION ====== */
const navBtns = document.querySelectorAll('.nav-btn');
const pages = document.querySelectorAll('.page');

navBtns.forEach(btn=>{
  btn.addEventListener('click', ()=>{
    const target = btn.dataset.page;
    // active nav
    navBtns.forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
    // show page
    pages.forEach(p=>{
      if (p.id === target) { p.classList.add('active') } else { p.classList.remove('active') }
    });
    // small focus animation for name shadow (visual feedback)
    const nameShadow = document.querySelector('.name-shadow');
    nameShadow.style.transform = 'translateY(0) rotateX(0) scale(1)';
    nameShadow.style.opacity = '0.95';
    setTimeout(()=>{ nameShadow.style.transform = ''; nameShadow.style.opacity = '' }, 450);
  });
});

/* ====== CONTACT BUTTON (demo) ====== */
document.getElementById('sendBtn').addEventListener('click', ()=>{
  const name = document.getElementById('name').value.trim();
  const email = document.getElementById('email').value.trim();
  const message = document.getElementById('message').value.trim();
  const sent = document.getElementById('sentNote');

  // simple validation
  if (!name || !email) {
    sent.textContent = 'Please enter name & email';
    sent.style.opacity = 1;
    setTimeout(()=> sent.style.opacity = 0, 2000);
    return;
  }

  // This is a demo: we simulate sending and show shadow-based animation
  sent.textContent = 'Simulating send...';
  sent.style.opacity = 1;

  // Pulsing shadow on send button
  const btn = document.getElementById('sendBtn');
  btn.animate([
    { boxShadow: '0 14px 45px rgba(110,231,255,0.09)' },
    { boxShadow: '0 24px 80px rgba(110,231,255,0.18)' },
    { boxShadow: '0 14px 45px rgba(110,231,255,0.09)' }
  ], { duration: 1100 });

  setTimeout(()=>{
    sent.textContent = 'Message ready (demo) — no external send configured';
    setTimeout(()=> sent.style.opacity = 0, 2200);
  }, 900);
});

/* ====== SNOW CANVAS (glacier snow animated) ====== */
(function(){
  const canvas = document.getElementById('snow-canvas');
  const ctx = canvas.getContext('2d');
  let W, H, particles;

  function resize(){
    W = canvas.width = innerWidth;
    H = canvas.height = innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  function rand(min,max){ return Math.random()*(max-min)+min; }

  function createParticles(){
    particles = [];
    const count = Math.round((W*H)/90000) + 80; // adapt to screen
    for(let i=0;i<count;i++){
      particles.push({
        x: Math.random()*W,
        y: Math.random()*H,
        r: rand(0.6,3.6),
        speedX: rand(-0.3,0.6),
        speedY: rand(0.2,1.2),
        opacity: rand(0.05,0.9),
        layer: Math.random() // 0..1 for parallax layering
      });
    }
  }
  createParticles();
  window.addEventListener('resize', ()=>{ createParticles(); });

  // subtle waving field to mimic glacier wind currents
  function windField(x,y,t){
    return Math.sin((x*0.002) + t*0.0008) * 0.6 + Math.cos((y*0.003) + t*0.0006)*0.4;
  }

  let t0 = performance.now();
  function step(now){
    const dt = now - t0;
    t0 = now;

    ctx.clearRect(0,0,W,H);

    // faint glacier background sheen (radial)
    const g = ctx.createLinearGradient(0,0,W,H);
    g.addColorStop(0, 'rgba(140,200,255,0.02)');
    g.addColorStop(0.6, 'rgba(10,20,28,0.03)');
    g.addColorStop(1, 'rgba(0,0,0,0.05)');
    ctx.fillStyle = g;
    ctx.fillRect(0,0,W,H);

    // draw subtle moving fog near bottom
    const fog = ctx.createLinearGradient(0,H*0.6,0,H);
    fog.addColorStop(0, 'rgba(220,235,255,0.01)');
    fog.addColorStop(1, 'rgba(0,0,0,0.06)');
    ctx.fillStyle = fog;
    ctx.fillRect(0,H*0.6,W,H*0.4);

    const t = now/16;
    for(let p of particles){
      // parallax effect: front layers move faster
      const wind = windField(p.x, p.y, now) * (0.2 + p.layer*0.9);
      p.x += p.speedX + wind*0.4;
      p.y += p.speedY + (p.layer*0.6);

      // wrap
      if (p.x > W + 30) p.x = -30;
      if (p.x < -30) p.x = W + 30;
      if (p.y > H + 20) { p.y = -10; p.x = Math.random()*W; }

      // glow for larger flakes
      if (p.r > 2.2){
        const rad = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.r*6);
        rad.addColorStop(0, `rgba(220,245,255,${0.12*p.opacity})`);
        rad.addColorStop(1, `rgba(220,245,255,0)`);
        ctx.fillStyle = rad;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.r*6, 0, Math.PI*2);
        ctx.fill();
      }

      // draw flake (slightly elongated to feel windblown)
      ctx.beginPath();
      ctx.ellipse(p.x, p.y, p.r*1.6, p.r, Math.sin(now/1000 + p.x)*0.2, 0, Math.PI*2);
      ctx.fillStyle = `rgba(230,245,255,${0.18*p.opacity})`;
      ctx.fill();

      // tiny reflective streak for sparkle
      if (Math.random() < 0.012) {
        ctx.beginPath();
        ctx.fillStyle = `rgba(255,255,255,${0.06 + Math.random()*0.14})`;
        ctx.fillRect(p.x - p.r*1.1, p.y - p.r*0.1, p.r*2.2, p.r*0.18);
      }
    }

    // subtle animated spotlight (glacier sheen moving)
    const sx = (Math.sin(now*0.0003) + 1)/2 * W;
    const sy = (Math.cos(now*0.00022) + 1)/2 * H;
    const spotlight = ctx.createRadialGradient(sx, sy, 0, sx, sy, Math.max(W,H)*0.9);
    spotlight.addColorStop(0, 'rgba(120,200,255,0.02)');
    spotlight.addColorStop(1, 'rgba(0,0,0,0)');
    ctx.fillStyle = spotlight;
    ctx.fillRect(0,0,W,H);

    requestAnimationFrame(step);
  }

  requestAnimationFrame(step);
})();

/* ====== SMALL ACCESSIBILITY: keyboard nav (1-4) ====== */
window.addEventListener('keydown', (e)=>{
  if (e.key >= '1' && e.key <= '4'){
    const idx = Number(e.key)-1;
    const btn = document.querySelectorAll('.nav-btn')[idx];
    if (btn) btn.click();
  }
});

/* ====== Nice initial tiny animation for name shadow (on load) ====== */
document.addEventListener('DOMContentLoaded', ()=>{
  const name = document.querySelector('.name');
  name.animate([
    { transform: 'translateY(-8px) scale(0.98)', opacity:0 },
    { transform: 'translateY(0) scale(1)', opacity:1 }
  ], { duration: 700, easing: 'cubic-bezier(.2,.9,.2,1)'});
});
</script>

</body>
</html>
