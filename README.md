# TP3 - Ansible

## Questions 

1. Document your inventory and base commands
L'inventaire est défini dans le fichier `setup.yml` sous le répertoire `inventories`. Dans cet inventaire :
- `ansible_user`: représente l'utilisateur avec lequel nous nous connectons à la machine distante, qui est `centos` dans notre cas.
- `ansible_ssh_private_key_file`: c'est le chemin vers notre clé SSH privée, qui est `~/.ssh/id_rsa` depuis que nous l'avons déplacée.
- Sous le groupe `prod`, nous avons défini notre serveur, qui est `kilian.codaccioni.takima.cloud`.
Commandes de base exécutées :
- `Ping`: Nous avons utilisé la commande `ansible all -i inventories/setup.yml -m ping` pour vérifier la connectivité à notre serveur. La réponse `pong` indique que la connexion est réussie.
- `Récupération des faits`: La commande `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"` a été utilisée pour récupérer des informations spécifiques sur la distribution du serveur. Il s'est avéré que notre serveur tourne sous CentOS 7.9.
- `Désinstallation d'Apache httpd`: Nous avons tenté de désinstaller Apache httpd avec la commande `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become`. Le résultat indique qu'Apache httpd n'est pas installé.

2. Document your playbook
Dans ce playbook, nous avons défini une série de tâches pour installer Docker sur notre serveur basé sur CentOS. Voici un résumé des étapes suivies :
- `Nettoyage des paquets` : Nous commençons par nettoyer les paquets en utilisant la commande `yum clean`.
- `Installation de device-mapper-persistent-data` : Ce paquet est essentiel pour le bon fonctionnement de Docker sur CentOS.
- `Installation de lvm2` : C'est un autre paquet nécessaire pour Docker, et nous nous assurons qu'il est installé et à jour.
- `Ajout du dépôt Docker` : Nous ajoutons le dépôt officiel de Docker pour CentOS afin de pouvoir installer Docker directement depuis ce dépôt.
- `Installation de Docker` : Après avoir ajouté le dépôt, nous procédons à l'installation de Docker.
- `Démarrage de Docker` : Enfin, nous nous assurons que le service Docker est démarré et fonctionne correctement sur le serveur.
Le playbook utilise l'option `become: yes` pour exécuter toutes les tâches avec des privilèges d'administrateur, garantissant ainsi qu'il a les permissions nécessaires pour installer et configurer Docker.

3. Document your docker_container tasks configuration.
Nous avons défini une tâche `docker_container` dans le rôle `launch_proxy` pour déployer un conteneur HTTPD. Dans cette tâche, nous avons spécifié le nom du conteneur comme `httpd` et utilisé l'image `jdoe/my-httpd:1.0`. Nous avons également exposé le port 80 du conteneur et l'avons lié au port 80 de l'hôte. De plus, ce conteneur est attaché au réseau nommé `mynetwork` que nous avons créé précédemment pour permettre la communication entre nos différents conteneurs.