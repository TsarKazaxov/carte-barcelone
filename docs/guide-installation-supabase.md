# Guide d'installation — Carte collective Barcelone (version en ligne)

La version collaborative fonctionne avec **Supabase** (gratuit) : base de données pour les messages + stockage pour les photos/vidéos. Sans configuration, le fichier tourne en **mode local** (test sur un seul appareil).

⏱️ Temps total : ~15 minutes. Aucune ligne de code à écrire, juste du copier-coller.

---

## Étape 1 — Créer le projet Supabase (5 min)

1. Va sur [supabase.com](https://supabase.com) → **Start your project** → crée un compte (GitHub ou email).
2. **New project** : nom `carte-barcelone`, mot de passe base de données (garde-le), région `West EU (Paris)`.
3. Attends ~1 min que le projet démarre.

## Étape 2 — Créer la table (2 min)

Dans le menu gauche : **SQL Editor** → **New query** → colle ceci → **Run** :

```sql
create table contributions (
  id uuid primary key default gen_random_uuid(),
  card_id uuid not null,
  name text not null,
  message text not null,
  media_path text not null,
  media_type text not null check (media_type in ('image','video')),
  created_at timestamptz default now()
);

alter table contributions enable row level security;

create policy "lecture publique" on contributions
  for select using (true);

create policy "ajout public" on contributions
  for insert with check (true);
```

## Étape 3 — Créer le bucket de stockage (2 min)

1. Menu gauche : **Storage** → **New bucket**.
2. Nom : `media` — coche **Public bucket** → **Create**.
3. Toujours dans SQL Editor, colle et **Run** (autorise l'upload anonyme) :

```sql
create policy "upload public media" on storage.objects
  for insert with check (bucket_id = 'media');
```

## Étape 4 — Récupérer tes clés (1 min)

Menu gauche : **Project Settings** → **API** :
- **Project URL** → ex. `https://abcdefgh.supabase.co`
- **anon public key** → longue chaîne `eyJhbGci...`

## Étape 5 — Configurer le fichier HTML (1 min)

Ouvre `index.html` dans un éditeur de texte, en haut du `<script>` :

```js
var SUPABASE_URL = 'https://abcdefgh.supabase.co';   // ta Project URL
var SUPABASE_ANON_KEY = 'eyJhbGci...';               // ta clé anon
```

## Étape 6 — Héberger le fichier (3 min)

**GitHub Pages** (choix retenu) : suis la section « Déployer sur GitHub Pages » du `README.md` à la racine du projet. Alternatives : Netlify Drop, Vercel, Cloudflare Pages.

## Étape 7 — Utiliser 🎉

1. Ouvre ton URL → **Créer une nouvelle carte** → un lien `?card=...` est généré.
2. **Copie le lien** et envoie-le aux 10 personnes (WhatsApp, mail…).
3. Chacun ouvre le lien, ajoute **sa photo/vidéo + son message + son prénom**.
4. N'importe qui clique **Voir la carte 🎬** : le carrousel affiche chaque souvenir avec son message. Les nouvelles contributions apparaissent automatiquement (vérification toutes les 12 s).

---

## Limites et notes

- Fichiers limités à 45 Mo (limite du plan gratuit Supabase : 50 Mo/fichier, 1 Go de stockage total).
- Les vidéos jouent muettes, en boucle sur leurs 5 premières secondes.
- Le lien de la carte n'est pas protégé par mot de passe : quiconque a le lien peut contribuer et voir la carte (c'est le principe, mais ne partage pas le lien publiquement).
- Quasi temps réel : rafraîchissement toutes les 12 s pendant la lecture (fiable partout, pas de websocket fragile).
