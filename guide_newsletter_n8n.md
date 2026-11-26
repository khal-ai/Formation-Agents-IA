# Tutoriel n8n : créer une newsletter automatique

Ce tutoriel s'adresse à des débutants qui n'ont jamais créé de workflow. Vous allez construire, étape par étape, une automatisation qui collecte des articles RSS, les fait résumer par une IA, crée une newsletter HTML et l'envoie aux abonnés actifs.

> **Important :** copiez exactement les noms des nœuds et des champs indiqués. Les expressions et le code font référence à ces noms. Une différence (même une majuscule, un espace ou un accent) peut empêcher le workflow de fonctionner.

## 1. Ce que le workflow va faire

1. Se déclencher tous les jours à 9 h.
2. Lire les URL de flux RSS dans Google Sheets.
3. définir le sujet de la newsletter et la période de collecte.
4. Lire les articles de chaque flux.
5. Filtrer les articles récents, nettoyer les URL et retirer les doublons.
6. Vérifier si des articles ont été trouvés.
7. Demander à une IA de rédiger la newsletter.
8. Générer le HTML de l'email.
9. Récupérer les abonnés actifs.
10. Envoyer la newsletter à chacun d'eux.

## 2. Préparer les outils

Vous aurez besoin de :

- Une instance n8n.
- Un compte Google et deux Google Sheets.
- Un compte Gmail autorisé à envoyer des emails.
- Une clé API OpenAI et un modèle accessible via votre API.

Les API OpenAI peuvent entraîner des frais distincts de l'abonnement ChatGPT. Ne partagez pas vos clés API et ne les publiez jamais dans GitHub.

### 2.1. Créer le Google Sheets « Newsletter - Données »

Ce fichier centralise les sites qui alimentent la newsletter. Créez un nouveau fichier Google Sheets et donnez-lui exactement le nom **Newsletter - Données**.

Dans la première feuille, copiez-collez le bloc ci-dessous dans la cellule **A1**. Il est au format tabulé : Google Sheets répartira les valeurs dans les deux colonnes.

#### Colonne A — `nom` (à coller en A1)

```text
nom
ActuIA
Next
Korben
Siècle Digital
Numerama IA
Developpez
ZDNet France
Le Monde Informatique
Techniques de l'Ingénieur
```

#### Colonne B — `rss` (à coller en B1)


```text
rss
https://www.actuia.com/feed/
https://next.ink/feed/
https://korben.info/feed
https://siecledigital.fr/feed/
https://www.numerama.com/tech/intelligence-artificielle/feed/
https://www.developpez.com/index/rss
https://www.zdnet.fr/feeds/rss/actualites/
https://www.lemondeinformatique.fr/flux-rss/thematique/toutes-les-actualites/rss.xml
https://www.techniques-ingenieur.fr/actualite/theme/informatique-et-numerique/feed/
```



Après le collage, vérifiez que vous avez bien deux colonnes :

- `nom` : nom de la source d'information.
- `rss` : adresse du flux RSS utilisée pour récupérer les articles.

**Important :** copiez le contenu du bloc de texte (et non le tableau Markdown). Les deux champs de chaque ligne sont séparés par une tabulation, afin que Google Sheets les place dans deux cellules distinctes. Si tout le contenu apparaît malgré tout dans une seule colonne, sélectionnez cette colonne, puis utilisez **Données → Scinder le texte en colonnes** et choisissez le séparateur **Tabulation**.

<text color="success" weight="medium">Le tableau est prêt à l'emploi : ne modifiez pas les noms des colonnes, car le workflow les utilise.</text>

### 2.2. Créer le Google Sheets « Abonnés »

Créez un second fichier Google Sheets et nommez-le **Abonnés** (ou choisissez le nom de fichier que vous souhaitez, en vous assurant de sélectionner ce fichier ensuite dans n8n).

Chaque lecteur doit utiliser ses propres adresses. Pour éviter de partager des données personnelles, ce guide ne fournit aucun exemple d'abonné. Copiez-collez uniquement la ligne d'en-têtes suivante dans la cellule **A1** de la première feuille :

```text
email	prenom	actif
```

Vous pourrez ensuite ajouter vos propres abonnés, une personne par ligne :

- `email` : adresse email du destinataire.
- `prenom` : prénom du destinataire (utilisable pour personnaliser le message, si le workflow le prévoit).
- `actif` : indique si la personne doit recevoir la newsletter. Le filtre de ce workflow utilise la valeur `oui` pour les abonnés actifs.

Respectez exactement les trois en-têtes `email`, `prenom` et `actif` (en minuscules). Lors de vos tests, utilisez uniquement une adresse que vous contrôlez.

## 3. Créer le workflow

1. Ouvrez n8n.
2. Créez un nouveau workflow.
3. Nommez-le `Newsletter automatique - simple`.
4. Enregistrez-le.

Dans n8n, les blocs sont appelés **nœuds** (nodes). Ils réalisent des actions. Les connexions entre nœuds transmettent les données. Nous allons les créer et les connecter progressivement.

## 4. Nœud 1 — Schedule Trigger

**Nom exact :** `Schedule Trigger`

### Rôle

Déclenche automatiquement le workflow chaque jour à 9 h.

### Création et configuration

1. Cliquez sur le bouton `+` pour ajouter un nœud.
2. Recherchez `Schedule Trigger` et sélectionnez-le.
3. Vérifiez que son nom est exactement `Schedule Trigger`.
4. Configurez un déclenchement quotidien (Every Day / Days, selon votre version).
5. Heure : `9`.
6. Minute : `0`, si ce champ est proposé.

Vérifiez le fuseau horaire du workflow ou de votre instance n8n. Pour la France métropolitaine, `Europe/Paris` est généralement approprié.

> Les libellés peuvent varier selon la version de n8n. Le résultat recherché est un déclenchement quotidien à 09:00.

Pour les premiers tests, utilisez l'exécution manuelle. Le déclenchement programmé ne fonctionnera en production que lorsque le workflow sera activé.

## 5. Nœud 2 — Lire sources

**Nom exact :** `Lire sources`

### Rôle

Lit les lignes du Google Sheets contenant les URL des flux RSS.

### Création et configuration

1. À partir du nœud `Schedule Trigger`, ajoutez un nœud `Google Sheets`.
2. Renommez-le exactement `Lire sources`.
3. Dans Credentials, connectez votre compte Google Sheets via OAuth et autorisez l'accès.
4. Configurez les champs (les libellés peuvent varier selon les versions) :

| Paramètre | Valeur |
|---|---|
| Resource | Sheet Within Document (ou équivalent) |
| Operation | Get Row(s) |
| Document | Votre fichier `Newsletter - Données` |
| Sheet | Feuille 1 |

5. Sélectionnez le fichier et la feuille qui contiennent les colonnes `nom` et `rss`.

### Test

Cliquez sur **Execute step**. Le résultat doit contenir les lignes et une propriété `rss`, par exemple :

```json
{
  "rss": "https://www.developpez.com/index/rss"
}
```

### Connexion

Connectez :

`Schedule Trigger → Lire sources`

> Ne renommez pas ce nœud : le nœud RSS suivant utilise le nom `Lire sources` dans une expression.

## 6. Nœud 3 — Constantes

**Nom exact :** `Constantes`

### Rôle

Définit les paramètres de la newsletter : son sujet et le nombre de jours d'actualité à collecter. Cela permet de modifier ces valeurs sans changer le code.

### Création et configuration

1. À la sortie de `Lire sources`, ajoutez `Edit Fields` (parfois appelé `Set` dans des versions antérieures).
2. Renommez-le exactement `Constantes`.
3. Choisissez le mode de saisie manuelle (Manual Mapping).
4. Ajoutez les deux champs suivants :

| Name | Type | Value |
|---|---|---|
| `sujet` | String | `Intelligence artificielle` |
| `jours` | Number | `7` |

Respectez précisément les noms `sujet` et `jours`, en minuscules.

### Test

Cliquez sur **Execute step**. Vous devez obtenir, entre autres, les valeurs suivantes :

```json
{
  "sujet": "Intelligence artificielle",
  "jours": 7
}
```

### Connexion

`Lire sources → Constantes`

> Le code du nœud `Filtrer` et le prompt de `Rédiger` font référence au nœud `Constantes` et à ces propriétés.

## 7. Nœud 4 — RSS Read

**Nom exact :** `RSS Read`

### Rôle

Récupère les articles publiés dans les flux RSS : titre, lien, date, extrait et autres informations disponibles.

### Création et configuration

1. À la sortie de `Constantes`, ajoutez `RSS Feed Read`.
2. Renommez-le exactement `RSS Read`.
3. Dans le champ URL, passez en mode **Expression** (et non Fixed).
4. Copiez-collez cette expression :

```javascript
={{ $('Lire sources').item.json.rss }}
```

Cette expression signifie :

- `$('Lire sources')` : référence le nœud nommé `Lire sources`.
- `.item` : récupère l'élément associé au contexte courant.
- `.json` : accède aux données de cet élément.
- `.rss` : récupère la valeur du champ `rss`, c'est-à-dire l'URL du flux.

L'URL n'est donc pas saisie en dur : le nœud utilise les URL récupérées dans Google Sheets.

Dans Settings, si votre version propose le réglage **On Error**, choisissez une option de continuation équivalente à `Continue` / `Continue using error output`, si vous souhaitez que les erreurs sur un flux ne bloquent pas nécessairement toute l'exécution. Le comportement précis dépend de la version.

### Test

Cliquez sur **Execute step**. Vérifiez que des articles sont récupérés. Selon le flux, les champs peuvent notamment ressembler à ceci :

```json
{
  "title": "Exemple d'article",
  "link": "https://example.com/article",
  "pubDate": "Fri, 20 Nov 2026 10:00:00 GMT",
  "contentSnippet": "Extrait de l'article..."
}
```

Les champs exacts dépendent du flux RSS.

### Connexion

`Constantes → RSS Read`

## 8. Nœud 5 — Filtrer

**Nom exact :** `Filtrer`

### Rôle

Ce nœud Code nettoie et sélectionne les articles. Il :

- conserve les articles de la période choisie ;
- exclut certains contenus publicitaires ou promotionnels selon des motifs simples ;
- nettoie les URL ;
- élimine les doublons ;
- détermine la source ;
- limite le nombre d'articles par source et le total ;
- attribue un identifiant à chaque article.

### Création et configuration

1. À la sortie de `RSS Read`, ajoutez un nœud `Code`.
2. Renommez-le exactement `Filtrer`.
3. Choisissez **Run Once for All Items** comme Mode.
4. Choisissez `JavaScript` comme Language.
5. Remplacez le code présent par celui-ci (copiez tout) :

```javascript
const jours = $('Constantes').first().json.jours || 7;
const limite = Date.now() - jours * 86400000;
const vus = new Set();
const articles = [];

for (const { json: j } of $input.all()) {
  const d = new Date(j.isoDate || j.pubDate);
  if (isNaN(d) || d.getTime() < limite || !j.link) continue;

  // Exclure les contenus sponsorisés ou publicitaires
  const titre = j.title || '';
  if (/\/(publicite|guide-achat|tribune)\//i.test(j.link) || /#sponso/i.test(titre)) continue;

  // Nettoyer l'URL : ancres de suivi et paramètres utm_
  let url = j.link.replace(/#.*$/, '');
  url = url
    .replace(/([?&])utm_[^&]*/g, '$1')
    .replace(/\?&+/, '?')
    .replace(/&{2,}/g, '&')
    .replace(/[?&]+$/, '');
  if (vus.has(url)) continue;
  vus.add(url);

  // Source = nom de domaine principal (developpez.com, zdnet.fr…)
  const host = (url.match(/^https?:\/\/([^\/?#:]+)/i) || [])[1];
  if (!host) continue;
  const source = host.replace(/^www\./, '').split('.').slice(-2).join('.');

  articles.push({
    id: 0,
    titre,
    extrait: (j.contentSnippet || j.content || '')
      .replace(/<[^>]*>/g, ' ').replace(/\s+/g, ' ').trim().slice(0, 300),
    url,
    date: d.toISOString().slice(0, 10),
    source
  });
}

articles.sort((a, b) => b.date.localeCompare(a.date));

// Maximum 12 articles par source
const parSource = {};
const retenus = articles.filter(a => {
  parSource[a.source] = (parSource[a.source] || 0) + 1;
  return parSource[a.source] <= 12;
}).slice(0, 100);

retenus.forEach((a, k) => a.id = k + 1);

return [{ json: { articles: retenus, total: retenus.length } }];
```

> **Important :** ne changez pas le mode d'exécution. `Run Once for All Items` permet au code de traiter tous les articles ensemble, notamment pour éliminer les doublons.

### Comprendre quelques parties du code

**Récupérer la période :**

```javascript
const jours = $('Constantes').first().json.jours || 7;
```

Récupère le champ `jours` dans le nœud `Constantes`. Si la valeur n'existe pas, utilise 7.

**Calculer la date limite :**

```javascript
const limite = Date.now() - jours * 86400000;
```

`86400000` est le nombre de millisecondes dans une journée. Le calcul détermine la date limite.

**Parcourir les articles :**

```javascript
for (const { json: j } of $input.all()) {
```

Parcourt tous les éléments reçus en entrée.

**Vérifier la date et le lien :**

```javascript
const d = new Date(j.isoDate || j.pubDate);
if (isNaN(d) || d.getTime() < limite || !j.link) continue;
```

Ignore les articles dont la date est invalide, trop ancienne ou qui n'ont pas de lien.

**Exclure certains contenus :**

```javascript
if (/\/(publicite|guide-achat|tribune)\//i.test(j.link) || /#sponso/i.test(titre)) continue;
```

Exclut certains contenus si l'URL ou le titre correspond à des motifs identifiés. Ce filtre n'est pas une détection universelle de publicité.

**Éliminer les doublons :**

Le `Set` appelé `vus` mémorise les URL déjà traitées. Si une URL a déjà été rencontrée, l'article est ignoré.

**Limiter le nombre d'articles :**

```javascript
return parSource[a.source] <= 12;
```

Conserve au maximum 12 articles par source, puis :

```javascript
.slice(0, 100);
```

Limite le résultat à 100 articles.

**Attribuer un identifiant :**

```javascript
retenus.forEach((a, k) => a.id = k + 1);
```

Chaque article reçoit un identifiant numérique unique (1, 2, 3…). L'IA utilisera ces identifiants pour référencer les articles.

### Test

Cliquez sur **Execute step**. La sortie doit contenir un seul élément avec les propriétés :

- `articles` : tableau d'articles retenus ;
- `total` : nombre d'articles retenus.

Structure simplifiée :

```json
{
  "articles": [
    {
      "id": 1,
      "titre": "Exemple de titre",
      "extrait": "Extrait de l'article",
      "url": "https://example.com/article",
      "date": "2026-11-26",
      "source": "example.com"
    }
  ],
  "total": 1
}
```

### Connexion

`RSS Read → Filtrer`

## 9. Nœud 6 — Des articles ?

**Nom exact :** `Des articles ?`

### Rôle

Vérifie si le nœud `Filtrer` a retenu au moins un article.

- Si le nombre d'articles est différent de zéro, le workflow continue sur la sortie `true`.
- Si le total est égal à zéro, il passe sur la sortie `false` et la suite n'est pas exécutée dans cette version.

### Création et configuration

1. À la sortie de `Filtrer`, ajoutez un nœud `IF`.
2. Renommez-le exactement `Des articles ?`.
3. Ajoutez une condition de type Number.

| Paramètre | Valeur |
|---|---|
| Value 1 | Expression |
| Operator | Not Equal |
| Value 2 | `0` |

Dans Value 1, activez le mode Expression et copiez-collez :

```javascript
={{ $json.total }}
```

Dans Value 2, saisissez le nombre `0`.

La condition vérifie que le champ `total` n'est pas égal à zéro.

### Connexion

`Filtrer → Des articles ?`

La sortie `true` sera connectée au nœud `Rédiger`. La sortie `false` reste sans connexion dans cette version.

## 10. Nœud 7 — Rédiger

**Nom exact :** `Rédiger`

### Rôle

Ce nœud utilise un modèle d'IA pour sélectionner les articles pertinents et rédiger la newsletter. Il reçoit le sujet et les articles filtrés, puis produit un contenu structuré (objet, accroche, introduction, articles résumés et conclusion).

### Création et configuration

1. Sur la sortie `true` de `Des articles ?`, ajoutez le nœud `Chain LLM` (ou LLM Chain selon votre version).
2. Renommez-le exactement `Rédiger`.
3. Choisissez `Define` comme Prompt Type.
4. Dans le champ Text, passez en mode **Expression**.
5. Copiez-collez le prompt suivant, en entier :

```text
=Tu es rédacteur en chef d'une newsletter francophone.

Règles :
- Utilise UNIQUEMENT les articles fournis, référencés par leur id. N'écris jamais d'URL.
- Retiens 5 à 8 articles, les plus pertinents pour le sujet et les plus récents. Ignore ceux qui n'ont aucun rapport avec le sujet.
- Résume chaque article de façon factuelle en 2-3 phrases, dans un ton clair et engageant.
- N'écris aucun chiffre, nom propre ou fait qui ne figure pas dans le titre ou l'extrait de l'article concerné. Si l'extrait est trop court, reste général plutôt que d'inventer.

Sujet : {{ $('Constantes').first().json.sujet }}

Articles : {{ JSON.stringify($('Filtrer').first().json.articles) }}
```

> **Attention :** gardez les expressions `{{ ... }}` intactes. Elles injectent le sujet et la liste d'articles dans le prompt. Le champ doit être en mode Expression.

6. Activez **Require Specific Output Format** (ou le réglage équivalent) pour utiliser un parser de sortie structurée.
7. Laissez les options de batching par défaut.

### Comprendre le prompt

- Le rôle demande à l'IA de se comporter comme un rédacteur de newsletter francophone.
- Les règles limitent la sélection à des articles pertinents et récents, et demandent des résumés factuels.
- L'expression `{{ $('Constantes').first().json.sujet }}` récupère le sujet défini dans le nœud `Constantes`.
- L'expression `{{ JSON.stringify($('Filtrer').first().json.articles) }}` récupère la liste des articles filtrés et la transforme en texte JSON pour l'injecter dans le prompt.

La consigne de sélectionner 5 à 8 articles ne peut pas être respectée si moins de cinq articles pertinents sont disponibles. Dans ce cas, le modèle peut produire moins d'articles.

## 11. Nœud complémentaire — OpenAI Chat Model

**Nom exact :** `OpenAI Chat Model`

### Rôle

Fournit le modèle de langage utilisé par le nœud `Rédiger`. Ce nœud se connecte au port AI dédié et non à la ligne principale du workflow.

### Création et configuration

1. Ajoutez un nœud `OpenAI Chat Model`.
2. Renommez-le exactement `OpenAI Chat Model`.
3. Dans Credentials, configurez votre connexion API OpenAI. Suivez la procédure n8n pour saisir votre clé API. Ne partagez ni ne publiez cette clé.
4. Sélectionnez un modèle disponible dans votre compte API.
5. Dans Options, réglez Temperature sur `0.3`, si le champ est disponible.

> L'export d'origine indique le modèle `gpt-6-luna`, qui peut ne pas être disponible dans votre installation ou votre compte API. Sélectionnez un modèle réellement disponible dans la liste proposée.

### Connexion

Connectez le port AI du nœud `OpenAI Chat Model` au port **Language Model** du nœud `Rédiger`.

Ne le raccordez pas à la ligne principale.

## 12. Nœud complémentaire — Structured Output Parser

**Nom exact :** `Structured Output Parser`

### Rôle

Définit la structure JSON que l'IA doit renvoyer. Le nœud `Générer HTML` dépend de cette structure et des noms exacts des propriétés.

### Création et configuration

1. Ajoutez un nœud `Structured Output Parser`.
2. Renommez-le exactement `Structured Output Parser`.
3. Choisissez le mode permettant de générer le schéma à partir d'un exemple JSON (par exemple `Generate From JSON Example`).
4. Copiez-collez cet exemple JSON dans le champ prévu :

```json
{
  "objet": "Titre de l'email",
  "accroche": "Phrase d'accroche",
  "intro": "Introduction courte",
  "articles": [
    {
      "id": 1,
      "titre": "Titre de l'article",
      "resume": "Résumé de 2 à 3 phrases"
    }
  ],
  "conclusion": "Conclusion courte"
}
```

> Le JSON doit être valide : ne mettez pas de commentaires, ne supprimez pas les guillemets et ne renommez pas les propriétés.

### Rôle des propriétés

| Propriété | Utilité |
|---|---|
| `objet` | Objet de l'email |
| `accroche` | Phrase d'accroche |
| `intro` | Introduction |
| `articles` | Liste des articles retenus |
| `id` | Identifiant de l'article d'origine |
| `titre` | Titre de l'article |
| `resume` | Résumé |
| `conclusion` | Conclusion |

### Connexion

Connectez le port **Output Parser** du `Structured Output Parser` au port **Output Parser** du nœud `Rédiger`.

> Les noms des propriétés sont essentiels. Si vous les modifiez, il faudra aussi modifier le code de `Générer HTML`.

## 13. Nœud 8 — Générer HTML

**Nom exact :** `Générer HTML`

### Rôle

Transforme la sortie de l'IA en email HTML. Le code associe les articles sélectionnés par l'IA aux articles originaux grâce à leur identifiant, retrouve les URL et les sources, puis construit une newsletter mise en forme.

### Création et configuration

1. À la sortie de `Rédiger`, ajoutez un nœud `Code`.
2. Renommez-le exactement `Générer HTML`.
3. Mode : `Run Once for All Items`.
4. Language : `JavaScript`.
5. Remplacez le code présent par le code complet suivant :

```javascript
const d = $('Rédiger').first().json.output;
const pool = $('Filtrer').first().json.articles;
const esc = s => String(s ?? '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
const fmt = x => new Date(x).toLocaleDateString('fr-FR', { day:'numeric', month:'long', year:'numeric' });

const invalides = [];
const cards = d.articles.map(a => {
  const s = pool.find(p => p.id === a.id);
  if (!s) { invalides.push(a.id); return ''; }
  return `<tr><td style="padding:0 32px 20px">
  <table width="100%" cellpadding="0" cellspacing="0" style="background:#f8fafc;border-radius:12px;border:1px solid #e5e7eb"><tr><td style="padding:20px">
    <div style="font-size:12px;color:#6366f1;font-weight:600;text-transform:uppercase;letter-spacing:.5px">${esc(s.source.split('.')[0])} · ${fmt(s.date)}</div>
    <h2 style="margin:8px 0;font-size:18px;line-height:1.3;color:#111827">${esc(a.titre)}</h2>
    <p style="margin:0 0 14px;font-size:15px;line-height:1.6;color:#374151">${esc(a.resume)}</p>
    <a href="${esc(s.url)}" style="display:inline-block;background:#4f46e5;color:#ffffff;text-decoration:none;padding:9px 16px;border-radius:8px;font-size:14px;font-weight:600">Lire l’article →</a>
  </td></tr></table></td></tr>`;
}).join('');

const html = `<!DOCTYPE html><html lang="fr"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"></head>
<body style="margin:0;background:#eef2ff;font-family:-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif">
<div style="display:none;max-height:0;overflow:hidden">${esc(d.accroche)}</div> <!--BANDEAU-->
<table width="100%" cellpadding="0" cellspacing="0"><tr><td align="center" style="padding:0">
<table width="100%" cellpadding="0" cellspacing="0" style="max-width:720px;width:100%;background:#ffffff;border-radius:16px;overflow:hidden">
 <tr><td style="background:#4f46e5;background-image:linear-gradient(135deg,#4f46e5,#7c3aed);padding:36px 32px;color:#ffffff">
  <div style="font-size:12px;opacity:.85;letter-spacing:1px;text-transform:uppercase">Newsletter · ${fmt(new Date())}</div>
  <h1 style="margin:10px 0 0;font-size:26px;line-height:1.25">${esc(d.accroche)}</h1></td></tr>
 <tr><td style="padding:28px 32px 20px;font-size:16px;line-height:1.6;color:#374151">${esc(d.intro)}</td></tr>
 ${cards}
 <tr><td style="padding:8px 32px 28px;font-size:15px;line-height:1.6;color:#374151">${esc(d.conclusion)}</td></tr>
 <tr><td style="background:#f3f4f6;padding:20px 32px;font-size:12px;color:#6b7280;text-align:center">Vous recevez cet email car vous êtes abonné(e). Pour vous désabonner, répondez « STOP ».</td></tr>
</table></td></tr></table></body></html>`;

return [{ json: { html, objet: d.objet, nb_articles: d.articles.length - invalides.length, ids_invalides: invalides } }];
```

> **Important :** copiez le code en entier. Il référence les nœuds `Rédiger` et `Filtrer`, ainsi que les propriétés du JSON définies précédemment.

### Comprendre le code

**Récupérer la sortie de l'IA :**

```javascript
const d = $('Rédiger').first().json.output;
```

Récupère l'objet produit par la chaîne de rédaction.

**Récupérer la liste originale :**

```javascript
const pool = $('Filtrer').first().json.articles;
```

Récupère les articles pour retrouver les URL et sources.

**Échapper les caractères HTML :**

La fonction `esc` remplace certains caractères spéciaux afin qu'ils ne soient pas interprétés comme du HTML.

**Formater les dates :**

La fonction `fmt` affiche les dates dans un format français lisible.

**Retrouver l'article original :**

```javascript
const s = pool.find(p => p.id === a.id);
```

Recherche l'article d'origine ayant le même identifiant que celui sélectionné par l'IA. Cela permet de retrouver le lien réel sans demander à l'IA de produire une URL.

**Construire les cartes :**

Le code utilise des balises HTML (`table`, `h2`, `p`, `a`) et des styles pour mettre en forme chaque article.

**Retourner le résultat :**

```javascript
return [{ json: { html, objet: d.objet, nb_articles: d.articles.length - invalides.length, ids_invalides: invalides } }];
```

Retourne le HTML, l'objet email, le nombre d'articles valides et la liste des identifiants non retrouvés.

### Test

Exécutez le nœud. La sortie doit contenir notamment :

- `html` : code HTML complet.
- `objet` : objet de l'email.
- `nb_articles` : nombre d'articles valides.
- `ids_invalides` : liste d'identifiants non retrouvés.

### Connexion

`Rédiger → Générer HTML`

## 14. Nœud 9 — Get row(s) in sheet

**Nom exact :** `Get row(s) in sheet`

### Rôle

Récupère les abonnés dont la colonne `actif` indique `oui`. Les lignes récupérées seront transmises au nœud Gmail.

### Création et configuration

1. À la sortie de `Générer HTML`, ajoutez un nœud `Google Sheets`.
2. Renommez-le exactement `Get row(s) in sheet`.
3. Connectez les credentials Google Sheets (vous pouvez réutiliser la connexion du nœud `Lire sources` si elle donne accès au fichier).
4. Configurez :

| Paramètre | Valeur |
|---|---|
| Resource | Sheet Within Document (ou équivalent) |
| Operation | Get Row(s) |
| Document | Votre fichier `Abonnés` |
| Sheet | Feuille 1 |

5. Dans Filters, ajoutez le filtre suivant :

| Paramètre | Valeur |
|---|---|
| Column | `actif` |
| Value | `oui` |

Le filtre doit respecter exactement les noms de colonne et la valeur indiqués.

### Test

Cliquez sur **Execute step**. Vérifiez que seules les lignes avec `actif` = `oui` sont récupérées.

Un résultat peut ressembler à ceci :

```json
{
  "email": "votre-adresse-de-test@example.com",
  "actif": "oui"
}
```

### Connexion

`Générer HTML → Get row(s) in sheet`

## 15. Nœud 10 — Envoyer aux abonnés

**Nom exact :** `Envoyer aux abonnés`

### Rôle

Envoie la newsletter par Gmail à chaque abonné actif, en utilisant son adresse email, l'objet généré et le HTML créé précédemment.

### Création et configuration

1. À la sortie de `Get row(s) in sheet`, ajoutez un nœud `Gmail`.
2. Renommez-le exactement `Envoyer aux abonnés`.
3. Dans Credentials, créez une connexion Gmail OAuth2 et autorisez l'accès au compte Gmail. Utilisez un compte que vous contrôlez.

### Destinataire (To / Send To)

Passez le champ en mode **Expression** et copiez-collez :

```javascript
={{ $json.email }}
```

Cette expression récupère l'adresse email de l'abonné courant. Le champ de la feuille doit être nommé exactement `email`.

### Objet (Subject)

Passez le champ en mode **Expression** et copiez-collez :

```javascript
={{ $('Générer HTML').first().json.objet }}
```

Cette expression récupère l'objet de la newsletter générée.

### Message

Passez le champ en mode **Expression** et copiez-collez :

```javascript
={{ $('Générer HTML').first().json.html }}
```

Cette expression récupère le HTML de la newsletter. Ne collez pas le code HTML lui-même dans le champ Message.

### Options

Si votre version propose `Append n8n Attribution`, désactivez cette option pour reproduire la configuration d'origine.

### Test prudent

> **N'envoyez pas de test à une liste complète.**

1. Dans le Google Sheets des abonnés, ne laissez temporairement qu'une adresse de test que vous contrôlez, avec `actif` = `oui`.
2. Exécutez le nœud.
3. Vérifiez que l'email arrive.
4. Vérifiez le sujet, la mise en page et les liens.

### Connexion

`Get row(s) in sheet → Envoyer aux abonnés`

## 16. Vérifier le workflow complet

Voici l'ordre des nœuds sur la ligne principale :

```text
Schedule Trigger
    ↓
Lire sources
    ↓
Constantes
    ↓
RSS Read
    ↓
Filtrer
    ↓
Des articles ?
    └── true
         ↓
       Rédiger
         ↓
       Générer HTML
         ↓
       Get row(s) in sheet
         ↓
       Envoyer aux abonnés
```

Les connexions complémentaires, sur les ports dédiés, sont :

```text
OpenAI Chat Model ──(port AI Language Model)──> Rédiger

Structured Output Parser ──(port Output Parser)──> Rédiger
```

Dans le JSON d'origine, un nœud `Google Gemini Chat Model` est également présent, mais il n'est pas connecté à `Rédiger`. Il n'est donc pas nécessaire de le créer pour reproduire le fonctionnement effectif de ce workflow.

## 17. Tests, dans l'ordre

Testez les nœuds progressivement, avant d'exécuter l'ensemble.

1. **Lire sources :** des lignes contenant `rss` sont récupérées.
2. **Constantes :** `sujet` et `jours` sont présents.
3. **RSS Read :** les articles sont récupérés.
4. **Filtrer :** un objet contenant `articles` et `total` est renvoyé.
5. **Des articles ? :** la sortie `true` est utilisée si `total` est supérieur à zéro.
6. **Rédiger :** l'IA produit une sortie structurée.
7. **Générer HTML :** `html`, `objet`, `nb_articles` sont présents.
8. **Get row(s) in sheet :** seuls les abonnés actifs sont récupérés.
9. **Envoyer aux abonnés :** l'email de test est bien reçu.

## 18. Erreurs fréquentes

| Erreur | Cause possible | Solution |
|---|---|---|
| Google Sheets ne renvoie rien | Mauvais document ou feuille | Vérifiez le fichier sélectionné et les permissions |
| RSS Read échoue | URL RSS invalide ou inaccessible | Testez le flux dans un navigateur et vérifiez l'URL |
| Filtrer ne renvoie aucun article | Articles trop anciens, date/link absent ou flux vide | Vérifiez les données RSS et la période |
| Filtrer affiche une erreur | Nom de nœud ou champs incorrects | Vérifiez `Constantes`, `jours`, `isoDate`, `pubDate` et `link` |
| Des articles ? passe sur False | `total` vaut zéro | Vérifiez le résultat de `Filtrer` |
| Rédiger échoue | Credentials, modèle ou prompt mal configuré | Vérifiez la connexion API et le modèle disponible |
| Parser échoue | Exemple JSON invalide | Vérifiez le JSON et les propriétés |
| Générer HTML échoue | Sortie de l'IA inattendue ou référence incorrecte | Vérifiez la sortie de `Rédiger` et les noms de propriétés |
| Aucun abonné n'est récupéré | Filtre ou en-tête incorrect | Vérifiez `actif` et la valeur `oui` |
| Gmail échoue | Credential ou expression incorrecte | Vérifiez Gmail et les expressions |
| Email mal affiché | Compatibilité du client email | Testez dans différents clients |
| Emails envoyés à trop de destinataires | Plusieurs abonnés actifs | Vérifiez la feuille avant les tests |

## 19. Noms à copier-coller : récapitulatif

### Noms des nœuds

| Nœud | Nom exact |
|---|---|
| Trigger | `Schedule Trigger` |
| Google Sheets sources | `Lire sources` |
| Paramètres | `Constantes` |
| RSS | `RSS Read` |
| Code de filtrage | `Filtrer` |
| Condition | `Des articles ?` |
| Chaîne IA | `Rédiger` |
| Modèle IA | `OpenAI Chat Model` |
| Parser | `Structured Output Parser` |
| Code HTML | `Générer HTML` |
| Google Sheets abonnés | `Get row(s) in sheet` |
| Gmail | `Envoyer aux abonnés` |

### Colonnes Google Sheets

Sources :

```text
rss
```

Abonnés :

```text
email
actif
```

### Champs du nœud Constantes

```text
sujet
jours
```

### Propriétés de la sortie structurée

```text
objet
accroche
intro
articles
id
titre
resume
conclusion
```

> **Ne renommez pas ces éléments à votre convenance.** Les expressions et les codes font référence aux noms exacts. Si vous en changez un, vous devrez également modifier les expressions et les codes qui l'utilisent.

## 20. Activer le workflow

Quand tous les tests sont concluants :

1. Vérifiez les credentials Google Sheets, OpenAI et Gmail.
2. Vérifiez le fuseau horaire.
3. Vérifiez la feuille des abonnés et les destinataires.
4. Enregistrez le workflow.
5. Activez-le dans n8n.

> **Attention :** une fois activé, le workflow peut envoyer automatiquement des emails à tous les abonnés actifs. Ne l'activez pas tant que vous n'avez pas vérifié les destinataires et le contenu.

## 21. Idées d'amélioration

Une fois ce workflow de base fonctionnel, vous pouvez l'améliorer :

- Ajouter un email de validation avant l'envoi à tous les abonnés.
- Enregistrer les newsletters envoyées dans Google Sheets.
- Mettre en place un mécanisme de désabonnement réellement fonctionnel.
- Ajouter des notifications en cas d'erreur.
- Personnaliser le nombre d'articles.
- Proposer plusieurs sujets de newsletter.
- Ajouter une validation humaine.
- Mémoriser les articles déjà envoyés pour éviter les doublons entre newsletters.
- Ajouter un journal des envois.

## 22. Ce que vous avez appris

| Concept | Compétence acquise |
|---|---|
| Trigger | Déclencher automatiquement un workflow |
| Google Sheets | Lire des données d'un tableur |
| Expressions n8n | Réutiliser les données d'autres nœuds |
| RSS | Collecter des contenus |
| Code JavaScript | Filtrer et transformer des données |
| IF | Conditionner la suite d'un workflow |
| LLM | Utiliser une IA dans une automatisation |
| Structured Output Parser | Structurer la sortie d'une IA |
| HTML | Créer un email mis en forme |
| Gmail | Envoyer des emails automatiquement |
| Workflow | Relier plusieurs étapes pour automatiser une tâche |

**À retenir :** un workflow n8n est une suite de nœuds qui transmettent des données. La configuration de chaque nœud, les connexions, les noms des nœuds et les noms des champs sont tous importants. Respectez les noms indiqués et copiez les expressions et codes sans les modifier pour réussir la reproduction.
