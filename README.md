# DevOps_Chloe_Depit_V4

##1-1
Dans un premier temps nous avons créeé un Dossier Database. Ce dossier contient un fichier Dockerfile qui contient les lignes suivantes : 
'''FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd'''
Ensuite nous avons construit une image grâce à la ligne :
'''docker build -t database .'''
Après nous avons construit le conteneur : 
'''docker run  -d -p 8080:8080  --name mydb database'''

Nous avons créé notre network : '''docker network create app-network'''
On avons démarrer l’adminer grâce à la commande : '''docker run -p 8080:8080 --net=app-network --name=adminer -d adminer'''
Nous avons redémarer notre base de données avec la commande : '''docker run -d  --network app-network --name mydb database'''
Tout au long de ce processus, j’ai utilisé la commande docker ps pour vérifier quel conteneur était en train de fonctionner. J’ai aussi utilisé docker rm pour supprimer les anciens conteneurs et pour pouvoir appliquer des modifications. 
Pour pouvoir exécuter tous les scripts sql dans le conteneur, on a ajouter la ligne suivante au Dockerfile : '''COPY *.sql /docker-entrypoint-initdb.d'''
On a ensuite ajouter 2 scripts dans le dossier Database : CreateScheme.sql et InsertData.sql.
On a rebuild notre image et run denouveau pour vérifier que les scripts ont bien été exécutés et que les données sont bien présentes dans le conteneur.
Pour que les données persistent et ne soient pas détruites à chaque fois qu’on redémarre la base de données, on va utiliser la ligne suivante : '''docker -v /var/lib/postgresql/data'''

##1-2 
Multi-stage builds permet d’optimiser des fichiers Dockerfile et de les garder facilement lisible et pour pour faciliter leur maintenance.
Build : lorsque l’image est build 
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build : première étape de construction de l’image
ENV MYAPP_HOME /opt/myapp : définition de la variable d’environnement MYAPP_HOME qui a pour valeur : /opt/myapp  pour spécifier le répertoire de bas de l’application
WORKDIR $MYAPP_HOME : définit le répertoire de travail de l’image c’est-à-dire MYAPP_HOME 
COPY pom.xml . : copie le fichier ‘pom.xml’ vers le répertoire de travail actuel
COPY src ./src : copie le répertoire ‘src’ vers le répertoire ‘src’ dans l’image Docker
RUN mvn package -DskipTests : exécute la commande Maven pour construire le package de l'application en utilisant le fichier pom.xml
Run : losque que l’image est run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp : Cette commande définit à nouveau la variable d'environnement MYAPP_HOME avec la valeur /opt/myapp.
WORKDIR $MYAPP_HOME : Cette commande définit le répertoire de travail actuel de l'image à $MYAPP_HOME.
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar : Cette commande copie le fichier JAR de l'application à partir du répertoire target dans l'image intermédiaire myapp-build vers le répertoire $MYAPP_HOME dans l'image finale.
ENTRYPOINT java -jar myapp.jar : Cette commande définit la commande d'entrée de l'image Docker, spécifiant la commande à exécuter lorsque le conteneur est lancé. Dans ce cas, il exécute l'application Java en exécutant le fichier JAR myapp.jar à l'aide de la commande java -jar.

##1-3
'''version: '3.7'

services:
    backend:
        build: .\Backend_API\simple-api\simple-api-student //chemin qui permet de retrouver où est le conteneur du back
        container_name: simple2 
//nom du conteneur du back
        networks:
        - app-network // networks utilisé
        depends_on:
        - database // le back dépend de la base de donnée

    database:
        build: .\Database // chemin qui permet de retrouver le conteneur de la base de données
        container_name: mydb // le nom du conteneur
        networks:
        - app-network // networks utilisé

    httpd:
        build: .\Backend_API\simple-api\simple-api-student-front // chemin qui permet de retrouver le conteneur du front
        ports: 
        - "8081:80" // le port utilisé
        networks:
        - app-network // nom du network utilisé
        depends_on:
        - backend // le front dépend du back

networks:
    app-network:'''

##1-5  
Je me suis mise dans le dossier qui contenait le conteneur de la base de données. J’ai fait les commandes : '''docker tag mydb chloe2607/my-database''' puis '''docker push chloe2607/my-database'''.
J’ai refait ses commandes dans les dossiers qui contenaient le conteneur du back et celui du front. Leur tag respectif sont : my-back et my-front. 

##2-1.
Un test container est un environnement d'exécution préconfiguré pour l'exécution de tests d'intégration ou de tests fonctionnels.
##2-2.
Mon fichier main.yml contient les lignes suivantes :

'''name: CI devops 2023
on:
  push:
    branches: 
      - main
      - develop
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with: 
          java-version: '17'
          distribution: 'adopt'

      - name: Build and test with Maven
        run: mvn -f Backend_API/simple-api/simple-api-student/pom.xml clean install'''

Tout d'abord on installe et on configure la version 17 de JDK.
On exécute la commande mvn pour construire et tester le projet à l'aide de Maven. L'option -f spécifie le chemin vers le fichier pom.xml que Maven 
doit utiliser pour le projet. L'option clean supprime les fichiers générés lors de la précédente construction. L'option install compile le projet et       installe les artefacts dans le référentiel local Maven.

    ##2-3. 
On a utilisé la ligne suivante :  
       '''run: mvn -B verify sonar:sonar -Dsonar.projectKey=devops-v2-chloe-depit_chloe-depit -Dsonar.organization=devops-v2-chloe-depit -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOCKEN }}  --file Backend_API/simple-api/simple-api-student/pom.xml'''
mvn :indique l'utilisation de l'outil Maven.
-B : Cela active le mode batch, ce qui permet à Maven de fonctionner en mode non interactif et de désactiver la sortie colorée.
-Dsonar.projectKey : Cela spécifie la clé du projet SonarQube.
-Dsonar.organization : Cela spécifie l'organisation associée au projet SonarQube.
-Dsonar.host.url : Cela spécifie l'URL de l'instance SonarQube à utiliser pour l'analyse.
-Dsonar.login : Cela spécifie le jeton d'accès à utiliser pour se connecter à SonarQube. 
--file Backend_API/simple-api/simple-api-student/pom.xml : Cela spécifie le chemin vers le fichier pom.xml à utiliser pour le projet Maven.

##3-1. 
Mon dossier inventories est constitué d'un fichier setup.yml qui contient les lignes suivantes : 
'''all:
 vars:
   ansible_user: centos
   ansible_ssh_private_key_file: ../../../../../../../../home/chloe2607/id_rsa
 children:
   prod:
     hosts: chloe.depit.takima.cloud'''
Ce fichier est utilisé pour spécifier les hôtes sur lesquels exécuter des tâches ou des jeux de rôles dans Ansible.
vars: : Cela indique le début de la section où sont définis des variables  utilisées dans l'ensemble du fichier d'inventaire ou dans les tâches/rôles associés.
ansible_user: centos : Cela indique que le nom d'utilisateur à utiliser lors de la connexion à l'hôte distant est "centos".
ansible_ssh_private_key_file: ../../../../../../../../home/chloe2607/id_rsa : Cela spécifie le chemin du fichier de clé privée SSH à utiliser pour l'authentification. Dans cet exemple, il pointe vers un fichier situé à home/chloe2607/id_rsa en remontant plusieurs niveaux de répertoires.
children: : Ceci est utilisé pour définir des groupes d'hôtes ou d'autres groupes d'inventaire, permettant d'organiser les hôtes en fonction de différents critères.
prod: : C'est le nom du groupe dans cet exemple. Les groupes peuvent être utilisés pour exécuter des tâches ou des rôles sur un ensemble spécifique d'hôtes.
hosts: chloe.depit.takima.cloud : Cela spécifie l'hôte inclus dans le groupe prod. Toutes les tâches ou tous les rôles associés au groupe prod seront exécutés sur cet hôte.
En combinant ces éléments, ce fichier d'inventaire permet à Ansible d'exécuter des tâches ou des rôles sur l'hôte chloe.depit.takima.cloud en utilisant l'utilisateur centos et la clé privée spécifiée.

##3-2. Mon playbook est constitué des lignes suivantes :
'''- hosts: all
  gather_facts: false
  become: yes
  roles:
    - docker
    - network
    - database
    - back
    - proxy'''
hosts: all : Cela spécifie que toutes les machines dans l'inventaire seront les cibles de ce playbook.
gather_facts: false : Cela désactive la collecte des faits sur les machines distantes. 
become: yes : Cela indique qu'Ansible doit exécuter les tâches avec les privilèges d'administration. 
roles: : Cette section spécifie les rôles à exécuter sur les machines cibles. Chaque rôle mentionné correspond à un ensemble de tâches et de fichiers organisés dans une structure de répertoires spécifique, qui peut être réutilisée et partagée entre différents playbooks.
Les rôles mentionnés (docker, network, database, back, proxy) représentent différentes fonctionnalités ou services à configurer sur les machines cibles. Chaque rôle aura son propre ensemble de tâches, de modèles et de fichiers de configuration spécifiques à ce rôle particulier.

#3-3. Dans le proxy, le main.yml contient les lignes : 
'''- name: Run HTTPD
  docker_container:
    name: httpd
    image: chloe2607/my-front:1.0
    recreate: yes
    ports:
      - "80:80"
    pull: true
    networks:
      - name: app-network
  vars:
    ansible_python_interpreter: /usr/bin/python3'''
name: Run HTTPD : C'est le nom de la tâche. Il s'agit d'un identifiant qui peut être utilisé pour référencer cette tâche dans les sorties de playbook ou pour le débogage.
docker_container: : C'est un module Ansible utilisé pour gérer les conteneurs Docker. Il permet de contrôler la création, la suppression, l'exécution et la gestion de conteneurs Docker sur les machines cibles.
Les éléments spécifiés sous docker_container sont des paramètres de configuration pour le conteneur Docker :
name: httpd : Cela spécifie le nom du conteneur comme "httpd".
image: chloe2607/my-front:1.0 : Cela spécifie l'image Docker à utiliser pour créer ce conteneur.
recreate: yes : Cela indique que le conteneur doit être recréé s'il existe déjà.
ports: - "80:80" : Cela indique que le port 80 du conteneur doit être exposé sur le port 80 de la machine hôte.
pull: true : Cela spécifie que l'image doit être récupérée depuis le registre Docker si elle n'est pas déjà présente sur la machine.
networks: - name: app-network : Cela attache le conteneur au réseau nommé "app-network".
ansible_python_interpreter: /usr/bin/python3 : Cela spécifie l'interpréteur Python à utiliser pour l'exécution de ce script. 

Dans le network, le main.yml contient les lignes : 
'''- name: Create a network
  docker_network:
    name: app-network
  vars:
    ansible_python_interpreter: /usr/bin/python3

  Dans le back, le main.yml contient les lignes :
  - name: Run Back
  docker_container:
    name: simple2
    image: chloe2607/my-back:1.0
    recreate: yes
    pull: true
    networks:
      - name: app-network
  vars:
    ansible_python_interpreter: /usr/bin/python3'''

Dans la base de données, le main.yml contient les lignes :
'''- name: Run Database
  docker_container:
    name: mydb
    image: chloe2607/my-database:1.0
    recreate: yes
    pull: true
    networks:
      - name: app-network
  vars:
    ansible_python_interpreter: /usr/bin/python3
  Dans le docker, le main.yml contient les lignes :
    - name: Install device-mapper-persistent-data
        yum:
          name: device-mapper-persistent-data
          state: latest

      - name: Install lvm2
        yum:
          name: lvm2
          state: latest

      - name: add repo docker
        command:
          cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

      - name: Install Docker
        yum:
          name: docker-ce
          state: present

      - name: Install python3
        yum:
          name: python3
          state: present

      - name: Install docker with Python 3
        pip:
          name: docker
          executable: pip3
        vars:
          ansible_python_interpreter: /usr/bin/python3

      - name: Make sure Docker is running
        service: name=docker state=started
        tags: docker'''


