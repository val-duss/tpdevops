# DEVOPS tp 01

## Pull des images
`docker pull httpd`
`docker pull openjdk`
`docker pull postgres`
`docker pull adminer`

## Creation du Dockerfile
``` FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd
```

Commandes pour build de l'image docker via le Dockerfile
` mkdir imagedck && imagedck`
`docker build -t imagename:v1 .`

Pour run l'image sur un réseau et se connecter sur le postgres avec adminer
création du réseau:
`docker network create app-network`
lancement du container:
`docker run --network=app-network --name containerdck imagename:v1`
lancement de adminer pour se connecter à la base de données
`sudo docker run --network=app-network -p 8080:8080 adminer`

Après nous pouvons observer la base sur http://127.0.0.1:8080 via adminer

## Init de la base via des fichiers sql externes
ajout des fichiers sql dans le dossier imagedck
ajout des ligne pour copié les script sql dans l'init de postgres:
`COPY 01-sqlschema.sql /docker-entrypoint-initdb.d/`
`COPY 02-sqlschema.sql /docker-entrypoint-initdb.d/`
ensuite rebuild de l'image:
`docker build -t imagename:v1 .`

## Ajout d'un volume persistant
ajouter la variable d'environnement suivante
`	PGDATA=/var/lib/postgresql/data/pgdata`
Ajouter au lancement l'ajout du volume
`docker run --network=app-network --name containername -v /custom/mount:/var/lib/postgresql/data imagename:v1`
Pour voir les différents volume
`docker volume ls`

## Application java
Création d'un nouveau dossier 
` mkdir javadck && javadck`
Ajout du Main.java dans le dossier
Edition d'un nouveau dockerfile
``` 
FROM openjdk:16-alpine3.13
COPY . /var/www/java  
WORKDIR /var/www/java  
RUN javac Main.java  
CMD ["java", "Main"] 
```
build de l'image 
`docker  build -t javadck:v1 .`
lancement de l'image
`docker run javadck:v1`

##  Application Spring
change Dockerfile:
```
FROM maven:3.6.3-jdk-11 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests
FROM openjdk:11-jre
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
ENTRYPOINT java -jar myapp.jar 
```

## Application Proxy
change dockerfile
``` FROM httpd:2.4
COPY ./public_html/ /usr/local/apache2/htdocs/
COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf 
```

### Dockercompose
le docker-compose est différent de docker, c'est un fichier de conf yaml qui permet de générer la config de différents docker

dans le compose on build les images des 3 services
```  postgres:
    container_name: postgres
    build: postgres/
    networks:
      - app-network
```
et le network
```
networks:
  app-network:
```
# TP 02
## Dockerhub
Après la création du compte dockerhub pour push l'image faut la build avec le nom d'utilisateur
``` docker build -t username/image:version . ```
se logger
` docker login `
et push l'image dans son registry
`docker push username/image:version `

## GITHUB CI-CD
Après la création d'un repo git de base
on peut déjà ajouter un .gitignore pour eviter de push les fichiers non désirés comme les dossiers /target
Ensuite on créé un fichier dans .github/workflow/main.yml pour définir les différents job que l'on peut mettre en place

exemple pour les test 
``` 
test-backend:
    runs-on:  ubuntu-18.04
    steps:
      #checkout your github code using actions/checkout@v2.3.3
      - uses: actions/checkout@v2.3.3
      #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
      - name:  Set  up  JDK  11
        uses: actions/setup-java@v2
        with:
          distribution: 'zulu'
          java-version: '11'
      #finally build your app with the latest command
      - name:  Build  and  test  with  Maven
        run:  mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=val-duss_tpdevops -Dsonar.organization=val-duss -Dsonar.login=${{secrets.SONAR_TOKEN}} --file simple-api/pom.xml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}  
   ```
   on a ajouter sonnar pour vérifier les différents tests et l'efficacité du process
   ajout de sonnar dans le pom.xml du projet maven et dans le main.yml

# TP 03
## Ansible 
Pour déployer tout notre projet sur un environnnement on doit utiliser Ansible
Après la création des dossiers et fichier:
``` 
my-project/playbook
my-project/inventories/setup.yml
```
Et mettre en place un playbook avec différentes taches

pour harmoniser et rendre propre ce projet ansible on utilise des roles pour différencier les différencier les différentes taches 
ici docker, network, http, database, app-java
