# Arborea — Principes du projet

> Ce document est la référence. Toute décision de conception doit le respecter.
> Il sert aussi de mémoire entre les sessions de travail : à lire en premier.

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

- `index.html` — site vitrine.
- `mes-tarifs.html` — grille de prix (V1, local-first, multi-métier). **Prêt.**
- `devis-modele.html` — générateur de devis ; sait déjà se pré-remplir par
  paramètres d'URL (fondation du flux « voix → devis »).
- `facture-modele.html`, `tva-modele.html` — maquettes provisoires (cf. §4).

## Reste à construire

1. Backend serverless « voix → JSON » (transcription + extraction respectant la
   grille, sans mémoire). Seule brique qui a besoin d'un serveur (clés API).
2. Branchement note vocale → écran de vérif → `devis-modele.html`.
3. Plus tard : abonnements (Stripe), nom de domaine, branchement facturation API.
