# Changelog

All notable changes to Z-GL Shadertoy are documented here.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).


## [2.1.2] — 20 juin 2026

### Fixed — menu Tools

- **Settings ne s'affichait pas** — le toggle JS (`toggleSettingsPanel`) fonctionnait correctement, mais `#settings-panel`/`.floating-panel` n'avait **aucune règle CSS nulle part dans le projet** : `display:block` sur une `<div>` non positionnée, sans `z-index`, sans fond, rendait le panneau invisible/inutilisable avant tout drag manuel. CSS complet injecté (même pattern que `modulation-panel.js`) : positionnement fixe, fond, bordure, z-index, et style des sections (langue/thème/accessibilité). Vérifié en live (capture + `getComputedStyle`).
- **Bouton HDR mort retiré** — contrairement à Settings, aucune implémentation n'a jamais existé (pas de fichier JS, pas de CSS, aucun listener) ; seul le bouton + l'entrée `panel-dock.js` (`root: '#hdr-panel'`) avaient été scaffoldés. Supprimé (markup + entrée du dock), suivant le même traitement que le bouton ISF en 2.1.0. Mentions obsolètes nettoyées au passage dans le tooltip d'onboarding et un commentaire de `panel-dock.js` (citaient aussi « node graph » et « volume rendering », déjà retirés depuis longtemps).

## [2.1.1] — 20 juin 2026

### Fixed — Pin slider

- **Le pin ne survivait pas à un reparse qui décalait la numérotation** — `state.pinnedIds` était indexé par l'`id` runtime volatil (`v0`, `v1`…), réassigné depuis zéro à *chaque* parse. Ajouter/retirer un `#define` ou un littéral *avant* un slider épinglé décalait sa numérotation, et le pin se retrouvait silencieusement attaché à un autre slider (ou à rien). Migré vers le `stableKey` déterministe (`def:NAME`/`const:NAME`/`lit:N`) introduit en Phase C — exactement le problème que cette infrastructure était censée résoudre, mais le pin n'avait jamais été migré. Tous les points de lecture/écriture corrigés : `parser.js` (calcul du flag `pinned` à l'analyse), `slider.js` (`togglePin`, les 3 rendus de ligne, randomize, reset au double-clic, fusion `syncSlidersFromCode`), `io/actions.js` (`resetSliders`), `context-menu.js` (libellé Pin/Unpin).
- **Icône du bouton pin corrompue après un clic** — `togglePin` faisait `btn.textContent = '&#128204;'` : `textContent` ne décode pas les entités HTML (contrairement au `innerHTML` du rendu initial), donc après le tout premier clic l'icône affichait le texte littéral `&#128204;` au lieu de 📌. Remplacé par les caractères Unicode directs (📌 / ⊙).
- Vérifié en live : un slider épinglé reste protégé même quand son `id` runtime change suite à l'ajout d'un `#define` avant lui dans le code ; l'icône reste propre après plusieurs bascules pin/unpin.

**Note pour les presets existants** : un preset sauvegardé *avant* ce correctif et contenant des `pinnedIds` au format `["v0", "v2"]` ne re-épinglera rien à son chargement (les `stableKey` actuels ne ressemblent jamais à `v0`) — dégradation silencieuse vers « rien d'épinglé » plutôt qu'un mauvais pin, pas de migration automatique écrite (cas marginal, faible valeur face à la complexité).

## [2.1.0] — 20 juin 2026

### Added — Système de sliders (roadmap complète, voir `ROADMAP-SLIDERS.md`)

- **Annotations GLSL** (`@range/@step/@group/@label/@color/@xy`) — overrides opt-in de l'inférence heuristique via commentaire `//`, 100% compatible shadertoy.com.
- **Persistance des personnalisations** (renommage, ordre, groupe, range manuel) — survit au reparse et aux presets/`.zgl`, via un `stableKey` déterministe (`def:NAME`/`const:NAME`/`lit:N`) indépendant de l'`id` runtime volatil. Bouton « ⎌ custom » pour réinitialiser.
- **Widgets spécialisés** — color swatch + picker, pad XY 2D, stepper entier, cadran d'angle — auto-détectés (annotation ou heuristique), tous branchés sur le même `onValChange`/`patchLine`/undo que les sliders classiques.
- **Ergonomie** — barre de recherche/filtre, bascule « modifié seulement », « Set range… » au menu contextuel, scrub clavier (`Alt+↑/↓` sur un nombre dans Monaco), icônes de catégorie.
- **Modulation logicielle** (`src/ui/slider-modulation.js` + popover par-slider + panneau `∿ Modulators`) — LFO/bruit/audio-réactif/random lissé/enveloppe, bouton global « 🎲 random ». Écriture throttlée (80 ms) pour ne pas re-compiler le shader à 60 fps.

### Removed — nettoyage de fonctionnalités mortes

- **Bouton et panneau ISF** — `toggleISFPreview` n'avait aucune implémentation JS depuis le début ; supprimés (markup, CSS, fichier `.html` orphelin, état `isfPreviewOpen`/`state.spout`/`state.ndi`).
- **Chapitres Help Center obsolètes** — Guide Timeline et Guide MIDI/OSC (fonctionnalités retirées depuis plusieurs versions) remplacés par un nouveau chapitre Modulation ; tables de canaux/uniformes/raccourcis et chapitre Export Guide réécrits pour matcher l'app réelle (suppression des mentions WebGPU/WGSL/HLSL/gamepad/mesh/face-tracking/GIF/tiled-render/batch-export).
- **Système de détection de conflits de raccourcis mort** (`shortcuts-manager.js` + `shortcut-conflicts.js`) — vérifiait une table de configuration jamais branchée à la vraie gestion clavier (son seul effet visible : un toast « ⚠ N conflits » à *chaque* démarrage). Supprimé.
- `src/style/modulators.css` (scaffold "Phase 16.3" jamais importé, remplacé par le panneau de modulation actuel).

### Fixed

- **`TAA: Auto` bouton d'en-tête** — `toggleTAAVelSource` n'avait aucune implémentation ; cliquer ne faisait rien. Implémenté pour cycler les options non désactivées du sélecteur `#taaVelSelect`.
- **Incohérence Image-pass dans le sélecteur TAA** — l'option « Image » apparaissait activée dans la liste mais `mpSetTAAVelocity` exigeait une condition différente (`p.rt`) que celle utilisée pour l'activer/désactiver (`p.rt || p.code`), donc la sélectionner échouait silencieusement (toast seul, UI non synchronisée). Conditions alignées.
- **Collision réelle de raccourci `Ctrl+Shift+L`** — bindé à la fois à « Select All Occurrences » (Monaco) et « Toggle LUT Library » via deux `state.editor.addAction()` séparés ; le second ne se déclenchait jamais. LUT Library déplacé sur `Ctrl+Shift+U` (libéré par le retrait du rendu volumétrique).
- **Icônes PWA/Open-Graph manquantes** — `index.html` référençait `/icons/icon-192.png` et `/icons/icon-512.png`, absents de `public/` (404 silencieux, image Open Graph cassée au partage de lien). Régénérés depuis les sources Tauri existantes (`src-tauri/icons/`).
- Lien `<link>` mort vers `phase11.css` (déjà supprimé) restant dans `src/ui.html`.
- `/public/icon-light-32x32.png` → `/icon-light-32x32.png` dans `titlebar.js` (les fichiers de `public/` sont servis à la racine par Vite).
- Liste de raccourcis du Help Center / écran d'accueil / overlay « which-key » — entrées obsolètes (`Ctrl+Shift+W` WGSL preview, `Ctrl+Shift+N` node graph, `Ctrl+Shift+D` GLSL debugger, `Ctrl+Shift+U` volume renderer) supprimées ; `F1` corrigé (ouvre Shader Docs, pas le Help Center).
- **Collision réelle des raccourcis fichier-projet (`Ctrl+O`, `Ctrl+Shift+S`, `Ctrl+N`)** — chacun était bindé par *trois* mécanismes indépendants (`viewport.js`, `io/project-ui.js`, `ui/menu-manager.js`) via des `document.addEventListener('keydown', …)` séparés, sans coordination. Tranché :
  - `ui/menu-manager.js` — le gestionnaire clavier Ctrl+N/Ctrl+S retiré : il dupliquait `io/project-ui.js` (la vraie implémentation, MRU/`.zgl`/Tauri) et `MENU_ACTIONS.newProject` était lui-même **cassé** (écrivait dans `state.code`, un champ qui n'existe nulle part dans `core/state.js` — le shader réel vit dans `state.editor` — donc l'éditeur n'était jamais vidé malgré le toast « New project created »). Rien d'autre ne déclenche jamais l'event `'menu-action'` que ce module écoute par ailleurs : sous-système mort dans son ensemble.
  - `ui/viewport.js` — `Ctrl+O`/`Ctrl+Shift+S` (bibliothèque / Save As modal) cèdent la priorité au flux projet natif `io/project-ui.js` quand `isTauri()` ; inchangés en navigateur/dev (où le flux projet ne ferait qu'afficher un toast « requires desktop app »).
  - **Bug latent démasqué par cette correction** : une fois `Ctrl+N` correctement acheminé vers le vrai `io/project.js`, son `_cb.setChannels({})` faisait planter le rendu (`state.channels.some is not a function`) — le callback `setChannels` dans `app/init.js` remplaçait carrément la référence du tableau `state.channels` (4 éléments, lu via `.some()`/`.forEach()` dans tout le reste de l'app) par l'objet brut reçu. Corrigé en fusionnant par index sur place (`chClear()` + `Object.assign`) au lieu de réassigner la référence — corrige aussi, au passage, le chargement de canaux depuis un `.zgl` (même callback, même bug).

## [2.0.0] — 18 juin 2026

### Renamed — Z-GL → **Z-GL Shadertoy**

L'application est rebaptisée **Z-GL Shadertoy** pour refléter son périmètre désormais 100 % dédié au format Shadertoy : `productName`, titre de fenêtre, titlebar, `<title>`/`og:title`, `startMenuFolder`, copyright et descriptions de bundle mis à jour dans `tauri.conf.json`, `Cargo.toml`, `index.html`, `src/ui.html` et `src/native/titlebar.js`.

### Changed

- Version synchronisée à **2.0.0** dans `package.json`, `src-tauri/tauri.conf.json` et `src-tauri/Cargo.toml` (la commande Tauri `app_version()` lit `Cargo.toml`, qui était resté bloqué sur `1.0.0`).
- `README.md` et `README_USER.md` entièrement réécrits : suppression de toute la documentation obsolète héritée des phases retirées (WebGPU, rendu différé G-Buffer/SSR/SSAO/GI/OIT, moteur de particules GPU, Scene Graph procédural, MIDI/Spout/DMX/capteurs, copilote IA local, path tracer…). Les deux fichiers décrivent maintenant fidèlement l'application telle qu'elle existe : éditeur Shadertoy multipass, assistants SDF/LUT/Ray March, export sans WASM, app Tauri.
- `vite.config.js` — `server.watch.ignored: ['**/src-tauri/**']` pour éviter le crash `EBUSY` du watcher Vite sur les binaires verrouillés par `cargo` pendant la compilation.
- `src/native/titlebar.js` — `/public/icon-light-32x32.png` → `/icon-light-32x32.png` (les fichiers de `public/` sont servis à la racine par Vite, pas sous `/public/`).

### Removed — nettoyage du code mort

Purge de **27 modules JS orphelins** (aucun import restant nulle part dans le code), reliquats des systèmes déjà retirés lors du passage à un Z-GL 100 % Shadertoy : outils de debug GPU avancés (`phase10.js`, `gpu-profiler.js`, `pixel-debugger.js`, `glsl-compile-manager.js`, `glsl-compile-worker.js`), pipeline post-process différé orphelin (`post-passes.js`, `post-passes-test.js`, `post-passes-examples.js`, `post-process-config.js`, `post-process-pipeline.js`, `perf-panel-ui.js`), heatmap GLSL (`heatmap.js`, `glsl-heatmap-analysis.js` + l'entrée de menu morte associée), TAA/multi-viewport/kiosk/feedback-latency, ainsi que des doublons non utilisés (`toast.js`, `layout-manager.js`, `window-manager.js`, `shader-diff.js`, `panel-tabs.js`, `shortcuts-panel.js`, `uniforms-panel.js`, `usage-profiles.js`).
- Test e2e `midi-learn.spec.js` (testait le panneau MIDI, déjà retiré).
- Assets `public/` inutilisés (icônes de gabarit, placeholders boilerplate, dossier `fonts/` vide).
- Config ESLint legacy `.eslintrc.cjs` (l'app utilise le flat config `eslint.config.js`).
- Lien `<link>` mort vers `phase11.css` dans `src/ui.html` (le fichier avait été supprimé sans que la référence soit nettoyée).

---

## [1.0.0] — initial release

- Real-time GLSL shader editor in the browser.
- ShaderToy-compatible `mainImage` convention.
- Standard uniforms: `iResolution`, `iTime`, `iTimeDelta`, `iFrame`, `iMouse`, `iChannel0–3`.
- Auto-extracted `#define` / `const float` sliders with undo/redo.
- 4 channels: noise, image, video, webcam, mic, audio FFT.
- Preset library with JSON import/export.
- Share URL (compressed base64 hash).
- Embed mode (`postMessage` API + `window['Z-GL']`).
