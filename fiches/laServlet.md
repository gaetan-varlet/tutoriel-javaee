# La servlet

## Derrière les rideaux

### Retour sur HTTP
C’est le langage qu’utilisent le client et le serveur pour s’échanger des informations. Ce langage ne comprend que quelques mots, appelés méthodes HTTP. Ce sont les mots qu’utilisent le navigateur pour poser des questions au serveur. Nous allons voir trois de ces mots : **GET**, **POST** et **HEAD**.

**GET**
C’est la méthode utilisée par le client pour récupérer une ressource web via une URL. Par exemple, on écrivant www.google.com dans la barre d’adresses du navigateur, celui-ci envoie une requête GET pour récupérer la page correspondant à cette adresse et le serveur lui renvoie. Idem en cliquant sur un lien.
Lorsqu’il reçoit une telle demande, le serveur en plus de retourner la ressource demandée, l’accompagne d’informations diverses à son sujet, dans les en-têtes HTTP : longueur des données renvoyés, date d’envoi.
Il est possible de transmettre des données au serveur lorsqu’on effectue une requête GET, au travers de paramètres directement placés après l’URL (paramètres nommés query strings) ou de cookies placés dans les en-têtes de la requête. La limite de ce système est que, comme la taille d’une URL est limitée, on ne peut pas utiliser cette méthode pour envoyer des données volumineuses au serveur.
Les gens qui ont écrit la norme décrivant le protocole HTTP ont émis des recommandations d’usage, qui dit qu’il est uniquement possible de récupérer ou lire des informations, sans que cela ait un quelconque impact sur la ressource. Une requête GET est donc censée pouvoir être répétée indéfiniment sans risques pour la ressource concernée.

**POST**
La taille du corps du message d’une requête POST n’est pas limitée, c’est la méthode à utiliser pour réaliser des opérations qui ont un effet sur la ressource, et qui ne peuvent par conséquent pas être répétées sans l’autorisation explicite de l’utilisateur. Le navigateur peut envoyer un message d’alerte par exemple après le rafraîchissement d’une page web qui renvoi les informations au serveur.

**HEAD**
Cette méthode est identique à la méthode GET, sauf qu’elle renvoie seulement les en-têtes HTTP. Il est ainsi possible par exemple de vérifier la validité d’une URL, ou de vérifier si le contenu d’une page a changé ou non sans avoir à récupérer la ressource elle-même. Il suffit de regarder ce que contiennent les différents champs des en-têtes.

### Pendant ce temps-là, sur le serveur

La requête HTTP part du client et arrive sur le serveur. L’élément qui entre en jeu est le **serveur HTTP** (on parle également de **serveur web**), qui ne fait qu’écouter les requêtes HTTP sur un certain port, en général le port 80.
Quand une requête lui parvient, il l’a transmet au **conteneur de servlets**, également appelé **conteneur web**. Celui-ci va alors créer deux nouveaux objets :
- *HttpServletRequest* : cet objet contient la requête HTTP et donne accès à toutes ses informations, telles que les en-têtes (headers) et le corps de la requête
- *HttpServletResponse* : cet objet initialise la réponse HTTP qui sera renvoyée au client, et permet de la personnaliser, en initialisant par exemple les en-têtes et le corps.

Ensuite, c’est le code de l’application qui entre en jeu. Le conteneur de servlet va les transmettre à notre application, plus précisément aux servlets et filtres que nous avons mis en place.


## Création d’une servlet

Une servlet est une simple classe Java, qui a la particularité de **permettre le traitement de requêtes et la personnalisation de réponses**. Pour faire simple, le plus souvent, c’est une classe capable de recevoir une requête HTTP envoyée depuis le navigateur de l’utilisateur, et de lui renvoyer une réponse HTTP.

En principe, une servlet dans son sens générique est capable de gérer n’importe quel type de requête, mais dans les faits il s’agit principalement de requêtes HTTP. L’usage veut qu’on ne s’embête pas à préciser “servlet HTTP”, il est donc commun de parler de servlets alors qu’il s’agit bien de servlets HTTP.

**L’interface Servlet est l’interface mère que toute servlet doit obligatoirement implémenter.** Certaines classes de bases l’implémentent déjà, il suffit donc d’hériter ce des classes pour créer une servlet. **Nous allons faire hériter notre servlet de la classe HttpServlet.**

```java
import javax.servlet.http.HttpServlet;
public class Test extends HttpServlet {    
}
```

Notre classe est vide pour le moment.
Intéressons-nous aux méthodes de la classe abstraite HttpServlet : elle propose les **méthodes Java nécessaire au traitement des requêtes HTTP** : **doGet()** (pour la méthode GET), **doPost()**, **doHead()**.
La méthode **service()** se charge de lire l’objet HttpServletRequest et de distribuer la requête HTTP à la méthode doXXX() correspondante.
Une servlet doit implémenter au moins une des méthodes doXXX() afin d’être capable de traiter une requête entrante.
Les servlets prennent en chargent les requêtes entrantes, elles sont donc les **points d’entrée de notre application web**. Contrairement au Java SE, il n’existe pas en Java EE d’entrée unique comme la méthode main().


## Mise en place

Les servlets sont des aiguilleurs de requêtes. Pour être reconnues en tant que telles, les servlets nécessitent d’être enregistré auprès de notre application. Il va falloir configurer que **notre servlet va être associée à une URL**, dans le fichier **web.xml**. Lorsque le client saisira l’URL, la requête HTTP sera automatiquement aiguillée par notre conteneur de servlet vers la bonne servlet.
Ce fichier est le coeur de notre application, où se trouve tous les paramètres qui contrôlent son cycle de vie.
Voici la structure du fichier vide :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
  version="3.0">
</web-app>
```

La mise en place d’une servlet se déroule en deux étapes : déclaration de la servlet puis correspondance à une URL

```xml
<servlet>
	<servlet-name>Test</servlet-name>
	<servlet-class>com.sdzee.servlets.Test</servlet-class>
</servlet>
```

- `<servlet>` : balise de définition d’une servlet
- `<servlet-name>` : permet de donner un nom à une servlet, pour y faire référence ensuite. Il est de bonne pratique de garder un nom de classe et un nom de servlet identique pour éviter des confusions
- `<servlet-class>` : sert à préciser le chemin de la classe de la servlet dans l’application (package dans lequel elle se situe).

Il est possible d’insérer au sein de la définition d’une servlet d’autres balises facultatives :
- `<description>` : décrire le rôle de la servlet (aucune utilité technique et visible que dans ce fichier)
- `<init-param>` : pour préciser des paramètres qui seront accessibles à la servlet lors de son chargement
- `<load-on-startup>` : permet de forcer le chargement de la servlet dès le démarrage du serveur

Il faut ensuite faire correspondre notre servlet à une URL (mapping de la servlet) :
```xml
<servlet-mapping>
	<servlet-name>Test</servlet-name>
	<url-pattern>/toto</url-pattern>
</servlet-mapping>
```

- `<servlet-mapping>` : balise de définition du mapping
- `<servlet-name>` : permet de préciser le nom de la servlet à laquelle faire référence
- `<url-pattern>` : permet de préciser la ou les URL relatives au travers desquelles la servlet sera accessible.

Notre servlet est joignable via l’URL : http://localhost:8080/nomAppli/toto. L’ordre des sections de déclaration au sein du fichier est important : il est impératif de définir une servlet avant de spécifier son mapping.


## Mise en service

Lorsque nous lançons le serveur et tentons d’accéder à l’URL en question, nous obtenons un code de statut HTTP, le code d’erreur 405 qui dit que la méthode GET n’est pas supportée par la servlet associée à l’URL.

Le conteneur de servlets :
- a reçu la requête HTTP depuis le serveur web
- a généré un couple requête/réponse
- a parcouru le fichier web.xml à la recherche de l’URL contenue dans l’objet requête
- a trouvé et identifié la servlet déclarée
- a contacté la servlet et transmis la paire d’objets requête/réponse

Afin que notre servlet soit capable de traiter une requête HTTP de type GET, il faut implémenter une méthode doGet(). La méthode service() de HttpServlet s’occupera de transmettre la requête GET vers la méthode doGet() de notre servlet.
Le comportement par défaut des méthodes doXXX() (notamment doGet() donc) de la classe mère HttpServlet est de renvoyer un code d’erreur HTTP 405. Si le développeur a bien implémenté la méthode doXXX() de la servlet, celle-ci sera bien appelée. S’il a oublié de la surcharger, la méthode doXXX() voulue, alors c’est la méthode de la classe mère HttpServlet qui sera appelée. La classe mère s’assure toujours que sa classe fille - notre servlet - surcharge bien la méthode doXXX() correspondant à la méthode HTTP traitée.

Le conteneur de servlets est aussi capable de générer des codes d’erreur HTTP. S’il parcourt le fichier web.xml à la recherche d’une entrée correspondant à l’URL envoyée par le client et qu’il ne trouve rien, c’est lui qui va se charger de générer le fameux code d’erreur 404.

Il faut donc surcharger la méthode doGet() de la classe HttpServlet dans notre servlet. Voici le code de notre servlet :

```java
public class Test extends HttpServlet {
	public void doGet(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException{		
	}
}
```

Si on contacte de nouveau notre servlet via l’URL, le message d’erreur disparaît et le navigateur nous affiche une page blanche, car il considère que la requête s’est terminée avec succès, sinon il afficherait un code et un message d’erreur HTTP. On peut voir dans les outils de développeur du navigateur que le code HTTP renvoyé est 200, ce qui signifie que la requête s’est effectuée avec succès.  
C’est encore une fois le conteneur de servlet qui a fait le boulot : quand il a généré la paire d’objets requête/réponse, il a initialisé la réponse avec une valeur par défaut à 200, c’est à dire que par défaut le conteneur de servlet crée un objet qui stipule que tout s’est bien passé. L’objet est ensuite transmis à la servlet qui peut le modifier. Lorsqu’il reçoit à nouveau l’objet en retour, si le code de statut n’a pas changé, c’est que tout s’est bien passé. Le conteneur adopte une certaine philosophie : pas de nouvelles, bonne nouvelle !  
Le serveur retourne toujours une réponse au client, peu importe ce que fait notre servlet avec la requête ! Dans notre cas, la servlet n’effectue aucune modification sur l’objet HttpServletResponse, d’où le code retour à 200 et la page blanche en guise de résultat.

**Cycle de vie d’une servlet**
Quand une servlet est demandée pour la première fois ou quand l’appli démarre, le conteneur de servlets va créer une instance de celle-ci et la garder en mémoire pendant toute l'existence de l’application. **La même instance sera réutilisée pour chaque requête entrante** dont l’URL correspond au pattern d’URL défini pour la servlet. Dans notre exemple, tous nos appels à monAppli/toto seront dirigés vers la même et unique instance de notre servlet générée par Tomcat lors du tout premier appel. L’instance de la servlet a été créé par défaut au premier appel de la servlet. Ce mode de fonctionnement est configurable, avec la balise <load-on-startup>N</load-on-startup> dans le fichier web.xml, où N est un entier positif. Dans ce cas, on ordonne au serveur de charger l’instance de la servlet en question directement pendant le chargement de l’application. N  correspond à la priorité à donner au chargement de la servlet, pour savoir l’ordre de chargement quand il y en a plusieurs.  La ou les servlets ayant un load-on startup à 0 sont les premiers à être chargées, puis 1, 2, 3…

**Envoyer des données au client**
Pour envoyer des données au client, il faut manipuler l’objet HttpServletResponse dans notre méthode doGet().

```java
public void doGet( HttpServletRequest request, HttpServletResponse response ) throws ServletException, IOException{
	response.setContentType("text/html");
	response.setCharacterEncoding( "UTF-8" );
	PrintWriter out = response.getWriter();
	out.println("<!DOCTYPE html>");
	out.println("<html>");
	out.println("<head>");
	out.println("<meta charset=\"utf-8\" />");
	out.println("<title>Test</title>");
	out.println("</head>");
	out.println("<body>");
	out.println("<p>Ceci est une page générée depuis une servlet.</p>");
	out.println("</body>");
	out.println("</html>");
}
```

- modification de l’en-tête **Content-Type** de la réponse HTTP pour préciser au client qu’on lui envoie un page HTML.
- modification de l’encodage utilisé (ISO-885-1 par défaut)
- création d’un objet PrintWriter qui va permettre d’envoyer du texte au client via la méthode getWriter() de l’objet HttpServletResponse.
- nous écrivons du texte via la méthode println() de l’objet PrintWriter

La langage Java n’est pas adapté à la rédaction de page web.
En suivant le pattern MVC, **la servlet n’est pas censée s’occuper de l’affichage, c’est la vue qui doit s’en charger !** Nous allons créer une vue et la mettre en relation avec notre servlet.
