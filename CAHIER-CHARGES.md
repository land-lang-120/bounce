# 📘 bounce — Cahier des charges

> Document rétrospectif. Source de vérité pour la vision, le périmètre et la roadmap.
> Lié à : [PROGRESS.md](PROGRESS.md) (suivi opérationnel)
> Mis à jour : **2026-04-23**

---

## 1. Vision

**Bounce ∞ Labyrinth** est un **runner / platformer arcade** en mode paysage où le joueur contrôle une boule qui rebondit dans un labyrinthe **infini procédural**. 5 zones thématiques (Donjon, Lave, Cyber, Abyssal sous-marin, Cristal) défilent en boucle, avec rings à collectionner (style Sonic), gemmes, ennemis, pièges électriques, power-ups, checkpoints et leaderboard.

**Tagline** : « Rebondis. Survis. Repousse les limites. »

**Promesse joueur** :
- Un platformer infini avec progression visible (zones, distance, gems)
- Paysage **forcé** (mode portrait = écran de rotation)
- Audio synthétisé (Web Audio API, 1 mélodie par zone)
- 5 skins déblocables (Fire / Ice / Gold / Neon / Void)
- 100% offline, 100% gratuit, install PWA en 2 secondes

---

## 2. Personas

### Persona 1 — Le runner casual (cible primaire)
- Joue 5–10 min en pause
- Cherche un platformer qui va vite, pas trop dur
- Apprécie l'aléatoire (pas 2 runs identiques grâce au labyrinthe procédural)
- KPI : sessions/semaine + zones max atteintes

### Persona 2 — Le complétionniste
- Veut **collectionner les 5 skins** (50 / 150 / 300 / 500 gemmes + 1 free)
- Suit son total gemmes (`SD.totalGems`)
- KPI : % skins débloqués + total gemmes lifetime

### Persona 3 — Le scoreur
- Veut son nom dans le top 3 du classement local
- Optimise la collecte des rings (+100 / ring) + bonus zone complète (+500)
- KPI : best score + position classement

### Persona 4 — Le joueur sensible aux ambiances
- Aime alterner les 5 thèmes (musique procédurale + palette + tile graphics)
- Apprécie particulièrement le mode sous-marin (Abyssal) avec gravité réduite + overlay water
- KPI : temps passé par zone + favori personnel

---

## 3. Périmètre fonctionnel

### Mécaniques core
- **Boule rebondissante** avec gravité directionnelle (`gravDir = ±1`, possibilité d'inverser)
- **Double saut** (2 jumps avant grounded reset)
- **Squash & stretch** (`ball.sqV` indicator) sur impacts
- **Joystick OU boutons** (paramétrable, 2 modes de contrôle)
- **Vitesse adaptative** selon thème
- **Mort instantanée** sur tiles spike (`TS`) ou contact ennemi/élec sans bouclier
- **Respawn checkpoint** ou point de départ si pas de checkpoint

### Génération procédurale du labyrinthe
- **Chunks** : largeur 22 tiles (`CHUNK_W`), 12 lignes hauteur (`ROWS = 12`)
- **Corridors verticaux** dynamiques (1 → 2 corridors avec split/merge aléatoire)
- **Décalages** chaque 3–7 colonnes (montée/descente)
- **Items** semés stochastiquement (skip 2 premiers chunks pour grace period) :
  - Gemme (TG, 2.8%)
  - Spike (TS, 0.6%)
  - Bumper (TB, 0.4%)
  - Power-Up (TPU, 0.3%)
  - Fan (TFN, 0.3%)
- **Checkpoints** (TCP) tous les 8 chunks
- **Génération à la volée** : `ensureGen(tileX + 65)` avec garbage collect (max 10 chunks en mémoire)

### Zones thématiques (5)
| # | Nom | Underwater | Tile colors | Accent | Music seq |
|---|---|---|---|---|---|
| 0 | DONJON | non | violet/dark | `#9c4de4` | Mineur médiéval |
| 1 | LAVE | non | rouge/orange | `#ff5500` | Tribal |
| 2 | CYBER | non | vert néon | `#00e676` | Synthwave |
| 3 | ABYSSAL | **oui** | bleu profond | `#1e88ff` | Ambient |
| 4 | CRISTAL | non | rose/violet | `#ea80fc` | Cristallin aigu |

- Changement automatique tous les 200 tiles (`distanceTiles`)
- Annonce zone `#zonev` overlay (3s)
- Mélodie change (`startMusic(theme.name)`)
- Mode underwater : gravité réduite + overlay bleu + son water

### Collectibles & ennemis
- **Rings** : 3–5 par chunk, +100 pts, +500 si zone complète, particules
- **Gems** : +10 pts, +1 total gem (skin unlock progress)
- **Enemies** : tués si stomp dorsal (+50 pts), sinon kill ball (sauf bouclier)
- **Electric bars** : pulse activé/inactif via period + phase, kill ball si actif
- **Spikes** : kill instantané, pas de bouclier qui sauve
- **Bumpers** : rebond × 1.4 vitesse jump, particle confetti

### Power-ups (4)
| Power-up | Durée | Effet |
|---|---|---|
| **Shield** 🛡️ | 360 frames | Bloque 1 hit (ennemi ou élec), absorbe et meurt |
| **Super Jump** ⬆️ | 240 frames + 3 charges | Saut 1.4× hauteur (3 utilisations) |
| **Magnet** 🧲 | 360 frames | Aimante gemmes dans 4 tiles autour |
| **Slowmo** ⏱️ | 380 frames | Ralenti le monde |

- Spawn random parmi 4 types
- Affichés dans `#pups` HUD avec progress bar décroissante

### Audio (Web Audio API)
- AudioContext synthé (oscillator + gain)
- 11 SFX : jump / jump2 / gem / ring / death / ckpt / pu / zone / enemy / fan / elec / water
- 5 mélodies procédurales (1 par zone, sequences MSEQS)
- Toggle ON/OFF dans paramètres (`SD.settings.sound`)

### Vibrations haptiques
- `navigator.vibrate()` sur jump, gem (16ms), ring (séquence), death (80/30/80), checkpoint, power-up

### UI / écrans
- **Splash** (load bar gradient orange/jaune)
- **Menu** : logo BOUNCE ∞ LABYRINTH + 4 boutons (Jouer / Skins / Classement / Paramètres)
- **Game** : canvas plein écran + HUD (zone / rings / gems / score / under-pill)
- **HUD power-ups** sous le HUD principal
- **Game Over** : titre rouge + info distance
- **Settings** : toggle Boutons/Joystick + Sons ON/OFF
- **Leaderboard** : top scores locaux (pas encore Firebase)
- **Skins** : 5 cartes (Fire free + 4 cost gems)
- **Zone announcement** : overlay ZONE X NOM HINT

### Persistance
- `localStorage.bounce_maze_v2` :
  ```json
  {
    "totalGems": 0,
    "scores": [],
    "unlockedSkins": [0],
    "selectedSkin": 0,
    "settings": { "joystick": false, "sound": true }
  }
  ```

### Orientation forcée
- **Paysage obligatoire** : si portrait → overlay #pw avec icône rotation animée, jeu masqué

### PWA
- `manifest.json` standalone
- Service worker `sw.js` enregistré
- 2 icons SVG (192/512)
- Theme color `#000000`

---

## 4. Architecture & stack

### Stack actuelle
- **HTML monolithique** : 1334 lignes dans `index.html`
- **CSS inline** : ~90 lignes dans `<style>`
- **JS vanilla** : ~1100 lignes, 61 fonctions globales
- **Canvas 2D** : tout le rendu (ball, tiles, rings, particles, enemies, eBars)
- **Web Audio API** : AudioContext synthé, MSEQS séquences notes
- **0 dépendance** npm

### Fichiers
```
bounce/
├── index.html              # Tout le jeu (1334 lignes)
├── manifest.json
├── sw.js
├── icons/
│   ├── icon-192.svg
│   └── icon-512.svg
├── PROGRESS.md
├── README.md
└── CAHIER-CHARGES.md       # ← ce document
```

### State globals
- `SD` : save data (totalGems, scores, unlockedSkins, selectedSkin, settings)
- `ball` : { x, y, vx, vy, r, sqV, grounded, jumpsLeft, dead }
- `generatedChunks[]` : labyrinth tiles (max 10)
- `rings[]`, `eBars[]`, `enemies[]`, `parts[]`, `floats[]`
- `PU` : { shield, superjump, magnet, slowmo } (durées)
- `currentZone`, `distanceTiles`, `score`, `gems`, `deaths`
- `K` : { fwd, bck } input keys
- `joyDX`, `joyTouchId`, `joyBaseX` : joystick state

### Cible (post-refactoring)
```
bounce/
├── src/
│   ├── main.ts                 # Entry point + game loop
│   ├── state.ts                # Save / runtime state
│   ├── canvas.ts               # Resize, ROWS, TILE constants
│   ├── physics/
│   │   ├── ball.ts             # Move, collide, gravity, double jump
│   │   ├── input.ts            # Keys + joystick + buttons
│   │   └── collision.ts        # Tile sampling, special tiles
│   ├── world/
│   │   ├── maze.ts             # Procedural generation
│   │   ├── chunks.ts           # generateChunk, getTile, setTile
│   │   ├── rings.ts            # Spawn + collect
│   │   ├── enemies.ts          # Spawn + AI + stomp detection
│   │   ├── ebars.ts            # Electric bars
│   │   └── powerups.ts         # 4 power-ups + HUD
│   ├── render/
│   │   ├── ball.ts             # Drawing skin + glow
│   │   ├── tiles.ts            # Wall, floor, corridor variants
│   │   ├── particles.ts        # spawnParts + render
│   │   ├── floats.ts           # +10 floating numbers
│   │   └── water.ts            # Underwater overlay
│   ├── audio/
│   │   ├── sfx.ts              # 11 SFX
│   │   ├── music.ts            # 5 zone melodies
│   │   └── synth.ts            # note() helper
│   ├── ui/
│   │   ├── menus/
│   │   │   ├── Splash.ts
│   │   │   ├── Menu.ts
│   │   │   ├── Settings.ts
│   │   │   ├── Leaderboard.ts
│   │   │   ├── Skins.ts
│   │   │   └── ZoneAnn.ts
│   │   ├── hud.ts              # Pills + power-ups bar
│   │   └── controls.ts         # Buttons / joystick render
│   ├── data/
│   │   ├── themes.ts           # 5 zones config
│   │   ├── skins.ts            # 5 skins config
│   │   └── sequences.ts        # MSEQS music
│   └── i18n/
├── public/
│   ├── manifest.json
│   ├── sw.js
│   └── icons/
├── tests/
│   ├── physics.test.ts
│   ├── maze.test.ts
│   └── powerups.test.ts
├── vite.config.ts
└── package.json
```

---

## 5. Charte UI / design

### Tile rendering
- Wall : 3-tone shading (wallC body + wallH highlight + wallS shadow)
- Floor : couleur `floorC`
- Empty : transparent
- Spikes : pointes rouges
- Bumper : ressort visuel
- Gem : losange coloré (+ aimanté visuellement quand magnet actif)
- Power-up : capsule pulsante
- Fan : ailettes animées
- Checkpoint : drapeau

### Couleurs zone (5 palettes)
- Tokens par thème : `wallC` / `wallH` / `wallS` / `floorC` / `acc` / `emC` / `gemC` / `bgT` / `bgB` / `ringC` / `elecC`
- Background : gradient vertical `bgT → bgB`

### Typographie
- **Display** : Courier New monospace (rétro arcade)
- **Letter-spacing positif** sur titres (3–12px) pour effet "écran"

### Effets visuels
- Logo BOUNCE : gradient `#ff6030 → #ffaa00 → #ff2200` + drop-shadow
- Sub-logo : `#180c..` violet pâle letter-spacing 9px
- HUD pills : backdrop-blur(4px) + transparent dark
- Power-up icons : circulaires avec progress bar bottom (transition .15s)
- Particles : spawnParts(N=18–24 par event)
- Float numbers : +10 +50 +100 +500 textes flottants 33–70 frames

### Contrôles
- **Boutons mode** : 62×62 px, ronds, semi-transparent
- **Joystick** : track 155×60, knob 52×52, drag horizontal -1..+1
- **Jump button** : 86×86 px orange, label "SAUT" + "✕✕" (jumps remaining)
- Touch + mouse + keyboard (Arrow keys + Space)

### Mobile
- `touch-action: none` partout
- Safe area inset bottom respectée
- Tap highlight transparent

---

## 6. Roadmap

### v0.1 — Présent ✅ (actuelle = v4)
- Jeu jouable, 5 zones thématiques, 5 skins, 4 power-ups, leaderboard local, 11 SFX, 5 mélodies, double jump, double contrôle (boutons/joystick), checkpoints, persistance localStorage

### v0.2 — Refactoring
- [ ] **Décomposer `index.html` (1334 lignes)** en `src/` modulaire :
  - Extraire CSS dans `<style>` séparés (~90 lignes)
  - Extraire JS (61 fonctions) en modules ES6 (physics, world, render, audio, ui)
  - Garder `index.html` minimal (shell only)
- [ ] **Setup Vite + TypeScript strict**
- [ ] Types : `Ball`, `Chunk`, `Theme`, `Skin`, `Ring`, `Enemy`, `EBar`, `PowerUp`, `SaveData`

### v1.0 — Tests + persistance étendue
- [ ] **Tests Vitest** ≥ 70% sur logique pure :
  - `maze.ts` : génération chunks, corridors split/merge
  - `physics.ts` : collisions tiles, gravity inversion, double jump
  - `powerups.ts` : durations, shield absorption
- [ ] **Persistance étendue** :
  - Best score par zone (actuellement seulement scores globaux)
  - Death counter total
  - Distance lifetime
  - Nombre de runs par skin
- [ ] **Audit Lighthouse ≥ 90**

### v1.1 — Backend Firebase
- [ ] **Leaderboard Firebase Realtime DB** :
  - Submit best score + pseudo + zone max
  - Top 100 mondial
  - Top 10 weekly + monthly
  - Pseudo modération (filtre profanité)
- [ ] **Anti-cheat Cloud Function** : cohérence score/temps/distance
- [ ] **i18n** : minimum FR + EN + ES + DE + PT (currently FR-only)

### v1.2 — Gameplay extension
- [ ] **Daily challenge** : seed labyrinthe fixé pour la journée (fair compete)
- [ ] **3 nouvelles zones** : Espace, Forêt enchantée, Volcan actif
- [ ] **Boss** tous les 5 zones (rencontre fixe au lieu de procédural)
- [ ] **Achievements** : 50 gems / 100 rings / 1km parcouru / etc.
- [ ] **5 nouveaux skins** premium déblocables par achievements

### v2.0 — APK + stores
- [ ] **Build APK Capacitor** + soumission Play Store
- [ ] **iOS** via Capacitor + soumission App Store
- [ ] Achievements natifs Play Games / Game Center
- [ ] Cloud save (Firebase Auth)

### Nice to have
- Multijoueur asynchrone (course de 2 joueurs sur le même seed)
- Custom skins (personnalisation couleurs c0/c1/c2/glow)
- Editeur de niveau communautaire
- Replay system (vidéo top run partageable)
- Page promo dédiée dans clonex-studio (existe déjà : `bounce-v4.html`)

---

## 7. Risques & dépendances

### Risques techniques
- **Tout en `index.html` (1334 lignes)** : maintenance difficile, conflits merge → refactoring critique avant ajout de feature majeure
- **61 fonctions globales** dans le scope window → risque collisions
- **Pas de TypeScript** : risque régression silencieuse sur physique (canPlace, collisions, jumps)
- **Pas de tests** : impossible de garantir non-régression sur génération labyrinthe ou physique
- **AudioContext** parfois bloqué par autoplay policy navigateur → user gesture requis
- **Vibrate API** non supportée iOS Safari (silently fails)
- **Generation chunks** : max 10 en mémoire → si joueur recule = chunks regenerated différemment (pas reproducible)

### Risques performance
- Canvas 2D non hardware-accelerated → potentiellement lag sur entrée de gamme
- Particles spawned 18–24 par event sans pool reuse
- 5 mélodies tick toutes les 270ms via `setTimeout` (drift possible vs RAF)
- HUD updates DOM à chaque tick → hits reflows

### Risques UX
- **Mode portrait bloqué** : refus net si phone en portrait → friction onboarding (pas d'instruction de tourner avant le splash)
- **Joystick / boutons toggle** dans settings → user pourrait pas le trouver au premier run
- Pas de tutorial in-game (jump, double jump, power-ups)

### Risques business
- **Pas de monétisation** : pas de pubs, pas d'IAP, pas de cosmétiques payants
- **Leaderboard local seulement** : pas de viralité naturelle
- **Genre saturé** : différenciation par procédural infini + 5 zones thématiques + audio synthé

### Dépendances externes
- Aucune actuellement (entièrement offline)
- Hébergement GitHub Pages (CDN clonex-studio)

---

## 8. Conventions

### Naming
- **Functions** : camelCase (`generateChunk`, `collectGem`, `bReset`)
- **Constants** : UPPER (`ROWS`, `TILE`, `CHUNK_W`, `COR_H`, `INT_MIN`, `INT_MAX`)
- **Tile types** : `TW` (wall), `TE` (empty), `TS` (spike), `TB` (bumper), `TG` (gem), `TPU` (power-up), `TCP` (checkpoint), `TFN` (fan)
- **CSS classes** : kebab-case courts (`#mv`, `#gov`, `#setv`, `.dbtn`, `.pill`)
- **State globals** : courtes sans préfixe (`ball`, `score`, `gems`, `PU`)
- **Storage key** : `bounce_maze_v2`

### Couleurs convention
- Hex 6 chars partout
- Dans thème : `wallC` (color principale) → `wallH` (highlight) → `wallS` (shadow)
- Skins : `c0` (lightest) → `c1` (mid) → `c2` (darkest) + `glow` + `shine`

### Tests (à créer)
- Unit Vitest : `physics.test.ts`, `maze.test.ts`, `powerups.test.ts`, `chunks.test.ts`
- E2E Playwright : start game → bounce 50 frames → collect ring → die → respawn

### Versioning
- Pas de cache busting actuel (HTML statique)
- À ajouter avec migration Vite

### Save versioning
- Clé `bounce_maze_v2` (incrémenter sur breaking change schema)

---

## 9. Résumé exécutif

Bounce ∞ Labyrinth est un **runner / platformer arcade** infini procédural en mode paysage forcé. Tout le jeu tient dans un `index.html` monolithique (1334 lignes, 61 fonctions JS, 5 zones thématiques avec changement musique synthé Web Audio API, 4 power-ups, double jump, leaderboard local, 5 skins déblocables par gemmes 0–500). **Génération procédurale** par chunks de 22 tiles avec corridors split/merge dynamiques. **Persistance** localStorage `bounce_maze_v2`. **PWA installable** avec manifest + sw + 2 icons SVG. **Dette technique majeure** : monolithe à décomposer en `src/` Vite + TypeScript avant toute nouvelle feature. **Roadmap** : refactoring → tests + persistance étendue → leaderboard Firebase + i18n → nouvelles zones + boss + daily challenge → APK Play Store. **Différenciation** : runner infini procédural + 5 ambiances avec audio synthé + double mode contrôle (boutons/joystick) + zone underwater Abyssal.
