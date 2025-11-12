/**
 * AK-GOAT Interactive README Animations
 * -------------------------------------
 * Animates:
 *  - 🐉 Dragon gliding between four buttons
 *  - 🐝 Bees swarming and orbiting around "Ak-GOAT" text
 *  - 🌈 "Ak-GOAT" text shimmer + glow synced with bees
 *
 * By aminkhan00478-art
 */


/** --- 🔧 HELPER FUNCTIONS --- **/

// Linear interpolation
function lerp(a, b, t) {
  return a + (b - a) * t;
}

// Ease-in-out cubic
function easeInOut(t) {
  return t < 0.5
    ? 4 * t * t * t
    : 1 - Math.pow(-2 * t + 2, 3) / 2;
}

// Random float in [min, max)
function rand(min, max) {
  return Math.random() * (max - min) + min;
}

// Clamp value
function clamp(val, min, max) {
  return Math.max(min, Math.min(max, val));
}

// Angle interpolation (handles wrapping at 2π)
function lerpAngle(a, b, t) {
  let d = b - a;
  while (d > Math.PI) d -= 2 * Math.PI;
  while (d < -Math.PI) d += 2 * Math.PI;
  return a + d * t;
}

// Detect tab visibility for throttling
let isTabActive = true;
document.addEventListener('visibilitychange', () => {
  isTabActive = !document.hidden;
});


/** --- 🐉 DRAGON ANIMATION --- **/

const dragon = document.getElementById('dragon');
const buttonIds = ['btn1', 'btn2', 'btn3', 'btn4'];
const buttons = buttonIds.map(id => document.getElementById(id));

// Dragon state
let dragonState = {
  currentTarget: 0,
  pos: { x: 0, y: 0 },
  prevPos: { x: 0, y: 0 },
  t: 0, // Progress between buttons
  pause: false,
  pauseTimer: 0,
  wingFlapTimer: 0,
  angle: 0,
  speed: rand(0.75, 1.5),
};

function getButtonCenter(idx) {
  const rect = buttons[idx].getBoundingClientRect();
  return {
    x: rect.left + rect.width / 2 + window.scrollX,
    y: rect.top + rect.height / 2 + window.scrollY
  };
}

// Initialize dragon at first button
(function initDragonPosition() {
  const start = getButtonCenter(0);
  dragonState.pos.x = start.x;
  dragonState.pos.y = start.y;
  dragonState.prevPos.x = start.x;
  dragonState.prevPos.y = start.y;
})();

// Animate a curved path from prev to target (quadratic Bezier)
function dragonPath(t, from, to) {
  // Control point for curve: mid-point, offset up/down by 40-80px, and add random lateral offset
  const mx = (from.x + to.x) / 2 + rand(-20, 20);
  const my = (from.y + to.y) / 2 + rand(-80, 80);
  const cx = mx;
  const cy = my;

  // Quadratic Bezier: (1-t)^2 * P0 + 2(1-t)t * CP + t^2 * P1
  const x = (1 - t) * (1 - t) * from.x + 2 * (1 - t) * t * cx + t * t * to.x;
  const y = (1 - t) * (1 - t) * from.y + 2 * (1 - t) * t * cy + t * t * to.y;
  return { x, y };
}

// Animate dragon
function animateDragon(dt) {
  if (dragonState.pause) {
    dragonState.pauseTimer -= dt;
    dragonState.wingFlapTimer += dt;
    // Flap effect: oscillate scale and rotate
    const flapScale = 1 + Math.sin(dragonState.wingFlapTimer * 8) * 0.10;
    const flapRotate = Math.sin(dragonState.wingFlapTimer * 7) * 8;
    dragon.style.transform = `translate(-50%, -50%) scale(${flapScale}) rotate(${flapRotate}deg)`;
    if (dragonState.pauseTimer <= 0) {
      // Resume flying to next button
      dragonState.pause = false;
      dragonState.speed = rand(0.8, 1.5);
      dragonState.prevPos = dragonState.pos;
      dragonState.currentTarget = (dragonState.currentTarget + 1) % buttons.length;
      dragonState.t = 0;
    }
    return;
  }

  // Get positions
  const targetIndex = dragonState.currentTarget;
  const fromIndex = (targetIndex === 0)
    ? buttons.length - 1
    : targetIndex - 1;
  const from = getButtonCenter(fromIndex);
  const to = getButtonCenter(targetIndex);

  // Advance `t` with some randomness
  let speed = dragonState.speed * dt * rand(0.95, 1.05) * 0.7;
  dragonState.t += speed;
  if (dragonState.t >= 1) {
    dragonState.t = 1;
    dragonState.pos = { ...to };
    dragonState.pause = true;
    dragonState.pauseTimer = rand(0.5, 1.1); // pause at button
    dragonState.wingFlapTimer = 0;
    dragon.style.transform = `translate(-50%, -50%) scale(1) rotate(0deg)`;
    return;
  }

  // Calculate Bezier curve position
  const progress = easeInOut(dragonState.t);
  const pos = dragonPath(progress, from, to);

  // Angle: direction of movement
  const dx = pos.x - dragonState.pos.x;
  const dy = pos.y - dragonState.pos.y;
  const angleTo = Math.atan2(dy, dx);
  dragonState.angle = lerpAngle(dragonState.angle, angleTo, 0.18);

  // Slight undulations for liveliness
  const undulateScale = 1 + Math.sin(Date.now() * 0.003 + dragonState.t * 6) * 0.045;
  const undulateRotate = Math.sin(Date.now() * 0.0027 + dragonState.t * 8) * 4;

  dragon.style.left = `${pos.x}px`;
  dragon.style.top = `${pos.y}px`;
  dragon.style.transform = `
    translate(-50%, -50%)
    rotate(${dragonState.angle * 180/Math.PI + undulateRotate}deg)
    scale(${undulateScale})
  `;
  dragonState.pos = pos;
}


/** --- 🐝 BEE SWARM ANIMATION --- **/

const beeElements = Array.from(document.getElementsByClassName('bee')) || [];
const akGoat = document.getElementById('akGoat');
const akGoatRect = akGoat.getBoundingClientRect();
const akGoatCenter = {
  x: akGoatRect.left + akGoatRect.width / 2 + window.scrollX,
  y: akGoatRect.top + akGoatRect.height / 2 + window.scrollY,
};

// Each bee gets its own motion parameters
const bees = beeElements.map((el, i) => ({
  el,
  angle: rand(0, 2 * Math.PI),
  baseRadius: rand(80, 120),
  floatRadius: rand(12, 28),
  speed: rand(0.22, 0.40),
  offset: rand(0, 1000), // random motion phase
  colorPhase: rand(0, 2 * Math.PI),
  toTextTimer: rand(0, 2 * Math.PI),
  flicker: 0,
}));

function animateBees(dt, timestamp) {
  bees.forEach((bee, idx) => {
    // Main motion: circular orbit + wavy float
    bee.angle += bee.speed * dt + Math.sin(timestamp * 0.001 + bee.offset) * 0.0006;

    // Occasionally "attract to text": reduce radius for a bit
    bee.toTextTimer += dt;
    let attracting = Math.sin(bee.toTextTimer + bee.offset) > 0.97;
    let radius = bee.baseRadius;
    if (attracting) {
      radius = lerp(bee.baseRadius, bee.baseRadius * 0.7, 0.5 + 0.5*Math.sin(bee.toTextTimer * 5 + bee.offset));
    }

    // Wobble layer: sinusoidal offset
    const floatX = Math.sin(timestamp * bee.speed * 0.7 + bee.offset) * bee.floatRadius;
    const floatY = Math.cos(timestamp * bee.speed * 0.8 + bee.offset) * bee.floatRadius * 0.8;

    // Final position: orbit center + float
    const bx = akGoatCenter.x + Math.cos(bee.angle) * radius + floatX;
    const by = akGoatCenter.y + Math.sin(bee.angle) * radius + floatY;

    bee.el.style.left = `${bx}px`;
    bee.el.style.top = `${by}px`;

    // Glow/color flicker: alternate white/green
    bee.colorPhase += dt * rand(2, 4); // flicker speed
    bee.flicker = Math.abs(Math.sin(bee.colorPhase));
    const glow = Math.round(bee.flicker * 6 + 3);
    bee.el.style.boxShadow =
      bee.flicker > 0.55
        ? `0 0 ${glow}px 2px #80ff80, 0 0 1px 0px #fff`
        : `0 0 ${glow}px 2px #fff, 0 0 1px 0px #80ff80`;

    bee.el.style.background = bee.flicker > 0.6 ? '#b3ffb3' : '#fff';
  });
}


/** --- 🌈 "Ak-GOAT" TEXT EFFECT --- **/

// Animate shifting linear gradient color and pulsing glow
let akGoatGradientPhase = 0;
function animateAkGoatText(dt, timestamp) {
  akGoatGradientPhase += dt * 0.18;

  // Colored gradient background with animated phase
  const gradAngle = Math.round((Math.sin(akGoatGradientPhase) + 1) * 45);
  const gradColors = [
    `#42f5e3 ${30 + Math.sin(akGoatGradientPhase) * 10}%`,
    `#9d42f5 ${60 + Math.cos(akGoatGradientPhase / 1.5) * 10}%`,
    `#ebf542 ${Math.abs(Math.sin(akGoatGradientPhase * 0.7)) * 100}%`
  ];
  akGoat.style.background =
    `linear-gradient(${gradAngle}deg, ${gradColors.join(', ')})`;

  // Shimmer text color (clip gradient for text fill)
  akGoat.style.webkitBackgroundClip = 'text';
  akGoat.style.webkitTextFillColor = 'transparent';
  akGoat.style.backgroundClip = 'text';
  akGoat.style.textFillColor = 'transparent';

  // Glow pulse tied to bee swarm phase
  const overallFlicker = bees.length
    ? bees.reduce((acc, b) => acc + b.flicker, 0) / bees.length
    : 0.5;
  const pulse = 0.4 + 0.7 * overallFlicker;
  akGoat.style.filter = `drop-shadow(0 0 ${10 + 45*pulse}px #b3ff30)`;
}


/** --- 🕓 MAIN ANIMATION LOOP --- **/

let lastTime = performance.now();
function animate() {
  const now = performance.now();
  const dt = isTabActive
    ? clamp((now - lastTime) / 1000, 0, 0.06)
    : 0; // Throttle if inactive

  animateDragon(dt);

  animateBees(dt, now);

  animateAkGoatText(dt, now);

  lastTime = now;
  window.requestAnimationFrame(animate);
}


/** --- 🚀 INIT SETUP --- **/

// Position dragon and bees absolutely/fixed (ensure CSS: position:absolute)
dragon.style.position = 'absolute';
dragon.style.pointerEvents = 'none';
dragon.style.zIndex = 9;

beeElements.forEach(b => {
  b.style.position = 'absolute';
  b.style.width = '16px';
  b.style.height = '16px';
  b.style.borderRadius = '50%';
  b.style.zIndex = 8;
  b.style.boxShadow = '0 0 8px 3px #fff';
  b.style.transition = 'box-shadow 0.1s, background 0.14s';
});

// Start animation
window.requestAnimationFrame(animate);


/** --- 📢 OPTIONAL: Responsive recalculation on resize (handles Ak-GOAT or button position changes) --- **/
window.addEventListener('resize', () => {
  const newRect = akGoat.getBoundingClientRect();
  akGoatCenter.x = newRect.left + newRect.width / 2 + window.scrollX;
  akGoatCenter.y = newRect.top + newRect.height / 2 + window.scrollY;
});
