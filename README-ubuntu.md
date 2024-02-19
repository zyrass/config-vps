# 📚 Guide Complet de Configuration et Sécurisation d'un VPS avec NGINX et NGINX Amplify

> Ce guide approfondi vous guide pas à pas dans le processus de configuration et de sécurisation de votre VPS. Il couvre tout, depuis la création des clés SSH jusqu'à l'installation de NGINX, en passant par NGINX Amplify – un agent performant et optimisé, ainsi que la configuration du pare-feu. Chaque étape est minutieusement détaillée pour assurer une mise en place efficace et une sécurité optimale de votre environnement serveur. 🚀💼🔧🌐🔒

---

## A EDITER

```sh
scp -r C:/Users/toto/Documents/www/Vue/projects/nom_projet/client-build/* demo@WW.XX.YY.ZZ:/var/www/nom_projet.fr/html
```

## 📋 Pré-requis

Avant de commencer, vérifiez que vous disposez de :

1. **Un VPS avec Ubuntu Server déjà installé**.
2. **Accès root ou avec des privilèges sudo**.
3. **Connaissances de base en ligne de commande Linux**.
4. **Un client SSH sur votre machine locale**

    - Par exemple, PuTTY pour **_Windows_** ou le terminal intégré dans Linux et macOS.
    - Sur VS Code, utilisez ces extensions utiles disponibles sur le [marketplace officiel](https://marketplace.visualstudio.com/vscode) :
        - [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
        - [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)
        - [Remote Exporer](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-explorer)
        - [Remote - SSH:Editing Configuration Files](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh-edit)
        - [Remote - Tunnels](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server)
        - [Remote Repositories](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-repositories)

5. **Notions de base sur Docker et NGINX**.

Avec ces pré-requis en place, vous êtes prêt à commencer la configuration de votre VPS.
Suivez les étapes décrites dans ce guide pour une mise en place réussie et sécurisée de votre environnement serveur. 🚀💼🔧

---

## 1 - Configuration Initiale de son VPS

1. **🌐 Connexion SSH Initiale**
    1. **⬆️ Mettre à jour son VPS**
    2. **🚪 Modifier le port d'écoute SSH par défaut**
    3. **🔑 Création et Configuration des Clés SSH**
    4. **🔒 Copie de la Clé Publique sur le Serveur**
    5. **🛠️ Installation Manuelle de la Clé (_si nécessaire_)**
    6. **👤 Création d'un Nouvel Utilisateur**
    7. **🔐 Désactivation de l'Authentification par Mot de Passe**
2. **🚀 Installation de NGINX Ampify**

### 1.1 - 🌐 Connexion SSH Initiale

```sh
# Connexion au VPS via SSH via un terminal
ssh utilisateur@adresse_ip_vps
```

-   Remplacez **_`utilisateur`_** par votre nom d'utilisateur (ex: **_`ubuntu`_** pour un système Ubuntu sur un VPS chez OVH).
-   Remplacez **_`adresse_ip_vps`_** par l'adresse IP de votre VPS qui vous aura été communiquée par email.

### 1.2 - Mettre à jour son VPS

Mettre à jour régulièrement son système est crucial pour la sécurité du VPS.
En effet, les développeurs de distributions et de systèmes d’exploitation proposent de fréquentes mises à jour de paquets, très souvent pour des raisons de sécurité.

```bash
# Mise à jour de la liste des paquets et des paquets eux-mêmes
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

🔥 **IMPORTANT** - **Effectuez régulièrement cette opération pour maintenir un système à jour**.

### 1.3 - 🚪 Modification du Port d'Écoute SSH par Défaut

Changer le port SSH par défaut (**22**) au profit d'un port différent réduit le risque d'attaques automatisées. (_tentatives de hack du serveur par des robots_)
Utilisez un port non-standard, de préférence entre **49152** et **65535**.

> La modification de ce paramètre, a, est une mesure simple pour renforcer la protection de votre serveur contre les attaques automatisées.
> **Pour cela, il faut modifier le fichier de configuration du service avec l'éditeur de texte de votre choix (\_**nano** est utilisé dans le terminal dans cet exemple\_) :**

**Exemple ici avec la définition du port 50001** :

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

Remplacez par

```bash
# ... d'autres lignes de code

Port 50001
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

# ... encore d'autres lignes de code
```

Donc ici, nous avons remplacé le nombre **_`22`_** par le numéro de port de suivant **_`50001`_**.
Enregistrez (**_`CTRL + S`_**) et quittez (**_`CTRL + X`_**). Puis effectuez la commande suivante :

```bash
# Redémarrer le service SSH pour appliquer les changements.
sudo systemctl restart sshd

# En cas de problème, redémarrez le VPS : sudo reboot
# Cela devrait être suffisant pour appliquer les changements.
# Dans le cas contraire, redémarrez le VPS avec cette commande :
sudo reboot
```

⚠️ **ATENTION** - Pour se connecter après ce changement :

```bash
# Connexion au VPS via SSH avec le nouveau port
ssh utilisateur@adresse_ip_vps -p 50001
```

_Vous devrez indiquer le nouveau port à chaque demande de connexion SSH à votre serveur._

### 1.4 - 🔑 Création et Configuration des Clés SSH

```bash
# Générez une paire de clés SSH sur votre machine locale
ssh-keygen -t ed25519 -C "description_de_la_cle"
```

### 1.5 - Copie de la Clé Publique sur le Serveur

> ⚠️ **ATTENTION** - Assurez-vous de copier uniquement la clé **id_ed25519.pub**

```bash
# Copier la clé publique sur le serveur VPS
ssh-copy-id -i ./id_ed25519.pub utilisateur@ip_ovh
```

### 1.6 - Installation Manuelle de la Clé (Si Nécessaire)

Sur Windows, si `ssh-copy-id` n'est pas disponible, suivez ces étapes :

```bash
# Connexion au serveur VPS via SSH
ssh utilisateur@ip_ovh

# Accéder au dossier .ssh de l'utilisateur
cd .ssh

# Afficher le contenue du répertoire caché ".ssh"
ls -lah

# Éditer le fichier authorized_keys pour ajouter la clé publique
sudo nano authorized_keys

# Copier manuellement la clé publique générée à l'étape 1
# Enregistrer avec "CTRL + S" et quitter avec "CTRL + X"

# Afficher le contenu de authorized_keys pour vérifier
cat authorized_keys
```

### 1.7 - 👤 Création d'un Nouvel Utilisateur

```bash
# Pour ajouter un utilisateur avec des droits restreints:
sudo adduser nom_utilisateur

# Pour ajouter un utilisateur avec des droits sudo :
sudo adduser nom_utilisateur
sudo usermod -aG sudo nom_utilisateur
```

### 1.8 - 🔐 Désactivation de l'Authentification par Mot de Passe

Pour plus de sécurité, désactivez l'authentification par mot de passe.
Pour celà, nous devons à nouveau modifier le fichier que l'on a modifié à l'étape `1.2`.

> ⚠️ **ATTENTION** - Assurez-vous d'avoir copier votre clé ssh. Auquel cas la connexion sera impossible.

```bash
# Déplacement dans le répertoire correspondant
cd /etc/ssh

# Ouvrir le fichier de configuration SSH pour modification
sudo nano sshd_config
```

Modifiez :

```bash
PasswordAuthentication yes
PermitRootLogin yes
```

en :

```bash
PasswordAuthentication no
PermitRootLogin no
```

Enregistrez (`CTRL + S`) et quittez (`CTRL + X`), puis redémarrez SSH.

---

## 2 - Installation et Configuration de NGINX

### 🌍 2.1 Installation de NGINX

```bash
# Mettre à jour les paquets et installer NGINX
sudo apt update && sudo apt upgrade -y && sudo apt install nginx

# Démarrer et activer NGINX
sudo systemctl start nginx # Démarre
sudo systemctl enable nginx # Active au démarrage
```

### 📄 2.2 Configuration de NGINX pour Plusieurs Sites

**NOTE:** - Répétez ces étapes pour chaque site :

```bash
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

# Vérification de l'installation
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

**Visitez `localhost` dans votre navigateur pour voir NGINX en action.**

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
sudo ufw allow 50002/tcp # Pour le second VPS (si il y en a un)

# Autoriser le trafic pour NGINX
sudo ufw allow 'Nginx Full'

# Activer UFW
sudo ufw enable

# Vérifier le statut du pare-feu
sudo ufw status
```

---

## 5 - Nginx Amplify

### 5.1 - Installation de NGINX Amplify 🔧

NGINX Amplify est un outil performant offrant des métriques détaillées pour surveiller l'état de votre serveur NGINX.

Pour l'installer :

1. **Créez un compte sur NGINX Amplify** 📝: Rendez-vous sur [le site de NGINX Amplify](https://www.nginx.com/products/nginx-amplify/) pour vous inscrire.
2. **Téléchargez et installez l'agent NGINX Amplify**

Connectez-vous en SSH à votre serveur et suivez ces étapes :

-   Connectez-vous en tant que root 👤:

```bash
# Au cas où, voici la commande pour se connecter en root
sudo su
```

-   Téléchargez 📥 le script d'installation.
-   Exécutez la commande d'installation en tant que root 👨‍💻.
-   Suivez les instructions à l'écran et cliquez sur `Continue` ✅.

Pour vérifier et gérer l'agent NGINX Amplify :

```bash
# Vérifier si l'agent NGINX Amplify est installé et actif 🔍
sudo service amplify-agent status

# Démarrer l'agent NGINX Amplify, si nécessaire 🟢
sudo service amplify-agent start

# Arrêter l'agent NGINX Amplify 🔴
sudo service amplify-agent stop
```

### 5.2 - Configuration Nginx pour visualiser les métriques 📊

Pour configurer NGINX afin de visualiser les métriques avec NGINX Amplify, suivez ces étapes : (_Header violet_)

Il faut simplement suivre chacune des étapes affiché à l'écran

1. Accédez au répertoire de configuration de NGINX 📁:
    ```bash
    cd /etc/nginx
    ```
2. Vérifiez que les fichiers de configuration additionnels sont inclus 🔍:
    ```bash
    grep -i include\.*conf nginx.conf
    ```
3. Créez une nouvelle configuration pour `stub_status` 📝:
    ```bash
    cat > conf.d/stub_status.conf
    ```
    Puis, copiez et collez le contenu suivant 📋:
    ```nginx
    server {
        listen 127.0.0.1:80;
        server_name 127.0.0.1;
        location /nginx_status {
            stub_status on;
            allow 127.0.0.1;
            deny all;
        }
    }
    ```
4. Vérifiez et affichez le contenu du fichier de configuration 🖥️:
    ```bash
    ls -la conf.d/stub_status.conf && cat conf.d/stub_status.conf
    ```
5. Rechargez la configuration NGINX 🔁:
    ```bash
    kill -HUP `cat /var/run/nginx.pid`
    ```
