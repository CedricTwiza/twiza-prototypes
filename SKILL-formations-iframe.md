# SKILL — Catalogue de formations Twiza (iframe WordPress)

## Vue d'ensemble

Ce document décrit le flux complet pour maintenir et mettre à jour le catalogue de formations Twiza, hébergé sur GitHub Pages et intégré dans WordPress via une `<iframe>`.

### Workflow de mise à jour (session Claude Code)

1. Fournir le CSV (fichier ou copie-colle)
2. Claude lit `twiza-formations.html` sur GitHub pour partir de l'état exact en production
3. Claude génère le HTML mis à jour et affiche un **aperçu visuel interactif**
4. Modifications éventuelles à la demande
5. **Validation explicite** par Cédric
6. Claude pousse automatiquement sur GitHub via l'API REST → GitHub Pages se met à jour en quelques secondes → l'iframe WordPress affiche la nouvelle version sans aucune intervention côté WordPress

- **Fichier source** : `twiza-formations.html`
- **URL GitHub Pages** : `https://cedrictwiza.github.io/twiza-prototypes/twiza-formations.html`
- **Dépôt** : `https://github.com/cedrictwiza/twiza-prototypes`
- **Intégration WordPress** : via `<iframe>` pointant vers l'URL GitHub Pages ci-dessus

---

## Architecture du fichier `twiza-formations.html`

Le fichier est un **fragment HTML autonome** (pas de `<html>`/`<body>`) composé de trois parties :

### 1. CSS (balise `<style>`)
Toutes les classes sont préfixées `.twiza-` pour éviter les conflits avec le thème WordPress.

| Classe | Rôle |
|--------|------|
| `.twiza-catalogue` | Conteneur principal, max-width 860px, police Georgia |
| `.twiza-intro` | Bandeau d'introduction beige avec bordure gauche brun |
| `.twiza-quick-links` | Boutons de navigation rapide (flex wrap) |
| `.twiza-btn-primary` | Bouton brun plein (`#8b6a3e`) |
| `.twiza-btn-secondary` | Bouton outline brun |
| `.twiza-theme` | Carte accordéon par thématique |
| `.twiza-theme-header` | En-tête cliquable de l'accordéon |
| `.twiza-theme-body` | Corps masqué/affiché par JS |
| `.twiza-formation` | Ligne d'une formation (nom + dates) |
| `.twiza-bientot` | Badge "Prochainement" |

**Couleur principale** : `#8b6a3e` (brun chaud)
**Fond** : `#f7f3ee` (crème)
**Responsive** : breakpoint à 580px (boutons empilés, padding réduit)

### 2. HTML (div `.twiza-catalogue`)
Structure par thématiques, chacune dans un accordéon :

| # | Thème | Emoji | Nb formations (2026) |
|---|-------|-------|----------------------|
| 1 | Enduits & Finitions | 🪨 | 6 |
| 2 | Tadelakt | ✨ | 1 |
| 3 | Bâti Ancien & Maçonnerie | 🏚️ | 5 |
| 4 | Toiture | 🏠 | 1 |
| 5 | Terre & Construction Naturelle | 🌱 | 1 |
| 6 | Autonomie | ⚡ | 3 |
| 7 | Rénovation & Conception Énergétique | 🔧 | 2 |
| 8 | Formation Spécifique | 👩 | 1 |

Chaque formation contient :
- **Nom** avec lien href vers la page Twiza (ou Airtable)
- **Dates** : liste de `.twiza-date-line` (lieu + date)
- **Badge** `.twiza-bientot` si pas encore planifiée

### 3. JavaScript
```js
function twizaToggle(btn) {
  // Ferme tous les accordéons ouverts
  // Ouvre celui cliqué (ou le ferme s'il était déjà ouvert)
}
```
Comportement : **un seul accordéon ouvert à la fois**.

---

## Procédure de mise à jour à partir d'un CSV

### Format CSV attendu

```csv
thematique,emoji,formation,url,lieu,dates,prochainement
"Enduits & Finitions",🪨,"Enduits et Joints à la Chaux","https://fr.twiza.org/...",Saint-Bazile-de-la-Roche (19),"16-18 mars 2026 | 19-21 oct 2026",non
"Autonomie",⚡,"Autonomie en Eau : ...","https://fr.twiza.org/...",,, oui
```

**Colonnes :**
| Colonne | Description |
|---------|-------------|
| `thematique` | Nom de la catégorie accordéon |
| `emoji` | Emoji associé à la thématique |
| `formation` | Titre complet de la formation |
| `url` | URL de la page d'inscription |
| `lieu` | Ville et département (vide si distanciel) |
| `dates` | Dates séparées par `\|` (pipe) |
| `prochainement` | `oui` → affiche badge, pas de dates |

### Étapes de mise à jour

#### Étape 1 — Recevoir le CSV
L'utilisateur fournit un fichier `.csv` (ou copie-colle le contenu).

#### Étape 2 — Parser et structurer les données
Pour chaque ligne du CSV :
1. Regrouper les formations par `thematique`
2. Compter le nombre de formations par thématique (pour le badge `.twiza-count`)
3. Séparer les dates (split sur `|`)
4. Détecter si `prochainement = oui`

#### Étape 3 — Régénérer le bloc HTML des accordéons
Pour chaque thématique, générer :

```html
<!-- ======= THÈME N : NOM THÉMATIQUE ======= -->
<div class="twiza-theme">
  <button class="twiza-theme-header" onclick="twizaToggle(this)">
    <span><span class="twiza-emoji">EMOJI</span> NOM THÉMATIQUE</span>
    <span class="twiza-count">N formation(s)</span>
    <span class="twiza-arrow">▼</span>
  </button>
  <div class="twiza-theme-body">

    <!-- Pour chaque formation sans badge "prochainement" -->
    <div class="twiza-formation">
      <p class="twiza-formation-name"><a href="URL">NOM FORMATION</a></p>
      <div class="twiza-dates">
        <div class="twiza-date-line"><span class="dot">•</span><span>DATE 1 — LIEU</span></div>
        <div class="twiza-date-line"><span class="dot">•</span><span>DATE 2</span></div>
      </div>
    </div>

    <!-- Pour chaque formation "prochainement" -->
    <div class="twiza-formation">
      <p class="twiza-formation-name"><a href="URL">NOM FORMATION</a></p>
      <span class="twiza-bientot">⏳ Prochainement — LIEU ou distanciel</span>
    </div>

  </div>
</div>
```

#### Étape 4 — Remplacer le contenu dans `twiza-formations.html`
- Conserver le `<style>` et le `<script>` intacts
- Remplacer uniquement le contenu entre `<div class="twiza-catalogue">` et `</div>` final
- Mettre à jour les commentaires de thème si les thématiques changent

#### Étape 5 — Aperçu visuel et validation
Claude affiche le rendu HTML dans une fenêtre d'aperçu interactif avant tout envoi.
Checklist de validation :
- [ ] Chaque formation a un lien `href` valide (non vide)
- [ ] Les badges `.twiza-count` reflètent le bon nombre
- [ ] Les formations "Prochainement" n'ont pas de section `.twiza-dates`
- [ ] L'accordéon JS fonctionne (pas de syntaxe cassée)
- [ ] **Validation explicite de Cédric obtenue** avant de passer à l'étape suivante

#### Étape 6 — Publication automatique sur GitHub
Claude pousse le fichier directement via l'API REST GitHub (token `GITHUB_TOKEN` configuré dans `~/.claude/settings.json`). Aucune action manuelle requise.

GitHub Pages se met à jour automatiquement en quelques secondes.
L'iframe WordPress affiche la nouvelle version sans intervention côté WordPress.

---

## Intégration WordPress (rappel)

Dans la page ou le widget WordPress, le code d'intégration est :

```html
<iframe
  src="https://cedrictwiza.github.io/twiza-prototypes/twiza-formations.html"
  width="100%"
  height="1200"
  frameborder="0"
  scrolling="no"
  style="border:none; overflow:hidden;">
</iframe>
```

> **Note** : ajuster `height` si le contenu s'allonge après mise à jour.
> Pour un iframe auto-redimensionnable, une lib JS comme `iFrameResizer` peut être envisagée.

---

## Points d'attention

- **Conflits CSS WordPress** : toutes les classes sont préfixées `.twiza-` — ne pas modifier cette convention
- **Liens** : certaines formations pointent vers Airtable (`airtable.com/...`), d'autres vers `fr.twiza.org`
- **Thématiques stables** : l'ordre des thématiques est fixe ; si une nouvelle thématique est ajoutée, mettre à jour aussi les boutons de navigation rapide (`.twiza-quick-links`) si présents
- **Cache navigateur** : après un push, vider le cache ou ajouter `?v=YYYYMMDD` à l'URL de l'iframe pour forcer le rechargement

---

*Dernière mise à jour : mars 2026 — push automatique via API GitHub activé (GITHUB_TOKEN)*
