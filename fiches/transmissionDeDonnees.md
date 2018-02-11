# Transmission de données

## Données issues du serveur : les attributs
Nous nous sommes pour l’instant contenté de transmettre la requête HTTP de la servlet à la JSP, alors que l’objet HttpServletRequest contient énormément de méthodes.

Par exemple, nous pouvons transmettre une chaîne de caractères depuis notre servlet à la vue pour affichage.

```java
public void doGet( HttpServletRequest request, HttpServletResponse response ) throws ServletException, IOException{
	String message = "Transmission de variables : OK !";
	request.setAttribute( "test", message );
	this.getServletContext().getRequestDispatcher( "/WEB-INF/test.jsp" ).forward( request, response );
}
```

Il suffit d’appeler la méthode setAttribute() de l’objet requête pour y enregistrer un attribut, qui prend en paramètre le nom que l’on souhaite donner à l’attribut et la valeur de cet attribut.
L’attribut créé ici est une simple chaîne de caractères (objet de type String).

Pour récupérer et afficher l’objet côté vue :

```html
<%@ page pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>Test</title>
    </head>
    <body>
        <p>Ceci est une page générée depuis une JSP.</p>
        <p>
            <%
            String attribut = (String) request.getAttribute("test");
            out.println( attribut );
            %>
        </p>
    </body>
</html>
```

- la technologie JSP permet d’inclure du code Java en l’entourant des balises <% et %>. A l’intérieur, tout se passe comme si on écrivait du code dans une servlet avec la méthode println() de l’objet PrintWriter out
- il suffit d’appeler la méthode getAttribute() pour récupérer un attribut depuis une JSP. Nous avons transmis un objet String, mais il est possible de transmettre n’importe quel objet, comme un entier ou une liste. Il faut convertir (cast) l’objet récupéré dans la JSP au type souhaité car la méthode getAttribute() renvoie un objet global de type Object.

Ecrire du Java dans une JSP n’a aucun sens car l’intérêt des pages JSP est de s’affranchir du langage Java.
Cependant cela fonctionne, et c’est important de le connaître. Au chapitre suivant, nous allons tout mettre en oeuvre pour **ne plus écrire de Java dans une JSP**.


## Données issues du client : les paramètres

Les paramètres sont des données transmises par le client au serveur en les incluant dans l’URL de la méthode GET du protocole HTTP.

**La forme de l’URL**

```html
<!-- URL sans paramètres -->
/page.jsp
<!-- URL avec un paramètre nommé 'cat' et ayant pour valeur 'java' -->
/page.jsp?cat=java
<!-- URL avec deux paramètres nommés 'lang' et 'admin', et ayant pour valeur respectivement 'fr' et 'true' -->
/page.jsp?lang=fr&admin=true
```

- 1er paramètre séparé du reste de l’URL par le caractère ?
- les paramètres sont séparés entre eux par le caractère &
- une valeur est attribuée à chaque paramètre via l’opérateur =

Il n’y a pas d’autres moyen de déclarer des paramètres dans une requête GET, ils doivent apparaître en clair dans l’URL. La taille d’une URL est limitée, la taille des données envoyé doit donc aussi l’être. La limite avec IE8 est de 2083 caractères. Lorsqu’on a beaucoup de contenu à transmettre, il faut préférer la méthode POST.
Lorsqu’on envoie des données qui vont **avoir un impact sur la ressource demandée** (entraîner une modification de la ressource), il est préférable de passer par la méthode POST.

N’importe quel client peut envoyer des paramètres au serveur, en ajoutant à une URL des paramètres. Si la page n’en tient pas compte, le site n’en fera rien mais le serveur les recevra bien.

**Récupération des paramètres par le serveur**
Modifions légèrement la méthode doGet() de notre servlet en ajoutant la méthode getParameter() de l’objet requête avec le nom du paramètre que l’on souhaite récupérer, et en l’ajoutant ensuite à l’attribut test que l’on va afficher dans la JSP.

```java
String paramAuteur = request.getParameter( "auteur" );
String message = "Transmission de variables : OK ! " + paramAuteur;
request.setAttribute( "test", message );
```

Il suffit d’ajouter le paramètre dans l’URL pour que ça fonctionne : http://localhost:8080/test/toto?auteur=Coyote

Lorsqu’on envoie un paramètre, il reste présent dans la requête HTTP durant tout son cheminement, on peut donc y accéder depuis notre JSP sans passer par la servlet. Si aucun paramètre n’est précisé, la méthode getParameter() renverra null.

```java
String parametre = request.getParameter( "auteur" );
```

C’est très imprudent de procéder à l’utilisation ou à l’affichage d’un paramètre transmis par le client sans contrôler et éventuellement sécuriser son contenu auparavant.
L’utilisation la plus courante des paramètres dans une application web est la récupération de données envoyées par le client via des formulaires.


## Résumé

- **les paramètres de requête** sont un concept du protocole HTTP, ils sont envoyés par le client au serveur au sein de l’URL sous forme de chaînes de caractères. Il n’est pas possible de forger des paramètres dans l’objet HttpServletRequest, uniquement possible d’y accéder en lecture. Concept non spécifique à Java EE mais commun à toutes les technologies du web. Il ne peut pas être objectifié, la méthode getParameter renvoie un type String.
- **les attributs de requête** sont un concept du conteneur Java, ils sont créés côté serveur. C’est dans le code de l’application qu’on les initialise et qu’on les insère dans l’objet HttpServletRequest. Ils ne sont pas présents directement dans la requête HTTP mais uniquement dans l’objet Java qui l’enveloppe, et peuvent contenir n’importe quel type de données. Ils sont utilisés pour permettre à une servlet de communiquer avec d’autres servlets ou pages JSP.
