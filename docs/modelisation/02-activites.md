# 2. Diagramme d'activités — « Jouer un niveau »

[← Cas d'utilisation](./01-cas-utilisation.md) · [Index](./README.md) · [Données →](./03-donnees.md)

> *Comment se déroule le processus, quelles décisions, quelles tâches concurrentes,
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

    %% Fork : deux écritures Mongo concurrentes (Promise.all) une fois le niveau réussi
    sauver --> fork(( ))
    fork --> achiev
    fork --> afficherFin["Afficher la réussite<br/>(1 à 3 étoiles)"]
    achiev --> join(( ))
    afficherFin --> join
    join --> fin([◉])
```

## Mécanismes UML illustrés

- **Decision / Merge** (`{ }`) : « Commande valide ? », « Objectif atteint ? », « Bloqué ? » — un seul chemin est suivi.
- **Fork / Join** (`(( ))`) : après la réussite, l'enregistrement de la progression et l'évaluation des achievements sont **lancés ensemble** (`Promise.all`), puis on **synchronise** (join) avant la fin.
  > ⚠️ **Précision technique** (back TypeScript / Node) : Node est **mono-thread**. Le fork ne représente **pas** du parallélisme CPU mais de la **concurrence asynchrone** : les deux I/O MongoDB sont émises simultanément et leurs temps d'attente se **recouvrent** via la boucle d'événements. C'est ce qu'exprime `await Promise.all([enregistrerProgression(), évaluerAchievements()])`. Du vrai parallélisme CPU exigerait des *Worker Threads*, inutiles ici (on attend des I/O, pas du calcul).
- **Swimlanes** : chaque action est rattachée à son responsable (Joueur / Front / Serveur / MongoDB), ce qui montre bien que **toute la validation et le calcul des étoiles sont côté serveur** (anti-triche).
