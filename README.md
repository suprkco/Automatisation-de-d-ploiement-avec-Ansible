# TP3 - Ansible

## Table des matières
1. [Partie 1: 3-tiers application](#partie-1-3-tiers-application)
2. [Partie 2: CI/CD](#partie-2-cicd)
3. [Partie 3: Ansible](#partie-3-ansible)

## Partie 1: 3-tiers application

### 1. Base de données

- Document your database container essentials: commands and Dockerfile:

    - on utilise l'image postgres:14.1-alpine comme base. Alpine est une version légère de Linux, ce qui rend l'image plus petite et plus rapide.
    - On définis des variables d'environnement pour le nom de la base de données, l'utilisateur et le mot de passe. Cela facilite la configuration initiale de PostgreSQL lors de la première exécution du conteneur.
    - On copie deux scripts SQL dans un répertoire spécial (/docker-entrypoint-initdb.d/). Tout script dans ce répertoire sera automatiquement exécuté par PostgreSQL lors du démarrage du conteneur, ce qui permet d'initialiser la structure de la base de données et d'y insérer des données initiales.

### 2. Construction multi-étapes

- Why do we need a multistage build? And explain each step of this dockerfile:
    - Le build multi-étapes en Docker sert principalement à deux choses :
        - Optimiser la taille de l'image finale : La première étape peut inclure tous les outils et fichiers nécessaires pour construire l'application (comme un JDK pour Java ou un compilateur pour C++), qui peuvent être volumineux. La seconde étape ne conserve que ce qui est nécessaire pour exécuter l'application (comme un JRE pour Java), réduisant ainsi la taille de l'image finale.
        - Séparer les étapes de construction et d'exécution : Cela permet d'avoir une séparation claire entre la compilation de l'application et son exécution, rendant le Dockerfile plus propre et organisé.

    - Étape de construction (build) pour l'API backend:
        - `FROM maven:3.8.6-amazoncorretto-17 AS myapp-build`: Cette ligne indique qu'on utilise l'image `maven:3.8.6-amazoncorretto-17` comme base pour cette étape de construction. C'est une image contenant Maven pour la gestion de projets Java, ainsi que l'Amazon Corretto 17 JDK (une distribution de OpenJDK). AS myapp-build donne un nom à cette étape, qui sera utilisé dans la suite du Dockerfile.

        - `ENV MYAPP_HOME /opt/myapp`: Cette ligne définit une variable d'environnement `MYAPP_HOME` avec la valeur `/opt/myapp`. Cette variable sera utilisée dans les étapes suivantes.

        - `WORKDIR $MYAPP_HOME`: Cette commande change le répertoire de travail dans l'image à `/opt/myapp` (la valeur de `MYAPP_HOME`).

        - `COPY pom.xml .`: Cette ligne copie le fichier `pom.xml` (qui est le fichier de configuration Maven de votre projet) du contexte de construction (généralement votre dossier local) vers le répertoire de travail dans l'image (c'est-à-dire `/opt/myapp`).

        - `COPY src ./src`: Cette commande copie le répertoire `src` (qui contient le code source de votre application) du contexte de construction vers le répertoire `/src` dans l'image

### 3. Commandes docker-compose

- Document docker-compose most important commands:
    - `docker-compose up` : Construire et démarrer les services.
    - `docker-compose down` : Arrête et supprime les services.
    - `docker-compose build` : Construit ou reconstruit les services.
    - `docker-compose logs` : Affiche la sortie des conteneurs.
    - `docker-compose ps` : Liste les conteneurs.

### 4. Fichier docker-compose

- Document your docker-compose file:
    Pour `build` :, les chemins (`./backend`, `./httpd`) doivent pointer vers les répertoires contenant les fichiers Docker respectifs.
    La directive `networks` : est utilisée pour connecter les services dans un réseau personnalisé pour la communication inter-conteneurs.

### 5. Publication sur Docker Hub

- Document your publication commands and published images in dockerhub:
    - Après vous être connecté à Docker Hub avec `docker login`, marquez votre image :
        ```bash
        docker tag my-database <USERNAME>/my-database:1.0
        ```
    - Poussez votre image :
        ```bash
        docker push <USERNAME>/my-database:1.0
        ```

### Tips questions

- Why should we run the container with a flag -e to give the environment variables?
    Pour garantir la sécurité et la flexibilité. En utilisant le drapeau `-e` lors de l'exécution du conteneur, nous pouvons définir des variables d'environnement spécifiques sans les inclure directement dans le Dockerfile. Cela permet d'éviter d'exposer des informations sensibles, telles que les mots de passe, dans des fichiers ou des images Docker, tout en permettant une configuration dynamique du conteneur lors de son exécution.

- Why do we need a volume to be attached to our postgres container?
    Pour garantir la persistance des données. Sans volumes, toutes les données à l'intérieur du conteneur sont éphémères et seront perdues si le conteneur est détruit. En montant un volume depuis l'hôte vers le conteneur, nous pouvons conserver les données du conteneur même après sa destruction, garantissant ainsi l'intégrité et la durabilité des données de la base de données.

- Why do we need a reverse proxy?
    Un proxy inverse offre plusieurs avantages :
        - `Sécurité` : Il masque la structure interne de votre réseau et empêche l'accès direct aux services backend.
        - `Optimisation des performances` : Il peut mettre en cache le contenu, réduisant ainsi la charge sur les serveurs backend.
        - `Équilibrage de charge` : Il peut distribuer les requêtes entrantes vers plusieurs serveurs backend pour équilibrer la charge.
        - `Centralisation` : Il offre un point d'accès unique aux clients et peut également gérer le SSL/TLS, libérant ainsi les applications backend de cette responsabilité.

- Why is docker-compose so important?
    Docker-compose simplifie la définition et l'exécution de plusieurs conteneurs Docker comme un ensemble unifié. Avec un seul fichier `docker-compose.yml`, vous pouvez définir des services, des réseaux et des volumes, et démarrer l'ensemble de l'application avec une seule commande (`docker-compose up`). Cela facilite la gestion, la mise à jour et la mise à l'échelle de services interdépendants dans des environnements de développement, de test et de production.

- Why do we put our images into an online repo?
    Pour faciliter la collaboration, la distribution et le déploiement. En stockant des images dans un dépôt en ligne comme Docker Hub, d'autres développeurs, systèmes ou services peuvent facilement télécharger et exécuter ces images sans avoir à les construire localement. Cela simplifie également le déploiement d'applications dans divers environnements ou sur plusieurs machines, car les images peuvent être rapidement récupérées et exécutées depuis le dépôt. De plus, cela offre une forme de sauvegarde pour vos images Docker.

## Partie 2: CI/CD

### 1. Testcontainers

- What are testcontainers ?

Les testcontainers sont des bibliothèques Java qui permettent d'exécuter des conteneurs Docker pendant les tests. 
Dans le contexte décrit, ils sont utilisés pour lancer un conteneur PostgreSQL pendant l'exécution des tests, fournissant ainsi une base de données dans laquelle les tests peuvent insérer ou extraire des données, garantissant ainsi que les fonctionnalités d'interaction avec la base de données de l'application fonctionnent comme prévu.

### 2. Configurations GitHub Actions

- Document your Github Actions configurations.

GitHub Actions est un service intégré à GitHub qui permet d'automatiser des workflows, comme la construction, les tests et le déploiement de votre code. Dans le contexte fourni, GitHub Actions est configuré pour:

    - Être déclenché lors des push et des pull requests.
    - Exécuter les jobs sur une machine virtuelle Ubuntu 22.04.
    - Récupérer le code source du dépôt.
    - Configurer JDK 17.
    - Construire l'application et exécuter les tests avec Maven.
    - Si les tests sont réussis, construire et pousser des images Docker vers Docker Hub.

La configuration utilise également des variables sécurisées pour stocker les informations d'identification de Docker Hub, garantissant que ces informations sensibles ne sont pas exposées.

Tout cela est défini dans le fichier main.yml sous le répertoire .github/workflows du dépôt. Ce fichier YAML décrit étape par étape comment le workflow doit s'exécuter, en utilisant à la fois des actions prédéfinies fournies par GitHub et des commandes personnalisées.

## Partie 3: Ansible

### 1. Inventaire et commandes de base

- Document your inventory and base commands

    L'inventaire est défini dans le fichier `setup.yml` sous le répertoire `inventories`. Dans cet inventaire :
    - `ansible_user`: représente l'utilisateur avec lequel nous nous connectons à la machine distante, qui est `centos` dans notre cas.
    - `ansible_ssh_private_key_file`: c'est le chemin vers notre clé SSH privée, qui est `~/.ssh/id_rsa` depuis que nous l'avons déplacée.
    - Sous le groupe `prod`, nous avons défini notre serveur, qui est `kilian.codaccioni.takima.cloud`.
    Commandes de base exécutées :
    - `Ping`: Nous avons utilisé la commande `ansible all -i inventories/setup.yml -m ping` pour vérifier la connectivité à notre serveur. La réponse `pong` indique que la connexion est réussie.
    - `Récupération des faits`: La commande `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"` a été utilisée pour récupérer des informations spécifiques sur la distribution du serveur. Il s'est avéré que notre serveur tourne sous CentOS 7.9.
    - `Désinstallation d'Apache httpd`: Nous avons tenté de désinstaller Apache httpd avec la commande `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become`. Le résultat indique qu'Apache httpd n'est pas installé.

### 2. Playbook

- Document your playbook

    Dans ce playbook, nous avons défini une série de tâches pour installer Docker sur notre serveur basé sur CentOS. Voici un résumé des étapes suivies :
    - `Nettoyage des paquets` : Nous commençons par nettoyer les paquets en utilisant la commande `yum clean`.
    - `Installation de device-mapper-persistent-data` : Ce paquet est essentiel pour le bon fonctionnement de Docker sur CentOS.
    - `Installation de lvm2` : C'est un autre paquet nécessaire pour Docker, et nous nous assurons qu'il est installé et à jour.
    - `Ajout du dépôt Docker` : Nous ajoutons le dépôt officiel de Docker pour CentOS afin de pouvoir installer Docker directement depuis ce dépôt.
    - `Installation de Docker` : Après avoir ajouté le dépôt, nous procédons à l'installation de Docker.
    - `Démarrage de Docker` : Enfin, nous nous assurons que le service Docker est démarré et fonctionne correctement sur le serveur.
    Le playbook utilise l'option `become: yes` pour exécuter toutes les tâches avec des privilèges d'administrateur, garantissant ainsi qu'il a les permissions nécessaires pour installer et configurer Docker.

### 3. Configuration des tâches docker_container

- Document your docker_container tasks configuration.

    Nous avons défini une tâche `docker_container` dans le rôle `launch_proxy` pour déployer un conteneur HTTPD. Dans cette tâche, nous avons spécifié le nom du conteneur comme `httpd` et utilisé l'image `jdoe/my-httpd:1.0`. Nous avons également exposé le port 80 du conteneur et l'avons lié au port 80 de l'hôte. De plus, ce conteneur est attaché au réseau nommé `mynetwork` que nous avons créé précédemment pour permettre la communication entre nos différents conteneurs.