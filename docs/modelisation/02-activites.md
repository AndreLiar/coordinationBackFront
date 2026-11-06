# 2. Diagramme d'activités — « Jouer un niveau »

[← Cas d'utilisation](./01-cas-utilisation.md) · [Index](./README.md) · [Données →](./03-donnees.md)

> *Comment se déroule le processus, quelles décisions, quelles tâches en parallèle,
> et **qui fait quoi** (swimlanes = responsabilités).*

```mermaid
flowchart TD
    start([●]) --> ouvrir

    subgraph J[" Joueur "]
        direction TB
        ouvrir["Ouvrir le niveau"]
        saisir["Saisir une commande<br/>ou éditer un fichier"]
        choixIndice{"Bloqué ?"}
        demIndice["Demander un indice"]
    end

    subgraph F[" Front (Nuxt) "]
        direction TB
        envoyer["Envoyer la commande<br/>à l'API"]
        afficher["Afficher le résultat<br/>+ animer la scène du port"]
    end

    subgraph B[" Serveur (API + Simulateur) "]
        direction TB
        parser["Analyser la commande"]
        valideCmd{"Commande valide ?"}
        erreur["Produire une erreur réaliste<br/>(état inchangé)"]
        muter["Mettre à jour l'état du port<br/>(images / conteneurs / ports…)"]
        objectif{"Objectif<br/>atteint ?"}
        etoiles["Calculer les étoiles<br/>(essais + indices)"]
    end

    subgraph DB[" MongoDB "]
        direction TB
        sauver["Enregistrer la progression<br/>+ étoiles"]
        achiev["Évaluer les achievements"]
    end

    ouvrir --> saisir
    saisir --> envoyer
    choixIndice -- oui --> demIndice --> saisir
    choixIndice -- non --> saisir

    envoyer --> parser --> valideCmd
    valideCmd -- non --> erreur --> afficher
    valideCmd -- oui --> muter --> objectif
    objectif -- non --> afficher --> choixIndice
    objectif -- oui --> etoiles --> sauver

    %% Fork : deux écritures en parallèle une fois le niveau réussi
    sauver --> fork(( ))
    fork --> achiev
    fork --> afficherFin["Afficher la réussite<br/>(1 à 3 étoiles)"]
    achiev --> join(( ))
    afficherFin --> join
    join --> fin([◉])
```

## Mécanismes UML illustrés

- **Decision / Merge** (`{ }`) : « Commande valide ? », « Objectif atteint ? », « Bloqué ? » — un seul chemin est suivi.
- **Fork / Join** (`(( ))`) : après la réussite, l'enregistrement de la progression et l'évaluation des achievements se font **en parallèle**, puis on **synchronise** (join) avant la fin.
- **Swimlanes** : chaque action est rattachée à son responsable (Joueur / Front / Serveur / MongoDB), ce qui montre bien que **toute la validation et le calcul des étoiles sont côté serveur** (anti-triche).
