# 📚 Guide de Configuration et Sécurisation de VPS

> Ce guide complet vous offre une vue d'ensemble des étapes nécessaires pour configurer et sécuriser vos VPS, de la création des clés SSH à l'installation de NGINX et Docker, en passant par la configuration du pare-feu. Suivez ces étapes pour une mise en place réussie et sécurisée de votre environnement serveur. 🚀💼🔧🌐🔒

---

## 📋 Pré-requis

Avant de commencer, assurez-vous de disposer des éléments suivants :

1. **Deux VPS avec Ubuntu Server pré-installé**.
2. **Accès root ou avec privilèges sudo**.
3. **Connaissance de base en ligne de commande Linux**.
4. **Un client SSH installé sur votre machine locale**.
    - Par exemple, PuTTY pour Windows ou le terminal intégré dans Linux et macOS.
    - Sur VS Code, 3 extensions bien utile:
        - Remote - SSH (_par Microsoft_)
        - Remote Explorer (_par Microsoft_)
        - Remote -- SSH:Editing Configuration Files
5. **Un compte GitHub pour les pipelines CI/CD avec GitHub Actions**.
6. **Connaissance de base de Docker et NGINX**.

Avec ces pré-requis en place, vous êtes prêt à commencer la configuration de vos VPS.
Suivez les étapes décrites dans ce guide pour une mise en place réussie et sécurisée de votre environnement serveur. 🚀💼🔧

---

## 🔑 1 - Création et Configuration des Clés SSH

### 1.1 Création d'une Paire de Clés SSH

```sh
# Générer une nouvelle paire de clés SSH
ssh-keygen -t ed25519 -C "un nom pour la décrire"
```

### 1.2 Copie de la Clé Publique sur le Serveur

```sh
# Copier la clé publique sur le serveur VPS
ssh-copy-id -i ./id_ed25519.pub utilisateur@ip_ovh
```

### 1.3 Installation Manuelle de la Clé (si nécessaire)

```sh
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

---

## 2 - Configuration Initiale des VPS

### 🌐 2.1 Connexion SSH Initiale

```sh
# Connexion au VPS via SSH
ssh ubuntu@adresse_ip_vps
```

-   Remplacez `adresse_ip_vps` par l'adresse IP de votre VPS.

### 🔐 2.2 Désactivation de l'Authentification par Mot de Passe

```sh
# Ouvrir le fichier de configuration SSH pour modification
sudo nano /etc/ssh/sshd_config

# Désactiver l'authentification par mot de passe
# Modifier les lignes suivantes :
PasswordAuthentication yes # Changez la valeur en mettant no
PermitRootLogin yes # Changez la valeur en mettant no

# Soit vous devriez obtenir:
PasswordAuthentication no
PermitRootLogin no

# Sauvegarder et fermer le fichier
```

### 🚪 2.3 Changement du Port SSH

> Veillez toutefois à ne pas renseigner un numéro de port déjà utilisé sur votre système.
> Pour plus de sécurité, utilisez un numéro entre **49152** et **65535**.

_Pour l'exemple nous utiliserons les port 50001 et 50002._

```bash
# Modifier le port SSH par défaut pour des raisons de sécurité
sudo nano /etc/ssh/sshd_config

# Changer le port SSH
# Pour VPS 1 : Port 50001
# Pour VPS 2 : Port 50002

# Redémarrer le service SSH pour appliquer les changements
sudo systemctl restart sshd
```

### 👤 2.4 Création d'un Nouvel Utilisateur

```sh
# Créer un nouvel utilisateur avec des privilèges sudo
sudo adduser nom_utilisateur
sudo usermod -aG sudo nom_utilisateur
```

### 2.5 Authentification différente

**ATENTION**, dorénavent pour se conecter au serveur SSH, il faudra faire ceci:

```bash
# Connexion au VPS n°1 via SSH avec le nouvel utilisateur et le nouveau port
ssh nom_utilisateur@adresse_ip_vps -p 50001

# Connexion au VPS n°1 via SSH avec le nouvel utilisateur et le nouveau port
ssh nom_utilisateur@adresse_ip_vps -p 50002
```

---

## 3 - Installation et Configuration de NGINX

### 🌍 3.1 Installation de NGINX

```bash
# Mettre à jour les paquets et installer NGINX
sudo apt update && sudo apt upgrade -y
sudo apt install nginx

# Démarrer et activer NGINX
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 📄 3.2 Configuration de NGINX pour Plusieurs Sites

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

## 4 - Installation de Docker sur le VPS

```bash
# Installer Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run

# Vérification que docker est bien installer
docker -v
```

[Source officiel](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)

### 🐳 4.1 Commandes Docker

```bash
# Télécharger une image nginx
docker pull nginx

# Lancer un conteneur avec l'image nginx
docker run --name docker-nginx -p 80:80 nginx

# Vérifier que NGINX fonctionne
sudo ps -ef | grep nginx

# Afficher les conteneurs actifs
docker container ls

# Afficher les images disponibles
docker images ls
```

### 4.2 Lancer le Service NGINX

```bash
# Démarrer le service NGINX
sudo service nginx start
```

---

## 🔥 5 - Sécurisation et Pare-feu

### 5.1 Configuration du Pare-feu UFW

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
