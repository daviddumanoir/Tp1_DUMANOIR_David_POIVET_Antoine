# Projet Docker - Configuration PrestaShop

## Description
Ce référentiel contient la configuration Docker et la mise en place pour le déploiement de PrestaShop avec une base de données PostgreSQL, ainsi qu'une configuration réseau à l'aide de Docker Compose. La configuration comprend des réseaux frontal et dorsal, ainsi qu'un routeur passerelle.

## Prérequis
- Docker installé sur votre système
- Docker Compose (en option, en fonction de votre configuration)

## Instructions

### Tâche 1 : Configuration de PrestaShop
1. Téléchargez la dernière image PrestaShop :
    ```bash
    docker pull prestashop/prestashop:latest
    ```

2. Créez un réseau Docker :
    ```bash
    docker network create prestashop_network
    ```

3. Exécutez le conteneur PostgreSQL :
    ```bash
    docker run --name database-container --privileged -d -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin -e POSTGRES_DB=studentdb -v tp_volume:/var/lib/postgresql/data -p 5433:5432 postgres
    docker network connect prestashop-network database-container
    ```

4. Exécutez le conteneur PrestaShop :
    ```bash
    docker run --name prestashop-container --privileged --network prestashop-network -e PRESTASHOP_DATABASE_PASSWORD=admin -d prestashop/prestashop
    docker exec -it prestashop-container bash
    apt-get update
    apt-get install postgresql-client
    psql -h database-container -U admin -d studentdb –W
    ```

### Tâche 2 : Configuration du réseau
1. Créez des réseaux frontal et dorsal :
    ```bash
    docker network create --subnet=10.0.0.0/24 ynov-frontend-network
    docker network create --subnet=10.0.1.0/24 ynov-backend-network
    ```

2. Connectez les conteneurs à leurs réseaux respectifs :
    ```bash
    docker network connect ynov-frontend-network prestashop-container
    docker network connect ynov-backend-network database-container
    docker network disconnect prestashop-network prestashop-container
    docker network disconnect prestashop-network database-container
    ```

3. Créez et connectez le routeur passerelle :
    ```bash
    docker run --name gateway-container --privileged --network ynov-frontend-network --cap-add NET_ADMIN --cap-add SYS_MODULE --sysctl net.ipv4.ip_forward=1 -v /lib/modules:/lib/modules:ro -d nginx:latest
    docker network connect ynov-backend-network gateway-container
    ```

4. Installez les outils nécessaires sur le conteneur passerelle :
    ```bash
    docker exec –it gateway-container bash
    ```
    ```bash
    apt-get update
    apt-get install –y iproute2
    exit
    ```

5. Installez la commande ping pour les tests :
    ```bash
    docker exec -it gateway-container bash
    ```
    ```bash
    apt-get update 
    apt-get install -y iproute2 
    apt-get install -y iputils-ping
    exit
    ```
6. Configuration des routes :
    ```bash
    docker exec -it gateway-container bash
    ```
    ```bash
    ip route replace 10.0.0.0/24 via 10.0.0.3
    ip route replace 10.0.1.0/24 via 10.0.1.3
    exit
    ```
    
## Tests
- Pinguez la base de données depuis le conteneur prestashop :
    ```bash
    docker exec -it prestashop-container bash
    ```
    ```bash
    apt-get update 
    apt-get install -y iproute2 
    apt-get install -y iputils-ping
    ping 10.0.1.2
    exit
    ```
    
- Pinguez PrestaShop depuis le conteneur database :
    ```bash
    docker exec -it database-container bash
    ```
    ```bash
    apt-get update 
    apt-get install -y iproute2 
    apt-get install -y iputils-ping
    ping 10.0.0.2
    exit
    ```



## Contributeurs
- David Dumanoir
- Antoine Poivet
