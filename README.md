# n8n-coolify

description: |
  Cette version non officielle de n8n permet de télécharger et d'installer la dernière image Docker de n8n en utilisant Docker Compose. Cette configuration est spécialement conçue pour les utilisateurs de **Coolify** ou ceux utilisant le reverse proxy  **Caddy**, permettant une gestion automatique des ports et de la sécurité SSL.

  Depuis la version **beta.237**, Coolify a ajouté la prise en charge de **Caddy** et **Traefik** comme reverse proxies. Vous pouvez basculer entre eux à tout moment selon vos préférences.

  Avant de changer de proxy et si vous avez une application qui a été créée avant la version **beta.237**, il est important de vérifier les points suivants :
  Si vous n'avez pas les libellés `caddy_*` ou `traefik_*` :
  1. **Automatiquement** : Un redémarrage de votre ressource ajoutera les libellés manquants.
  2. **Manuellement** :
     - **Pour les Applications** : Cliquez sur le bouton *Reset to Coolify Default Labels*.
     - **Pour les Services** : Enregistrez simplement le service - il ajoutera automatiquement les libellés requis.

  N'oubliez pas de redémarrer votre service pour que les nouvelles étiquettes soient appliquées.

  Si vous souhaitez autohéberger n8n, cette configuration est idéale et vous permet de déployer n8n en toute simplicité. Il suffit de quelques commandes et configurations pour lancer et gérer n8n sur votre serveur.

---

## Prérequis

  Avant de commencer, vous devez avoir une certaine expérience avec :
  - La configuration et gestion de serveurs Docker
  - L'utilisation de reverse proxies comme **Caddy**, **Nginx** ou **Traefik**
  - La gestion des ressources et des applications en production
  - La sécurisation des serveurs et des applications

  **Note** : Cette solution est destinée aux utilisateurs avancés. Si vous n'êtes pas à l'aise avec ces technologies, il est recommandé d'utiliser la version **n8n Cloud**.

---
# Étapes d'installation pour déployer une application depuis un repository public avec Coolify

instructions:
  - step: "1. Accédez à votre instance Coolify"
    description: "Connectez-vous à l’interface de Coolify via votre navigateur (exemple : http://your-coolify-instance.com)."

  - step: "2. Ajoutez une nouvelle application"
    actions:
      - "Cliquez sur 'Applications' dans le menu principal."
      - "Cliquez sur 'Create Application' pour ajouter une nouvelle application."

  - step: "3. Configurez l'application"
    actions:
      - "Choisissez le type d'application : Sélectionnez 'Git Repository' comme source pour le déploiement."
      - "Configurez le dépôt Git :"
        details:
          repository_url: "https://github.com/stranxik/n8n-coolify-public.git"
          branch: "main (ou modifiez selon vos besoins)"
      - "Spécifiez le chemin Docker Compose :"
        details:
          docker_compose_path: "docker-compose.yml"
      - "Ajoutez les variables d'environnement :"
        variables:
          - "DOMAIN_NAME : example.com"
          - "SUBDOMAIN : n8n.example.com"
          - "GENERIC_TIMEZONE : Europe/Paris"
          - "DATABASE_TYPE : postgres"
          - "N8N_BASIC_AUTH_ACTIVE : true"

  - step: "4. Configurez le reverse proxy Caddy"
    actions:
      - "Activer le proxy pour cette application :"
        details:
          enable_proxy: true
          subdomain: "n8n.example.com"
      - "Configurer un certificat SSL :"
        details:
          auto_ssl: true

  - step: "5. Déployez l’application"
    actions:
      - "Cliquez sur 'Deploy' pour lancer le déploiement."
      - "Coolify récupérera automatiquement le repository, générera les conteneurs Docker, et configurera le reverse proxy."

  - step: "6. Accédez à votre application"
    description: "Une fois le déploiement terminé, vous pourrez accéder à votre application en visitant l’URL configurée."
    url_example: "https://n8n.example.com"

summary:
  - "Simplification du déploiement : Coolify gère l’ensemble du processus, de la récupération du repository à la configuration du proxy."
  - "Configuration centralisée : Les variables d’environnement et les configurations sont toutes gérées via l’interface de Coolify."
  - "Gestion automatisée des certificats : Caddy s’occupe de générer et renouveler les certificats SSL."

---

## Étapes d'installation sur serveur avec reverse proxy Caddy

1. **Clonez ce repository** :
    ```bash
    git clone https://github.com/stranxik/n8n-coolify-public.git
    cd n8n-coolify
    ```

2. **Copiez le fichier `.env.example`** et renommez-le en `.env`. Complétez ensuite les variables de configuration dans ce fichier selon vos besoins :
    ```bash
    cp .env.example .env
    ```

3. **Complétez votre fichier `.env`** : Assurez-vous de configurer les variables suivantes dans votre fichier `.env` :
    - `DOMAIN_NAME` : Votre domaine principal, par exemple : `example.com`.
    - `SUBDOMAIN` : Le sous-domaine que vous souhaitez utiliser pour accéder à n8n, par exemple : `n8n.example.com`.
    - `GENERIC_TIMEZONE` : La timezone de votre serveur, par exemple : `Europe/Paris`.
    - D'autres variables d'environnement spécifiques au service ou à la configuration peuvent être ajoutées ici. Voici quelques exemples :
      - `DATABASE_TYPE` : Si vous utilisez une base de données, par exemple `postgres`.
      - `N8N_BASIC_AUTH_ACTIVE` : Pour activer l'authentification de base, définissez-le sur `true` ou `false`.

4. **Lancer n8n avec Docker Compose** :
    Il vous suffit d'exécuter la commande suivante pour démarrer n8n avec toutes les configurations nécessaires :
    ```bash
    docker-compose up -d
    ```
---

## Fonctionnement

- **Caddy** (ou un autre reverse proxy comme **Traefik** ou **Nginx**) gère automatiquement les ports et la redirection HTTPS. Vous n'avez pas besoin de définir explicitement les ports dans le fichier `docker-compose.yml` :
    - **Caddy** détecte automatiquement les services et s'assure qu'ils sont accessibles via HTTPS.
    - Les ports pour n8n sont automatiquement gérés par le reverse proxy, donc vous n'avez pas à les spécifier dans la configuration de Docker.

- **Caddyfile** : Vous n'avez pas besoin de configurer Caddy manuellement dans la plupart des cas. **Caddy** gère automatiquement la sécurité SSL. Si vous souhaitez personnaliser des aspects comme les domaines ou les redirections, vous pouvez ajuster le fichier `Caddyfile` dans le dossier `caddy_config`.

---

## Correction des permissions de fichier

- Lors de l'exécution de n8n, vous pourriez voir une alerte concernant les permissions des fichiers de configuration :
    ```
    Permissions 0644 for n8n settings file /home/node/.n8n/config are too wide.
    ```

    Pour corriger cela, vous pouvez définir la variable suivante dans votre fichier `.env` :
    ```bash
    N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
    ```

    Cela permettra de forcer les bonnes permissions sur les fichiers de configuration à chaque démarrage de n8n.

---

## Problèmes connus

- **Note importante** : Par défaut, cette configuration utilise le port interne **5678** dans le fichier `Caddyfile` et dans `docker-compose.yml`. Si vous obtenez une erreur de port déjà attribué lors du démarrage de n8n, il est probable que le port 5678 soit déjà utilisé sur votre machine. Si vous souhaitez changer de port, vous devez modifier ces deux fichiers, mais ne touchez pas aux autres configurations. Assurez-vous que le port est bien mis à jour à ces deux endroits.

---

## Résumé

Cette configuration simplifie l'auto-hébergement de n8n, particulièrement pour les utilisateurs de **Coolify** ou ceux utilisant le reverse proxy  **Caddy**. Il vous suffit de modifier quelques variables d'environnement, de copier le fichier `.env.example` en `.env`, et d'exécuter `docker-compose up`. Cela permet de lancer facilement n8n avec une gestion automatique des ports, la dernière image de n8n et une configuration SSL sécurisée.

---

## Auteurs

Ce projet est maintenu par **[stranxik](https://github.com/stranxik)**.
