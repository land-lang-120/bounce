# 📊 bounce — Suivi

> Voir aussi : [CAHIER-CHARGES.md](CAHIER-CHARGES.md) (vision/spec complète)
> Mis à jour : **2026-04-23**

| | |
|---|---|
| **Stack** | Vanilla JS + Canvas 2D + Web Audio API monolithique (1334 lignes dans `index.html`, 61 fonctions JS) |
| **Statut** | 🔴 Prototype monolithique (jouable, polish abouti, mais 0 modularité, 0 tests) |
| **Type** | Runner / platformer arcade — labyrinthe infini procédural en mode paysage |
| **Tagline** | Rebondis. Survis. Repousse les limites. |
| **URL** | (déployable via `clonex-studio/bounce-v4.html`) |

---

## ✅ Fait

### Gameplay core
- Boule rebondissante avec gravité directionnelle (`gravDir = ±1`)
- **Double saut** (2 jumps avant grounded reset)
- Squash & stretch sur impacts
- 2 modes contrôle (boutons `◀ ▶ SAUT` ou joystick analogique horizontal)
- Clavier (Arrow keys + Space) en plus du tactile/souris
- Mort instantanée sur spike, ennemi, électricité (sans bouclier)
- Respawn checkpoint (ou point de départ)
- **Mode paysage forcé** (overlay rotation si portrait)

### Génération procédurale
- **Chunks 22 tiles wide × 12 lignes** avec corridors dynamiques (split/merge)
- Décalages stochastiques 3–7 colonnes
- 6 types de tiles : Wall, Empty, Spike, Bumper, Gem, Power-Up, Checkpoint, Fan
- Items semés probabiliste (gem 2.8%, spike 0.6%, bumper 0.4%, PU 0.3%, fan 0.3%)
- Checkpoints tous les 8 chunks
- Garbage collect chunks (max 10 en mémoire)

### 5 zones thématiques
| # | Nom | Underwater | Mélodie |
|---|---|---|---|
| 0 | Donjon (violet) | non | Mineur médiéval |
| 1 | Lave (orange) | non | Tribal |
| 2 | Cyber (vert néon) | non | Synthwave |
| 3 | Abyssal (bleu profond) | **oui** | Ambient + overlay water |
| 4 | Cristal (rose/violet) | non | Cristallin aigu |

- Changement automatique tous les 200 tiles
- Annonce zone overlay 3s
- Mélodie change automatiquement
- Overlay water + son water sur Abyssal

### Collectibles
- **Rings** (3–5/chunk) : +100 pts + particules + son séquence 4 notes ; bonus +500 si zone complète
- **Gems** : +10 pts + +1 totalGems lifetime + skin unlock progress
- **Magnet** aimante gemmes dans 4 tiles autour

### Ennemis & pièges
- Enemies : tués par stomp dorsal (+50 pts), sinon kill ball (sauf bouclier)
- Electric bars : pulse activé/inactif via period + phase
- Spikes : kill instantané (pas absorbé par bouclier)
- Bumpers : rebond × 1.4 vitesse jump

### Power-ups (4)
- 🛡️ **Shield** (360 frames) : absorbe 1 hit
- ⬆️ **Super Jump** (240 frames + 3 charges) : saut 1.4× hauteur
- 🧲 **Magnet** (360 frames) : aimante gemmes
- ⏱️ **Slowmo** (380 frames) : ralenti

### Audio (Web Audio API)
- AudioContext synthé : oscillator + gain
- 11 SFX : jump / jump2 / gem / ring / death / ckpt / pu / zone / enemy / fan / elec / water
- 5 mélodies procédurales (1 par zone, sequences MSEQS, tick 270ms)
- Toggle ON/OFF dans paramètres

### Vibrations haptiques
- `navigator.vibrate()` sur jump (28ms), gem (16), ring (séquence), death (80/30/80), checkpoint, power-up

### Skins (5 déblocables par gemmes)
- 🔥 **Fire** (free, par défaut)
- ❄️ **Ice** (50 gems)
- 🥇 **Gold** (150 gems)
- 💚 **Neon** (300 gems)
- 🌌 **Void** (500 gems)
- Auto-unlock quand totalGems atteint le seuil

### UI / écrans
- Splash avec logo gradient + load bar
- Menu : 4 boutons (Jouer / Skins / Classement / Paramètres)
- HUD pills : zone / rings / gems / score + under-pill (sous-marin)
- HUD power-ups secondaire avec progress bars
- Game Over avec info distance + tap to replay
- Settings : toggle Boutons/Joystick + Sons ON/OFF
- Leaderboard local (top scores triés)
- Skins screen : grid 3 cols + total gems
- Zone announcement overlay

### Persistance
- `localStorage.bounce_maze_v2` : `{ totalGems, scores, unlockedSkins, selectedSkin, settings }`

### PWA
- `manifest.json` standalone
- `sw.js` service worker enregistré
- 2 icons SVG (192/512), theme color `#000000`

### Documentation
- CAHIER-CHARGES.md rétrospectif (vision, personas, périmètre, architecture, charte UI, roadmap, risques)

## 🔄 En cours

- Aucun (état figé en prototype monolithique)

## 📋 À faire (priorité décroissante)

### Critique
1. ~~Créer `CAHIER-CHARGES.md`~~ ← fait dans cette session
2. **Décomposer `index.html` (1334 lignes)** en `src/` modulaire :
   - Extraire CSS dans `<style>` séparés (~90 lignes)
   - Extraire JS (61 fonctions) en modules ES6 thématiques (`physics/`, `world/`, `render/`, `audio/`, `ui/`, `data/`)
   - Garder `index.html` minimal (shell only)
3. **Setup Vite + TypeScript strict** + types (`Ball`, `Chunk`, `Theme`, `Skin`, `Ring`, `Enemy`, `EBar`, `PowerUp`, `SaveData`)

### Important
4. **Tests Vitest** ≥ 70% sur logique pure :
   - `maze.ts` : génération chunks, corridors split/merge
   - `physics.ts` : collisions tiles, gravity inversion, double jump
   - `powerups.ts` : durations, shield absorption
5. **Persistance étendue** :
   - Best score par zone (actuellement seulement scores globaux)
   - Death counter total
   - Distance lifetime
   - Nombre de runs par skin
6. **Audit Lighthouse ≥ 90** + accessibilité WCAG AA
7. **Leaderboard Firebase Realtime DB** :
   - Submit best score + pseudo + zone max
   - Top 100 mondial + Top 10 weekly + monthly
   - Pseudo modération (filtre profanité)
   - **Anti-cheat Cloud Function** (cohérence score/temps/distance)
8. **i18n** : minimum FR + EN + ES + DE + PT (currently FR-only)

### Nice to have
9. **Daily challenge** : seed labyrinthe fixé pour la journée (fair compete entre joueurs)
10. **3 nouvelles zones** : Espace, Forêt enchantée, Volcan actif
11. **Boss** tous les 5 zones (rencontre fixe au lieu de procédural)
12. **Achievements** : 50 gems / 100 rings / 1km parcouru / etc.
13. **5 nouveaux skins premium** déblocables par achievements (pas par gemmes)
14. **Build APK Capacitor** + soumission Play Store
15. **iOS** via Capacitor + soumission App Store
16. Achievements natifs Play Games / Game Center
17. Cloud save (Firebase Auth)
18. Multijoueur asynchrone (course de 2 joueurs sur le même seed)
19. Custom skins (personnalisation couleurs)
20. Editeur de niveau communautaire
21. Replay system (vidéo top run partageable)
22. Page promo dédiée dans clonex-studio (existe : `bounce-v4.html`, à enrichir)
