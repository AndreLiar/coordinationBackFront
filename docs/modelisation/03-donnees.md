# 3. Modèle de données — du conceptuel aux collections MongoDB

[← Activités](./02-activites.md) · [Index](./README.md) · [Séquence →](./04-sequence.md)

## 3.1 MCD (niveau métier)

Entités métier et cardinalités (notation Merise `0,N` / `1,1`).

```mermaid
erDiagram
    JOUEUR ||--o{ PROGRESSION : "possède"
    NIVEAU ||--o{ PROGRESSION : "est joué dans"
    STAGE  ||--o{ CHAPITRE : "contient"
    CHAPITRE ||--o{ NIVEAU : "regroupe"
    JOUEUR ||--o{ SCORE_EPREUVE : "réalise"
    EPREUVE ||--o{ SCORE_EPREUVE : "reçoit"
    NIVEAU ||--o| EPREUVE : "sert de base à"
    JOUEUR }o--o{ ACHIEVEMENT : "débloque"

    JOUEUR {
        string pseudo
        string email
        string motDePasseHash
    }
    STAGE { int numero  string titre }
    CHAPITRE { int numero  string titre }
    NIVEAU {
        int ordre
        string consigne
        json etatInitial
        json conditionsReussite
    }
    PROGRESSION {
        string statut
        int etoiles
        int nbEssais
        int nbIndices
    }
    EPREUVE { string titre }
    SCORE_EPREUVE { int tempsMs  int score }
    ACHIEVEMENT { string code  string titre }
```

**Règles métier (à valider avant implémentation) :**

- Un **Joueur** a 0..N **Progressions** ; chaque Progression appartient à **exactement 1** Joueur et **1** Niveau.
- Un **Stage** contient plusieurs **Chapitres** ; un Chapitre regroupe plusieurs **Niveaux** (débloqués dans l'ordre).
- Un **Joueur** peut débloquer plusieurs **Achievements** (relation N..N → collection de jonction ou tableau de refs).
- Une **Épreuve** s'appuie généralement sur un **Niveau** « cassé à réparer ».

## 3.2 Diagramme de classes UML (niveau application / domaine)

La grande différence avec le MCD : on ajoute les **comportements** (méthodes) et les **invariants**.
La classe `Niveau` sait **valider un objectif** ; `Progression` protège le cycle de vie d'une tentative.

```mermaid
classDiagram
    class Joueur {
        +id: string
        +pseudo: string
        +email: string
        -motDePasseHash: string
        +verifierMotDePasse(clair) bool
    }

    class Progression {
        +id: string
        -statut: StatutProgression
        -etoiles: int
        -nbEssais: int
        -nbIndices: int
        -meilleurTempsMs: int
        +debloquer() void
        +enregistrerEssai() void
        +reussir(etoiles) void
    }

    class Niveau {
        +id: string
        +ordre: int
        +consigne: string
        +etatInitial: EtatPort
        +conditionsReussite: Objectif[]
        +indices: Indice[]
        +verifierObjectif(etat) bool
        +calculerEtoiles(essais, indices) int
    }

    class EtatPort {
        +images: Image[]
        +conteneurs: Conteneur[]
        +volumes: Volume[]
        +reseaux: Reseau[]
        +appliquer(cmd) ResultatCommande
    }

    class Conteneur {
        +id: string
        +nom: string
        +image: string
        -statut: StatutConteneur
        +portsExposes: PortMapping[]
        +demarrer() void
        +arreter() void
        +supprimer() void
    }

    class Image { +nom: string  +tag: string }
    class Volume { +nom: string }
    class Reseau { +nom: string  +conteneurs: string[] }
    class Indice { +palier: int  +texte: string  +penaliteEtoile: int }
    class Achievement { +code: string  +titre: string  +estDebloque(joueur) bool }

    Joueur "1" --> "0..*" Progression : suit
    Niveau "1" --> "0..*" Progression : concerne
    Niveau "1" *-- "1" EtatPort : état de départ
    Niveau "1" o-- "0..*" Indice : propose
    EtatPort "1" *-- "0..*" Conteneur : contient
    EtatPort "1" *-- "0..*" Image
    EtatPort "1" *-- "0..*" Volume
    EtatPort "1" *-- "0..*" Reseau
    Joueur "0..*" --> "0..*" Achievement : débloque
```

- **Composition** (`*--`) : un `EtatPort` **possède** ses conteneurs/volumes/réseaux — ils n'existent pas hors de l'état de la partie (dépendance de cycle de vie).
- **Agrégation** (`o--`) : un `Niveau` **référence** des indices réutilisables.
- **Association** (`-->`) : `Joueur` ↔ `Progression`, `Niveau` ↔ `Progression`.

## 3.3 MLD → collections MongoDB

Le projet utilise **MongoDB** : pas de clés étrangères SQL, mais des références par `_id` et
quelques documents **embarqués** quand les données sont lues ensemble.

| Collection | Contenu | Références |
| --- | --- | --- |
| `joueurs` | pseudo, email, `motDePasseHash`, dates | — |
| `niveaux` | `chapitreId`, ordre, consigne, `etatInitial`, `conditionsReussite`, `indices[]`, `dialogues[]` | `chapitreId` |
| `chapitres` | `stageId`, numero, titre | `stageId` |
| `progressions` | `joueurId`, `niveauId`, statut, etoiles, nbEssais, nbIndices, meilleurTempsMs | `joueurId`, `niveauId` |
| `epreuves` | `niveauId`, titre, règles de score | `niveauId` |
| `scores` | `joueurId`, `epreuveId`, tempsMs, score, date | `joueurId`, `epreuveId` |
| `achievements` | code, titre, condition | — |
| `joueurs.achievements[]` | tableau embarqué `{ code, dateObtention }` dans `joueurs` | — |

Exemple de document `niveaux` (**le niveau est une donnée, pas du code** — ajouter un niveau
= ajouter un document, sans toucher au simulateur) :

```json
{
  "_id": "n-ch1-02",
  "chapitreId": "ch-1",
  "ordre": 2,
  "titre": "Premier bateau",
  "consigne": "Lance un conteneur nginx nommé 'site'.",
  "etatInitial": { "images": [{ "nom": "nginx", "tag": "latest" }],
                   "conteneurs": [], "volumes": [], "reseaux": [] },
  "conditionsReussite": [
    { "type": "conteneurExiste", "nom": "site", "statut": "RUNNING", "image": "nginx" }
  ],
  "indices": [
    { "palier": 1, "texte": "Il faut 'lancer' une image.", "penaliteEtoile": 0 },
    { "palier": 2, "texte": "Regarde la commande 'docker run'.", "penaliteEtoile": 1 },
    { "palier": 3, "texte": "docker run --name site nginx", "penaliteEtoile": 2 }
  ]
}
```
