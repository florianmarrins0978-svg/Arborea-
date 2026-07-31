# Arborea — Principes du projet

> Ce document est la référence. Toute décision de conception doit le respecter.
> Il sert aussi de mémoire entre les sessions de travail : à lire en premier.

## Mode de travail : autonomie complète

Le patron a délégué l'exécution technique de bout en bout (le 2026-07-31).
Concrètement : construire → committer → ouvrir la Pull Request → **fusionner
soi-même** → vérifier la mise en ligne → prévenir « c'est publié ». Pas de
demande de confirmation à chaque étape. Le patron garde un droit de regard
total (tout reste visible et réversible sur GitHub).

Après chaque PR fusionnée, repartir de `main` à jour pour la suite (ne pas
empiler sur une branche déjà fusionnée).

## Passation entre sessions — état au 2026-07-31

À lire en premier dans une nouvelle session pour reprendre sans rien réapprendre.

**Séparation appli / site vitrine (décidé).**
- Le **site vitrine** (pour les clients) reste dans le dépôt **`Arborea-`**
  (`index.html`, `index-classic.html`, `Le vrai .html`, `images/`).
- L'**appli du patron** doit vivre dans son **propre dépôt : `atlas-app`**
  (`github.com/florianmarrins0978-svg/atlas-app`), avec sa propre adresse
  `https://florianmarrins0978-svg.github.io/atlas-app/`.

**Migration à faire (dès qu'une session a accès à `atlas-app`).**
Copier vers `atlas-app` UNIQUEMENT l'appli :
`app.html`, `devis-vocal.html`, `devis-modele.html`, `facture-modele.html`,
`tva-modele.html`, `mes-tarifs.html`, `nav.js`, `PRINCIPES.md`, `CAPACITOR.md`,
`package.json`, `capacitor.config.json`, `scripts/`, `tests/`, `.gitignore`
— **plus** un `index.html` racine qui redirige vers `app.html`, **plus** le
workflow Pages (`.github/workflows/pages.yml`, déclenché sur `main`).
**Ne pas** copier le site vitrine (`index.html` vitrine, `index-classic.html`,
`Le vrai .html`, `images/`). Puis retirer l'appli de `Arborea-` (source unique).

**Nom du produit — question ouverte.** Le dépôt s'appelle « atlas-app » ;
l'interface affiche encore « Arborea ». À confirmer avec le patron : renommer
l'appli à l'écran en « Atlas », ou garder « Arborea » dans l'appli ?

**Objectif : App Store + Google Play.** La coque Capacitor est déjà en place
(cf. `CAPACITOR.md`). Le web reste la version de test (lien instantané) ; les
apps natives se buildent depuis le même code, le jour venu (Mac + Apple
Developer 99 $/an ; Android Studio + Google Play 25 $).

**État V1 (fait et testé, 52 tests e2e au vert).**
Dictée (transcription navigateur) → devis (depuis la grille de tarifs) → PDF →
**envoi au client** (partage natif mobile / e-mail sur PC). Grille de tarifs,
navigation commune, identité charbon/pin, coque Capacitor iOS+Android scaffoldée.

**Reste à faire.**
1. **Dictée intelligente (Claude)** pour comprendre toutes les tournures.
   Nécessite une clé API. Modèle commercial confirmé : **une seule clé, côté
   serveur du vendeur** (jamais chez les patrons) ; les patrons paient un
   **abonnement** (Stripe) ; le backend reste **sans mémoire** des clients des
   patrons. Point de branchement prêt : `extractDevis()` dans `devis-vocal.html`.
2. Micro natif dans l'app (plugin `@capacitor-community/speech-recognition`).
3. Plus tard : abonnements, domaine, branchement facturation par API.

## La valeur du produit

Le cœur, c'est le maillon **chantier → devis** : le patron enregistre une note
vocale sur place, et l'assistant rédige le devis tout seul. Personne ne fait
bien ça aujourd'hui. Tout le reste (facturation, TVA, calendrier) est secondaire
et ne doit **pas** être recodé à la main.

Parcours V1 :
1. Note vocale enregistrée sur le chantier.
2. Transcription + extraction structurée (client, adresse, prestations, durée, moyens).
3. Écran de vérification : le patron corrige en 30 secondes.
4. Devis PDF généré et envoyé par mail.

Calendrier, validation de fin de chantier et facture viennent en V2/V3.

## Principes non négociables

### 1. L'IA n'invente RIEN

C'est la règle absolue. L'assistant ne fabrique jamais une information.

- **Jamais un prix.** Les prix viennent uniquement de la grille « Mes tarifs »
  remplie par le patron. Si un moyen n'est pas dans la grille, le prix reste
  **vide et signalé en rouge** sur l'écran de vérif — jamais deviné. Le patron
  l'ajoute une fois, c'est mémorisé.
- **Jamais une donnée client.** Nom, adresse, téléphone : uniquement ce qui est
  dit dans la note vocale. Un champ absent reste vide (« à compléter »), jamais
  inventé.
- **Jamais une prestation.** Seulement ce qui a été dit. En cas d'ambiguïté,
  l'assistant signale au lieu de supposer.
- **Traçabilité.** Chaque ligne extraite doit pouvoir être rattachée à ce que le
  patron a réellement dit et à une entrée réelle de la grille.
- **L'écran de vérification est obligatoire.** Rien ne part avant validation
  humaine.

Concrètement : le modèle d'extraction ne produit que des données ancrées dans
(a) la transcription et (b) la grille de tarifs. Tout ce qui n'y figure pas est
marqué « à compléter », pas complété.

### 2. Cloisonnement des données

- L'app est **local-first** : la grille de tarifs et les données des clients du
  patron restent sur son appareil (`localStorage`), jamais sur nos serveurs.
- **Le vendeur (nous) n'accède jamais aux clients de ses patrons.** Nous ne
  connaissons que nos abonnés (les patrons).
- Le futur backend « voix → JSON » sera **sans mémoire** : il traite l'audio,
  renvoie le texte structuré, puis oublie tout.

### 3. Multi-métier par construction

La grille de tarifs est **générique** : postes libres + unités libres (jour,
heure, forfait, m², ml…). Aucun champ spécifique à l'arboriculture. Un plombier
ou un maçon utilise le même moteur avec son propre vocabulaire.

### 4. Ne pas recoder la facturation ni la TVA légale

La facturation électronique conforme (numérotation inviolable, archivage,
Factur-X/UBL/CII, obligations 2026/2027) est un projet à part et un risque
juridique. On s'y branche par **API** sur un outil existant (Pennylane, Evoliz,
Tiime, Sellsy, Abby…) en V2/V3. Les écrans `facture-modele.html` et
`tva-modele.html` sont des maquettes **provisoires**, destinées à être
remplacées par ce branchement.

## Direction artistique

Minimaliste, luxueux, très vendeur, très simple pour le patron. Identité déjà
posée : palette crème/os/forêt (`--bone #f5f3ee`, `--pine #2f3b2f`,
`--clay #b5562a`), Playfair Display (titres) + Inter (texte). Beaucoup d'espace,
gestes évidents, utilisable au téléphone.

## État des fichiers

- `index.html` — **site vitrine** (pour les clients). Volontairement **séparé
  de l'appli** : l'appli ne renvoie jamais vers lui.
- `app.html` — **porte d'entrée de l'appli** (espace pro du patron). Tableau de
  bord : Nouveau devis (dictée), Mes tarifs, Devis, Factures, TVA. C'est le lien
  à ouvrir/épingler. Le logo de la nav pointe ici (plus vers la vitrine).
- `mes-tarifs.html` — grille de prix (V1, local-first, multi-métier). **Prêt.**
- `devis-vocal.html` — **cœur du produit.** Dictée (transcription in-browser via
  l'API Web Speech, gratuite, sans clé) → extraction rule-based rapprochée de la
  grille (aucun prix inventé : un moyen absent est signalé « à définir ») →
  aperçu → passage au devis pré-rempli. La fonction `extractDevis()` est le point
  de branchement pour une extraction par LLM (Claude) plus tard.
- `devis-modele.html` — générateur de devis ; se pré-remplit par paramètres
  d'URL, par un **devis vocal** (`arborea_voice_devis`), et via le bouton
  **« ＋ Depuis mes tarifs »** (prix issu de la grille, TVA par défaut reprise) ;
  bouton « Valider → créer la facture ».
- `facture-modele.html`, `tva-modele.html` — maquettes provisoires (cf. §4).
- `nav.js` — barre de navigation partagée, incluse par tous les écrans outils
  (Nouveau devis · Devis · Factures · TVA · Mes tarifs). Ajouter un écran = une
  ligne d'include.

Connexions en place : Site → Devis, Voix → Devis, Devis → Facture,
Mes tarifs → Devis, et une navigation commune entre les écrans.

## Reste à construire

1. **Extraction par LLM (Claude).** L'extraction actuelle est rule-based (bonne
   pour démarrer, sans clé). Pour comprendre finement le langage naturel du
   chantier, la brancher sur Claude via un petit backend serverless sans
   mémoire. Nécessite une clé API (côté patron/abonnement). Point de branchement
   déjà prêt : `extractDevis()` dans `devis-vocal.html`.
2. Envoi du devis par mail (aujourd'hui : PDF à télécharger puis joindre).
3. Plus tard : abonnements (Stripe), nom de domaine, branchement facturation API
   (Pennylane, Evoliz, Tiime…), calendrier et validation de fin de chantier.
