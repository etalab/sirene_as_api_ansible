# SIRENE_as_API_ansible

Bienvenue !

Ce répertoire ansible a pour but de vous permettre de déployer une
infrastructure capable de faire tourner
[SIRENE_as_API](https://github.com/sgmap/sirene_as_api) sur un de vos serveurs.
SIRENE_as_API se met à jour automatiquement grâce à des taches cron.

Nous partons du principe que vous avez une distribution serveur de type Debian,
il vous faudra potentiellement contribuer au code si vous souhaitez ajouter la
compatibilité pour d'autres distributions.

## Installation Ansible

Il vous faut la version 2.2 d'Ansible pour pouvoir exécuter le playbook
sirene_as_api.yml. Ansible s'installe sur votre machine de développement et ne
nécessite pas d'installation sur votre serveur distant.

#### Mac OS X

Vous pouvez utiliser le gestionnaire de paquets
[Brew](https://brew.sh/index_fr.html). Une fois installé, tapez la commande
suivante :

    brew install ansible

#### Linux

La documentation officielle est votre amie. Vous y trouverez les
[instructions d'installation](http://docs.ansible.com/ansible/intro_installation.html#getting-ansible).


#### Windows

Rebootez sous linux ou mac os et reportez vous aux sections précédentes =)


## Installation de la machine distante via Ansible

Les scripts de déploiements ansible sont appelés des playbooks. Le playbook
nécessaire que nous utiliserons est sirene_as_api.yml. Dans un playbook on
spécifie entre autres sur quelle(s) type de machines distantes l'éxécuter ainsi que
le compte utilisateur de connexion sur ces machines : Il faut donc que
Ansible puisse se connecter.

Nous utilisons des connexions à base de clé ssh seules, et sans mots de passe.
Si d'aventure vous avez besoin d'un mot de passe pour vous connecter à root, il
vous faudra utiliser l'option `--ask-sudo` dans vos appels à ansible-playbook.
Ansible vous donnera la main pour taper votre mot de passe au bon moment

Notre playbook utilise des roles de deux sortes :
* définis localement
* disponibles sur ansible galaxy

Installons donc ces roles sur notre machine

    ansible-galaxy install geerlingguy.ruby
    ansible-galaxy install geerlingguy.redis

En fonction de votre systeme d'exploitation / distrubution, ansible va piocher les
différents hôtes à des endroits différents nommés inventaires. Vous pouvez
[jeter un oeil ici](http://docs.ansible.com/ansible/intro_inventory.html)

Le plus simple est encore d'exécuter
    echo 'MON_IP_OU_DOMAINE' >> inventories/sirene_as_api

Nous allons maintenant lancer la commande Ansible en lui indiquant quel
inventaire choisir

    ansible-playbook sirene_as_api.yml -i inventories/sirene_as_api

Tout devrait maintenant s'installer d'un tour de main. Les étapes concernant le
logiciel a proprement parler sont sur le repo Github
[sirene_as_api](https://github.com/sgmap/sirene_as_api)

#### Quelques précisions

Si l'on regarde en détail, notre script installe des paquets cosmétiques, de la
configuration vim, zsh etc. Vous pouvez tout aussi bien vous en passer en
commentant les appels à ces tâches dans les fichiers.
Les commentaires se font grâce au '#'
