# SIRENE_as_API_ansible

Bienvenue !

Ce répertoire ansible a pour but de vous permettre de déployer une
infrastructure capable de faire tourner
[SIRENE_as_API](https://github.com/sgmap/sirene_as_api) sur un de vos serveurs.
SIRENE_as_API se met à jour automatiquement grâce à des taches cron.

Nous partons du principe que vous avez une distribution serveur de type Ubuntu 16.04,
il vous faudra potentiellement contribuer au code si vous souhaitez ajouter la
compatibilité pour d'autres distributions.

## Installation Ansible

Il vous faut la version 2.3 ou supérieure d'Ansible pour pouvoir exécuter le playbook
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

En fonction de votre systeme d'exploitation / distrubution, ansible va piocher les
différents hôtes à des endroits différents nommés inventaires. Vous pouvez
[jeter un oeil ici](http://docs.ansible.com/ansible/intro_inventory.html)

Le plus simple est encore d'exécuter

    echo 'MON_IP_OU_DOMAINE' >> inventories/sirene_as_api

Vous pouvez maintenant lancer la commande Ansible en lui indiquant d'utiliser
l'inventaire que vous venez d'ajouter :

    ansible-playbook sirene_as_api.yml -i inventories/sirene_as_api

L'infrastructure devrait maintenant s'installer d'un tour de main. Le déploiement de
l'application se fait par la gem [Mina](https://github.com/mina-deploy/mina)
(cf. plus bas).

## Configuration du playbook

### Configuration des variables de sécurité [!IMPORTANT!]

Rails nécessite une clef secrete pour mettre l'application en production.
Vous pouvez générer une clef en tapant "rake secret" dans n'importe quel projet rails.

De même, vous devez rentrer un mot de passe pour votre base de données. Le playbook
s'arretera si ces données sont manquantes.
Ces informations sont à renseigner dans les champs correspondants dans le fichier
sirene_as_api.yml.

Bien évidemment, il est très déconseillé de rendre ces informations publiques.
Ne réutilisez pas un mot de passe qui peut vous compromettre par ailleurs.

### User deploy

Suivant les bonnes pratiques, notre script ansible n'installe pas sur le server distant en root
mais crée un utilisateur "deploy" et lui donne les accès pour installer l'application.

### Installation de Python 2

Ansible a besoin de Python installé sur la machine distante pour fonctionner.
Cependant, dans certaines configurations de serveurs vides, il est possible
que Python ne soit pas présent.
Le playbook commence donc par un role qui va installer Python 2 sans avoir
besoin de Python. Vous pouvez commenter ce rôle si votre machine est déjà
équipée.

### Configuration Nginx

L'API est installée par défaut en environnement de production. Le web-server
Nginx sera également installé et configuré pour être immédiatement accessible.
Pour accéder à votre server web depuis votre nom de domaine, vous devez
le renseigner dans la variable client_server_name dans le fichier
sirene_as_api.yml.

## Deploiement de l'API par Mina

Une fois votre infrastructure installée, vous pouvez utiliser [Mina](https://github.com/mina-deploy/mina)
pour déployer l'application sur votre serveur. Le principe est le meme que pour Ansible.

Clonez le repo sirene_as_api sur votre ordinateur :

    git clone git@github.com:sgmap/sirene_as_api.git && cd sirene_as_api

Lancez bundle install pour récupérer les gems nécessaires, ceci peut prendre un
peu de temps en fonction de votre installation. Vous pouvez omettre l'installation de
bundler s'il est déjà présent sur la machine

    gem install bundler && bundle install

Mina devrait s'installer avec le reste des gems.
Ouvrez le fichier config/deploy.rb dans le repository sirene_as_api et renseignez
votre nom de domaine dans la ligne adéquate.

Lancez la création des dossiers par Mina :

    mina setup to=production

Vous pouvez ensuite lancer le déploiement :

    mina deploy to=production

L'application devrait s'installer correctement sur l'infrastructure préalablement déployée
par le playbook Ansible.

Attention, Mina peut vous demander un mot de passe pour l'utilisateur deploy.
Vous pouvez copier votre clef privée SSH pour l'utilisateur deploy, ou plus simplement
définir à cet utilisateur un mot de passe. En tant que root sur votre serveur, executez :

    passwd deploy

C'est prêt, votre application est fonctionnelle en production ! Les mises à jour de la base
se feront automatiquement par la suite.
