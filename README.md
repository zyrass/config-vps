# 📚 Guide de Configuration et Sécurisation d'un VPS

> Ce guide complet vous offre une vue d'ensemble des étapes nécessaires pour configurer et sécuriser vos VPS, de la création des clés SSH à l'installation de NGINX et Docker, en passant par la configuration du pare-feu. Suivez ces étapes pour une mise en place réussie et sécurisée de votre environnement serveur. 🚀💼🔧🌐🔒

---

## 📋 Pré-requis

Avant de commencer, assurez-vous de disposer des éléments suivants :

1. **Un VPS avec Ubuntu Server pré-installé**.
2. **Accès root ou avec privilèges sudo**.
3. **Connaissance de base en ligne de commande Linux**.
4. **Un client SSH installé sur votre machine locale**.
    - Par exemple, PuTTY pour Windows ou le terminal intégré dans Linux et macOS.
    - Sur VS Code, 6 extensions bien utile que vous trouverez sur le [marketplace officiel](https://marketplace.visualstudio.com/vscode):
        - [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
        - [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)
        - [Remote Exporer](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-explorer)
        - [Remote - SSH:Editing Configuration Files](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh-edit)
        - [Remote - Tunnels](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server)
        - [Remote Repositories](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-repositories)
5. **Connaissance de base de Docker et NGINX**.

Avec ces pré-requis en place, vous êtes prêt à commencer la configuration de vos VPS.
Suivez les étapes décrites dans ce guide pour une mise en place réussie et sécurisée de votre environnement serveur. 🚀💼🔧

---

## 1 - Configuration Initiale de son VPS

1. **🌐 Connexion SSH Initiale**
2. **Mettre à jour son VPS**
3. **🚪 Modifier le port d'écoute SSH par défaut**
4. **🔑 Création et Configuration des Clés SSH**
5. **Copie de la Clé Publique sur le Serveur**
6. **Installation Manuelle de la Clé (_si nécessaire_)**
7. **👤 Création d'un Nouvel Utilisateur**
8. **🔐 Désactivation de l'Authentification par Mot de Passe**

### 1.1 - 🌐 Connexion SSH Initiale

```sh
# Connexion au VPS via SSH
ssh utilisateur@adresse_ip_vps
```

-   Remplacez `utilisateur` par le nom d'utilisateur qui vous a été attribué (**_ubuntu_**) sur ovh.
-   Remplacez `adresse_ip_vps` par l'adresse IP de votre VPS qui vous a été transmit par mail.

### 1.2 - Mettre à jour son VPS

**Avoir votre distribution ou système d'exploitation à jour est un point essentiel pour sécuriser votre VPS.**
En effet, les développeurs de distributions et de systèmes d’exploitation proposent de fréquentes mises à jour de paquets, très souvent pour des raisons de sécurité.

-   Deux étapes sont nécessaires:
    -   **Etape 1** - Mise à jour de la liste des paquets : `sudo apt update`
    -   **Etape 2** - Mise à jour des paquets à proprement parler : `sudo apt upgrade -y`

```bash
# Bonus - Mix des deux commandes en une
sudo apt update && sudo apt upgrade -y
```

🔥 **IMPORTANT** - **Cette opération doit être effectuée régulièrement afin de maintenir un système à jour**.

### 1.3 - Modifier le port d'écoute SSH par défaut 🚪

L'une des premières actions à effectuer sur votre serveur est la configuration du port d'écoute du service SSH.
**Par défaut, celui-ci est défini sur le port 22 donc les tentatives de hack du serveur par des robots vont cibler ce port en priorité**. La modification de ce paramètre, au profit d'un port différent, est une mesure simple pour renforcer la protection de votre serveur contre les attaques automatisées.

Pour cela, modifiez le fichier de configuration du service avec l'éditeur de texte de votre choix (_**nano** est utilisé dans cet exemple_) :

```bash
# Passer en administrateur
sudo su

# Edition du fichier sshd_config se trouvant dans le répertoire /etc/ssh/
cd /etc/ssh

# Voir le contenu du répertoire ssh
ls ssh

# Edition du fichier de configuration avec nano
nano sshd_config
```

Vous devriez trouver les lignes suivantes ou équivalentes :

```sh
# ... d'autres lignes de code

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

# ... encore d'autres lignes de code
```

> Veillez toutefois à ne pas renseigner un numéro de port déjà utilisé sur votre système.
> Pour plus de sécurité, utilisez un numéro entre **49152** et **65535**.

_Pour l'exemple nous utiliserons le port 50001 pour nos tests, mais avant nous aurons décommentez la ligne correspondante soit :_

```bash
# ... d'autres lignes de code

Port 50001
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

# ... encore d'autres lignes de code
```

Remplacez le nombre `22` par le numéro de port de **votre choix** (**_pour nous ici il s'agira du port 50001_**).
Enregistrez avec la combinaison suivante `CTRL + S` et quittez le fichier de configuration avec la combinaison suivante `CTRL + X`.

```bash
# Redémarrer le service SSH pour appliquer les changements
sudo systemctl restart sshd

# Cela devrait être suffisant pour appliquer les changements.
# Dans le cas contraire, redémarrez le VPS avec cette commande :
sudo reboot
```

⚠️ **ATENTION**
Vous devrez indiquer le nouveau port à chaque demande de connexion SSH à votre serveur, par exemple :

```bash
# Connexion au VPS via SSH avec le nouveau port
ssh utilisateur@adresse_ip_vps -p 50001
```

### 1.4 - 🔑 Création et Configuration des Clés SSH

```bash
# Générer une nouvelle paire de clés SSH sur sa machine en local
ssh-keygen -t ed25519 -C "un nom pour la décrire"
```

### 1.5 - Copie de la Clé Publique sur le Serveur

> ⚠️ _**ATTENTION** A BIEN COPIER EXCLUSIVEMENT LA CLE **id_ed25519.pub**_

```bash
# Copier la clé publique sur le serveur VPS
ssh-copy-id -i ./id_ed25519.pub utilisateur@ip_ovh
```

### 1.6 - Installation Manuelle de la Clé (si nécessaire)

sur windows, il est possible que la commande `ssh-copy-id` ne soit pas reconnue, ainsi donc veillez suivre les étapes suivantes:

```bash
# Connexion au serveur VPS via SSH
ssh utilisateur@ip_ovh

# Accéder au dossier .ssh de l'utilisateur
cd ~/.ssh

# Éditer le fichier authorized_keys pour ajouter la clé publique
sudo nano authorized_keys

# Copier manuellement la clé publique générée à l'étape 1
# Enregistrer avec "CTRL + S" et quitter avec "CTRL + X"

# Afficher le contenu de authorized_keys pour vérifier
cat authorized_keys
```

### 1.7 - 👤 Création d'un Nouvel Utilisateur

```bash
# Créer un nouvel utilisateur avec des droits restreints
sudo adduser nom_utilisateur

# Créer un nouvel utilisateur avec des privilèges sudo
sudo adduser nom_utilisateur
sudo usermod -aG sudo nom_utilisateur
```

### 1.8 - 🔐 Désactivation de l'Authentification par Mot de Passe

Après avoir défini nos clés SSH et explusivement nos clé SSH, on pourra supprimer la connexion par mot de passe.
Pour celà, nous devons à nouveau modifier le fichier que l'on a modifié à l'étape `1.2`.

```bash
# Déplacement dans le répertoire correspondant
cd /etc/ssh

# Ouvrir le fichier de configuration SSH pour modification
sudo nano sshd_config
```

Dans le fichier, modifié l'authentification par mot de passe :

```bash
# Modifier les lignes suivantes : (Origine)
PasswordAuthentication yes # Changez la valeur en mettant no
PermitRootLogin yes # Changez la valeur en mettant no

# Soit vous devriez obtenir: (Modifié)
PasswordAuthentication no
PermitRootLogin no
```

Enregistrer de nouveau le fichier avec la combinaison suivante `CTRL + S` et fermer nano avec la cette autre combinaison de touches `CTRL + X`.

---

## 2 - Installation et Configuration de NGINX

### 🌍 2.1 Installation de NGINX

```bash
# Mettre à jour les paquets et installer NGINX
sudo apt update && sudo apt upgrade -y
sudo apt install nginx

# Démarrer et activer NGINX
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 📄 2.2 Configuration de NGINX pour Plusieurs Sites

> **NOTE:**
> Répétez chacune des étapes suivantes pour chaque site

```sh
# Créer un répertoire pour le site et configurer les permissions
sudo mkdir -p /var/www/site1.com/html
sudo chown -R $USER:$USER /var/www/site1.com/html

# Créer et éditer la configuration du site dans NGINX
sudo nano /etc/nginx/sites-available/site1.com

# Activer le site en créant un lien symbolique
sudo ln -s /etc/nginx/sites-available/site1.com /etc/nginx/sites-enabled/

# Tester et redémarrer NGINX pour appliquer les changements
sudo nginx -t
sudo systemctl restart nginx
```

---

## 3 - Installation de Docker sur le VPS

```bash
# Installer Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run

# Vérification que docker est bien installer
docker -v
```

[Source officiel](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)

### 🐳 3.1 Commandes Docker

#### 3.1.1 Windows

```bash
# Télécharger une image nginx
docker pull nginx

# Lancer un conteneur avec l'image nginx
docker run --name docker-nginx -p 80:80 nginx

# Vérifier que NGINX fonctionne
sudo ps -ef | grep nginx

# Afficher toutes les VM lancés
docker ps

# Afficher les conteneurs actifsAjout du fichier changelog
docker container ls

# Afficher les images disponibles
docker images ls

# Stoper un container (ici docker-nginx défini plus haut)
docker stop docker-nginx

# Supprimer un container
docker rm docker-nginx
```

**Ouvrir un navigateur et saisir localhost dans la barre d'adresse afin de voir le serveur ngnx de lancé**

### 3.2 Lancer le Service NGINX

Uniquement si le serveur n'est pas démarré.

```bash
# Démarrer le service NGINX
sudo service nginx start
```

---

## 🔥 4 - Sécurisation et Pare-feu

### 4.1 Configuration du Pare-feu UFW

```bash
# Installer et configurer UFW (Uncomplicated Firewall)
sudo apt install ufw

# Autoriser le trafic SSH sur les nouveaux ports
sudo ufw allow 50001/tcp # Pour le premier VPS
sudo ufw allow 50002/tcp # Pour le second VPS

# Autoriser le trafic pour NGINX
sudo ufw allow 'Nginx Full'

# Activer UFW
sudo ufw enable

# Vérifier le statut du pare-feu
sudo ufw status
```
