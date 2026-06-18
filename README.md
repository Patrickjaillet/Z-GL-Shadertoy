<p align="center">
  <img src="public/icon.svg" width="72" alt="Z-GL Shadertoy">
</p>

<h1 align="center">Z-GL Shadertoy</h1>

<p align="center">
  <strong>Éditeur de shaders Shadertoy temps réel — 100 % hors-ligne, zéro WASM, zéro compte, zéro cloud.</strong><br>
  Pour les artistes génératifs, les chercheurs en rendu et toute personne qui écrit des shaders au format Shadertoy.
</p>

<p align="center">
  <img alt="Plateforme" src="https://img.shields.io/badge/Windows-desktop-blue">
  <img alt="Offline" src="https://img.shields.io/badge/offline-first-green">
  <img alt="Licence" src="https://img.shields.io/badge/licence-propriétaire-red">
</p>

---

## Qu'est-ce que Z-GL Shadertoy ?

Z-GL Shadertoy est un éditeur de shaders GLSL desktop, **100 % dédié au format Shadertoy** : écrivez une fonction `mainImage`, utilisez les uniforms standards (`iTime`, `iResolution`, `iMouse`, `iChannel0-3`…), branchez des passes Buffer A-D / Cube / Sound, et importez/exportez directement depuis/vers shadertoy.com — le tout sans connexion internet et sans aucun module WebAssembly.

L'application n'embarque **aucun** moteur de rendu alternatif (pas de WebGPU, pas de WGSL), aucun éditeur de node-graph, aucun moteur de particules, aucun SDK de plugins/scripting, aucun protocole de contrôle matériel (MIDI/OSC/NDI/Spout/DMX), aucune édition collaborative, aucune télémétrie. C'est un outil simple et focalisé.

---

## Fonctionnalités

### Édition

- **Éditeur Monaco** — coloration syntaxique GLSL ES, autocomplétion, documentation au survol, marqueurs d'erreur en direct, minimap, valeurs d'uniforms inline
- **Sliders automatiques** — chaque `#define` ou `const float/int` de premier niveau devient un slider glissable, avec annulation/rétablissement (Ctrl+Z / Ctrl+Y)
- **Bibliothèque de snippets** — extraits GLSL prêts à insérer (bruit, SDF, post-effets…)
- **Documentation GLSL intégrée** — panneau de référence des fonctions et types GLSL ES
- **Historique de versions** — chaque modification significative est conservée, restaurable depuis le panneau Version History
- **Palette de commandes** — `Ctrl+,` ou le bouton dédié pour accéder à tous les réglages

### Rendu Shadertoy complet

- **Multipass** — passes Image, Buffer A–D (avec feedback inter-frame), Cube A–F, et Sound
- **Tous les uniforms standards** — `iResolution`, `iTime`, `iTimeDelta`, `iFrame`, `iMouse`, `iDate`, `iSampleRate`, `iChannel0-3`, `iChannelResolution`
- **Tous les types de canaux (iChannel)** — image statique, vidéo, webcam, micro/fichier audio (FFT), cubemap, frame précédente (feedback), texture clavier, bruit procédural

### Outils d'assistance à l'écriture de shaders

- **Ray March Assistant** — détecte les patterns SDF/raymarching et propose un panneau de réglage (`MAX_STEPS`, `MAX_DIST`, `SURF_DIST`…)
- **SDF Library + Visualizer + Composer** — bibliothèque de 35+ primitives SDF, 4 modes de visualisation (iso-distance, field, normals, step count), compositeur visuel de scène SDF
- **LUT grading** — bibliothèque de LUTs intégrées + éditeur de courbes 1D + import `.cube`
- **Panneau Modulation** — sources de modulation pour piloter dynamiquement vos paramètres
- **Mode Daltonisme** — simulation Protanopie / Deutéranopie / Tritanopie en post-pass (`Ctrl+Shift+B`)

### Import / Export Shadertoy

- **Import ZIP** — glissez-déposez un export `.zip` depuis shadertoy.com : les passes Image/Buffer A–D/Sound/Cube sont détectées et reconstituées automatiquement
- **Export format ShaderToy** — nettoie les déclarations d'uniforms/precision locales pour un collage direct dans l'éditeur shadertoy.com
- **Export complet** — capture PNG, vidéo MP4/WebM (via `MediaRecorder` natif du navigateur, sans WASM/ffmpeg), GLSL pur/minifié, snippet Three.js, page HTML autonome, projet ZIP `.zgl`, sketch p5.js, format GLSL Sandbox
- **Lien de partage** — sérialise le shader et les valeurs de sliders dans l'URL (hash compressé), y compris en version multi-passes (v2)

### Bureau natif (Windows, via Tauri)

- Fenêtre sans bordure avec barre de titre personnalisée, effet Mica/Acrylic
- Icône dans la zone de notification (tray), liste des fichiers récents (MRU)
- Boîtes de dialogue natives, glisser-déposer, association de fichiers `.glsl`/`.frag`
- Watch de fichier externe (éditez dans VS Code/Neovim/Zed, prévisualisez en direct)
- Sauvegarde automatique avec récupération après crash

### Accessibilité et personnalisation

- **Internationalisation** — 9 langues embarquées (EN, FR, DE, JA, ZH, PT-BR, ES, RU, KO), zéro appel réseau
- **Thèmes** — plusieurs thèmes intégrés, panneau Settings dédié
- **Layouts** — dispositions de panneaux sauvegardables

### Rendu headless (CLI)

```bash
# Rendu PNG d'un shader GLSL
z-gl --headless --shader plasma.frag --out plasma.png

# Voir toutes les options
z-gl --headless --help
```

---

## Pour qui ?

- Artistes génératifs qui écrivent ou collectionnent des shaders Shadertoy
- Développeurs qui veulent un éditeur Shadertoy local, rapide, sans navigateur
- Chercheurs en rendu qui veulent itérer sur du GLSL sans dépendance cloud

---

## Compatibilité

- **Plateforme** : Windows (app native via Tauri) ou navigateur (Chrome/Edge/Firefox récents)
- **Aucune dépendance réseau** requise pour l'usage courant — tout fonctionne hors-ligne
- **Aucun WASM** — tout le pipeline de rendu, d'export et d'analyse est en JavaScript/GLSL natif

---

## Philosophie

Z-GL Shadertoy fait un seul métier et le fait bien : être le meilleur éditeur Shadertoy local possible. Pas de fonctionnalités annexes qui alourdissent l'app ou complexifient la maintenance — si une fonctionnalité ne sert pas directement l'écriture, le test ou le partage de shaders au format Shadertoy, elle n'a pas sa place ici.

---

## ⚙️ Dépannage

### L'app de production s'ouvre mais ne s'initialise pas

**Cause :** les appels `import('@tauri-apps/...')` dynamiques ne sont pas regroupés par Vite en production.

**Solution :** tous les appels à l'API Tauri passent par `window.__TAURI__.*` directement, toujours disponible grâce à `withGlobalTauri: true` dans `tauri.conf.json`.

### Erreur `EvalError` au démarrage de la version de production

**Cause :** `@shaderfrog/glsl-parser` utilise `new Function()` en interne ; la CSP de Tauri doit donc inclure `'unsafe-eval'`.

**Solution :** déjà configuré dans `tauri.conf.json` (`script-src 'self' 'unsafe-eval' ...`).

### `tauri.conf.json` — `expected value at line 1 column 1`

**Cause :** PowerShell (`ConvertTo-Json` / `Set-Content`) écrit un BOM UTF-8 que le parseur JSON de Tauri rejette.

**Solution :** utilisez `[System.IO.File]::WriteAllText` avec un encodage sans BOM (le script `build.ps1` le fait automatiquement).

### `tauri dev` plante avec `EBUSY: resource busy or locked`

**Cause :** le watcher de fichiers de Vite tentait de surveiller `src-tauri/target/`, verrouillé par `cargo` pendant la compilation.

**Solution :** `vite.config.js` ignore désormais `**/src-tauri/**` dans `server.watch.ignored`.

---

Voir [CHANGELOG.md](./CHANGELOG.md) pour l'historique complet des versions.
