# AGENTS.md

Guidance for working in this repository. Project language: **French** (code comments, UI, docs). Keep it that way unless asked otherwise.

## Le projet en une phrase

Un **jeu web pour apprendre Docker sans rien installer** : Docker est entièrement **simulé côté serveur** et représenté visuellement par un **port de commerce** (bateaux = conteneurs, plans = images, quais = ports exposés, coffres = volumes, canaux = réseaux). Inspiré de Flexbox Froggy et Learn Git Branching, appliqué à Docker.

Public cible : étudiant·es en informatique et développeur·ses juniors (18–30 ans), niveau Docker débutant à intermédiaire. Sessions de 10 à 30 minutes.

## Stack technique (imposée)

| Couche | Techno |
| --- | --- |
| Front | **Nuxt** (Vue 3), scène 2D en grille, transitions CSS + `<Transition>` de Vue (pas de 3D) |
| Back | **Express en TypeScript** |
| BDD | **MongoDB** (comptes, progression, étoiles, scores/temps) |
| Auth | Session par **jeton (JWT)**, mot de passe chiffré (bcrypt) |

Contraintes fortes :
- **Aucun vrai Docker n'est exécuté.** Toutes les commandes sont simulées par un simulateur dans Express. Aucun risque sécurité, aucun coût d'hébergement, comportement identique pour tous.
- **Validation côté serveur** : objectifs de niveau, scores et temps des épreuves sont calculés par le serveur (anti-triche sur le classement). Le front ne fait qu'afficher l'état du port qu'il reçoit.
- **Périmètre réaliste sur 2 mois.**

## Architecture

```
Navigateur (Nuxt)  ──commande──▶  API Express  ──▶  Simulateur Docker (état fictif)
      ▲                                │                     │
      └────── état du port (JSON) ◀────┴── vérifie l'objectif du niveau
                                             enregistre la progression ▶ MongoDB
```

Flux d'un niveau :
1. Un client arrive avec une demande (bulle de dialogue).
2. Le joueur tape des commandes dans un terminal simulé (ou écrit un Dockerfile / docker-compose.yml à partir du chapitre 6).
3. Le serveur analyse la commande, met à jour l'état du port, renvoie résultat ou **erreur réaliste**.
4. La scène s'anime (bateau apparaît, passerelle se pose, bateau coule en cas d'erreur).
5. Objectif atteint → **1 à 3 étoiles** selon essais et indices utilisés.

## Principe clé : les niveaux sont des données

Chaque niveau est un **document** (état de départ, consigne, dialogues, conditions de réussite, indices). **Ajouter un niveau ne doit pas demander de nouveau code.** Le simulateur et le moteur de niveaux lisent ces données. Respecter ce principe dans toute évolution.

Le simulateur tient un état fictif : images, conteneurs (avec statut en marche/arrêté), ports, volumes, réseaux. Il comprend un **sous-ensemble** de commandes Docker et doit produire des erreurs réalistes (ex. conflit de port, conteneur introuvable).

## Mapping Docker → jeu

| Notion Docker | Représentation |
| --- | --- |
| Hôte (machine) | Le port et sa capitainerie |
| Image | Un plan de bateau dans l'entrepôt |
| Conteneur | Un bateau construit depuis un plan, avec son état |
| Port exposé (`-p`) | Un quai relié au bateau par une passerelle |
| Volume | Un coffre à quai qui reste quand le bateau part |
| Réseau Docker | Un canal privé entre bateaux, invisible de la ville |
| Dockerfile | Le chantier naval |
| docker-compose | La flotte décrite sur un ordre de mission |
| Swarm, registre | L'amirauté et ses flottes |

## Contenu : campagne (~35 niveaux, 3 stages)

Stage 1 « Les bases » (ch. 0–7), Stage 2 « Creuser » (ch. 8), Stage 3 « Vers le staging » (ch. 9). Chapitres débloqués dans l'ordre. Indices en 3 paliers (piste → commande → solution), chacun réduit les étoiles. Commande `help` partout + aide terminal pour débutant·es.

Autres modes : **épreuves chronométrées** (situation cassée à réparer, classement par épreuve, score serveur, pénalité de temps par indice) et **achievements**.

## Priorisation du rendu (MVP)

**Indispensable** — c'est ce qu'on livre d'abord, un jeu complet et jouable :
- Inscription / connexion
- Terminal simulé + simulateur des commandes de base
- Scène du port animée
- Campagne chapitres **0 à 2** (4–5 niveaux : terminal/help, image vs conteneur, `pull`/`run`/`ps`/`stop`/`start`/`rm`/`--name`/`-d`, puis `-p` et conflits de ports)
- Sauvegarde progression + étoiles
- Indices et aide au terminal

**Important** (ensuite) : ch. 3–5 (logs/exec/inspect, volumes, réseaux), épreuves chronométrées + classement, achievements + page profil.

**Bonus / feuille de route** : ch. 6–7 (Dockerfile, compose) + éditeur de fichiers, stages 2 et 3, carnet de bord.

Les fonctions indispensables suffisent au rendu ; le reste s'ajoute **sans changer l'architecture** grâce aux niveaux en données.

## Objectifs pédagogiques (ce que le joueur doit savoir à la fin)

Image vs conteneur · cycle de vie des conteneurs · ports/exposition · volumes & bind mounts · réseaux privés · Dockerfile & build · docker-compose · workflow de dev · mise en prod (staging/Swarm/sécurité) · diagnostic/réparation.

## Conventions de travail

- Structure monorepo probable : séparer clairement `front` (Nuxt) et `back` (Express/TS). Confirmer l'organisation avant de scaffolder si ce n'est pas encore fait.
- TypeScript strict côté back. Le simulateur et la validation sont le cœur métier : les tester.
- Ne jamais introduire d'exécution Docker réelle, même "pour tester".
- Toute logique de score / réussite / déblocage reste **côté serveur**.

## État actuel

Dépôt vierge (greenfield). Rien n'est encore scaffoldé. Ce fichier décrit l'intention ; il sera mis à jour au fur et à mesure que l'arborescence et les choix concrets se fixent.
