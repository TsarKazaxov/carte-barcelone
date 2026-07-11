# 🎉 Carte de vœux collective · Barcelone

Une carte de vœux animée et collaborative : chaque personne ajoute **sa photo ou sa mini-vidéo (5 s) + son message + son prénom**, où qu'elle soit dans le monde. La carte finale est un carrousel coverflow plein écran — slide centrale nette, voisines floutées — avec le message de chaque auteur animé sous sa photo, confettis mosaïque et skyline de la Sagrada Família.

## Structure du projet

| Fichier | Rôle |
|---|---|
| `index.html` | **Version collaborative** (à héberger). Lien partagé `?card=...`, backend Supabase. Sans clés Supabase : mode local de test. |
| `carte-locale.html` | Version v1 hors-ligne, monoposte (message global, pas de partage). |
| `docs/guide-installation-supabase.md` | Configuration Supabase pas-à-pas (~15 min, copier-coller). |
| `docs/prompt-v2-collab.md` | Prompt détaillé de la version collaborative (spec complète). |
| `docs/prompt-v1.md` | Prompt détaillé de la v1 locale. |

## Déployer sur GitHub Pages (~10 min)

> Prérequis : avoir configuré Supabase d'abord (voir `docs/guide-installation-supabase.md`) et collé tes clés dans `index.html` (lignes `SUPABASE_URL` et `SUPABASE_ANON_KEY` en haut du `<script>`).

1. Va sur [github.com/new](https://github.com/new) : nom du repo `carte-barcelone`, visibilité **Public** (requis pour Pages en plan gratuit), **Create repository**.
2. Sur la page du repo : **uploading an existing file** → glisse `index.html` (et le dossier `docs` si tu veux) → **Commit changes**.
3. **Settings** → **Pages** (menu gauche) → Source : **Deploy from a branch** → Branch : `main` + `/ (root)` → **Save**.
4. Attends 1-2 min. Ton site est en ligne sur `https://<ton-user>.github.io/carte-barcelone/`.
5. Ouvre l'URL → **Créer une nouvelle carte** → copie le lien `?card=...` → envoie-le au groupe. C'est parti 🎉

⚠️ Le repo est public : n'y mets que `index.html` et la doc. La clé `anon` de Supabase est faite pour être exposée côté client (la sécurité vient des policies RLS), c'est normal qu'elle soit visible.

### Mettre à jour le site

Sur GitHub : ouvre `index.html` → icône crayon ✏️ → colle la nouvelle version → **Commit changes**. Pages se redéploie tout seul en ~1 min.

## Fonctionnement

1. **Créer** : ouvre le site → « Créer une nouvelle carte ✨ » → un lien unique est généré.
2. **Contribuer** : chaque personne ouvre le lien, ajoute prénom + photo/vidéo + message (90 car. max).
3. **Regarder** : « Voir la carte 🎬 » → coverflow auto (5 s/slide), flèches, swipe, clavier (←/→, Espace, Échap). Les nouvelles contributions apparaissent pendant la lecture (vérification toutes les 12 s).

## Limites connues

- Fichiers ≤ 45 Mo (plan gratuit Supabase : 50 Mo/fichier, 1 Go total).
- Vidéos : muettes, bouclées sur leurs 5 premières secondes.
- Quiconque a le lien peut voir et contribuer — ne pas le partager publiquement.
