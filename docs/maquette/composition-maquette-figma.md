# Maquette Figma — Docker Harbor : composition

**Lien du fichier Figma : https://www.figma.com/design/Q1OyhskvN7SOBA726LDSZP**

Ce document décrit **comment la maquette est constituée** : organisation du fichier, système visuel,
composants, écrans transverses et écrans de chacun des 10 chapitres. Sources fonctionnelles :
[`docs/modelisation/`](../modelisation/RENDU.md) et [`docs/simulateur/`](../simulateur/commandes-par-chapitre.md).

| Paramètre | Choix retenu |
| --- | --- |
| Format | Desktop **1440 × 900** |
| Thème | **Marine** uniquement |
| Langue | **Français** (les commandes et sorties Docker restent en anglais, comme le vrai Docker) |
| Style de la scène | Illustration **2D flat**, vue de côté |
| Capitaine | **Humain** (marin barbu, casquette), 4 expressions |
| Éditeur (ch. 6 à 8) | **Écran divisé** scène / éditeur |
| Épreuve chronométrée | Rattachée à son chapitre (niveau `type = EPREUVE`), badge chrono sur la carte |

---

## 1. Organisation du fichier

Le fichier contient **3 pages**, chacune organisée en sections.

| Page | Sections / contenu |
| --- | --- |
| **Fondations & composants** | Fondations (couleurs, typographie) · Composants |
| **Écrans transverses** | Accès · Carte de campagne · Profil et classement · Épreuve, victoire, aide |
| **Chapitres 0 à 9** | Une section par chapitre, écrans alignés horizontalement |

---

## 2. Fondations

**Couleurs** (collection de variables « Couleurs Marine ») :

| Famille | Variables |
| --- | --- |
| Marine | `marine/900` `#0B2545` · `marine/800` `#13315C` · `marine/700` `#1B4079` · `marine/600` `#2A5A9E` |
| Eau | `eau/700` `#0E7C9B` · `eau/500` `#13A4C8` · `eau/300` `#7FD3E8` |
| Bois / sable | `sable` `#F4E3B1` · `bois` `#B07A43` · `bois-sombre` `#7A4F27` |
| Statuts | `succes` `#3CB371` · `corail` `#E85D4A` (erreur) · `or` `#F5C542` (étoiles, achievements) |
| Texte / terminal | `texte` `#EAF2FB` · `texte-doux` `#8DA9C4` · `terminal` `#07182E` |

**Typographie** : *Nunito* (titres Bold/ExtraBold 20–72, corps Regular 12–16) et *JetBrains Mono*
(terminal, tableaux, code, 11–14).

---

## 3. Métaphore visuelle

| Docker | Représentation |
| --- | --- |
| Image | **Plan** (rouleau bleu) rangé à l'**entrepôt** |
| Conteneur | **Bateau**, nom écrit sur la coque |
| Port exposé `-p` | **Quai** numéroté + passerelle |
| Volume | **Coffre** en bois |
| Réseau | **Canal** (bande d'eau colorée, une couleur par réseau) |
| Registre | **Amirauté** (bâtiment à l'horizon) |
| Erreur | Bateau en pointillés / quai barré / coffre coulé + terminal en corail |

---

## 4. Composants

| Composant | Variantes |
| --- | --- |
| **Bateau** | Statut : Créé (cale sèche) · En marche (voiles et pavillon) · Arrêté (amarré) · Supprimé (fantôme en pointillés) |
| **Quai** | État : Libre · Occupé (passerelle) · Conflit (✖ rouge) |
| **Coffre** | État : Vide · Branché (câble) · Coulé |
| **Canal** | Couleur : Eau · Corail · Vert |
| **Plan (image)** | — |
| **Étoiles** | 0 · 1 · 2 · 3 |
| **Capitaine** | Expression : Neutre · Explique · Inquiet · Content |
| **Bulle capitaine** | Texte du dialogue / de l'indice |
| **Bouton** | Primaire · Secondaire |
| **Terminal** | Barre de titre + 4 lignes de sortie (commande, sortie, erreur, succès) |

---

## 5. Écran de niveau (gabarit commun)

Tous les écrans de chapitre reprennent le même gabarit **1440 × 900** :

| Zone | Contenu |
| --- | --- |
| **En-tête** (64 px) | Retour carte · titre « Chapitre N · nom — Niveau x/y » · étoiles · bouton Indice (0/3) |
| **Scène du port** (840 × 556, gauche) | Illustration de l'état du simulateur : ciel, skyline, eau, bâtiments, bateaux, quais, coffres, canaux… |
| **Capitaine** (droite, haut) | Portrait + bulle de dialogue (fond corail clair en cas d'erreur) |
| **Objectifs** (droite, milieu) | Cases à cocher des conditions de réussite |
| **Panneau contextuel** (droite, bas) | Onglets et tableau / logs / JSON, spécifiques au chapitre |
| **Terminal** (bas, pleine largeur) | Commande saisie et sortie |

**Variante écran divisé** (ch. 6, 7, 8) : la scène est réduite à 420 px de large et un **éditeur de
fichiers** (Dockerfile, docker-compose.yml, `.env`, `devcontainer.json`) occupe 400 px à côté, avec
numéros de ligne, coloration et bandeau d'erreur.

---

## 6. Écrans transverses

| Écran | Contenu |
| --- | --- |
| **Accueil** | Scène du port, titre, accroche, boutons *Jouer* / *Se connecter*, capitaine |
| **Inscription** | Formulaire pseudo / email / mot de passe, avec erreur de validation (pseudo déjà pris) |
| **Connexion** | Formulaire email / mot de passe, avec erreur d'identifiants |
| **Carte de campagne** | 10 îlots (chapitres) reliés par un chemin, regroupés en 3 zones : Stage 1 *Les bases* (ch. 0–7), Stage 2 *Creuser* (ch. 8), Stage 3 *Vers le staging* (ch. 9). États : terminé (vert + étoiles), en cours (halo), verrouillé (cadenas). Badge chrono sur les chapitres ayant une épreuve. Légende et message du capitaine |
| **Profil** | Carte joueur (pseudo, étoiles, niveaux, essais, indices), grille d'achievements (obtenus / verrouillés), progression par chapitre |
| **Classement** | Onglets par épreuve, tableau rang / joueur / temps / score / essais, ligne du joueur mise en évidence |
| **Épreuve chronométrée** | Écran de niveau avec chrono en en-tête, sans étoiles ni indices, bouton *Abandonner* |
| **Victoire** | Superposition : 1 à 3 étoiles, essais / indices, achievement débloqué, *Rejouer* / *Niveau suivant* |
| **Besoin d'aide** | Superposition : 3 paliers d'indices avec leur coût en étoiles (0, −1, −2) |

---

## 7. Écrans par chapitre

Chaque chapitre présente le gabarit rempli pour les moments clés (consigne, succès, erreurs réalistes
du simulateur).

### Stage 1 — Les bases

| Chapitre | Écrans | Scène / panneau |
| --- | --- | --- |
| **0 · Bienvenue au port** (1 niveau) | Tutoriel (terminal surligné) · Après `help` · `docker run hello-world` réussi | Port vide, entrepôt, petit bateau qui entre et salue. Panneau tutoriel guidé |
| **1 · Premiers bateaux** (4) | Gabarit niveau 2 · `pull` · `run` · `stop`/`start` · erreur `rm` sur conteneur en marche | Entrepôt + plans, bateaux. Onglets *Registre des bateaux* / *Entrepôt* |
| **2 · Ouvrir les quais** (3) | Porte fermée (client sans passerelle) · passerelle posée sur le quai 8080 · plusieurs bateaux / quais · conflit de port | Quais numérotés, passerelles, client. Onglets *Quais* / *Bateaux* |
| **3 · Inspecter et dépanner** (3) | Journal de bord (`logs`) · à bord (`exec`) · plan détaillé (`inspect`) · variables `-e` | Livre « Journal », bateau agrandi, étiquettes d'état. Onglets *Journal* / *Inspection* / *Consignes scellées* |
| **4 · Les coffres** (3) | Coffre créé · coffre branché · bind mount · le coffre coule (achievement *Naufrageur*) | Coffres, câbles, « entrepôt de la ville ». Onglets *Coffres* / *Fichiers* |
| **5 · Les canaux privés** (3) | Canal creusé · même canal (ping réussi) · canaux différents (résolution échouée) · `network connect` | Canaux colorés, bulles entre bateaux. Onglets *Canaux* / *Bateaux* |
| **6 · Le chantier naval** (4) | Éditeur Dockerfile · build couche par couche · re-build avec `CACHED` · `docker tag` · erreur `no build stage` | **Écran divisé.** Chantier naval, pile de couches. Onglets *Couches* / *Images* |
| **7 · La flotte** (4) | Éditeur compose · `compose up` · `compose down` · `compose ps` · YAML invalide | **Écran divisé.** Bateaux regroupés en flotte, liens `depends_on`. Onglets *Flotte* / *Journaux* |

### Stage 2 — Creuser

| Chapitre | Écrans | Scène / panneau |
| --- | --- | --- |
| **8 · L'atelier** (4) | Fichier `.env` et `--env-file` · `devcontainer.json` (« Rouvrir dans le conteneur ») · coque Windows incompatible · diagnostic (logs / inspect / exec) · avarie réparée | Atelier du capitaine (écran divisé pour les fichiers), trois cartes de diagnostic côte à côte. Onglets *Fichiers* / *Diagnostic* |

### Stage 3 — Vers le staging

| Chapitre | Écrans | Scène / panneau |
| --- | --- | --- |
| **9 · L'amirauté** (6) | `login` · `push` · `pull` · `swarm init` (manager) · `service create --replicas 3` · `stack deploy` · audit de sécurité (root, image lourde) | Amirauté à l'horizon, câbles d'expédition, escadrille de réplicas, carte d'audit. Onglets *Amirauté* / *Escadrille* / *Sécurité* |

---

## 8. Feedback d'erreur (commun à tous les chapitres)

1. **Terminal** : `stderr` en corail, message identique au vrai Docker.
2. **Scène** : réaction visuelle (bateau fantôme, passerelle barrée, coffre coulé, bulle d'erreur).
3. **Capitaine** : expression *Inquiet* et bulle sur fond corail clair avec le message ciblé.
4. **Objectifs** : cases cochées au fil de l'état atteint, sans afficher la commande à taper.
