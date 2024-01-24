# Guide Complet de Configuration et Sécurisation d'un VPS sur Rocky Linux 9

> Ce guide approfondi vous guide pas à pas dans le processus de configuration et de sécurisation de votre VPS sous Rocky Linux 9. Il couvre tout, depuis la création des clés SSH jusqu'à la création d'un compte utilisateur en passant par la suppression d'une authentification par mot de passe. Chaque étape est minutieusement détaillée pour assurer une mise en place efficace et une sécurité optimale de votre environnement de test ou de production. 🚀💼🔧🌐🔒

---

## 📋 Pré-requis

Avant de commencer, vérifiez que vous disposez de :

1. **Un VPS avec Rocky Linux 9 déjà installé**.
2. **Accès root ou avec des privilèges sudo**.
3. **Connaissances de base en ligne de commande Linux**.
5. **Un client SSH sur votre machine locale**

    - Par exemple, PuTTY pour Windows ou le terminal intégré dans Linux et macOS.
    - Sur VS Code, utilisez ces extensions utiles disponibles sur le [marketplace officiel](https://marketplace.visualstudio.com/vscode) :
        - [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
        - [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)
        - [Remote Exporer](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-explorer)
        - [Remote - SSH:Editing Configuration Files](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh-edit)
        - [Remote - Tunnels](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server)
        - [Remote Repositories](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-repositories)

Avec ces pré-requis en place, vous êtes prêt à commencer la configuration de votre VPS. Suivez ce guide, étape par étape pour une mise en place réussie et sécurisée de votre environnement. 🚀💼🔧

---

> **Pour sécuriser le contenu qui sera le mien, je vais utiliser un pseudo et une ip random.**

-   **Pseudo utilisé prochainement** : `guidevps` (_en minuscule_)
-   **IP utilisée** : `50.60.70.80` (_Avec mes tests cette ip n'est pas utilisé_)

🔴 **NOTE IMPORTANTE** 🔴
**Il va de soit qu'il vous faudra utiliser vos identifiants qui vous auront été transmis par mail. (par défaut sur OVH, `rocky` sera le nom d'utilisateur lambda)**

---

## 1 - Configuration Initiale de son VPS

1. **🌐 Connexion SSH Initiale**

    1. **🔄 Mettre à jour son VPS** - (_à effectuer régulièrement_)
    2. **📦 Paquets à installer sur Rocky Linux 9**
        - nano (_éditeur en ligne de commande_)
    3. **👤 Création d'un Nouvel Utilisateur**
    4. **🔑 Création et Configuration des Clés SSH**
    5. **🔗 Copie de la Clé Publique sur le Serveur**
    6. **🔧 Installation Manuelle de la Clé (_si nécessaire_)**
    7. **🚪 Modifier le port d'écoute SSH par défaut**
    8. **🔐 Désactivation de l'Authentification par Mot de Passe**
    2. **Installation de NGINX Ampify**

### 1.1 - 🌐 Connexion SSH Initiale

✅ _unique rappel sur le faite que nous utiliserons uniquement un pseudo et une ip fictive tout au long de ce guide et qu'il devra être remplacer par le votre_ (**guidevps**).
_Veillez à remplacer ces identifiants de connexion par les vôtres_.

```sh
# Connexion au VPS via SSH (Toute première fois)
ssh rocky@50.60.70.80
```

-   Le nom d'utilisateur est toujours le compte fourni par votre prestataire. (ex: `rocky` pour un serveur VPS chez OVH).
-   Remplacez `50.60.70.80` par l'adresse IP de votre VPS, qui vous aura été communiquée par email.

### 1.2 - Mettre à jour son VPS

🔥 Mettre à jour régulièrement votre système est crucial pour la sécurité du VPS.
En effet, les développeurs de distributions et de systèmes d’exploitation proposent de fréquentes mises à jour de paquets, très souvent pour des raisons de sécurité.

```bash
# Mise à jour de la liste des paquets, mise à niveaux des paquets eux-mêmes et suppressions des paquets inutiles
sudo dnf update -y && sudo dnf upgrade -y && sudo dnf autoremove -y
```

> **DNF** est le gestionnaire de paquets pour rocky linux 9

### 1.3 - Installer des paquets facultatif mais utile

Je vous propose ici d'installer quelques paquets absents de la distribution `Rocky Linux 9`.

```sh
# Nano - Editeur en ligne de commande permettant de remplacer vim qui n'est pas simple à prendre en main pour un non initié.
sudo dnf install nano -y
```

_D'autres viendront si le besoin s'en fait ressentir_

### 1.4 - 👤 Création d'un Nouvel Utilisateur

#### Pourquoi, j'ai déjà accès au compte rocky ?

Dans la gestion des serveurs et la pratique de la cybersécurité, il est généralement recommandé de créer et d'utiliser un nouvel utilisateur avec des droits sudo pour l'administration du système, plutôt que d'utiliser directement l'utilisateur par défaut (comme `rocky` dans le cas de Rocky Linux) ou, l'utilisateur `root` qui correspond au super administrateur. Voici pourquoi :

1. **Sécurité Améliorée :** L'utilisation d'un utilisateur spécifique pour les tâches d'administration réduit les risques associés à l'utilisation du compte `root` en permanence. Les comptes `root` ou les comptes par défaut sont souvent ciblés par des attaques automatisées.

2. **Suivi des Activités :** Avoir des utilisateurs distincts pour différentes personnes ou différents rôles facilite le suivi et l'audit des activités sur le serveur.

3. **Réduction des Erreurs :** Travailler en tant qu'utilisateur non-root oblige à une réflexion supplémentaire avant d'exécuter des commandes potentiellement dangereuses, ce qui peut aider à prévenir des erreurs accidentelles.

4. **Conformité aux Bonnes Pratiques :** C'est une bonne pratique en matière de gestion de serveur et de sécurité informatique de ne pas utiliser les comptes root ou par défaut pour les opérations quotidiennes.

**Pour ces raisons, il est conseillé de se connecter et de gérer votre serveur en utilisant le nom d'utilisateur que vous avez créé, qui dispose des droits sudo. Cela vous offre un équilibre entre la puissance nécessaire pour administrer le système et les contrôles de sécurité appropriés.**

#### Ok, super mais comment faire ?

```sh
# Dans un premier temps, il est possible de lister les utilisateurs sous linux
cat /etc/passwd

# Nous pouvons voir à quel groupe appartient le compte rocky fourni par défaut
groups rocky
# Affichera:
rocky : rocky adm systemd-journal

# Nous pouvons voir l'identifiant de l'utilisateur
id rocky
uid=1000(rocky) gid=1000(rocky) groups=1000(rocky),4(adm),190(systemd-journal)
```

Pour ajouter un utilisateur, (pour nous, il s'agit ici de guidevps) saisisez:

```sh
sudo adduser guidevps
```

Pour ajouter des droits sudo à notre utilisateur:

```sh
sudo usermod -aG adm guidevps
sudo usermod -aG systemd-journal totovps

# A utiliser avec prudence, permet de donner le droit d'utiliser "su"
sudo usermod -aG wheel guidevps

# Si vous souhaitez retirer un groupe, exemple adm
sudo usermod -G guidevps,adm guidevps

# Contrôler les changements en réutilisant de nouveau de la commande "id user"
id guidevps
# Affichera:
uid=1001(guidevps) gid=1001(rocky) groups=1001(guidevps),190(systemd-journal)

# Si vous souhaitez en revanche retirer tous les groupes que vous avez rejoints,
# il faudra saisir
sudo usermod -G guidevps guidevps

# Pour de nouveau contrôler les changements effectués, on saisiera de nouveau "id user"
id guidevps
# Affichera:
uid=1001(guidevps) gid=1001(rocky) groups=1001(guidevps)
```

Il est possible de créer un mot de passe por notre utilisateur

```sh
sudo passwd guidevps
# Après avoir exécuté cette commande, vous serez invité à entrer et confirmer le nouveau mot de passe.
# ⚠️ Sur linux les caractères ne s'affiche pas lors de la saisie d'un mot de passe ⚠️
```

Vérification du compte (Teste de la connexion avec un accès en **root** notre nouvel utilisateur "guidevps")

```sh
sudo su - guidevps
# Il faudra saisir le mot de passe fraichement créé.
# remarquez le nom d'utilisateur dans le terminal qui va changer en passant de rocky@xxxxxx à guidevps@xxxxxxx

# Vérification accès root
sudo whoami
# Si la commande retourne root, cela signifie que cet utilisateur a bien des droits de superadministrateur.
```

### 1.5 - 🔑 Création et Configuration des Clés SSH

```sh
# Générez une paire de clés SSH sur votre machine locale
ssh-keygen -t ed25519 -C "description_de_la_cle"

# remplacer "description_de_la_cle" par ce que vous voulez
```

Voir le contenu de cette clé (_besoin un peu plus tard_)

```sh
cat cat C:/Users/votre_user_windows/.ssh/id_ed25519.pub

# Ce qui affichera quelque chose du genre :
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAC+16EGsoJR0t3A1wGhZ0uYur7JkE3jNxiRtl5uCexS description_de_la_cle
```

✅ - En tant que clé publique elle peut être divulgué sans risque, **ce qui est important c'est de ne jamais divulguer la clé privé. Elle se trouve au même endroit dépouvue du .pub**.

### 1.6 - Copie de la Clé Publique sur notre VPS (server)

> ⚠️ **ATTENTION** - Assurez-vous de copier uniquement le contenu de la clé **id_ed25519.pub**

```sh
# Copie de la clé publique sur le serveur VPS
ssh-copy-id -i ./id_ed25519.pub guidevps@50.60.70.80
# Le mot de passe du compte sera potentiellement demandé.
```

### 1.7 - Installation Manuelle de la Clé (Si Nécessaire)

Sur Windows, si `ssh-copy-id` n'est pas disponible, suivez ces étapes :

```sh
# Connexion au serveur VPS via SSH
ssh guidevps@50.60.70.80

# Accéder au dossier .ssh de l'utilisateur (répertoire caché)
cd .ssh

# Afficher le contenue du répertoire caché ".ssh"
ls -lah

# Éditer le fichier authorized_keys pour ajouter la clé publique (nano sur windows)
sudo nano authorized_keys

# Copier manuellement la clé publique générée à l'étape 1.5 (id_ed25519.pub)
# Enregistrer avec "CTRL + S" et quitter avec "CTRL + X"

# Afficher le contenu de authorized_keys pour vérifier
cat authorized_keys
```

✅ - Si vous disposez de plusieurs ordinateurs et que vous souhaitez accedez en ssh sur votre serveur, chaque clé devra être ajouter à la suite de la première.

### 1.7 - 🚪 Modification du Port d'Écoute SSH par Défaut

Vis à vis de la cybersécurité, changer le port SSH par défaut (**22**) au profit d'un port différent réduit le risque d'attaques automatisées. (_tentatives de hack du serveur par des robots_).
Utilisez un port non-standard, de préférence entre **49152** et **65535**.

> La modification de ce paramètre, est une mesure simple pour renforcer la protection de votre serveur contre les attaques automatisées. Pour cela, modifiez le fichier de configuration du service avec l'éditeur de texte de votre choix (_**nano** est utilisé dans le terminal dans cet exemple_) :

**Je prendrais l'exemple du port 50001 pour oute la durée de ce guide.** :

```sh
# Passer en superadministrateur (root)
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

Décommentez en retirant le **#** et remplacez par :

```sh
# ... d'autres lignes de code

Port 50001
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

# ... encore d'autres lignes de code
```

Donc ici, nous avons décommenté et remplacé le nombre `22` par le numéro de port de suivant `50001`. En effet le port `22` était commenté mais surtout il est le port par défaut.

> Enregistrez (`CTRL + S`) et quittez (`CTRL + X`). Puis redémarrer le service SSH pour appliquer les changements :

```sh
# sudo n'est pas recquis car nous nommes toujours connecté en root (super administrateur)
systemctl restart sshd

# Cela devrait être suffisant pour appliquer les changements.
# Dans le cas contraire, redémarrez le VPS avec cette commande :
reboot
```

⚠️ **ATENTION** - Pour se connecter après ce changement :

```bash
# Connexion au VPS via SSH avec le nouveau port
ssh guidevps@50.60.70.80 -p 50001
# Saisissez votre mot de passe
```

_Vous devrez indiquer le nouveau port à chaque demande de connexion SSH à votre serveur._

### 1.8 - 🔐 Désactivation de l'Authentification par Mot de Passe

Pour plus de sécurité, désactivez l'authentification par mot de passe.
Pour celà, nous devons à nouveau modifier le fichier que l'on a modifié précédemment.

> ⚠️ **ATTENTION** - Assurez-vous d'avoir copier votre clé ssh. Auquel cas la connexion sera impossible.

```sh
# Déplacement dans le répertoire correspondant:
cd /etc/ssh

# Ouvrir le fichier de configuration SSH pour modification (tojours avec nano)
sudo nano sshd_config
```

Modifiez :

```sh
PasswordAuthentication yes # disponible sous rocky linux 9
PermitRootLogin yes # non disponible sous rocky linux 9 (mais on peut l'ajouter)
```

en :

```bash
PasswordAuthentication no
```

Enregistrez (`CTRL + S`) et quittez (`CTRL + X`), puis redémarrez SSH.

---

## 2 - Installation et Configuration de NGINX

### 2.1 - Avant l'installation

Pour installer la dernière version LTS (Long Term Support) de NGINX sur Rocky Linux 9, tu devras suivre quelques étapes. Voici les étapes à suivre :

#### 2.1.1 - Ajout du dépôt NGINX

Tout d'abord, tu dois ajouter le dépôt officiel de NGINX à ton système pour t'assurer d'obtenir la dernière version. NGINX ne fournit pas directement un dépôt pour Rocky Linux, mais tu peux utiliser le dépôt pour RHEL (**Red Hat Enterprise Linux**) qui est compatible.

```sh
# Créer le fichier de dépôt NGINX
sudo nano /etc/yum.repos.d/nginx.repo

# Ajouter les informations de dépôt suivantes dans le fichier :

[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/rhel/9/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

Sauvegarde le fichier (`Ctrl + s` et ferme l'éditeur `Ctrl+X`).

### 2. Installation de NGINX

Après avoir ajouté le dépôt, tu peux installer NGINX :

```sh

# Mettre à jour les paquets et installer NGINX
sudo dnf update -y && sudo dnf upgrade -y && sudo dnf autoremove -y

# Installation du référentiel de logiciels et nginx
sudo dnf install epel-release
sudo dnf install nginx -y
```

Cette commande va installer la dernière version de NGINX disponible dans le dépôt que tu viens d'ajouter.

### 3. Démarrage et Activation de NGINX

Une fois NGINX installé, tu dois le démarrer et l'activer pour qu'il se lance au démarrage du système :

```sh
# Plusieurs démarrage possible (veillez choisir un choix parmis les 3 commandes ci-dessous)
sudo systemctl start nginx
sudo service nginx start
sudo nginx

# Activer nginx pour relancer nginx au démarage
sudo systemctl enable nginx
```

### 4. Vérification

Pour vérifier que NGINX fonctionne correctement :

```bash
sudo systemctl status nginx
```

Et tu peux aussi vérifier la version de NGINX installée :

```bash
nginx -v

# Tester votre site directement via une ligne de commande
curl -I http://votre_ip

# Affichera quelque chose de similaire
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Wed, 17 Jan 2024 23:09:07 GMT
Content-Type: text/html
Content-Length: 7620
Last-Modified: Thu, 02 Feb 2023 21:29:03 GMT
Connection: keep-alive
ETag: "63dc2b1f-1dc4"
Accept-Ranges: bytes
```

### Conclusion

En suivant ces étapes, tu auras installé la dernière version LTS de NGINX sur Rocky Linux 9. N'oublie pas de configurer ton pare-feu pour autoriser le trafic HTTP et HTTPS si nécessaire.

### 📄 2.2 Configuration de NGINX pour Plusieurs Sites

Afin de faciliter l'édition des répertoires, veuillez modifier le groupe de quelques répertoires et ajouter le droit d'écriture pour le compte utilisé (`guidevps`)

```sh
# voir les droits associés aux fichiers
ls -lah /etc/nginx
# On constatera que nous n'avons pas les droits d'écriture, que le groupe et propriétaire est root

# Déplacement dans le répertoire nginx situé dans /etc/nginx
cd /etc/nginx

# Il est probable que des répertoires sont absents à l'initialisation d'nginx
# Création de deux répertoires qui seront utilisés
sudo mkdir sites-available sites-enabled

# voici la liste des fichiers et répertoires qui doivent être modifiés dans le répertoire nginx
# Fichier(s)
sudo chown -R $USER:$USER nginx.conf
# Répertoires(s)
sudo chown -R $USER:$USER conf.d/
sudo chown -R $USER:$USER sites-available/
sudo chown -R $USER:$USER sites-available/
# Ou
sudo chown -R $USER:$USER sites-*

# Contrôle de l'existance du répertoire "www"
ls -lah /var | grep www

# Si une ligne est retournée c'est que le répertoire existe.
drwxr-xr-x.  3 root root   28 Jan 17 22:38 www

# auquel cas, il faut créer celui-ci
sudo mkdir /var/www

# On va lui attribuer les mêmes droits que précédemment afin de pouvoir aisément intervenir dessus.
cd /var
sudo chmod -R g+w www/
sudo chown -R $USER:$USER www/

# On obtiendra ainsi
drwxrwxr-x.  3 guidevps guidevps   28 Jan 17 22:45 www
```

### 📄 2.3 Configuration de NGINX pour Plusieurs Sites

**NOTE:** - Répétez ces étapes pour chacun des sites que vous concevrez :

---

**`scp -r client-build/* guidevps@50.60.70.80:/var/www/site1.fr/html/`**
**`sudo chcon -Rt httpd_sys_content_t /var/www`**

---

```sh
# Créer un répertoire pour le site et configurer les permissions
sudo mkdir -p /var/www/nom_site1.com/html
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

## 3 - Utilisation de let's Encrypt pour passez en HTTPS

```sh
# On passe en superadministrateur (root)
sudo su

# Installation de certbot
dnf install certbot python3-certbot-nginx -y

# Utilisation de certbot avec nginx ou quelques questions seront posés
certbot --nginx

# Une fois terminé, on peu voir le contenu de la connexion avec TLS
openssl s_client -connect site1.fr:443
```

Ici je vous propose un lien pour tester votre server que vous aurez configurez.
Pas mal de tests sur des potentiels failles connu sont testés.
[https://www.ssllabs.com](https://www.ssllabs.com/index.html)
Et ci dessous un lien pour connaitre la meilleur configuration possible pour son serveur nginx.
[https://ssl-config.mozilla.org/#server=nginx&version=1.24&config=modern&openssl=1.1.1k&guideline=5.7](https://ssl-config.mozilla.org/#server=nginx&version=1.24&config=modern&openssl=1.1.1k&guideline=5.7)

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
