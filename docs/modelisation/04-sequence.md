# 4. Diagramme de séquence — « Exécuter une commande dans un niveau »

[← Données](./03-donnees.md) · [Index](./README.md) · [États-transitions →](./05-etats-transitions.md)

> *Qui appelle qui, dans quel ordre, avec quelles réponses.* On distingue les couches
> **Controller → Use Case → Domaine (Niveau/EtatPort) → Repository (MongoDB)**.

## Chemin nominal + erreurs (fragments `alt` / `par`)

```mermaid
sequenceDiagram
    actor Joueur
    participant Front as Front (Nuxt)
    participant API as CommandeController
    participant UC as JouerNiveauUseCase
    participant Sim as Simulateur / EtatPort
    participant Niv as Niveau (domaine)
    participant DB as MongoDB

    Joueur->>Front: tape « docker run --name site nginx »
    Front->>API: POST /niveaux/n-ch1-02/commande { cmd }
    activate API
    API->>UC: executer(niveauId, joueurId, cmd)
    activate UC
    UC->>DB: charger niveau + progression
    DB-->>UC: niveau, etatCourant
    UC->>Sim: appliquer(cmd)
    activate Sim

    alt commande invalide / erreur réaliste
        Sim-->>UC: Resultat(erreur, état inchangé)
        UC->>DB: incrémenter nbEssais
        UC-->>API: { stderr, etatPort }
        API-->>Front: 200 { erreur, etatPort }
        Front-->>Joueur: animation « le bateau coule », message d'erreur
    else commande valide
        Sim-->>UC: Resultat(stdout, nouvelEtatPort)
        deactivate Sim
        UC->>Niv: verifierObjectif(nouvelEtatPort)
        activate Niv
        alt objectif NON atteint
            Niv-->>UC: false
            UC->>DB: sauver nouvelEtatPort + nbEssais
            UC-->>API: { stdout, etatPort, reussi:false }
            API-->>Front: 200 { etatPort, reussi:false }
            Front-->>Joueur: scène mise à jour (bateau amarré)
        else objectif atteint
            Niv->>Niv: calculerEtoiles(essais, indices)
            Niv-->>UC: true, etoiles
            deactivate Niv
            par Persistance en parallèle
                UC->>DB: enregistrer progression (statut=REUSSI, étoiles)
            and
                UC->>DB: évaluer + débloquer achievements
            end
            UC-->>API: { stdout, etatPort, reussi:true, etoiles }
            deactivate UC
            API-->>Front: 200 { reussi:true, etoiles }
            deactivate API
            Front-->>Joueur: 🎉 1 à 3 étoiles, niveau suivant débloqué
        end
    end
```

## Fragments combinés utilisés

| Fragment | Usage ici |
| --- | --- |
| `alt` | commande invalide vs valide ; objectif atteint vs non |
| `par` | enregistrement de la progression **et** évaluation des achievements en parallèle |

> La **vérification de l'objectif**, le **calcul des étoiles** et l'**état du port** sont produits
> par le serveur. Le front ne fait qu'afficher ce qu'il reçoit → impossible de tricher côté client.

## Participants et couches

| Participant | Responsabilité |
| --- | --- |
| `CommandeController` | Entrée HTTP, (dé)sérialisation, codes de statut |
| `JouerNiveauUseCase` | Orchestration : charger, simuler, valider, persister |
| `Simulateur / EtatPort` | Applique la commande, maintient l'état fictif du port |
| `Niveau` | Règles métier : objectif atteint ? combien d'étoiles ? |
| `MongoDB` | Persistance progression, scores, achievements |
