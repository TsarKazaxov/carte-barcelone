# Prompt détaillé v2 — « Carte de vœux collective animée · Barcelone »

> Prompt prêt à copier-coller dans Lovable, v0, Claude, etc. Remplace la v1 : ajoute la collaboration à distance et le message par photo.

---

## Rôle

Tu es Product Designer senior et développeur front-end expert. Construis une application **collaborative** de carte de vœux animée au thème de Barcelone, sous forme d'un **fichier HTML unique** (HTML + CSS + JS vanilla, sans framework, sans build). Backend : **Supabase en REST pur via `fetch`** (pas de librairie supabase-js). Le fichier doit fonctionner en **mode local dégradé** si les clés Supabase ne sont pas renseignées.

## Concept

**1 photo/vidéo = 1 message = 1 auteur.** Une personne crée une carte et partage un lien ; jusqu'à ~10 personnes, où qu'elles soient, ouvrent ce lien et ajoutent chacune leur souvenir (photo ou vidéo de 5 s) avec leur message personnel et leur prénom. La carte finale est un carrousel coverflow plein écran : chaque slide affiche le média de la personne avec **son** message animé en dessous. Il n'y a **pas de message global**.

## Architecture

- **Identité de carte** : paramètre d'URL `?card=<uuid>` (générer via `crypto.randomUUID`, fallback manuel).
- **Table Supabase `contributions`** : `id uuid pk`, `card_id uuid`, `name text`, `message text`, `media_path text`, `media_type 'image'|'video'`, `created_at timestamptz`. RLS activée : `select` et `insert` publics (clé anon).
- **Storage** : bucket public `media`, chemin `<card_id>/<uuid>.<ext>`, policy d'insert anonyme. Limite client : 45 Mo/fichier.
- **API REST** : `POST /rest/v1/contributions`, `GET /rest/v1/contributions?card_id=eq.X&order=created_at.asc`, `POST /storage/v1/object/media/<path>`, lecture via `/storage/v1/object/public/media/<path>`. Headers `apikey` + `Authorization: Bearer <anon>`.
- **Quasi temps réel** : polling toutes les 12 s pendant la lecture ; si de nouvelles lignes arrivent, reconstruire le carrousel en conservant l'index courant et afficher un toast « Nouveau souvenir reçu ! 💌 ». Pas de websocket (fiabilité d'abord).
- **Mode local** (clés vides) : mêmes écrans, contributions stockées en mémoire avec `URL.createObjectURL`, suppression possible. Jamais de `localStorage`.

## Écrans

### 1. Accueil (cloud uniquement, sans `?card`)
Titre fun, bouton « Créer une nouvelle carte ✨ » → génère l'uuid, `history.replaceState` vers `?card=<uuid>`, passe à l'écran contribution.

### 2. Contribution (`?card=<uuid>` ou mode local)
- Badge de mode : « ☁️ Carte partagée en ligne » / « 💻 Mode local ».
- **Panneau lien** (cloud) : champ readonly avec l'URL complète + bouton « Copier 📋 » (`navigator.clipboard` avec fallback `execCommand`).
- **Formulaire** : prénom (requis, 40 car.), dropzone un seul média (drag & drop + clic, `accept="image/*,video/*"`, aperçu avec badge 🎬 5s pour vidéo, remplacement si nouveau fichier), message (requis, 90 car. + compteur). Bouton « Ajouter à la carte 🎁 » actif seulement si les 3 champs sont remplis ; pendant l'upload : désactivé + « Envoi… ⏳ » ; succès → toast vert + reset du formulaire (le prénom reste).
- **Galerie des contributions** : vignette + prénom + message de chacun (rafraîchie après envoi ; suppression possible en mode local uniquement).
- Boutons : « Voir la carte 🎬 » (recharge la liste avant de lancer) et « Voir une démo 🏖️ » (4 slides SVG inline avec messages/auteurs fictifs, zéro réseau).

### 3. Lecture plein écran
- **Coverflow 3D** : slide centrale nette (`scale 1`, sans filtre), voisines floutées (`blur 3–7 px`, `brightness 0.65`), inclinées (`rotateY ±35°`), reculées (`translateZ` négatif, `scale 0.78`), `perspective 1200px`, transition `0.6s cubic-bezier(0.22,1,0.36,1)`, offset circulaire (la dernière slide est voisine gauche de la première), `opacity 0` au-delà de ±3.
- **Bandeau par slide** : message de l'auteur en dégradé jaune→rose, animé lettre par lettre à chaque changement de slide, « — Prénom 💛 » en dessous, indicateur de position `●○○○`.
- **Vidéos** : `muted playsinline`, lecture seulement au centre, boucle limitée aux 5 premières secondes via `timeupdate`, `pause()` en quittant le centre.
- **Navigation** : autoplay 5 s/slide, flèches, clic sur slide latérale, swipe (seuil 40 px), clavier (←/→, Espace pause, Échap retour), toute action manuelle réinitialise le timer. Contrôles masqués après 3 s d'inactivité.
- **Ambiance** : confettis mosaïque canvas (70 carrés, couleurs trencadís, `requestAnimationFrame`, stoppés à la sortie), skyline SVG Sagrada Família en fond.

## Direction artistique

Palette trencadís : `#1273b8`, `#18a999`, `#ffc93c`, `#e4572e`, `#ef6da0`, fond `#2a1a5e → #0e4a86`. Titres 900 en dégradé animé, boutons pill, cartes bordées de blanc avec grosse ombre, emojis généreux, micro-animations « pop » à l'ajout.

## Anti-bugs (obligatoire)

- Gérer : 0 contribution (bouton refuse avec toast), 1 seule (flèches cachées, pas de voisins), fichier non média (toast), fichier > 45 Mo (toast), erreur réseau (toast, bouton réactivé, aucune perte de saisie), Échap conserve l'état.
- Nettoyage : `revokeObjectURL`, `clearInterval` (autoplay + polling), `cancelAnimationFrame` à la sortie.
- `canvas.getContext` gardé par null-check ; promesse `play()` catchée.
- Accessibilité : `aria-label` sur tous les boutons icône, focus visible, contrastes AA.

## Critères d'acceptation

1. A crée une carte à Paris, copie le lien ; B (Tokyo) et C (New York) ouvrent le lien, ajoutent chacun photo + message + prénom.
2. A clique « Voir la carte » : 3 slides, chacune avec le bon message et le bon prénom animés.
3. B ajoute une 4e contribution pendant que A regarde : elle apparaît en ≤ 12 s avec un toast.
4. Une vidéo de 20 s ne joue que ses 5 premières secondes, muette, en boucle, uniquement au centre.
5. Sans clés Supabase, le fichier reste utilisable en local (ajout, suppression, lecture, démo) sans aucune erreur console.
