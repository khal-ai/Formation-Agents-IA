# Guide complet : connecter n8n à Google Sheets, Drive et Gmail (OAuth)

> **Pour qui ?** Pour les débutants complets. Aucune connaissance de Google Cloud n'est nécessaire.
> **Durée :** 30 à 45 minutes la première fois.
> **Coût :** 0 €. Aucune carte bancaire n'est demandée. Si Google vous propose un « essai gratuit à 300 $ », **ignorez-le**.
> **Résultat :** un couple **Client ID / Client Secret** qui connecte n8n à Google Sheets, Drive et Gmail, et qui **ne s'expire pas au bout de 7 jours**.

> [!NOTE]
> Les écrans de Google Cloud changent souvent (noms de menus, emplacement des boutons). Ce guide donne les noms en **anglais** et leur équivalent **français** quand je le connais. Si un libellé diffère légèrement chez vous, cherchez le mot-clé le plus proche.

---

## Sommaire

1. [Comprendre en 2 minutes](#1-comprendre-en-2-minutes)
2. [Ce dont vous avez besoin](#2-ce-dont-vous-avez-besoin)
3. [Étape 1 : créer un projet Google Cloud](#étape-1--créer-un-projet-google-cloud)
4. [Étape 2 : activer les trois API](#étape-2--activer-les-trois-api)
5. [Étape 3 : configurer l'écran de consentement](#étape-3--configurer-lécran-de-consentement)
6. [Étape 4 : créer le Client ID et le Client Secret](#étape-4--créer-le-client-id-et-le-client-secret)
7. [Étape 5 : se déclarer comme utilisateur de test](#étape-5--se-déclarer-comme-utilisateur-de-test)
8. [Étape 6 : connecter n8n](#étape-6--connecter-n8n)
9. [Étape 7 : passer en production (clés permanentes)](#étape-7--passer-en-production-clés-permanentes)
10. [Dépannage](#dépannage)
11. [Sécurité : ne publiez jamais vos secrets](#sécurité--ne-publiez-jamais-vos-secrets)
12. [Checklist finale](#checklist-finale)
13. [Glossaire](#glossaire)

---

## 1. Comprendre en 2 minutes

### Pourquoi tout cela ?

Quand n8n veut lire votre Google Sheet ou envoyer un email depuis votre Gmail, Google doit être sûr que **vous êtes d'accord**. Google ne vous laisse pas simplement donner votre mot de passe à n8n. Il utilise un système appelé **OAuth**.

**Analogie :** imaginez que vous confiez un badge d'accès à un livreur. Vous ne lui donnez pas les clés de votre maison (votre mot de passe) : vous lui remettez un badge qui ouvre seulement la porte du garage, et que vous pouvez désactiver à tout moment. OAuth fonctionne de la même façon.

### Les mots à connaître

| Mot | Ce que c'est |
|---|---|
| **Projet Google Cloud** | Un « dossier » dans lequel vous rangez vos réglages Google. Il est gratuit. |
| **API** | Une « porte d'accès » à un service Google (Sheets, Gmail, Drive). Il faut l'ouvrir (« activer ») avant de s'en servir. |
| **Écran de consentement** | La page que Google vous montre pour demander « Autorisez-vous cette application ? ». |
| **Client ID** | L'identifiant public de votre application (comme un nom d'utilisateur). |
| **Client Secret** | Le mot de passe de votre application. **À garder confidentiel.** |
| **URI de redirection** | L'adresse où Google renvoie n8n une fois que vous avez cliqué sur « Autoriser ». |

### Une seule application pour trois services

Vous créez **un seul** projet et **un seul** couple Client ID / Client Secret. Vous le réutilisez pour **Google Sheets** et **Gmail**. Google Drive est utilisé par n8n en coulisses pour lister vos fichiers : il suffit de l'activer.

> [!IMPORTANT]
> **Gemini ne passe pas par OAuth.** Il utilise une **clé API** distincte, créée gratuitement sur [aistudio.google.com](https://aistudio.google.com) (bouton *Get API key*). Ce guide ne couvre pas cette clé.

---

## 2. Ce dont vous avez besoin

- [ ] Un **compte Google (Gmail)**. Ce sera le compte qui enverra les emails et qui possède le Google Sheet.
- [ ] **n8n** déjà installé et ouvert dans votre navigateur.
- [ ] Un navigateur dans lequel vous êtes connecté avec le bon compte Google.
- [ ] **Une deuxième fenêtre de navigateur** : une pour la console Google Cloud, une pour n8n. Vous allez copier-coller entre les deux.

> [!TIP]
> Si vous avez plusieurs comptes Google, vérifiez en haut à droite de chaque page que c'est **le bon compte** qui est connecté. C'est la cause d'erreur la plus fréquente chez les débutants.

---

## Étape 1 : créer un projet Google Cloud

1. Allez sur **<https://console.cloud.google.com>** et connectez-vous.
2. Si Google vous demande d'accepter les conditions d'utilisation, cochez la case et validez.
3. **Ignorez** toute proposition d'« essai gratuit » ou d'activation de la facturation.
4. En haut de la page, à côté du logo « Google Cloud », cliquez sur le **sélecteur de projet** (il affiche « Select a project » ou le nom d'un projet existant).
5. Dans la fenêtre qui s'ouvre, cliquez sur **New project** (*Nouveau projet*).
6. **Project name** : saisissez `n8n-newsletter` (ou le nom de votre choix).
7. Laissez **Organization / Location** par défaut, puis cliquez sur **Create** (*Créer*).
8. Patientez quelques secondes, puis **sélectionnez le projet** : cliquez de nouveau sur le sélecteur en haut et choisissez `n8n-newsletter`.

✅ **Vous devez voir** le nom `n8n-newsletter` en haut de la page, à côté du logo Google Cloud.

> [!WARNING]
> Pour toutes les étapes suivantes, vérifiez que **ce nom est bien affiché en haut**. Si c'est un autre projet, vous activerez les API au mauvais endroit.

---

## Étape 2 : activer les trois API

Une API est une porte d'accès. On ouvre ici trois portes : **Google Sheets**, **Gmail** et **Google Drive**.

1. Dans le menu de gauche (icône ☰ en haut à gauche), allez dans **APIs & Services** (*API et services*), puis **Library** (*Bibliothèque*).
2. Dans la barre de recherche, tapez **`Google Sheets API`** et cliquez sur le résultat.
3. Cliquez sur le bouton bleu **Enable** (*Activer*). Attendez que la page se recharge.
4. Revenez à la bibliothèque (flèche de retour ou menu ☰ → *APIs & Services* → *Library*).
5. Recommencez avec **`Gmail API`** → **Enable**.
6. Recommencez avec **`Google Drive API`** → **Enable**.

✅ **Vérification :** menu ☰ → *APIs & Services* → **Enabled APIs & services** (*API et services activés*). Vous devez voir les trois API dans la liste.

> [!NOTE]
> **Raccourci possible.** Vous pouvez aussi coller ces adresses dans votre navigateur. Elles ouvrent directement la page de chaque API pour le projet sélectionné :
> `https://console.cloud.google.com/apis/library/sheets.googleapis.com`,
> `https://console.cloud.google.com/apis/library/gmail.googleapis.com` et
> `https://console.cloud.google.com/apis/library/drive.googleapis.com`.

---

## Étape 3 : configurer l'écran de consentement

C'est la page que Google affichera quand vous cliquerez sur « Sign in with Google » dans n8n. Il faut lui donner un nom et une adresse de contact.

L'interface actuelle s'appelle **Google Auth Platform**. Si vous ne la trouvez pas dans le menu, tapez **`Google Auth Platform`** dans la barre de recherche en haut de la console.

1. Ouvrez **Google Auth Platform**.
2. Si vous voyez un bouton **Get started** (*Commencer*), cliquez dessus. Un assistant s'ouvre en quatre parties :
   1. **App information** (*Informations sur l'application*)
      - **App name** : `n8n newsletter`
      - **User support email** : sélectionnez votre adresse Gmail.
   2. **Audience** (*Public cible*)
      - Choisissez **External** (*Externe*) si vous utilisez un compte Gmail personnel.
      - Choisissez **Internal** (*Interne*) uniquement si vous avez un compte **Google Workspace** (professionnel ou scolaire) et que tous vos utilisateurs sont dans la même organisation.
   3. **Contact information** (*Coordonnées*) : saisissez votre adresse email (Google y envoie les notifications importantes).
   4. **Finish** (*Terminer*) : cochez la case d'acceptation de la politique des données utilisateur, puis cliquez sur **Continue** puis **Create**.

✅ **Vous devez voir** le tableau de bord *Google Auth Platform*, avec dans le menu de gauche : **Overview, Branding, Audience, Clients, Data Access**.

> [!IMPORTANT]
> **Pourquoi « Internal » change tout.** Avec un compte Workspace et le type **Internal**, il n'y a **pas de mode Testing** et **pas d'expiration à 7 jours** : vous pouvez sauter l'étape 7. Seuls les comptes de votre organisation pourront se connecter.

---

## Étape 4 : créer le Client ID et le Client Secret

C'est l'étape la plus importante. Pour la réussir, il faut d'abord **récupérer l'adresse de redirection dans n8n**.

### 4.1 Récupérer l'URI de redirection dans n8n

1. Dans **n8n**, ouvrez un workflow (ou créez-en un).
2. Ajoutez un nœud **Google Sheets** (*Get row(s) in sheet*), ou ouvrez-en un existant.
3. Dans le champ **Credential to connect with**, cliquez sur **Create new credential**.
4. Une fenêtre s'ouvre. Repérez le champ **OAuth Redirect URL**. Il ressemble à :
   - n8n en local : `http://localhost:5678/rest/oauth2-credential/callback`
   - n8n en ligne : `https://votre-instance.app.n8n.cloud/rest/oauth2-credential/callback`
5. **Copiez cette adresse** avec l'icône de copie. **Laissez cette fenêtre ouverte** : vous y reviendrez.

> [!WARNING]
> **Ne retapez pas cette adresse à la main.** Une seule lettre de différence et Google refusera la connexion (erreur `redirect_uri_mismatch`). Utilisez toujours le copier-coller.

### 4.2 Créer le client dans Google Cloud

1. Dans la console Google Cloud, ouvrez **Google Auth Platform → Clients** (*Clients*).
   *(Alternative : menu ☰ → APIs & Services → Credentials → Create credentials → OAuth client ID.)*
2. Cliquez sur **Create client** (*Créer un client*).
3. **Application type** (*Type d'application*) : choisissez **Web application** (*Application Web*).
4. **Name** : `n8n client`.
5. Descendez jusqu'à **Authorized redirect URIs** (*URI de redirection autorisés*) et cliquez sur **Add URI** (*Ajouter un URI*).
6. **Collez** l'adresse copiée à l'étape 4.1.
7. Laissez **Authorized JavaScript origins** vide.
8. Cliquez sur **Create** (*Créer*).

### 4.3 Copier le Client ID et le Client Secret

Une fenêtre affiche vos deux valeurs :

- **Client ID** : une longue chaîne qui se termine par `.apps.googleusercontent.com`
- **Client secret** : une chaîne plus courte, souvent commençant par `GOCSPX-`

**Copiez-les immédiatement** dans un endroit sûr (gestionnaire de mots de passe, fichier local **hors de GitHub**).

> [!WARNING]
> **Google peut ne plus réafficher le secret après la fermeture de cette fenêtre.** Si vous l'avez perdu, ce n'est pas grave : dans la page *Clients*, ouvrez votre client et créez un **nouveau secret** (*Add secret*). Il faudra alors mettre à jour le credential dans n8n.

✅ **Vous devez voir** votre client `n8n client` listé dans la page *Clients*.

---

## Étape 5 : se déclarer comme utilisateur de test

Tant que l'application est en mode **Testing**, Google n'autorise que les adresses de votre liste. Il faut donc vous y ajouter vous-même.

1. Ouvrez **Google Auth Platform → Audience** (*Public cible*).
2. Repérez la section **Test users** (*Utilisateurs test*) et cliquez sur **Add users** (*Ajouter des utilisateurs*).
3. Saisissez **votre adresse Gmail** (celle qui se connectera), puis **Save** (*Enregistrer*).

> [!TIP]
> Si vous formez d'autres personnes, **chacune** doit créer son propre projet avec son propre compte. Elle s'ajoute alors elle-même comme utilisateur de test. Il n'y a pas besoin de gérer une liste commune.

---

## Étape 6 : connecter n8n

Vous allez créer **deux credentials** dans n8n (un pour Sheets, un pour Gmail) avec **le même** Client ID et **le même** Client Secret.

### 6.1 Credential Google Sheets

1. Revenez à la fenêtre n8n laissée ouverte à l'étape 4.1 (*Google Sheets OAuth2 API*).
2. Collez le **Client ID** et le **Client Secret**.
3. Cliquez sur **Sign in with Google**. Une fenêtre Google s'ouvre.
4. Choisissez **votre compte**.
5. Google affiche probablement : **« Google n'a pas validé cette application »**. **C'est normal** pour un projet personnel. Cliquez sur **Advanced** (*Avancé*), puis sur **Go to n8n newsletter (unsafe)** (*Accéder à n8n newsletter*).
6. Google liste les accès demandés. Cochez toutes les cases proposées, puis cliquez sur **Continue** (*Continuer*).
7. La fenêtre se ferme.

✅ **Vous devez voir** dans n8n : **« Account connected »** et le credential passe en **vert**. Cliquez sur **Save**.

### 6.2 Credential Gmail

1. Ajoutez un nœud **Gmail** (par exemple *Send a message*).
2. **Credential to connect with** → **Create new credential** (*Gmail OAuth2 API*).
3. Collez **le même Client ID et le même Client Secret**.
4. L'**OAuth Redirect URL** affichée est **identique** à celle de l'étape 4.1 : elle est déjà enregistrée chez Google.
5. Cliquez sur **Sign in with Google** et refaites les mêmes étapes d'autorisation (compte → *Advanced* → *Go to…* → *Continue*).

✅ **Vous devez voir** « Account connected » en vert.

> [!NOTE]
> **Google Drive** n'a pas besoin de credential séparé pour le nœud Sheets : n8n s'en sert automatiquement à partir de l'API activée à l'étape 2. Si un jour vous utilisez le nœud *Google Drive*, créez un troisième credential avec les mêmes identifiants.

### Test rapide

Dans le nœud Sheets, choisissez *Document → From list* : votre liste de Google Sheets doit s'afficher. Si c'est le cas, la connexion fonctionne.

---

## Étape 7 : passer en production (clés permanentes)

### Le problème

Tant que l'application est en mode **Testing**, Google **invalide l'autorisation au bout de 7 jours** : n8n affiche alors une erreur (`invalid_grant` ou `Token has been expired or revoked`) et il faut cliquer à nouveau sur *Sign in with Google*.

Pour un projet qui dure plusieurs semaines, ce n'est pas pratique. La solution est de passer l'application en **« In production »** (*En production*).

### Ce que change la mise en production (et ce que ça ne change pas)

| | Testing | In production |
|---|---|---|
| Expiration de l'autorisation | **7 jours** | **Pas d'expiration** automatique |
| Utilisateurs autorisés | Seulement ceux de la liste *Test users* | Tout compte Google |
| Écran « application non validée » | Oui | **Oui, il reste** (normal) |
| Validation par Google (« verification ») | Non requise | **Non requise pour un usage personnel** |

> [!NOTE]
> Une application non validée par Google est limitée à environ **100 utilisateurs**. Pour un usage personnel ou une formation où chacun crée son propre projet, cette limite n'est jamais atteinte.

### 7.1 Compléter la page Branding (obligatoire avant de publier)

Le bouton **Publish app** reste **grisé** tant que ces informations manquent. Ouvrez **Google Auth Platform → Branding** (*Image de marque*) et renseignez :

| Champ | Quoi mettre |
|---|---|
| **App name** | `n8n newsletter` |
| **User support email** | votre adresse Gmail |
| **Application home page** | l'adresse d'une page web publique (voir 7.2) |
| **Application privacy policy link** | l'adresse d'une page web publique de politique de confidentialité (voir 7.2) |
| **Authorized domains** | le domaine de ces pages (par exemple `github.io`), si Google le demande |
| **Developer contact information** | votre adresse Gmail |

Cliquez sur **Save**. Vérifiez qu'**aucun message d'erreur rouge ou orange** n'apparaît sous les champs.

### 7.2 Créer les deux pages web (gratuit, sans compétence technique)

Google exige une page d'accueil et une page « confidentialité » pour toute application publiée. Ce sont deux petites pages publiques : n'importe quel texte simple convient. Choisissez **l'une** des deux méthodes.

**Méthode A : Google Sites (la plus simple).**

1. Allez sur **<https://sites.google.com>** et créez un site vide intitulé `n8n newsletter`.
2. Sur la page d'accueil, écrivez une ligne de présentation, par exemple : *« Outil personnel d'automatisation de newsletter. »*
3. Ajoutez une deuxième page `Confidentialité` et collez le texte modèle ci-dessous.
4. Cliquez sur **Publish** (*Publier*) et choisissez une adresse. Vous obtenez des adresses de la forme `https://sites.google.com/view/votre-site`.

**Méthode B : GitHub Pages.**

1. Dans votre dépôt GitHub, créez un fichier `index.md` (page d'accueil) et un fichier `confidentialite.md` (texte ci-dessous).
2. Dans le dépôt : **Settings → Pages**, choisissez la branche à publier (par exemple `main`) et le dossier racine, puis **Save**.
3. Après une ou deux minutes, vos pages sont accessibles à `https://votre-nom.github.io/nom-du-depot/`.

> [!WARNING]
> **Si Google refuse l'enregistrement** à cause du domaine (message du type « domain not authorized » ou « not registered »), essayez l'autre méthode. Google applique parfois des règles différentes selon le domaine utilisé. Si le message persiste, copiez-le tel quel dans une recherche : c'est la piste la plus rapide.

**Texte modèle pour la page de confidentialité :**

```text
Politique de confidentialité — n8n newsletter

Cette application est un outil personnel d'automatisation de newsletter.

Données consultées : elle accède uniquement aux fichiers Google Sheets
et à la messagerie Gmail du propriétaire du compte Google qui l'autorise,
afin de lire une liste d'abonnés et d'envoyer des emails.

Partage : aucune donnée n'est partagée avec des tiers.

Conservation : aucune donnée n'est conservée en dehors de l'instance n8n
de l'utilisateur.

Suppression : l'utilisateur peut retirer l'accès à tout moment depuis
https://myaccount.google.com/permissions

Contact : votre-adresse@gmail.com
```

### 7.3 Publier l'application

1. Ouvrez **Google Auth Platform → Audience** (*Public cible*).
2. Dans la section **Publishing status** (*État de la publication*), vous voyez **Testing**.
3. Cliquez sur **Publish app** (*Publier l'application*), puis confirmez dans la fenêtre qui s'ouvre.

✅ **Vous devez voir** l'état passer à **In production** (*En production*).

> [!IMPORTANT]
> **Si le bouton « Publish app » est absent ou grisé**, voici les causes les plus fréquentes, de la plus probable à la moins probable :
> 1. **Rechargez la page** (`Ctrl+F5`) après avoir enregistré *Branding*.
> 2. Un **champ obligatoire** de *Branding* est vide, ou son enregistrement a échoué (relisez les messages sous les champs).
> 3. Le **domaine** de vos pages n'est pas accepté : voir l'avertissement du 7.2.
> 4. Votre type d'utilisateur est **Internal** (compte Workspace) : il n'y a alors **ni bouton ni expiration**. Rien à faire.
> 5. L'état affiche déjà **In production** : c'est terminé. Le bouton disponible s'appelle alors *Back to testing*. **Ne l'utilisez pas.**

### 7.4 Reconnecter n8n une dernière fois

Une autorisation obtenue **avant** la publication peut garder son expiration à 7 jours. Il faut donc en créer une nouvelle **après** la publication :

1. Dans n8n, ouvrez votre credential **Google Sheets**.
2. Cliquez de nouveau sur **Sign in with Google** et refaites l'autorisation (compte → *Advanced* → *Go to…* → *Continue*).
3. Faites de même pour le credential **Gmail**.
4. Enregistrez.

✅ **C'est terminé** : ces autorisations n'expireront plus toutes seules.

> [!TIP]
> **Ordre idéal pour une nouvelle installation (le plus rapide) :** projet → API → écran de consentement → Client ID/Secret → *Branding* → **publier l'application** → et seulement **ensuite** créer les credentials dans n8n. Vous ne vous authentifiez ainsi qu'une seule fois.

---

## Dépannage

| Message ou symptôme | Cause probable | Solution |
|---|---|---|
| `redirect_uri_mismatch` (Erreur 400) | L'URI de redirection de Google ne correspond pas exactement à celle de n8n | Étape 4.2 : supprimez l'URI saisi et **recollez** celui affiché par n8n. Pas d'espace, pas de `/` en trop. |
| `access_denied` / « Accès bloqué : n8n newsletter n'a pas terminé la procédure de validation » | Votre adresse n'est pas dans *Test users* (mode Testing) | Étape 5 : ajoutez votre adresse. |
| `Gmail API has not been used in project …` ou `API has not been enabled` | L'API n'est pas activée dans le bon projet | Étape 2 : activez l'API **dans le projet sélectionné** (vérifiez le nom en haut). |
| `invalid_grant` ou `Token has been expired or revoked` | L'autorisation a expiré (7 jours en mode Testing), ou a été révoquée | Étape 7.3 pour publier, puis 7.4 pour se reconnecter. |
| `invalid_client` ou `The OAuth client was not found` | Client ID ou Secret mal copié, ou client supprimé | Recopiez les deux valeurs **sans espace** au début ou à la fin. Au besoin, créez un nouveau secret. |
| Le bouton **Publish app** est grisé | Informations de *Branding* incomplètes | Étape 7.1. Voir aussi l'encadré du 7.3. |
| Une page blanche ou une erreur s'affiche après « Sign in with Google » | Bloqueur de fenêtres pop-up | Autorisez les pop-ups pour l'adresse de n8n, puis recommencez. |
| Vous êtes connecté au mauvais compte Google | Plusieurs comptes ouverts dans le navigateur | Utilisez une **fenêtre de navigation privée**, connectez-vous uniquement avec le bon compte. |
| Impossible de lister vos Google Sheets | API **Google Drive** non activée | Étape 2 : activez *Google Drive API*. |
| « Google n'a pas validé cette application » | **Message normal** pour un projet personnel | *Advanced* → *Go to n8n newsletter*. |

---

## Sécurité : ne publiez jamais vos secrets

> [!CAUTION]
> Ce guide sera peut-être hébergé dans un dépôt **public**. Respectez ces règles pour ne pas exposer votre compte Google.

- **Ne mettez jamais votre Client Secret dans un fichier de dépôt GitHub** (ni dans un `.md`, ni dans un export de workflow n8n, ni dans une capture d'écran).
- **Masquez** le Client ID et le Secret sur toute capture d'écran partagée.
- **Les exports de workflow n8n** (`.json`) ne contiennent normalement pas les secrets des credentials, mais **vérifiez** avant de publier.
- **Si un secret a été exposé** : dans *Google Auth Platform → Clients*, ouvrez votre client, **créez un nouveau secret**, **désactivez ou supprimez l'ancien**, puis mettez à jour n8n.
- **Pour retirer l'accès d'une application** à votre compte Google : <https://myaccount.google.com/permissions>.
- Gardez votre secret dans un **gestionnaire de mots de passe**, pas dans un fichier texte sur le bureau.

---

## Checklist finale

Cochez au fur et à mesure.

**Préparation**
- [ ] Projet `n8n-newsletter` créé et **sélectionné** en haut de la console
- [ ] **Google Sheets API**, **Gmail API** et **Google Drive API** activées

**Écran de consentement**
- [ ] Application configurée dans *Google Auth Platform* (nom, email, type **External**)
- [ ] Votre adresse ajoutée dans **Test users**

**Identifiants**
- [ ] Client créé (type **Web application**) avec l'URI de redirection **copiée depuis n8n**
- [ ] Client ID et Client Secret **sauvegardés** dans un endroit sûr

**Connexion n8n**
- [ ] Credential **Google Sheets** : vert (« Account connected »)
- [ ] Credential **Gmail** : vert (« Account connected »)
- [ ] Liste de vos Google Sheets visible dans le nœud Sheets

**Clés permanentes**
- [ ] Page *Branding* complète (nom, email, page d'accueil, politique de confidentialité, domaine)
- [ ] Application **publiée** : état **In production**
- [ ] Credentials n8n **reconnectés après la publication**

---

## Glossaire

| Terme | Définition |
|---|---|
| **API** | Interface qui permet à un programme (n8n) d'utiliser un service (Gmail, Sheets). |
| **Authorized redirect URI** | Adresse où Google renvoie l'utilisateur après l'autorisation. Elle doit être **identique** à celle de n8n. |
| **Client ID / Client Secret** | Identifiant et « mot de passe » de votre application auprès de Google. |
| **Consent screen** | Écran qui demande à l'utilisateur d'autoriser l'application. |
| **Credential (n8n)** | Réglage enregistré dans n8n qui contient les informations de connexion à un service. |
| **External / Internal** | Types de public : *External* pour tout compte Google, *Internal* pour les comptes d'une organisation Workspace. |
| **OAuth 2.0** | Système d'autorisation qui évite de donner son mot de passe à une application. |
| **Publishing status** | État de l'application : **Testing** (liste restreinte, expire à 7 jours) ou **In production**. |
| **Scope (portée)** | Permission précise demandée (par exemple « envoyer des emails »). n8n les demande automatiquement. |
| **Test users** | Liste des comptes autorisés tant que l'application est en mode Testing. |
| **Token** | « Badge » temporaire que Google délivre à n8n après votre autorisation. |

---

*Guide rédigé en novembre 2026 pour un usage avec n8n. Les écrans de Google Cloud évoluent régulièrement : en cas de différence, fiez-vous au sens des étapes plutôt qu'à la position exacte des boutons.*
