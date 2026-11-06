# 0. Acteurs et périmètre

[← Index](./README.md)

| Acteur | Rôle |
| --- | --- |
| **Visiteur** | Personne non connectée : peut s'inscrire et se connecter. |
| **Joueur** | Visiteur authentifié : joue la campagne, les épreuves, consulte son profil. *(hérite de Visiteur)* |
| **Serveur de jeu** (API Express + Simulateur) | Simule Docker, valide les objectifs, calcule scores/étoiles. Acteur **interne** au système. |
| **MongoDB** | Système externe de persistance (comptes, progression, scores). |

Le système ne lance **aucun vrai Docker** : le « Simulateur » tient un état fictif
(images, conteneurs, ports, volumes, réseaux) et répond comme le ferait Docker, erreurs comprises.

```mermaid
flowchart LR
    Visiteur([Visiteur])
    Joueur([Joueur])
    Joueur -. "hérite de" .-> Visiteur

    subgraph SYS[" Système — Docker Harbor "]
        Serveur[["Serveur de jeu<br/>(API Express + Simulateur)"]]
    end

    MongoDB[(MongoDB)]

    Visiteur --- Serveur
    Joueur --- Serveur
    Serveur --- MongoDB
```

> Cette vue pose le cadre : les autres diagrammes détaillent **ce que** le joueur peut faire
> (cas d'utilisation), **comment** (activités, séquence) et **sur quelles données** (données, états).
