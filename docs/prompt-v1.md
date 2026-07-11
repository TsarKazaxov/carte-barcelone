# Prompt détaillé — MVP « Carte de vœux animée · Barcelone »

> Prompt prêt à copier-coller dans Lovable, v0, Claude, Figma Make, etc.

---

## Rôle

Tu es Product Designer senior et développeur front-end expert. Construis un MVP **fonctionnel, sans bug, sans backend**, sous forme d'un **fichier HTML unique** (HTML + CSS + JS vanilla, aucune librairie externe, aucun localStorage), utilisable hors ligne dans Chrome/Safari/Firefox desktop et mobile.

## Produit

Un mini-site pour créer une **carte de vœux animée, fun et colorée au thème de Barcelone** : l'utilisateur ajoute des photos et/ou des vidéos courtes (traitées comme des « mèmes » de 5 s max), écrit un message, puis lance une **vue plein écran** où les médias défilent en **carrousel coverflow** avec le message animé par-dessus.

## Parcours utilisateur (2 écrans)

### Écran 1 — Éditeur
1. **Header** : titre fun (« Ta carte de Barcelone 🎉 »), sous-titre court.
2. **Zone d'upload** : drag & drop + clic, `accept="image/*,video/*"`, multi-fichiers. Les vidéos sont automatiquement **bouclées sur leurs 5 premières secondes** (pas de découpe réelle : lecture limitée à 0–5 s, muette, en boucle). Utiliser `URL.createObjectURL` (jamais de base64 pour les vidéos), `revokeObjectURL` à la suppression.
3. **Grille de vignettes** : aperçu de chaque média (badge 🎬 pour vidéo), boutons **supprimer** et **réordonner** (‹ ›). Pas de drag-reorder (source de bugs).
4. **Champs texte** : message principal (max 90 caractères, compteur) + signature « De la part de… » (optionnelle).
5. **CTA** « Lancer la carte ✨ » — actif dès qu'il y a ≥ 1 média ; sinon proposer un **mode démo** avec 4 slides SVG intégrées (aucun réseau requis).

### Écran 2 — Lecture plein écran
1. **Carrousel coverflow 3D** : la slide centrale est **nette, grande, face caméra** ; les slides voisines sont **derrière sur les côtés, floutées** (`filter: blur(4–6px)`, `brightness(0.7)`), réduites (`scale ~0.75`), inclinées (`rotateY ±35°`), décalées (`translateX ±55%`, `translateZ` négatif). `perspective: 1200px` sur le conteneur, `transition 0.6s cubic-bezier(0.22, 1, 0.36, 1)`. Au-delà de ±3 positions : `opacity: 0`.
2. **Défilement automatique** toutes les **5 s**, bouclage infini. Vidéos : `muted playsinline`, `play()` quand centrales, `pause()` + `currentTime = 0` quand elles quittent le centre, boucle 0→5 s via `timeupdate`.
3. **Navigation manuelle** : flèches ‹ ›, clic sur une slide latérale, swipe tactile, clavier (← → naviguent, Espace = pause/lecture, Échap = retour éditeur). Toute action manuelle **réinitialise le timer** d'autoplay.
4. **Message animé** : bandeau en bas, texte en dégradé chaud animé, apparition lettre par lettre à l'ouverture, légère pulsation ensuite ; signature en dessous.
5. **Contrôles discrets** : pause/lecture, quitter (✕). Masqués après 3 s d'inactivité, réaffichés au mouvement de souris/touch.

## Direction artistique « Barcelone »

- **Palette trencadís (Gaudí)** : bleu Méditerranée `#1273b8`, turquoise `#18a999`, jaune soleil `#ffc93c`, orange-rouge `#e4572e`, rose `#ef6da0`, fond nuit chaude en dégradé `#2a1a5e → #0e4a86`.
- **Skyline** : silhouette SVG inline (Sagrada Família + tours + palmiers) en bas de page, semi-transparente.
- **Confettis** : canvas léger, petits carrés « mosaïque » aux couleurs de la palette qui tombent en continu en mode lecture (≤ 80 particules, `requestAnimationFrame`, stoppé hors mode lecture).
- **Typo** : system-ui en fallback ; titres très ronds et gras (900), boutons pill.
- **Micro-interactions** : hover scale sur les boutons, vignettes qui « pop » à l'ajout, emojis 🎉🏖️☀️.

## Contraintes qualité (anti-bugs)

- Zéro dépendance réseau : pas de CDN, pas de Google Fonts bloquantes, pas d'API.
- Interdits : `localStorage`/`sessionStorage`, drag-reorder, découpe vidéo réelle, autoplay avec son.
- Gérer : 0 média (mode démo), 1 seul média (pas de voisins dupliqués buggés — désactiver le coverflow latéral), fichiers non image/vidéo (ignorés avec toast), portrait/paysage (`object-fit: cover`), mobile (breakpoint ≤ 640 px : cartes plus étroites, flèches plus grandes).
- Nettoyage : `revokeObjectURL` sur suppression, `clearInterval`/`cancelAnimationFrame` en quittant le mode lecture.
- Accessibilité : boutons avec `aria-label`, contraste AA sur les contrôles, focus visible.

## Critères d'acceptation

1. J'ajoute 3 photos + 1 vidéo → la carte se lance, la vidéo joue muette 5 s en boucle quand elle est au centre.
2. La slide centrale est nette, les latérales floutées et inclinées, la transition est fluide (60 fps sur laptop récent).
3. Flèches, swipe, clavier et clic latéral fonctionnent ; Échap revient à l'éditeur sans perdre les médias.
4. Sans aucun upload, le mode démo tourne seul, hors ligne.
5. Aucune erreur console pendant tout le parcours.
