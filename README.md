# IAMD Simulation - Integrated Air & Missile Defense

![p5.js](https://img.shields.io/badge/p5.js-v1.9.4-ED225D?style=flat-square&logo=p5dotjs)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat-square&logo=javascript)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-deployed-22863a?style=flat-square&logo=github)
![License: MIT](https://img.shields.io/badge/License-MIT-00cc40?style=flat-square)

> A browser-based 2D air and missile defense simulation with **layered interceptor tiers**, **Shoot-Look-Shoot engagement doctrine**, **Proportional Navigation guidance**, **Kalman filter tracking**, and **probabilistic radar detection**. Everything runs in a single `index.html` with no build tools.

[**Try the live simulation**](https://definitelynotguru.github.io/IAMD-Simulation---Integrated-Air-Missile-Defense/)

---

<!-- HERO GIF PLACEHOLDER - record with LICEcap or ShareX and replace below -->
![IAMD Simulation Demo](https://media.discordapp.net/attachments/692810348650299533/1480192508096151662/brave_nlkn4ivOvf.gif?ex=69aec832&is=69ad76b2&hm=ac36c3502af21af820e5c514768eb3e5e3ddd51764f1a6df42cd9fdaa9e9a4ef&=&width=974&height=552)
<!-- Suggested recording: ~20s clip showing tiered interceptors engaging a mixed threat wave -->

---

## Features

### Layered Defense Architecture
Three interceptor tiers modeled after real-world IAMD layered defense:

| Tier | Analog | Color | Speed | SSPK | Preferred Threats |
|------|--------|-------|-------|------|-------------------|
| **UPPER** | THAAD | Red | 14 | 80% | Ballistic, Hypersonic |
| **MIDDLE** | Patriot PAC-3 | Blue | 10 | 75% | Cruise, Ballistic |
| **LOWER** | SHORAD | Green | 8 | 70% | Drones, Cruise |

Each tier has its own engagement zone (HIMEZ / LOMEZ / SHORADEZ), magazine capacity, and cooldown timing. Zone bands are drawn on screen so you can see where each tier operates.

### Shoot-Look-Shoot (SLS) Doctrine
The C2 system fires, waits for an assessment window, then re-engages with the next tier down if the first shot missed. This cascading logic means a ballistic missile might get a THAAD shot first, then a PAC-3 follow-up if it survives, rather than wasting two THAAD rounds.

### Proportional Navigation Guidance
All interceptors steer using the standard PN law:

```
a_m = N * V_c * (d_theta/dt)
```

- `a_m` - commanded lateral acceleration
- `N` - navigation constant (3.5 to 4.5 depending on tier)
- `V_c` - closing velocity
- `d_theta/dt` - line-of-sight rate

If the LOS angle stops changing, both objects are on a collision course. PN exploits this geometry to guide intercepts without needing to predict exact future positions.

### Kalman Filter Tracking
Noisy radar returns are fused into smooth tracks using a linear Kalman filter:

```
Predict:  x_pred = F * x
          P_pred = F * P * F^T + Q
Update:   K = P_pred * H^T * (H * P_pred * H^T + R)^(-1)
          x = x_pred + K * (z - H * x_pred)
          P = (I - K * H) * P_pred
```

Each track maintains its own covariance matrix. You can see the uncertainty ellipses around tracks shrink as the filter converges.

### Radar Detection Model
Detection is probabilistic, not binary. Each scan computes:

```
SNR = (P_tx * G^2 * lambda^2 * RCS) / ((4*pi)^3 * k_B * T_s * R^4)
P_d = 1 - exp(-SNR/2)
```

Drones with high RCS light up early. HGVs with low RCS can slip through several scans before being acquired.

---

## Threat Types

| Threat | Speed | RCS | Behavior |
|--------|-------|-----|----------|
| **Ballistic** | Fast (8-12) | High | Parabolic arc, gravity-driven re-entry |
| **Cruise** | Medium (4-6) | Med | Low-altitude weaving, terrain-following |
| **HGV** | Very fast (14-18) | Low | Skip-glide trajectory, hard to track |
| **Drone Swarm** | Slow (2-3) | Tiny | Boids flocking (separation + alignment + cohesion) |

### Boids Flocking (Drone Swarms)
Drone swarms use Craig Reynolds' three-rule algorithm:
- **Separation** - avoid crowding neighbors
- **Alignment** - steer toward average heading
- **Cohesion** - move toward average position

Each drone is individually tracked by radar and requires its own interceptor.

---

## HUD and Stats

The simulation displays real-time information:
- **Per-tier magazine bars** showing remaining rounds for UPPER, MIDDLE, and LOWER
- **Fire / Hit / Miss counters** broken down by tier
- **SLS re-engage counter** tracking how many cascade re-engagements have occurred
- **Kill feed** with tier labels showing which interceptor type scored each kill
- **Engagement zone bands** (HIMEZ / LOMEZ / SHORADEZ) drawn on the canvas
- **Radar sweep** with detection probability visualization
- **Track uncertainty ellipses** from the Kalman filter covariance

---

## Controls

| Key | Action |
|-----|--------|
| `R` | Restart simulation |
| `1` | Spawn ballistic missile wave |
| `2` | Spawn cruise missile wave |
| `3` | Spawn hypersonic glide vehicle |
| `4` | Spawn drone swarm (Boids AI) |
| `+` / `-` | Increase / decrease auto-wave frequency |

**C2 is fully automatic.** The system detects tracks via Kalman filter, classifies threats by altitude band, selects the best-fit interceptor tier, and fires using the SLS doctrine.

---

## Deploy

### GitHub Pages (one click)

```bash
# 1. Fork this repo
# 2. Settings > Pages > Source: "Deploy from branch" > main > / (root)
# 3. Live URL in ~60 seconds
# 4. Share the link - runs in any browser, no installs
```

### Local

```bash
git clone https://github.com/definitelynotguru/IAMD-Simulation---Integrated-Air-Missile-Defense.git
cd IAMD-Simulation---Integrated-Air-Missile-Defense
open index.html   # macOS
# or just drag index.html into your browser
```

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Rendering** | [p5.js](https://p5js.org) v1.9.4 (Canvas 2D) |
| **Language** | JavaScript ES6+ (~1,800 lines) |
| **Architecture** | Single `index.html`, zero build tools, one external dependency |
| **Styling** | Inline CSS, CRT terminal aesthetic with scanline overlay and phosphor glow |
| **Font** | Share Tech Mono (Google Fonts) |
| **CI/CD** | GitHub Actions (deploy, lint, test) |

---

## CI/CD

Three GitHub Actions workflows run on push:

| Workflow | What it does |
|----------|-------------|
| `deploy.yml` | Auto-deploys to GitHub Pages on push to main |
| `lint.yml` | HTMLHint linting + Lychee link checking |
| `test.yml` | JS syntax validation (Node.js 20), component checks, performance checks |

---

## License

MIT - do whatever you want, just don't blame me if it doesn't actually shoot down real missiles.

---

*Built with [p5.js](https://p5js.org) - the creative coding library that makes Canvas fun.*
