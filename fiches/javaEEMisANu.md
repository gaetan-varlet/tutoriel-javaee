# Le Java EE mis à nu !

## Principes de fonctionnement
Le client et le serveur communiquent via HTTP. Côté client, le navigateur, s’en occupe. Côté serveur, c’est le serveur HTTP. Son travail est d’écouter tout ce qui arrive sur le port utilisé par le protocole HTTP, le port 80, et scruter chaque requête entrante.  
Nous n’allons pas utiliser un **serveur HTTP** car nous voulons un serveur capable d’effectuer d’autres tâches. Une fois la requête lue et analysée, il faut encore traiter son contenu et éventuellement renvoyer une réponse au client. C’est le code que l’on écrit qui va décider ce qu’il faut faire lorsqu’une telle requête arrive. Un serveur HTTP ne peut pas gérer notre application, ce n’est pas son travail.  
Il nous faut une solution plus globale, qui va se charger d’exécuter le code en plus de faire le travail du serveur HTTP, qui se nomme **serveur d’applications**. Un tel serveur inclut un serveur HTTP et y ajoute la gestion d’objets de diverses natures au travers d’un composant que nous allons appeler pour le moment **le conteneur**.

Le serveur d’applications va :
- récupérer les requêtes HTTP issues des clients
- les mettre dans des objets que notre code sera capable de manipuler
- faire passer ces objets dans la moulinette qu’est notre application, via le conteneur
- renvoyer des réponses HTTP aux clients, en se basant sur les objets retournés par notre code

Il existe plusieurs solutions sur le marché :
- les solutions propriétaires et payantes ; WebLogic (Oracle) et WebSphere (IBM) sont les références dans le domaine, massivement utilisées dans les banques
- les solutions libres et gratuites : Apache Tomcat, JBoss, GlassFish et Jonas sont les principaux représentants. Nous utiliserons Apache Tomcat.


## Le modèle MVC : en théorie

Un modèle de conception, design pattern en anglais, (ou encore patron de conception) est une simple **bonne pratique** de conception d’une application, considéré comme standard.

Le développement en entreprise implique entre autres :
- travailler à plusieurs sur un même projet
- maintenir une application que l’on a pas créée soi-même
- faire évoluer une application que l’on a pas créée soi-même

Pour toutes ces raisons, il est nécessaire d’adopter une architecture standard, dans laquelle tout développeur sait se repérer. Un modèle permet de répondre à ces besoins : **le modèle MVC (Modèle-Vue-Contrôleur)**. Il découpe l’application en couches distinctes, ce qui impacte très fortement l’organisation du code.
- ce qui concerne le traitement, le stockage et la mise à jour des données de l’application doit être contenu dans la couche nommée “Modèle”
- ce qui concerne l’interaction avec l’utilisateur et la présentation des données (mise en forme, affichage) doit être contenu dans la couche nommée “Vue”
- ce qui concerne le contrôle des actions de l’utilisateur et des données doit être contenu dans la couche nommée “Contrôle”


## Le modèle MVC : en pratique
Le schéma précédent est très global, et encore assez abstrait. Détailler plus finement les blocs d’une application n’est possible qu’au cas par cas, idem pour les relations entre ceux-ci et dépend fortement de l’utilisation ou non d’un framework.

Un **framework** est un ensemble de composants qui servent à créer l’architecture et les grandes lignes d’une application. C’est un genre de boîte à outils géante conçue par des développeurs et mise à disposition d’autres développeurs afin de faciliter leur travail. Il existe par exemple pour Java EE les framework JSF, Spring, Struts, Hibernate.

Voyons d’abord les applications Java EE nues, sans frameworks, en voyons chacune des couches composant les applications web suivant le modèle MVC.

### Modèle : des traitements et des données
On trouve les données et les traitements à appliquer à ces données. Ce bloc contient :
- des objets Java qui peuvent contenir des attributs (données) et des méthodes (traitements) qui leur sont propres
- un système capable de stocker des données

### Vue : des pages JSP
Une page JSP est destinée à la vue, est exécutée côté serveur et permet l’écriture de gabarits (pages en langage “client” comme HTML, CSS, Javascript, XML). Elle permet au concepteur de la page d’appeler de manière transparente des portions de code Java via des balises et expressions ressemblant fortement aux balises de présentation HTML.

### Contrôleur : des servlets
Une servlet est un objet qui permet d’intercepter les requêtes faites par un client et qui peut personnaliser une réponse en conséquence. Il fournit pour cela des méthodes permettant de scruter les requêtes HTTP. Cet objet n’agit jamais directement sur les données, il faut le voir comme un simple aiguilleur : il intercepte une requête issue d’un client, appelle éventuellement des traitements effectués par le modèle et ordonne en retour d’afficher le résultat au client.
