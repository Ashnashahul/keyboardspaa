<!-- Digital Soft Keyboard Cleaner: Single-file Web version + Python/Pygame script

Save the HTML below as `keyboard-cleaner.html` and open it in a modern browser.
The Python script requires Python 3, pygame, and numpy (for sound synthesis). Save as `keyboard_cleaner_pygame.py`.

This file contains both the web implementation and the Python implementation (below the HTML) for convenience.
-->

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title> KEYBOARD SPA🌊✨</title>
<style>
  :root{
    --bg:#0b1222; --card:#0f1b2b; --accent:#76e4a6; --muted:#9fb3c8;
  }
  html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto; background:linear-gradient(180deg,#051020 0%, #071826 60%); color:#e6f2fb}
  .app{max-width:980px;margin:28px auto;padding:24px;border-radius:16px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));box-shadow:0 8px 40px rgba(2,6,23,0.6)}
  h1{margin:0 0 12px;font-weight:700;letter-spacing:0.2px}
  p.lead{margin:0 0 18px;color:var(--muted)}
  .controls{display:flex;gap:12px;align-items:center;margin-bottom:16px}
  .toggle{display:inline-flex;align-items:center;gap:8px}
  .kbd{background:linear-gradient(180deg,#091429,#07121e);padding:18px;border-radius:12px;display:grid;grid-template-columns:repeat(15,1fr);gap:8px}
  .key{background:linear-gradient(180deg,#122638,#0d1f2b);padding:10px 8px;border-radius:8px;text-align:center;box-shadow:inset 0 -6px 14px rgba(0,0,0,0.5);user-select:none;cursor:default;position:relative}
  .key.small{grid-column:span 1}
  .key.wide{grid-column:span 2}
  .key.extra{grid-column:span 3}
  .key.active{transform:translateY(-6px) scale(1.03);box-shadow:0 10px 30px rgba(0,0,0,0.6);}
  .status{display:flex;align-items:center;gap:12px;margin-top:14px}
  .progress-wrap{flex:1;background:rgba(255,255,255,0.04);height:12px;border-radius:8px;overflow:hidden}
  .progress{height:100%;width:0%;background:linear-gradient(90deg,var(--accent),#88f0c9);transition:width 0.12s linear}
  .timer{min-width:110px;text-align:right;color:var(--muted)}
  .bubbles{position:relative;height:140px;pointer-events:none}
  .bubble{position:absolute;border-radius:50%;opacity:0.9;mix-blend-mode:screen}
  .overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:none}
  .finale{background:linear-gradient(180deg, rgba(255,255,255,0.06), rgba(255,255,255,0.02));padding:28px 44px;border-radius:16px;border:1px solid rgba(255,255,255,0.06);box-shadow:0 20px 60px rgba(0,0,0,0.6);text-align:center;opacity:0;transform:scale(0.95);transition:all 400ms ease}
  .finale.show{opacity:1;transform:scale(1)}
  .sparkle{position:absolute;width:12px;height:12px;border-radius:50%;background:radial-gradient(circle,#fff, #ffd1f1 40%, transparent 60%);opacity:0.95}
  .muted{color:var(--muted);font-size:13px}
  .controls button{background:#0e2a2a;border:1px solid rgba(255,255,255,0.04);padding:8px 12px;border-radius:10px;color:#dff7ea;cursor:pointer}
  .note{font-size:13px;color:var(--muted);margin-top:8px}
  footer{margin-top:18px;font-size:13px;color:var(--muted);text-align:center}

  @keyframes bubbleUp{0%{transform:translateY(0) scale(0.6);opacity:0.9}100%{transform:translateY(-120px) scale(1.1);opacity:0}}
  @keyframes sparkleMove{0%{transform:translateY(0) scale(0.5);opacity:0}50%{opacity:1}100%{transform:translateY(-40px) scale(1.3);opacity:0}}
</style>
</head>
<body>
  <div class="app">
    <h1>WELCOME TO KEYBOARD SPA🌊✨</h1>
    <p class="lead">Press any key — watch it get pampered. Cleaning runs for exactly <strong>10 seconds</strong> after the last key press.</p>
    <div class="controls">
      <label class="toggle"><input id="musicToggle" type="checkbox"/> Elevator music (pop / hubble-bubble)</label>
      <button id="resetBtn">Reset timer</button>
      <div style="flex:1"></div>
      <div class="muted">Last key: <span id="lastKey">—</span></div>
    </div>

    <div class="kbd" id="keyboard"></div>

    <div class="status">
      <div class="progress-wrap"><div id="progress" class="progress"></div></div>
      <div class="timer">Cleaning in <span id="countdown">10.0</span>s</div>
    </div>

    <div class="bubbles" id="bubbles"></div>

    <div class="note">Tip: type anything — each keypress restarts the 10s cleaning timer. This is purely cosmetic. ✨</div>
    <footer>Made for fun — does not disinfect real keyboards.</footer>
  </div>

  <div class="overlay" id="overlay" style="display:none;">
    <div class="finale" id="finaleCard">
      <h2>Keyboard Cleaned! ✨</h2>
      <p class="muted">Enjoy your freshly *virtually* sanitized keyboard.</p>
    </div>
  </div>

<script>
// Basic keyboard layout rows for visuals
const layout = [
  ['Esc','F1','F2','F3','F4','F5','F6','F7','F8','F9','F10','F11','F12','Prt','Del'],
  ['`','1','2','3','4','5','6','7','8','9','0','-','=','Backspace'],
  ['Tab','Q','W','E','R','T','Y','U','I','O','P','[',']','\\'],
  ['Caps','A','S','D','F','G','H','J','K','L',';','\'', 'Enter'],
  ['Shift','Z','X','C','V','B','N','M',',','.','/','Shift'],
  ['Ctrl','Win','Alt','Space','Alt','Fn','Menu','Ctrl']
];

const keyboard = document.getElementById('keyboard');
const bubbles = document.getElementById('bubbles');
let lastKeySpan = document.getElementById('lastKey');
let progressEl = document.getElementById('progress');
let countdownEl = document.getElementById('countdown');
let overlay = document.getElementById('overlay');
let finale = document.getElementById('finaleCard');
let musicToggle = document.getElementById('musicToggle');
let resetBtn = document.getElementById('resetBtn');

// Build key elements
function makeKeyLabel(t){
  const k = document.createElement('div');
  k.className = 'key';
  if(t.length>1) k.classList.add('small');
  if(t==='Backspace' || t==='Enter' || t==='Shift' || t==='Space') k.classList.add('wide');
  if(t==='Space') { k.classList.add('extra'); k.style.gridColumn='span 5'; }
  k.textContent = t;
  k.dataset.key = t.toLowerCase();
  return k;
}

layout.forEach(row => row.forEach(k => keyboard.appendChild(makeKeyLabel(k))));

// WebAudio setup for sounds
const AudioCtx = window.AudioContext || window.webkitAudioContext;
const actx = new AudioCtx();

function playPop(){
  const o = actx.createOscillator();
  const g = actx.createGain();
  o.type='square'; o.frequency.value=600; g.gain.value=0.06;
  o.connect(g); g.connect(actx.destination);
  o.start(); o.stop(actx.currentTime + 0.06);
}

function playBubble(){
  // short descending sine + noise
  const osc = actx.createOscillator(); const gain = actx.createGain();
  osc.type='sine'; osc.frequency.setValueAtTime(420, actx.currentTime);
  osc.frequency.exponentialRampToValueAtTime(120, actx.currentTime+0.26);
  gain.gain.setValueAtTime(0.06, actx.currentTime);
  osc.connect(gain);
  // add subtle noise
  const bufferSize = 2*actx.sampleRate; const noiseBuffer = actx.createBuffer(1, bufferSize, actx.sampleRate);
  const data = noiseBuffer.getChannelData(0);
  for(let i=0;i<bufferSize;i++) data[i] = (Math.random()*2-1)*0.3;
  const noise = actx.createBufferSource(); noise.buffer=noiseBuffer; const noiseGain = actx.createGain(); noiseGain.gain.value=0.005;
  noise.connect(noiseGain); noiseGain.connect(gain);
  gain.connect(actx.destination);
  osc.start(); noise.start();
  osc.stop(actx.currentTime+0.28); noise.stop(actx.currentTime+0.28);
}

let musicNode = null;
function startMusic(){
  if(musicNode) return;
  const master = actx.createGain(); master.gain.value=0.02; master.connect(actx.destination);
  // simple gentle arpeggio using oscillators
  let i = 0;
  const notes = [440,550,660,880];
  function schedule(){
    const o = actx.createOscillator(); const g = actx.createGain();
    o.type='triangle'; o.frequency.value = notes[i%notes.length];
    g.gain.value=0.03; o.connect(g); g.connect(master);
    const t = actx.currentTime;
    o.start(t); o.stop(t+0.4);
    i++;
    musicNode = setTimeout(schedule, 320);
  }
  schedule();
}
function stopMusic(){ if(musicNode){ clearTimeout(musicNode); musicNode=null; } }

// timer logic
const CLEAN_SECONDS = 10.0; // exactly 10s after last key
let timer = null; let remaining = CLEAN_SECONDS; let lastPress = 0;

function startOrResetTimer(){
  lastPress = performance.now();
  remaining = CLEAN_SECONDS;
  if(timer) cancelAnimationFrame(timer);
  animate();
}

function animate(){
  const now = performance.now();
  const elapsed = (now - lastPress)/1000.0;
  const rem = Math.max(0, CLEAN_SECONDS - elapsed);
  const pct = (1 - rem/CLEAN_SECONDS)*100;
  progressEl.style.width = pct + '%';
  countdownEl.textContent = rem.toFixed(1);
  if(rem<=0){ showFinale(); stopMusicIfNeeded(); return; }
  timer = requestAnimationFrame(animate);
}

function stopMusicIfNeeded(){ if(!musicToggle.checked) stopMusic(); }

function showFinale(){ overlay.style.display='flex'; setTimeout(()=> finale.classList.add('show'), 20);
  // sparkles
  for(let i=0;i<22;i++){
    const s = document.createElement('div'); s.className='sparkle';
    s.style.left = (40 + Math.random()*60) + '%'; s.style.top = (40 + Math.random()*20) + '%';
    s.style.width = (6+Math.random()*18)+'px'; s.style.height = s.style.width;
    s.style.animation = `sparkleMove ${0.9 + Math.random()*0.9}s linear ${Math.random()*0.2}s forwards`;
    overlay.appendChild(s);
    setTimeout(()=> s.remove(), 2400);
  }
  // hide after a while
  setTimeout(()=>{ finale.classList.remove('show'); overlay.style.display='none'; progressEl.style.width='0%'; countdownEl.textContent = CLEAN_SECONDS.toFixed(1); }, 2600);
}

// bubble visual effect for a keypress target
function spawnBubblesForKey(keyEl){
  const rect = keyEl.getBoundingClientRect(); const parentRect = bubbles.getBoundingClientRect();
  const x = rect.left - parentRect.left + rect.width/2; const y = rect.top - parentRect.top + rect.height/2;
  const count = 6 + Math.floor(Math.random()*6);
  for(let i=0;i<count;i++){
    const b = document.createElement('div'); b.className='bubble';
    const size = 6 + Math.random()*28; b.style.width = size+'px'; b.style.height = size+'px';
    const cx = x + (Math.random()-0.5)*rect.width*0.8; const cy = y + (Math.random()-0.5)*rect.height*0.7;
    b.style.left = cx + 'px'; b.style.top = cy + 'px';
    b.style.background = `radial-gradient(circle, rgba(255,255,255,0.9), rgba(255,255,255,0.08))`;
    b.style.opacity = 0.9; b.style.transform = `translateY(0) scale(${0.6+Math.random()*0.8})`;
    b.style.animation = `bubbleUp ${0.9 + Math.random()*0.9}s cubic-bezier(.2,.9,.1,1)`;
    bubbles.appendChild(b);
    setTimeout(()=> b.remove(), 1400 + Math.random()*900);
  }
}

// highlight key element matched by event
function findKeyElement(k){
  const txt = k.length===1? k.toUpperCase() : k;
  // try matching data-key or textContent
  const els = Array.from(document.querySelectorAll('.key'));
  return els.find(e=> e.textContent.toLowerCase()===txt.toLowerCase() || e.dataset.key===txt.toLowerCase() ) || els.find(e=> e.textContent.toLowerCase().includes(txt.toLowerCase()));
}

window.addEventListener('keydown', (ev)=>{
  // ignore modifier-only repeated events
  if(ev.key===undefined) return;
  const keyName = ev.key;
  lastKeySpan.textContent = keyName;
  const el = findKeyElement(keyName.length===1? keyName : keyName);
  if(el){ el.classList.add('active'); setTimeout(()=>el.classList.remove('active'),180); spawnBubblesForKey(el); }
  // sounds
  playPop(); setTimeout(playBubble, 40);
  if(musicToggle.checked) startMusic();
  startOrResetTimer();
});

resetBtn.addEventListener('click', ()=>{
  lastPress = performance.now(); startOrResetTimer(); overlay.style.display='none'; stopMusicIfNeeded();
});

musicToggle.addEventListener('change', ()=>{
  if(!musicToggle.checked) stopMusic();
});

// start with no timer, but show 10s idle
countdownEl.textContent = CLEAN_SECONDS.toFixed(1);

// make clicking keys simulate press
document.querySelectorAll('.key').forEach(k=>{
  k.addEventListener('click', ()=>{
    const label = k.textContent; lastKeySpan.textContent = label; k.classList.add('active'); setTimeout(()=>k.classList.remove('active'),180);
    playPop(); setTimeout(playBubble, 40); spawnBubblesForKey(k);
    if(musicToggle.checked) startMusic(); startOrResetTimer();
  });
});

</script>


<!--
------------------------------
Optional Python + Pygame version (save as keyboard_cleaner_pygame.py)
Requires: pygame, numpy
Run: python keyboard_cleaner_pygame.py
This script synthesizes simple pop/bubble tones using numpy and pygame.sndarray and shows a minimal GUI.
------------------------------
-->


<pre style="display:none">
# ---------------------- keyboard_cleaner_pygame.py ----------------------
# Minimal Pygame implementation (optional)
# Requires: pip install pygame numpy

"""
Digital Soft Keyboard Cleaner - Pygame version
Press keys — each press restarts a 10s cleaning timer with sounds and animated bubbles.
"""

import pygame
import numpy as np
import time

WIDTH, HEIGHT = 900, 420
CLEAN_SECONDS = 10.0

pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Digital Soft Keyboard Cleaner - Pygame')
clock = pygame.time.Clock()

# sound generation helpers
def make_tone(freq, length=0.12, volume=0.3, sr=44100):
    t = np.linspace(0, length, int(sr*length), False)
    wave = 0.5*np.sin(2*np.pi*freq*t)
    # apply quick envelope
    env = np.ones_like(wave)
    env *= np.linspace(1.0, 0.001, env.size)
    wave = wave * env * volume
    audio = np.int16(wave * 32767)
    return pygame.sndarray.make_sound(audio)

pop_sound = make_tone(700, 0.06, 0.4)
bubble_sound = make_tone(420, 0.22, 0.25)

# key visuals
font = pygame.font.SysFont('Segoe UI', 18)
small = pygame.font.SysFont('Segoe UI', 14)
keys = []

# simple layout for visuals
rows = [ ['Q','W','E','R','T','Y','U','I','O','P'], ['A','S','D','F','G','H','J','K','L'], ['Z','X','C','V','B','N','M'] ]
start_y = 80
for ri, row in enumerate(rows):
    start_x = 60 + ri*18
    y = start_y + ri*70
    for i,k in enumerate(row):
        rect = pygame.Rect(start_x + i*62, y, 56, 48)
        keys.append((k, rect))

last_press = time.time()
running = True
bubbles = []

def spawn_bubbles(x, y):
    for i in range(np.random.randint(6,12)):
        vx = np.random.uniform(-0.3,0.3)
        vy = np.random.uniform(-2.2,-0.6)
        life = np.random.uniform(0.8,1.6)
        size = np.random.randint(6,24)
        bubbles.append([x,y,vx,vy,life,size])

while running:
    dt = clock.tick(60)/1000.0
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            last_press = time.time()
            pop_sound.play()
            pygame.time.delay(30)
            bubble_sound.play()
            # spawn bubbles near center
            spawn_bubbles(WIDTH//2, HEIGHT//2)

    # update bubbles
    for b in bubbles[:]:
        b[0] += b[2]*60*dt
        b[1] += b[3]*60*dt
        b[4] -= dt
        if b[4] <= 0:
            bubbles.remove(b)

    # cleaning timer
    rem = max(0, CLEAN_SECONDS - (time.time()-last_press))
    progress = (1 - rem/CLEAN_SECONDS)

    screen.fill((6,12,24))
    # draw keys
    for k, r in keys:
        pygame.draw.rect(screen, (18,36,56), r, border_radius=8)
        pygame.draw.rect(screen, (6,18,30), r.inflate(-6,-6), border_radius=6)
        txt = font.render(k, True, (220,245,240))
        screen.blit(txt, txt.get_rect(center=r.center))

    # draw bubbles
    for b in bubbles:
        x,y,_,_,life,size = b
        alpha = int(max(0, min(255, 255*(life/1.6))))
        surf = pygame.Surface((size,size), pygame.SRCALPHA)
        pygame.draw.circle(surf, (255,255,255,alpha), (size//2, size//2), size//2)
        screen.blit(surf, (x-size//2, y-size//2))

    # progress bar
    pygame.draw.rect(screen, (22,40,60), (60, 30, WIDTH-120, 18), border_radius=9)
    pygame.draw.rect(screen, (120,228,166), (60, 30, int((WIDTH-120)*progress), 18), border_radius=9)
    time_text = small.render(f'Cleaning in {rem:.1f}s', True, (160,190,200))
    screen.blit(time_text, (WIDTH-220, 52))

    if rem <= 0 and len(bubbles)==0:
        # show finale for a brief time
        finale_text = font.render('Keyboard Cleaned! ✨', True, (255,255,255))
        screen.blit(finale_text, finale_text.get_rect(center=(WIDTH//2, HEIGHT//2)))

    pygame.display.flip()

pygame.quit()
# ----------------------------------------------------------------------
</pre>

</body>
</html>
