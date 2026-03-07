# ◈ IAMD Simulation - Integrated Air & Missile Defense

![p5.js](https://img.shields.io/badge/p5.js-v1.9.4-ED225D?style=flat-square&logo=p5dotjs)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat-square&logo=javascript)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-deployed-22863a?style=flat-square&logo=github)
![License: MIT](https://img.shields.io/badge/License-MIT-00cc40?style=flat-square)

> A browser-based 2D air & missile defense simulation featuring **Proportional Navigation guidance**, **Kalman filter tracking**, and **probabilistic radar detection** - all running in a single `index.html` with no build tools. Click to test.

[Click here to test the simulation](https://definitelynotguru.github.io/IAMD-Simulation---Integrated-Air-Missile-Defense/)
---

<!-- HERO GIF PLACEHOLDER — record with LICEcap or ShareX and replace below -->
![IAMD Simulation Demo](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExNDM1MHNvZHR5NnZsOHgzNjZ3azA1M3Q2cmFzbWxsOGM2amxsNml0aSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/7oXHjHjxaj5FwTj08p/giphy.gif)
<!-- Suggested recording: ~20s clip showing a drone swarm wave intercepted + one HGV leaking through -->

---

## ▶ One-Click Deploy (GitHub Pages)

```bash
# 1. Fork this repo
# 2. Go to Settings → Pages → Source: "Deploy from branch" → main → / (root)
# 3. GitHub will give you a live URL in ~60 seconds
# 4. Share the link - it runs in any browser, no installs
```

Or clone and open directly:
```bash
git clone https://github.com/definitelynotguru/IAMD-Simulation---Integrated-Air-Missile-Defense.git
cd iamd-sim
open index.html   # macOS
# or just drag index.html into your browser
```

---

## 🎮 Controls

| Key | Action |
|-----|--------|
| `R` | Restart simulation |
| `1` | Spawn ballistic missile wave |
| `2` | Spawn cruise missile wave |
| `3` | Spawn hypersonic glide vehicle |
| `4` | Spawn drone swarm (Boids AI) |
| `+` / `-` | Increase / decrease auto-wave frequency |

**C2 is automatic** - the Command & Control system detects tracks via Kalman filter and auto-assigns interceptors using a greedy assignment algorithm.

---

## ⚙ Physics Explainer

This simulation implements real aerospace guidance and sensor mathematics - simplified for real-time browser rendering but structurally faithful to operational systems.

### Proportional Navigation (PN Guidance)
All interceptors use the standard military guidance law:

```
a_m = N · V_c · (dθ/dt)
```

- `a_m` - commanded acceleration (perpendicular to line-of-sight)
- `N` - navigation constant (set to 4; real SAMs use 3–5)
- `V_c` - closing velocity between missile and target
- `dθ/dt` - line-of-sight angular rate (computed each frame)

**Key insight:** if the LOS angle isn't changing (`dθ/dt = 0`), both objects are guaranteed to collide - PN exploits this geometry to steer intercepts without predicting exact future position.

### Kalman Filter Tracking
Radar measurements are noisy (±4 px Gaussian noise added per detection). A two-state Kalman filter estimates true position and velocity from these noisy returns:

```
Predict:  x̂ = F·x̂_prev,    P = F·P·Fᵀ + Q
Update:   K = P·Hᵀ·(H·P·Hᵀ + R)⁻¹
          x̂ = x̂ + K·(z − H·x̂)
```

Green crosshairs on-screen show the Kalman-estimated track position - watch them lag slightly behind fast hypersonic threats!

### Radar Detection Probability
Based on the radar range equation (R⁴ power law):

```
P_detect = 1 / (1 + (R/R_max)^4) × (σ/σ_nom)
```

- `σ` (RCS) varies by threat type: drones `σ=0.05` (stealthy), ballistic `σ=0.9` (easy to detect)
- Low RCS threats may ghost through radar coverage undetected → leakage

### Threat Types
| Threat | RCS | Speed | Behavior |
|--------|-----|-------|---------|
| Ballistic ICBM | 0.9 | Medium | Parabolic arc via `y = vy·t − ½g·t²` |
| Cruise missile | 0.4 | Slow | Constant altitude + sinusoidal undulation |
| Hypersonic HGV | 0.6 | Very fast | Shallow glide with evasive wobble |
| Drone swarm | 0.05 | Slow | Boids flocking (separation + alignment + cohesion) |

---

## 🗂 Architecture (Modular - easy to extend)

```
index.html
├── CFG {}              — all tunable params (one place)
├── KF1D / KalmanTrack  — decoupled 2D Kalman filter
├── Radar               — detection prob + track management
├── Threat subclasses   — Ballistic, Cruise, Hypersonic, Drone
├── Interceptor         — PN guidance law
├── Particle            — explosion effects
├── C2 (runC2)          — greedy interceptor assignment
└── setup/draw          — p5.js main loop
```

### Planned Phase 2 Additions
- [ ] Jamming / ECM reducing radar `P_detect`
- [ ] MIRV decoys splitting from ballistic re-entry
- [ ] Auction-based interceptor assignment (replace greedy)
- [ ] Realistic acceleration limits with energy management
- [ ] ML-based threat classification (TensorFlow.js)

---

## 📜 License

MIT — fork freely, credit appreciated.

---

*Built with [p5.js](https://p5js.org/) · Physics from open aerospace textbooks · No external APIs, no backend, no npm — just open `index.html`*
