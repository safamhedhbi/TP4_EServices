How to: Dockeriser des microservices
=======
<!-- TOC -->
- [Environnement](#Environnement)
- [Architecture Microservices](#microservices-to-deploy-on-docker)
- [Dockeriser les Microservices](#dockeriser-microservices)
- [Cr�ation des Dockerfiles](#Dockerfile)
- [Cr�ation du docker-compose.yml](#Docker-compose)
- [R�sultat final](#final)

<!-- /TOC -->

## Environnement
On a besoin de :
- Intellij
- JDK 8 
- Docker (Docker Toolbox si vous n'�tes pas windows10 PRO)

(plus de d�tails ici --> https://insatunisia.github.io/TP-eServices/tp4/ )

## Architecture Microservices
- Product Service : Service principal, qui offre une API REST pour lister une liste de produits.
- Config Service : Service de configuration, dont le r�le est de centraliser les fichiers de configuration des diff�rents microservices dans un endroit unique.
- Proxy Service : Passerelle se chargeant du routage d'une requ�te vers l'une des instances d'un service, de mani�re � g�rer automatiquement la distribution de charge.
- Discovery Service: Service permettant l'enregistrement des instances de services en vue d'�tre d�couvertes par d'autres services.
**L'architecture r�sultante aura l'allure suivante:**


![Architecture](images/archi.png)

## Dockeriser les Microservices
Pour dockerize les microservices, nous devons g�n�rer une image Docker � partir de chaque application. 
Ceci peut �tre r�alis� par la cr�ation d'un fichier Dockerfile pour chaque microservice.

## Cr�ation des Dockerfiles
On cr�e pour chaque projet son Dockerfile:
Exemple de **Proxy Service:**


```
FROM maven:3.6.0-jdk-8 AS build
WORKDIR /usr/src/app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

FROM openjdk:8-jre
COPY --from=build /usr/src/app/target/proxy-service.jar /usr/app/
EXPOSE 9999
RUN apk add --no-cache bash
ADD wait-for-it.sh /wait-for-it.sh
CMD ["java","-jar","/usr/app/proxy-service.jar","--spring.profiles.active=docker"] 
```


Puis on ex�cute la commande  ``docker build -t proxy-service . ``


On v�rifie l'ajout des images avec ``docker images``


![Dockerimages](images/dockerimg.png)

## Cr�ation du docker-compose.yml
**Documentation ->** https://docs.docker.com/compose/


```
version: '3'
services:
  config-service:
    build: config-service
    ports: 
     - "8888:8888"
  
  discovery-service:
    build: discovery-service
    ports: 
     - "8761:8761"
    depends_on:
      - config-service
    entrypoint: /wait-for-it.sh config-service:8888 -t 300 --
    command: java -jar /usr/app/discovery-service-0.0.1-SNAPSHOT.jar --spring.profiles.active=docker
  
  proxy-service:
    build: proxy-service
    ports: 
     - "9999:9999"
    depends_on:
      - discovery-service
    entrypoint: /wait-for-it.sh discovery-service:8761 -t 300 --
    command: java -jar /usr/app/proxy-service-0.0.1-SNAPSHOT.jar --spring.profiles.active=docker
  
  product-service-8080:
    build: product-service
    ports: 
     - "8080:8080"
    depends_on:
      - proxy-service
    entrypoint: /wait-for-it.sh proxy-service:9999 -t 300 --
    command: java -jar /usr/app/TP4-Microservices-0.0.1-SNAPSHOT.jar --spring.profiles.active=docker
  
  product-service-8081:
    build: product-service
    ports: 
     - "8081:8080"
    depends_on:
      - proxy-service
    entrypoint: /wait-for-it.sh proxy-service:9999 -t 300 --
    command: java -jar /usr/app/TP4-Microservices-0.0.1-SNAPSHOT.jar --spring.profiles.active=docker
  
  product-service-8082:
    build: product-service
    ports: 
     - "8082:8080"
    depends_on:
      - proxy-service
    entrypoint: /wait-for-it.sh proxy-service:9999 -t 300 --
    command: java -jar /usr/app/TP4-Microservices-0.0.1-SNAPSHOT.jar --spring.profiles.active=docker 
```
    
	
A partir du dossier o� se trouve **docker-compose.yml** on ex�cute ``docker-compose up`` 


![docker compose](images/compose.png)

## R�sultat final

Voil� �a marche!


![time](images/resultat.png)