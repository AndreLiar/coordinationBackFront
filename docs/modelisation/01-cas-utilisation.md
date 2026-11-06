# 1. Diagramme de cas d'utilisation

[← Index](./README.md) · [← Acteurs](./00-acteurs-perimetre.md) · [Activités →](./02-activites.md)

> *Qui utilise le jeu, quelles actions sont possibles, où s'arrêtent les responsabilités du système.*
> Mermaid n'a pas de notation « use case » native : les acteurs sont des nœuds à l'extérieur,
> les cas d'utilisation sont des ellipses `(( ))` dans la frontière du système.

```mermaid
flowchart LR
    Visiteur([Visiteur])
    Joueur([Joueur])
    Joueur -. généralisation .-> Visiteur

    subgraph SYS[" Système — Docker Harbor "]
        UC_insc(("S'inscrire"))
        UC_conn(("Se connecter"))

        UC_jouer(("Jouer un niveau"))
        UC_cmd(("Exécuter une commande simulée"))
        UC_valid(("Vérifier l'objectif du niveau"))
        UC_indice(("Demander un indice"))
        UC_editer(("Éditer un fichier<br/>Dockerfile / compose"))

        UC_epreuve(("Jouer une épreuve<br/>chronométrée"))
        UC_score(("Calculer le score / le temps"))
        UC_classement(("Consulter le classement"))

        UC_profil(("Consulter son profil<br/>& sa progression"))
        UC_achiev(("Débloquer un achievement"))
    end

    MongoDB[(MongoDB)]

    Visiteur --- UC_insc
    Visiteur --- UC_conn

    Joueur --- UC_jouer
    Joueur --- UC_indice
    Joueur --- UC_editer
    Joueur --- UC_epreuve
    Joueur --- UC_classement
    Joueur --- UC_profil

    %% include = comportement obligatoire réutilisé
    UC_jouer -. include .-> UC_cmd
    UC_jouer -. include .-> UC_valid
    UC_epreuve -. include .-> UC_cmd
    UC_epreuve -. include .-> UC_score

    %% extend = comportement conditionnel / optionnel
    UC_indice -. extend .-> UC_jouer
    UC_editer -. extend .-> UC_jouer
    UC_achiev -. extend .-> UC_jouer

    UC_valid --- MongoDB
    UC_score --- MongoDB
    UC_profil --- MongoDB
```

## Lecture des relations

| Relation | Exemple dans le jeu | Pourquoi |
| --- | --- | --- |
| `<<include>>` | « Jouer un niveau » **inclut toujours** « Exécuter une commande » et « Vérifier l'objectif » | comportement **obligatoire** à chaque partie |
| `<<extend>>` | « Demander un indice » **étend** « Jouer un niveau » | comportement **conditionnel** : seulement si le joueur bloque |
| `<<extend>>` | « Débloquer un achievement » étend le jeu | se déclenche **uniquement si** une condition est remplie |
| Généralisation | `Joueur` hérite de `Visiteur` | tout Joueur est aussi un Visiteur |

> Cette vue est **fonctionnelle**, pas technique. Elle ne montre ni MongoDB en détail, ni les
> routes API, ni le simulateur interne — ces éléments arrivent dans les vues suivantes.
