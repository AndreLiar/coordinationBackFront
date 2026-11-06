# Commandes simulées par chapitre

Source de vérité du **simulateur** (Express). Elle définit, chapitre par chapitre :

- les **commandes** et **options** que le simulateur doit comprendre ;
- leur **effet** sur l'état du port (images, conteneurs, ports, volumes, réseaux) ;
- les **erreurs réalistes** à reproduire (comme le vrai Docker) ;
- la **métaphore** visuelle correspondante (scène du port).

> Principe : le jeu de commandes est **cumulatif** — chaque chapitre ajoute des commandes/options
> à celles déjà apprises. Tout ce qui n'est pas dans la table d'un chapitre produit une erreur
> « commande inconnue » ou « option non supportée ».

**Légende** — Priorité : 🟢 Indispensable (rendu) · 🟡 Important · 🔵 Bonus / feuille de route.

---

## Commandes transverses (disponibles partout) 🟢

| Commande | Effet | Erreurs |
| --- | --- | --- |
| `help` | Liste les commandes disponibles **dans le niveau courant** | — |
| `<cmd> --help` | Aide courte d'une commande | — |
| `clear` | Vide le terminal (affichage uniquement) | — |
| `docker ps` / `docker ps -a` | Liste les conteneurs en marche / tous | — |
| `docker images` | Liste les images (plans dans l'entrepôt) | — |

> Le parseur doit tolérer `docker ps` **et** `docker container ls`, `docker images` **et**
> `docker image ls` (alias officiels), au moins pour les listings de base.

---

## Stage 1 — Les bases

### Chapitre 0 — Bienvenue au port 🟢 *(1 niveau)*

Notions : terminal, `help`, premier conteneur.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `help` | — | affiche l'aide | — | la capitainerie explique |
| `docker run hello-world` | — | pull implicite de `hello-world` si absente, crée un conteneur, affiche le message de bienvenue, le conteneur **se termine** (statut `Exited`) | `Unable to find image` → **suivi** du pull simulé (pas une erreur bloquante) | un petit bateau entre, salue, repart |

### Chapitre 1 — Premiers bateaux 🟢 *(4 niveaux)*

Notions : image vs conteneur, cycle de vie.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker pull <image>[:tag]` | `:tag` (défaut `latest`) | ajoute une **image** à l'entrepôt | `repository does not exist` si image hors catalogue simulé | un plan de bateau arrive à l'entrepôt |
| `docker run <image>` | `--name <nom>`, `-d` | crée **+ démarre** un conteneur ; `-d` = en arrière-plan | `Conflict. The container name "/x" is already in use` ; image absente → pull simulé | un bateau est construit et navigue |
| `docker ps` | `-a` | liste conteneurs (marche / tous) | — | registre des bateaux |
| `docker stop <nom\|id>` | — | `En marche` → `Arrêté` | `No such container: x` | le bateau s'arrête à quai |
| `docker start <nom\|id>` | — | `Arrêté`/`Créé` → `En marche` | `No such container: x` | le bateau repart |
| `docker rm <nom\|id>` | `-f` | supprime un conteneur **arrêté** ; `-f` force même en marche | `You cannot remove a running container ... Stop the container before attempting removal or force (-f)` | le bateau quitte le port |

> Voir la machine à états du conteneur :
> [`../modelisation/05-etats-transitions.md`](../modelisation/05-etats-transitions.md).

### Chapitre 2 — Ouvrir les quais 🟢 *(3 niveaux)*

Notions : exposition de ports, plusieurs conteneurs, conflit de ports.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker run ... -p <hôte>:<conteneur>` | `-p` (répétable) | mappe un port hôte → port conteneur ; réserve le port hôte | `Bind for 0.0.0.0:8080 failed: port is already allocated` si port hôte déjà pris | une passerelle relie le bateau au quai `8080` |
| *(plusieurs `docker run`)* | — | plusieurs conteneurs coexistent | nom déjà utilisé ; port déjà alloué | plusieurs bateaux à plusieurs quais |

> Erreur pédagogique clé : un service **non** exposé = le client « frappe à une porte fermée »
> (pas de passerelle). À exploiter dans les dialogues de niveau.

### Chapitre 3 — Inspecter et dépanner 🟡 *(3 niveaux)*

Notions : logs, exécution dans un conteneur, inspection, variables d'environnement.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker logs <nom>` | `-f` (affichage), `--tail N` | affiche les logs **simulés** du conteneur | `No such container` | le journal de bord du bateau |
| `docker exec <nom> <cmd>` | `-it` | exécute une commande simulée dans le conteneur (ex. `ls`, `cat`, `env`) | conteneur arrêté → `Container x is not running` | on monte à bord |
| `docker inspect <nom>` | — | renvoie un JSON simulé (état, ports, volumes, réseaux) | `No such object: x` | plan détaillé du bateau |
| `docker run ... -e CLE=VALEUR` | `-e` (répétable) | injecte une variable d'env dans le conteneur | — | consignes scellées remises au capitaine |

### Chapitre 4 — Les coffres 🟡 *(3 niveaux)*

Notions : volumes nommés, bind mounts, perte de données.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker volume create <nom>` | — | crée un **volume** | nom déjà existant (avertissement) | un coffre est posé à quai |
| `docker volume ls` | — | liste les volumes | — | inventaire des coffres |
| `docker run ... -v <volume>:/chemin` | `-v` | monte un volume nommé → **données persistées** après `rm` | volume absent → créé implicitement | le bateau branche un coffre |
| `docker run ... -v /hôte:/chemin` | `-v` | **bind mount** d'un dossier hôte (simulé) | chemin invalide | lien direct vers un entrepôt de la ville |
| `docker volume rm <nom>` | — | supprime un volume | `volume is in use` si monté | on retire le coffre |

> Erreur pédagogique clé : écrire des données **sans** volume puis `docker rm` → **données perdues**
> (le « coffre coule »). Déclenche l'achievement *Naufrageur*.

### Chapitre 5 — Les canaux privés 🟡 *(3 niveaux)*

Notions : réseaux, résolution par nom.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker network create <nom>` | — | crée un **réseau** privé | nom déjà existant | un canal privé est creusé |
| `docker network ls` | — | liste les réseaux | — | carte des canaux |
| `docker run ... --network <nom>` | `--network` | attache le conteneur au réseau | réseau absent → `network x not found` | le bateau rejoint le canal |
| `docker network connect <net> <nom>` | — | connecte un conteneur existant | objets absents | on relie un bateau au canal |
| *(ping par nom)* via `exec` | — | un conteneur joint un autre **par son nom** sur le même réseau | pas sur le même réseau → échec de résolution | les bateaux se parlent sans passer par la ville |

> Erreur pédagogique clé : deux conteneurs qui ne sont **pas** sur le même réseau ne se
> résolvent pas par nom → message « name resolution failed » simulé.

### Chapitre 6 — Le chantier naval 🔵 *(4 niveaux)*

Notions : Dockerfile, build, tags, cache des couches. **Éditeur de fichiers requis.**

Instructions Dockerfile supportées : `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `EXPOSE`, `CMD`.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker build -t <nom>:<tag> .` | `-t`, `-f <Dockerfile>` | lit le Dockerfile de l'éditeur, construit une **image** couche par couche | `FROM` manquant → `no build stage in current context` ; instruction inconnue | on assemble un nouveau plan au chantier |
| *(re-build)* | — | **cache des couches** : les étapes inchangées sont réutilisées (`CACHED`) | — | on réutilise les pièces déjà fabriquées |
| `docker tag <img> <nom>:<tag>` | — | ajoute un tag à une image | image absente | on étiquette le plan |

### Chapitre 7 — La flotte 🔵 *(4 niveaux)*

Notions : docker-compose. **Éditeur `docker-compose.yml` requis.**

Clés compose supportées : `services`, `image` / `build`, `ports`, `volumes`, `networks`,
`environment`, `env_file`, `depends_on`.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker compose up` | `-d`, `--build` | crée/démarre **tous les services** décrits ; respecte `depends_on` | YAML invalide ; service/image inconnu ; port déjà alloué | la flotte entière appareille sur un seul ordre |
| `docker compose down` | `-v` | arrête et supprime les conteneurs (et volumes si `-v`) | — | toute la flotte rentre au port |
| `docker compose ps` | — | liste les services de la flotte | — | registre de la flotte |
| `docker compose logs` | `-f`, `<service>` | logs agrégés des services | service inconnu | journaux de bord cumulés |

> Le simulateur doit supporter la double syntaxe `docker compose` (v2) **et** `docker-compose` (v1).

---

## Stage 2 — Creuser

### Chapitre 8 — L'atelier 🔵 *(4 niveaux)*

Notions : workflow de dev, `.env`, `devcontainer.json`, Windows containers, débogage.

| Élément | Support attendu | Erreurs simulées | Métaphore |
| --- | --- | --- | --- |
| `--env-file <.env>` sur `run` | charge des variables depuis un fichier `.env` (éditeur) | fichier absent | consignes lues depuis un carnet |
| `env_file:` en compose | idem côté flotte | — | — |
| `devcontainer.json` | éditeur + commande « rouvrir dans le conteneur » (simulée) | JSON invalide | l'atelier du capitaine |
| Windows containers | `docker run` avec image Windows simulée ; notion d'OS du conteneur | OS mismatch → `image operating system "windows" cannot be used on this platform` | cale adaptée à un autre type de coque |
| Débogage | lecture `logs` + `inspect` + `exec` pour diagnostiquer | — | inspection de l'avarie |

### Stage 3 — Vers le staging

### Chapitre 9 — L'amirauté 🔵 *(6 niveaux)*

Notions : registre, staging, Swarm, sécurité, images allégées.

| Commande | Options | Effet sur l'état | Erreurs simulées | Métaphore |
| --- | --- | --- | --- | --- |
| `docker login <registre>` | `-u`, `-p` | authentifie (simulé) | mauvais identifiants | laissez-passer de l'amirauté |
| `docker push <nom>:<tag>` | — | envoie une image vers le registre simulé | non authentifié ; image absente | on expédie le plan à l'amirauté |
| `docker pull <registre>/<img>` | — | récupère depuis le registre | image inexistante | on reçoit un plan d'un autre port |
| `docker swarm init` | — | active le mode Swarm (simulé) | déjà actif | fondation de l'amirauté |
| `docker service create` | `--replicas N`, `-p` | crée un service répliqué | Swarm inactif → `This node is not a swarm manager` | une flotte dupliquée en escadrille |
| `docker stack deploy -c <compose>` | `-c` | déploie une stack depuis un compose | Swarm inactif ; YAML invalide | déploiement d'une armada |
| *(sécurité)* `USER` non-root, images `alpine`/multi-stage | validé via Dockerfile | image qui tourne en root signalée (avertissement pédagogique) | coque allégée et équipage réglementaire |

---

## Récapitulatif — couverture par priorité

| Priorité | Chapitres | Ce que le simulateur doit couvrir |
| --- | --- | --- |
| 🟢 **Indispensable (rendu)** | 0, 1, 2 | `help`, `pull`, `run` (`--name`, `-d`, `-p`), `ps`/`-a`, `stop`, `start`, `rm`/`-f`, `images` + erreurs : image absente, nom déjà utilisé, port déjà alloué, conteneur introuvable |
| 🟡 **Important** | 3, 4, 5 | `logs`, `exec`, `inspect`, `-e`, volumes (`-v`), bind mounts, perte de données, réseaux (`--network`), résolution par nom |
| 🔵 **Bonus / roadmap** | 6, 7, 8, 9 | Dockerfile + `build` + cache, compose (`up`/`down`), `.env`/devcontainer/Windows, registre/Swarm/stack/sécurité |

## Catalogue d'erreurs réalistes (réutilisable)

| Code interne | Message (style Docker) | Déclencheur |
| --- | --- | --- |
| `IMAGE_NOT_FOUND` | `Unable to find image 'x:tag' locally` | image hors catalogue au `run`/`pull` |
| `NAME_CONFLICT` | `Conflict. The container name "/x" is already in use` | `--name` déjà pris |
| `PORT_ALLOCATED` | `Bind for 0.0.0.0:P failed: port is already allocated` | port hôte déjà réservé |
| `NO_SUCH_CONTAINER` | `Error: No such container: x` | `stop`/`start`/`rm`/`logs` sur inconnu |
| `RUNNING_RM` | `You cannot remove a running container ... (-f)` | `rm` sans `-f` sur conteneur en marche |
| `NOT_RUNNING_EXEC` | `Error response from daemon: Container x is not running` | `exec` sur conteneur arrêté |
| `NETWORK_NOT_FOUND` | `network x not found` | `--network` sur réseau inexistant |
| `NAME_RESOLUTION` | `could not resolve host: x` | ping par nom hors réseau commun |
| `NOT_SWARM_MANAGER` | `This node is not a swarm manager` | `service`/`stack` sans `swarm init` |
| `COMPOSE_INVALID` | `yaml: ...` | `compose up` sur YAML invalide |

## Pour l'implémentation

- Chaque entrée de cette table = au moins **un cas de test** du simulateur (succès **et** erreur).
- Les `conditionsReussite` d'un niveau ([exemple de document](../modelisation/03-donnees.md#33-mld--collections-mongodb))
  s'expriment en termes d'**état final attendu** (ex. « conteneur `site` en marche, port 8080 exposé »),
  pas en termes de commande tapée → plusieurs solutions valides sont acceptées.
- Le parseur doit distinguer **commande inconnue** (hors périmètre) et **option non supportée**
  (commande connue, flag non géré) pour des messages d'aide ciblés.
