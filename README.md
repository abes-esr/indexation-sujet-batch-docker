# indexation-sujet-batch-docker

Configuration docker 🐳 pour déployer le batch **indexation-sujet-batch** de l'application **indexation-sujet** (IA Rameau). Ce README fait également office de fiche d'exploitation.

## Prérequis

Disposer de :

- `docker`
- `docker-compose` (ou plugin `docker compose`)

## Installation

Déployer la configuration docker dans un répertoire :

```bash
# adaptez /opt/pod/ avec l'emplacement où vous souhaitez déployer l'application
cd /opt/pod/
git clone https://github.com/abes-esr/indexation-sujet-batch-docker.git
```

Configurer l'application depuis l'exemple du [fichier `.env-dist`](./.env-dist) (ce fichier contient la liste des variables avec des explications et des exemples de valeurs) :

```bash
cd /opt/pod/indexation-sujet-batch-docker/
cp .env-dist .env
# personnaliser alors le contenu du .env
```

Démarrer l'application :

```bash
cd /opt/pod/indexation-sujet-batch-docker/
docker compose up -d
```

## Démarrage et arrêt

Pour démarrer l'application :

```bash
cd /opt/pod/indexation-sujet-batch-docker/
docker compose up -d
```

Pour arrêter l'application :

```bash
cd /opt/pod/indexation-sujet-batch-docker/
docker compose stop
```

## Supervision

Pour vérifier que l'application est démarrée, on peut consulter l'état des conteneurs :

```bash
cd /opt/pod/indexation-sujet-batch-docker/
docker compose ps
```

Pour vérifier que l'application est bien lancée, on peut consulter les logs :

```bash
cd /opt/pod/indexation-sujet-batch-docker/
docker compose logs --tail=50 -f
```

À noter que ces logs sont envoyées automatiquement au puits de log de l'Abes à l'aide du client Filebeat installé sur le nœud Docker et grâce aux labels configurés (`abes_appli=indexation-sujet`).

## Sauvegardes

Pour sauvegarder l'application, il faut :

- **Sauvegarder le fichier de configuration** :
  - Le fichier `/opt/pod/indexation-sujet-batch-docker/.env` qui est non versionné et qui permet de configurer le conteneur (les autres fichiers étant versionnés sur le dépôt GitHub `indexation-sujet-batch-docker`).

Ces opérations sont prises en charge dans la politique de sauvegarde du SIAT (sauvegardes régulières sur sotora/socorro).

## Restauration

### Réinstallation de l'application

- Se connecter avec son compte développeur sur la machine de déploiement (via SSH) :

- Se positionner dans le répertoire des applications :

```bash
cd /opt/pod
```

- Récupérer le projet `indexation-sujet-batch-docker` et se positionner dans le répertoire :

```bash
git clone https://github.com/abes-esr/indexation-sujet-batch-docker.git
cd indexation-sujet-batch-docker
```

- Récupérer le fichier `.env` depuis le serveur de sauvegarde sotora (authentification nécessaire) :

```bash
rsync -av devel@sotora.v104.abes.fr:/backup_pool/<nom-machine>/daily.0/racine/opt/pod/indexation-sujet-batch-docker/.env /opt/pod/indexation-sujet-batch-docker/.env
```

- Lancer le conteneur :

```bash
docker compose up -d
```

## Déploiement continu

Les objectifs des déploiements continus de indexation-sujet-batch sont les suivants (cf. [poldev](https://github.com/abes-esr/abes-politique-developpement/blob/main/01-Gestion%20du%20code%20source.md#utilisation-des-branches)) :

- Un `git push` sur la branche `develop` provoque un déploiement automatique sur le serveur de développement (`dev`).
- Un `git push` (ou merge) sur la branche `main` provoque un déploiement automatique sur le serveur de test (`test`).
- Un `git tag X.X.X` (associé à une release) sur la branche `main` permet un déploiement sur le serveur de production (`prod`).

indexation-sujet-batch s'appuie sur **WUD (What's Up Docker?)** pour la surveillance et la mise à jour automatique des conteneurs via les labels Docker configurés dans [docker-compose.yml](./docker-compose.yml) :

```yaml
labels:
  - "wud.watch=true"
  - "wud.watch.digest=true"
```

Le fonctionnement de WUD consiste à surveiller régulièrement la publication de nouvelles versions d'images sur Docker Hub pour `indexation-sujet-batch`. Lorsqu'une nouvelle image est détectée pour le tag ou le digest suivi, WUD la télécharge, arrête l'ancien conteneur et recrée le nouveau conteneur avec les mêmes paramètres d'environnement.

Pour le développeur, il suffit de pousser son code (ex: sur `develop`), d'attendre la complétion de la GitHub Action qui compile et publie l'image sur Docker Hub, puis WUD met à jour le service automatiquement.

## Mise à jour manuelle

Pour récupérer et appliquer manuellement la dernière version de l'image :

```bash
docker compose pull
docker compose up -d
```

Le `pull` téléchargera la dernière image disponible correspondant à la version définie dans votre `.env` (`INDEXATION_SUJET_BATCH_DUMP_VERSION`). Sans le pull, Docker réutilise l'image déjà présente localement.

## Architecture

Le service orchestré par Docker Compose :

- **indexation-sujet-batch** : Application Java Spring Boot chargée de l'extraction et du traitement batch des données RAMEAU.

### Dépôts sources et images Docker

Les images Docker de indexation-sujet-batch sont issues des dépôts GitHub de l'organisation [abes-esr](https://github.com/abes-esr) :

- [abes-esr/indexation-sujet-batch](https://github.com/abes-esr/indexation-sujet-batch) : Code source du batch Java Spring Boot
- [abes-esr/indexation-sujet-batch-docker](https://github.com/abes-esr/indexation-sujet-batch-docker) : Configuration Docker Compose de déploiement
- Images publiées sur Docker Hub : [abesesr/indexation-sujet](https://hub.docker.com/r/abesesr/indexation-sujet)
