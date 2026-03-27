# Directives de conception des présentations MédECN

> Ce fichier décrit exhaustivement toutes les règles de conception, de contenu et de style
> à respecter pour chaque nouvelle présentation HTML du projet MédECN.

---

## 1. Architecture technique

### Fichier HTML unique
Chaque cours est un fichier HTML autonome nommé `presentation_[sujet].html`.
Il contient en inline : le CSS, les slides, le JS d'interaction — aucune dépendance externe.

### Approche de génération
- Écriture directe dans la conversation (partie par partie) via des scripts Python pickle-chain
- Un script par groupe de 2–3 slides + QCM → sauvegarde dans `/tmp/[cours]_pN.pkl`
- Un script assembleur final lit le dernier `.pkl` et génère le HTML complet
- Vérification systématique de l'équilibre `<div>` / `</div>` après assemblage

### Structure d'un slide content-slide
```
1. h2 — Titre de la diapo
2. h3 "📦 Prérequis" — 2 à 4 def-box
3. h3 "📖 Explication" — bloc explication exhaustive + SVG + suite
4. h3 "🃏 Flashcards" — exactement 3 flash-blocks
```

---

## 2. Calibration du nombre de slides

| Source | Ratio | Exemple |
|--------|-------|---------|
| Cours hémato | 1 page PDF = 1 slide | 14 pages → 14 slides |
| Cours néphro | 2 pages PDF = 1 slide | 26 pages → 13 slides contenu |
| Règle générale | pages × 3.6 = slides (référence) | 28 pages → ~101 slides |

### Structure standard d'une présentation (néphro / cours dense)
```
Slide 0    — Titre
Slides 1–3 — Chapitre 1 (2 pages/slide)
QCM 1      — 10 questions + 1 DP 5 étapes
Slides 4–6 — Chapitre 2
QCM 2      — 10 questions + 1 DP 5 étapes
...
Slide N    — Tableau bilan
QCM final  — 10 questions de récapitulatif + 1 DP final
Slide Fin  — Conclusion
```

---

## 3. Design visuel

### Thème
- Fond : `#0d0d1a` (noir bleuté)
- Fond secondaire : `#1a1a2e`
- Texte principal : `#e0e0f0`
- Texte secondaire : `#c0c0d8`
- Texte grisé : `#9090b0`

### Couleurs d'accentuation
| Couleur | Code | Usage |
|---------|------|-------|
| Orange | `#f39c12` | h2, titres principaux, def-box |
| Rouge | `#e74c3c` | h3, urgences, warning-box |
| Bleu | `#3498db` | h4, info-box, schémas secondaires |
| Vert | `#2ecc71` | highlight-box, résultats corrects, validation |
| Violet | `#9b59b6` | DP steps, cellules intercalaires, éléments tertiaires |

### Topbar fixe
```css
height: 50px; background: rgba(13,13,26,0.97);
```
Contenu : [← Menu] [titre centré] [N / Total] [◁ ▷]

### Barre de progression
```css
position: fixed; top: 50px; height: 3px;
background: linear-gradient(90deg, #e74c3c, #f39c12);
```

---

## 4. Boîtes de contenu obligatoires

### def-box (termes techniques)
```css
background: rgba(243,156,18,0.12); border-left: 4px solid #f39c12;
```
**Règle :** obligatoire pour TOUT terme technique à sa PREMIÈRE occurrence dans une slide.

### highlight-box (concept clé)
```css
background: rgba(46,204,113,0.1); border-left: 4px solid #2ecc71;
```
**Règle :** 1 par slide contenu, pour le concept le plus important à retenir.

### warning-box (piège / erreur fréquente)
```css
background: rgba(231,76,60,0.1); border: 2px solid #e74c3c;
```
**Règle :** pour les pièges pédagogiques classiques.

### urgence-box (urgence clinique)
```css
background: rgba(231,76,60,0.2); border: 2px solid #e74c3c;
```
**Règle :** pour les urgences vitales et les erreurs graves à ne jamais faire.

---

## 5. Schémas SVG (OBLIGATOIRES)

### Règles absolues
- **Format SVG inline uniquement** — pas d'ASCII, pas de HTML+CSS schema-flow
- Un schéma SVG par slide contenu minimum
- Placé **APRÈS** l'explication exhaustive (voir §6)
- Le schéma résume visuellement le mécanisme expliqué par le texte

### Template SVG standard
```html
<svg viewBox="0 0 700 280" xmlns="http://www.w3.org/2000/svg"
     style="width:100%;max-height:280px;margin:1rem 0;display:block">
  <defs>
    <marker id="aN" markerWidth="8" markerHeight="6" refX="8" refY="3" orient="auto">
      <polygon points="0 0,8 3,0 6" fill="#f39c12"/>
    </marker>
  </defs>
  <!-- Éléments : rect + line + text + polygon -->
</svg>
```

### Conventions SVG
- Rectangles avec `fill="rgba(R,G,B,0.15..0.25)"` et `stroke` coloré
- Flèches via `<line marker-end="url(#aN)">` avec polygone dans `<defs>`
- Texte : `font-family="Segoe UI,sans-serif"`, labels blancs (`fill="white"`),
  sous-labels en `fill="#b0b0c0"`
- Un marker différent par couleur (éviter la collision d'IDs entre slides)
- `viewBox` : largeur 700, hauteur adaptée (240–380 selon la complexité)

---

## 6. Structure de l'explication exhaustive (OBLIGATOIRE)

### Règle fondamentale
Chaque slide contenu doit avoir **les deux** :
1. Une explication exhaustive structurée (AVANT le SVG)
2. Le schéma SVG (APRÈS l'explication)

### Format de l'explication
```html
<div class="expl-wrap">

<h4>1. 🦠 Titre de la section</h4>
<p>Explication en langage simple et accessible.
   Tous les termes techniques sont expliqués avec des mots du quotidien.
   Les mécanismes sont décomposés étape par étape.</p>
<div class="expl-bq">Ex : exemple concret tiré de la pratique clinique ou du quotidien.</div>

<hr class="expl-sep">

<h4>2. 🩸 Deuxième section</h4>
<p>...</p>
<div class="expl-bq">Ex : ...</div>

<hr class="expl-sep">

<!-- Répéter pour chaque mécanisme, 3 à 6 sections par slide -->

<div class="expl-warn"><!-- ou expl-info --><strong>⚠️ Point important / piège</strong><br>...</div>

</div>
```

### CSS associé (à inclure dans chaque présentation)
```css
.expl-wrap { margin: 0 0 1.5rem 0; }
.expl-wrap h4 { color: #e0e0f0; font-size: 0.97rem; margin: 1rem 0 0.4rem; font-weight: 700; }
.expl-wrap p { color: #c0c0d8; font-size: 0.9rem; line-height: 1.75; margin-bottom: 0.4rem; }
.expl-wrap strong { color: #f0f0ff; }
.expl-bq {
  border-left: 3px solid rgba(243,156,18,0.5); padding: 0.4rem 1rem;
  color: #9898b8; font-style: italic; font-size: 0.88rem; margin: 0.3rem 0 0.9rem 0;
  background: rgba(243,156,18,0.04); border-radius: 0 6px 6px 0;
}
.expl-sep { border: none; border-top: 1px solid rgba(255,255,255,0.07); margin: 1.1rem 0; }
.expl-warn {
  background: rgba(231,76,60,0.08); border-left: 4px solid #e74c3c;
  border-radius: 0 8px 8px 0; padding: 0.6rem 1rem;
  margin: 0.75rem 0; font-size: 0.88rem; color: #c8c8d8; line-height: 1.6;
}
.expl-info {
  background: rgba(52,152,219,0.08); border-left: 4px solid #3498db;
  border-radius: 0 8px 8px 0; padding: 0.6rem 1rem;
  margin: 0.75rem 0; font-size: 0.88rem; color: #c8c8d8; line-height: 1.6;
}
```

### Règles de rédaction de l'explication
- **Langage simple** : expliquer comme à un étudiant de 2e année, sans présupposer de connaissances
- **Exhaustivité** : couvrir TOUT ce qui est dans les pages sources du cours
- **Exemples concrets** : au moins 1 exemple clinique ou du quotidien par section
- **Séparateurs** : `<hr class="expl-sep">` entre chaque section numérotée
- **Numérotation** : sections numérotées `1.`, `2.`, `3.`… avec emoji thématique
- **Longueur** : 3 à 6 sections, chaque section = 3 à 8 lignes de texte

---

## 7. Flashcards (3 par slide)

```html
<div class="flash-block" data-answer="Réponse complète et précise">
  <div class="q-text">❓ Question ? (niveau N)</div>
  <div class="flash-input-row">
    <input class="flash-input" type="text" placeholder="Votre réponse…"
           onkeydown="if(event.key==='Enter')validateFlash(this)">
    <button class="qcm-validate" onclick="validateFlash(this)">Vérifier</button>
  </div>
  <div class="qcm-explanation"></div>
</div>
```

### Règles
- **Exactement 3 flashcards** par slide contenu
- Niveaux progressifs : niveau 1 (définition/factuel), niveau 2 (mécanisme), niveau 3 (application clinique)
- La réponse dans `data-answer` doit être complète et auto-suffisante

---

## 8. QCM (après chaque chapitre)

### Structure standard
```html
<div class="slide content-slide" id="slide-qcmN">
<h2>🎯 QCM — Chapitre N : [Titre]</h2>
<!-- 10 qo() blocks -->
<!-- 1 dp-block avec 5 dp() steps -->
</div>
```

### Helpers Python
```python
def qo(num, question, options, correct_letter, explication):
    # Génère un bloc QCM à 4 choix (A/B/C/D)
    # correct_letter : 'A', 'B', 'C' ou 'D'

def dp(step_num, titre_etape, question, options, correct_letter, explication):
    # Génère une étape de dossier progressif

def fl(question, reponse):
    # Génère une flashcard texte libre
```

### Règles QCM
- **10 questions** par QCM (couvrant les notions des slides précédentes)
- **1 dossier progressif** (DP) de **5 étapes** par QCM
- Distracteurs plausibles (pas de "réponse évidente")
- Explications obligatoires : claires, pédagogiques, incluant pourquoi les mauvaises réponses sont fausses
- Le DP doit suivre un cas clinique cohérent de bout en bout (un seul patient)

---

## 9. Tableau Bilan (obligatoire en fin de présentation)

Une slide `id="slide-tableau"` doit être insérée avant `slide-fin`. Elle contient :
- Un tableau des valeurs normales à retenir
- Un tableau récapitulatif des transporteurs/mécanismes (si applicable)
- Un tableau de diagnostic différentiel (si applicable)
- Un tableau UNa/FENa d'interprétation (si néphro)
- Une boîte urgence-box avec les 5 "points rouges" à l'examen

---

## 10. JavaScript obligatoire

```javascript
let current = 0;
const total = N;  // nombre de slides
const slides = document.querySelectorAll('.slide');

function showSlide(n) { /* active/désactive + scroll top + counter + progress */ }
function changeSlide(d) { showSlide(current + d); }
function selectOption(el) { /* sélectionne une option QCM */ }
function validateQCM(btn) {
  // Détecte format lettre (A/B/C/D) ou index (0/1/2/3)
  const correctIdx = /^[0-9]+$/.test(answer)
    ? parseInt(answer)
    : letters.indexOf(answer.toUpperCase());
}
function validateFlash(el) { /* révèle la réponse + désactive l'input */ }
// Clavier : ArrowRight/Left = slide suivante/précédente
// Touch : swipe left/right
```

---

## 11. CSS critique — display des slides

```css
/* Règle impérative pour éviter le bug d'affichage simultané */
.slide { display: none !important; }
.slide.active { display: block !important; }
.slide.active.title-slide { display: flex !important; }
/* Ne JAMAIS mettre display:flex sur .title-slide sans .active */
```

---

## 12. Slide titre (slide-0)

```html
<div class="slide title-slide active" id="slide-0">
  <a class="menu-link" href="index.html">← Menu</a>
  <div>
    <div class="title-tag">UE[N] · [Matière] · [Professeur]</div>
    <h1>[Titre du cours]</h1>
    <div class="title-subtitle">[Sous-titre : thèmes couverts]</div>
    <div class="title-meta">
      <span>🧪 [Matière]</span>
      <span>📚 [N] diapos</span>
      <span>⚡ Quiz intégrés</span>
    </div>
  </div>
</div>
```

---

## 13. Slide fin (slide-fin)

```html
<div class="slide content-slide" id="slide-fin">
  <div style="display:flex;flex-direction:column;align-items:center;
              justify-content:center;min-height:60vh;text-align:center;padding:2rem">
    <div style="font-size:3rem;margin-bottom:1rem">🎉</div>
    <h2 style="color:#f39c12;margin-bottom:0.5rem">Cours terminé !</h2>
    <p style="color:#b0b0c0;font-size:1.1rem;max-width:500px">
      [Résumé : N diapos · M QCM · P DP · Q flashcards]
    </p>
    <a href="index.html" style="/* bouton retour menu */">← Retour au menu</a>
  </div>
</div>
```

---

## 14. Mise en ligne (GitHub Pages)

```bash
git add presentation_[sujet].html index.html
git commit -m "Add [Titre] — [N slides, M QCM, P DP]"
git push
```

Après chaque nouvelle présentation, mettre à jour `index.html` :
- Ajouter la carte dans la section matière correspondante
- Incrémenter le numéro de cours
- Mettre à jour le compteur "N cours disponibles" dans la matière-card

---

## 15. Checklist avant publication

- [ ] `<div>` et `</div>` comptés et équilibrés (diff = 0)
- [ ] Slide-0 a la classe `active`, les autres non
- [ ] Chaque slide contenu a : def-box + expl-wrap + SVG + highlight-box + 3 flashcards
- [ ] Chaque QCM a : 10 questions + 1 DP 5 étapes
- [ ] Slide tableau-bilan présente
- [ ] `const total = N` dans le JS correspond au nombre réel de slides
- [ ] Topbar counter affiche `1 / N` correctement
- [ ] index.html mis à jour avec la nouvelle carte
- [ ] Git commit + push effectués
