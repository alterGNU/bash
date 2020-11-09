# 0 - Installation
Cette configuration personnelle de bash déclare et utilise une variable d'ENVIRONNEMENT nommée `BASH_DIR` correspondant
à la localisation du dossier `bash/`

## Installation standart - $BASH\_DIR:`$HOME/.bash/*`
Cloner le repo dans le home en tant que dotfile puis installer configuration :
```bash
$ git clone https://github.com/alterGNU/bash.git $HOME/.bash && $HOME/.bash/install.sh
```
## Installation en changeant localisation et nom par défaut:
- 1 : Cloner le repo. renommé à l'endroit souhaité:`$ git clone https://github.com/alterGNU/bash.git <emplacement>/<nom_dossier>`
    Example avec **localisation=~/.dotfiles** et **nom_dossier=bash_config**
    ```bash
    $ git clone https://github.com/alterGNU/bash.git ~/.dotfiles/bash_config
    ```
- 2 : Modifier la variable d'environnement $BASH\_DIR dans le fichier `bash_profile`, ainsi remplacer la ligne 11 par:
    ```bash
    export BASH_DIR="<localisation>/<nom_dossier>"
    ```
    _Avec notre exmaple précédent cela donne : `export BASH_DIR="~/.dotfiles/bash_config"`_
- 3 : Lancer l'installation : `$ <emplacement_choisi>/<nom_choisi>/install.sh`:
    example avec : localisation et nom = ~/.dotfiles/bash_config
    ```bash
    $ ~/.dotfiles/bash_config/install.sh
    ```

# 1 - BASH repo
Ce dossier regroupant l'ensemble de mes configurations personnels du shell bash contient :
- les fichiers de configuration de bash (**BDF** ~ _**B**ash **D**ot**F**iles_ ) :
    - `bash_profile` : **BDF** chargé à chaque session interactive loguée (bash source bash\_login).
    - `bashrc` : **BDF** chargé à chaque session interactive non-loguée.
    - `bash_logout` : **BDF** chargé à chaque fermeture de session bash loguée.
- des scripts :
    - `bash.sh` : script d'installation des **BDF**
    - `restore_bash.sh` : script permettant de remettre à l'état initial (avant installation) l'ensemble des **BDF**
- une collection de **shell-fct** (dossier `bash/bin/*`):
    - `coucou` : créée pour apprendre à déclarer mes propres fcts....
    - `nbr_agent-ssh` : Retourne le nombre d'agent ssh actif (methode:parcours `procfs`)
    - `check_git` : vérifie qu'un dossier est bien dans l'arborescence d'un dépôt git local

## 1.1 - Bash\_DotFiles
### RAPPELS : Fonctionnement et Ordre de chargement des BDF
Les fichiers lus dépendent du type de shell:
- **Non-interactive** : shell non associé à un terminal (cas lors d'éxécution de script)
- **Interactive** : shell qui lis et écris dans un terminal d'utilisateur (Interpréteur de commande)
    - Loging : cas de Co. locale ou SSH ou `$ bash --login`
    ORDRE : /etc/profile >> ~/.bash_profile >> ~/.bash_login >> ~/.profile ( -f .bash_profil => .profil pas sourcé)
    - Non\_loging: `$ bash`
    ORDRE : ~/.bashrc >> ~/.bash_login >> ~/.profile

### Le fichier `~/.bash_profile` _(Interactive loging shell=>Link ds le HOME)_
Ce fichier est le premier sourcé au démarrage d'une session interactive loguée :`$ bash --login`
On doit y placer les commandes ne devant s'exécutées qu'une seule fois, il faut donc y placer :
- les déclarations des **VarENV** :
    - BASH\_DIR : localisation des BDF personnalisés
    - HISTFILE : localisation du fichier historique `.bash_history`
- les commandes **sourçants** les fichiers devant l'être :
    - `${BASH_DIR}/bashrc` : pour charger les configurations de bash
    - `${BASH_DIR}/aliases`: pour charger les alias de bash
- les commandes ajoutant des dossiers au **$PATH**
    - `${BASH_DIR}/bin` : pour charger les fonctions personnelles de bash

### Le fichier `bashrc` : *(Sourcé par bash_profile)*
Y placé les commandes devant s'éxécutées à chaque fois qu'on démarre le shell comme les **options et configurations** de
bash.

Normalement le bashrc se trouve dans le home et est chargé lors des Interactive non-loging shell.
Cependant dans notre installation, il est sourcé pas bash\_profile et reste dans le $BASH\_DIR afin de limiter le nombre
de dotfiles présent dans le $HOME...

Cela à pour principal conséquence de le rendre inutilisé par les session interactive non loguée du shell :
- Exécution de script
- Lancement de bash via la commande `$ bash`

Il est bon de noter qu'il est facile de corriger ce problème en appliquant une des deux solutions suivantes:
- 1) Créer un lien symbolique pointant vers lui dans le home : `$ ln -s $BASH_DIR/bashrc $HOME/.bashrc`
- 2) Utiliser l'option suivante: `$ bash --rcfile "$BASH_DIR/bashrc`

#### Personnalisation du prompt
Dans le bashrc se trouvent deux fonctions (`git_color`;`git_branch`) permettant d'afficher dans le prompt l'état du
dépot git local:
- vert: le dépot local est à jour
- rouge: le dépot local n'est pas à jour (commit nécessaire)

### Le fichier `/aliases` : *(Sourcé par bash\_profile)*
Contient l'ensemble des alias (certains définis conditionnellement grace à l'encapsulation dans des fonctions applées
après des tests.

#### Cas particulier, alias alert utilisant notify-send sous WSL2...
Si on veux utiliser cet alias (mis par défaut) on se rend compte qu'il faut installer 415
paquets (dont ceux pour serveur X11) et le tout sans grande chance de fonctionner!
Une alternative serait d'utiliser les outils analogue sous windows comme [BurntToast](https://github.com/Windos/BurntToast)
(Voir l'astuce de [Blog CodeLearn](https://codelearn.me/2019/01/13/wsl-windows-toast.html)

### `~/.bash_logout` : _(login shell=>Link ds le HOME)_
Y placer les commandes que l'on souhaite exécuter lors de la déconnexion de la session.

#### tuer le ssh-agent!
A la déconnexion, en utilisant la fct-bash `check_ssh` dans un test, on peut tuer l'agent ssh en cours!
```bash
[[ check_ssh ]] && eval $(ssh-agent -k)
```
### Le fichier `history`
Correspond au fichier contenant l'historique des commandes de bash.
Par défaut ce fichier est `$HOME/.bash_history`, cependant, pour ne pas surcharger le $HOME de dotfiles on préfère le
placé dans le `$BASH_DIR/history`

## 1.2 - Fonctionnement des SCRIPTS
### `install.sh`
Ce script permet l'installation des configurations personnelles de bash, pour cela il:
- 1 : **Vérifie** que le shell utilisé est bien **bash**, sinon quitte l'éxécution en produisant l'erreur 66
- 2 : **Archive** dans le dossier ~/.backupfiles/ (le crée si besoin) l'ensemble des BDF actuels.
- 3 : **Crée** deux liens symboliques (indispensable) dans le $HOME : `~/.bash_profil` & `~/.bash_logout`

Afin de limiter au maximum la surcharge de $HOME par des dotfiles inutiles, on se limite aux deux liens ci-dessous.
Ceci est possible car `bash_login` source les fichiers `bashrc` et `aliases` _(inutile donc qu'ils soient présent dans
le $HOME)_

### `restore_bash.sh`
Ce script permet "d'annuler" les modifications apportées par `install.sh`, pour ce faire il:
- 1 : **Supprime** les liens symboliques pointant vers des BDF s'ils existent
- 2 : **Restaure** dans le home l'ancienne version des BDF souhaitèes 
- 3 : **Supprime** le dossier de sauvegarde ayant été créé aujourdhui... (utile pour phase de test successifs)

_Pour choisir le dossier contenant les bashdotfiles s'il en existe plusieurs, changer ds le script `${OriginFolder}`_

## 1.3 Fonctions et Commandes BASH
### `$ coucou`
Fonction créée pour apprendre les règles de base d'écriture et d'éxécution de commande shell.(utilisation argument)

- Si aucun argument, dit bonjour à l'utilisateur
    ```bash
    $ coucou 
    Bonjour alter.GNU
    ```

- Si argument passé (autre que les options acceptées) fait le perroquet!
    ```bash
    $ coucou "Y a quelqu'un qui ma dit que..."
    "Y a quelqu'un qui ma dit que..."
    ```

- La commande accepter deux options `-q/--quit` ou `-h/--help`
    ```bash
    $ coucou -q "Y a quelqu'un qui ma dit que..."
    ssshhut!
    $ coucou -h
    ...
    AFFICHE L'USAGE
    ...
    ```
### `$ nbr_agent-ssh` 
**Retourne le nombre d'agent ssh actif (methode parcourant procfs).**

Cette fonction, peut donc être utiliser comme une fonctionnelle/un test!

Elle parcours le **procfs** à la recherche de processus s'appelant `(ssh-agent)`, si elle en trouve elle incrémente une
variable...puis la retourne.

Ainsi s'il n'y a pas d'SSH-AGENT en cours, elle retourne 0, sinon elle retourne le nombre de processus portant ce nom!

À l'instar des autre fonctions elle accepte en options:
- `-v/--verbose`: la rendant tchatty!
-  `-h/--help`: affiche l'usage

### `$ check_git` 
**Vérifie qu'un dossier est bien dans l'arborescence d'un dépôt git local**:
- **Sans argument** : 
    - retourne 0 si on se trouve dans un dossier git
    - retourne 8 si pas dans un dossier git
- **Avec argument** : 
    - retoune 0 si l'argument est un dossier git
    - retourne 28 si le dossier passé en paramètre n'est pas un dossier git

_Ainsi tout autre retour d'erreurs (différents de 8 ou 28) correspond à une mauvaise utilisation de la commande_

Elle admet aussi deux options :
- `-v/--verbose`: la rendant tchatty!
-  `-h/--help`: affiche l'usage

# Sources
- [Bash startup files](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html#Bash-Startup-Files)
- [bashrc vs bash_profile](https://linuxize.com/post/bashrc-vs-bash-profile/)
- [Create bash aliases](https://linuxize.com/post/how-to-create-bash-aliases/)
- [Create bash functions](https://linuxize.com/post/bash-functions/)
