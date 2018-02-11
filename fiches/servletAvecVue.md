# Servlet avec vue

Le modèle MVC nous conseille de placer ce qui touche à l’affichage dans une couche à part : la vue. La technologie utilisée pour réaliser une vue est la **page JSP**.


## Introduction aux JSP

C’est un document qui ressemble à première vue beaucoup à une page HTML, mais qui est différent :
- l’extension devient .jsp et non plus .html
- elle peut contenir des balises HTML et aussi des balises JSP qui appellent du code Java
- contrairement à une page HTML statique directement envoyé au client, **une page JSP est exécutée côté serveur**, et génère alors une page renvoyée au client

L’intérêt d’une page dynamique est qu’il est possible de faire varier l’affichage et d’interagir avec l’utilisateur en fonction de la requête et des données reçues !

Il est possible de générer n’importe quel type de format avec une page JSP : HTML, CSS, XML… Dans la grande majorité des cas, il s’agira de pages HTML destinées à l’affichage sur le navigateur du client.

Les JSP sont une des technologies de la plateforme Java EE, elles se présentent sous la forme d’un fichier texte contenant des balises et respectant une syntaxe. Le langage JSP combine les technologies HTML, XML, servlet et JavaBeans (c’est un objet Java).

Intérêt des JSP :
- la technologie servlet est trop difficile d’accès et ne convient pas à la génération du code de présentation : écrire une page web en langage Java est très pénible. Les pages JSP sont une **technologie qui joue le rôle de simplification de l’API servlet**, c’est une abstraction de “haut niveau” de la technologie servlet
- **le modèle MVC recommande une séparation nette entre le code de contrôle et la présentation**. Il est théoriquement possible d’utiliser des servlets pour le contrôle et d’autres pour l’affichage mais nous rejoignons alors le point précédent, la servlet n’est pas adaptée à la prise en charge de l’affichage
- **le modèle MVC recommande une séparation nette entre le code métier et la présentation**. Dans le modèle, on doit trouver le code Java responsable de la génération des éléments dynamiques, et dans la vue l’interface utilisateur.

**La technologie JSP offre les capacités dynamiques des servlets tout en permettant une approche naturelle pour la création de contenus statiques**, grâce à :
- un **langage dédié**
- la **simplicité d’accès aux objets Java**, grâce à des balises du langage
- des **mécanismes permettant l’extension du langage** utilisé au sein des pages JSP, en mettant en place des balises qui n’existent pas dans le langage JSP pour augmenter les fonctionnalités accessibles (cf JSTL)


## Mise en place d’une JSP

### Création de la vue

Création d’un fichier JSP par défaut dans le dossier WebContent.

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>Test</title>
    </head>

    <body>
        <p>Ceci est une page générée depuis une JSP.</p>
    </body>
</html>
```

En se rendant à l’URL de la page, on constate que les caractères accentués sont mal affichés. Dans le précédent exemple, nous avions modifié l’encodage par défaut directement dans la servlet. Cette fois, nous ne l’avons pas fait dans la JSP, par conséquent la réponse créée par notre page utilise la valeur par défaut, c’est-à-dire l’encodage latin ISO-8859-1.
- en ce qui concerne le serveur, si la configuration dans Eclipse est bonne, nos fichiers sources sont encodés en UTF-8.
- en ce qui concerne le navigateur, celui-ci va uniquement se baser sur le contenu de l’en-tête Content-type de la réponse HTTP afin d’interpréter les données reçues.

Par défaut, la page JSP a envoyé la réponse au client en spécifiant l’encodage latin dans l’en-tête HTTP. Le navigateur a essayé d’afficher des caractères encodés en UTF-8 en utilisant l’encodage latin ISO-8859-1 alors que ce sont 2 encodages différents.
Il faut donc modifier l’en-tête HTTP depuis notre page JSP afin de faire savoir au navigateur qu’il doit utiliser l’encodage UTF-8 pour interpréter les données reçues, en ajoutant une instruction en tête de notre page JSP.

```java
<%@ page pageEncoding="UTF-8" %>
```

### Cycle de vie d’une JSP

**En théorie**
Quand la JSP est demandée pour la première fois, **le conteneur de servlets va vérifier, traduire puis compiler la page JSP en une classe héritant de HttpServlet**, et l’utiliser durant l’existence de l’application.
**La JSP est transformée en servlet par le serveur.**
La technologie JSP est une abstraction de la technologie servlet, ce qui signifie que **les JSP permettent au développeur de faire du Java sans avoir à écrire de code Java**. Le code JSP est une succession de raccourcis qui dans les coulisses appellent en réalité des portions de code Java toutes prêtes.

![Structure d'une application Web](images/cycleDeVieJsp.png)

Le processus de vérification/traduction/compilation n’est pas effectué à chaque appel ! La servlet générée et compilée étant **sauvegardée**, les appels suivants à la JSP sont beaucoup plus rapide car le conteneur se contente d’exécuter directement l’instance de la servlet stockée en mémoire.

**En pratique**
Ont peut trouver le code source de la servlet générée à partir de la JSP dans le répertoire de travail du serveur Tomcat /work.
C’est celui-ci qui est compilé et exécuté lorsque notre JSP est appelée.


## Mise en relation avec notre servlet

### Garder le contrôle

Dans l’exemple précédent, nous avons appelé directement la JSP depuis son URL. C’est pratique pour l’exemple mais il ne faut jamais faire cela dans une application Java EE car on ne respecte pas le modèle MVC. Il faut mettre en place un contrôleur, en associant une servlet à la vue.
Il faudra systématiquement créer une servlet lorsque nous créons une page JSP, ce qui permet de garder le contrôle en s’assurant que la vue ne sera jamais appelée par le client sans être passée à travers une servlet. **La servlet est le point d’entrée de notre application.**

Nous allons déplacer notre page JSP dans le répertoire **/WEB-INF**, qui cache les ressources qu’il contient. Une page présente sous ce répertoire n’est plus accessible directement par une URL pour donner l’accès à cette page, il devient nécessaire de passer par une servlet côté serveur pour donner l’accès à cette page, plus d’oubli possible !

Nous devons donc associer notre servlet à notre vue, opération réalisée depuis la méthode doGet() de notre servlet car c’est elle qui décide d’appeler la vue.

```java
public void doGet( HttpServletRequest request, HttpServletResponse response )	throws ServletException, IOException {
	this.getServletContext().getRequestDispatcher( "/WEB-INF/test.jsp" ).forward( request, response );
}
```

- depuis notre instance de servlet (this), nous appelons la méthode getServletContext() qui nous retourne un objet ServletContext qui fait référence au contexte commun à toute l’application, qui contient un ensemble de méthode pour communiquer avec le conteneur de servlets.
- la méthode getRequestDispatcher(), que nous appliquons à notre page JSP, permet de manipuler une ressource. Elle retourne un objet RequestDispatcher qui agit comme une enveloppe autour de notre JSP. C’est grâce à cet objet que notre servlet est capable de faire suivre nos objets requête et réponse à une vue. Il est impératif d’y préciser le chemin complet vers la JSP.
- la méthode forward() du dispatcher permet de réexpédier la paire requête/réponse HTTP vers la page JSP.

La page JSP devient accessible au travers de la servlet, son seul rôle est actuellement de transférer le couple requête/réponse vers la JSP.
Le navigateur nous affiche bien maintenant le contenu de la page.

## Résumé
- une page JSP ressemble en apparence à une page HTML mais elle est plus proche d’une servlet : elle contient des balises derrière lesquelles se cache du code Java
- une JSP est exécutée sur le serveur et la page finale générée et envoyée au client est une page HTML, le client ne voit pas le code de la JSP
- dans le modèle MVC, une JSP est accessible à l’utilisateur à travers une servlet et non pas directement
- le répertoire /WEB-INF cache les fichiers qu’il contient à l’extérieur de l’application
- la méthode forward() de l’objet RequestDispatcher permet depuis une servlet de rediriger la paire requête/réponse HTTP vers une autre servlet ou vers une page JSP
