# La technologie JSP (1/2)

## Les balises

### Balises de commentaires
Les commentaires JSP doivent être compris entre les balises <%-- ert →

```html
<%-- Ceci est un commentaire JSP, non visible dans la page HTML finale.  --%>
<!-- Ceci est un simple commentaire HTML. -->
```

### Balises de déclaration

Permet de déclarer une variable à l’intérieur d’une JSP. C’est déconseillé d’écrire du java dans une JSP mais il faut la connaître.

```html
<%! String chaine = "Salut les zéros.";  %>
```

### Balises de scriptlet
Mélange entre “script” et “servlet”, elle permet d’inclure du code Java. L’exemple ci-dessous est très moche car il mélange du code Java et du code HTML.

```html
<form action="/tirage" method="post">
  <%
    for(int i = 1; i < 3; i++){
      out.println("Numéro " + i + ": <select name=\"number"+i+"\">");
      for(int j = 1; j <= 10; j++){
        out.println("<option value=\""+j+"\">"+ j + "</option>");
      }
      out.println("</select><br />");
    }
    %>
    <br />
    <input type="submit" value="Valider" />
</form>
```

### Balises d’expression
C’est un raccourci de la scriptlet.
```html
<% out.println("Bip bip !"); %>
```

Elle retourne simplement le contenu d’une chaîne :
```html
<%= "Bip bip !" %>
```

Absence de point-virgule lors de l’utilisation de ce raccourci


## Les directives
Les directives JSP permettent :
- d’importer un package
- d’inclure d’autres pages JSP
- d’inclure des bibliothèques de balises (cf autre chapitre plus tard)
- de définir des propriétés et informations relatives à une page JSP

Pour généraliser, elles contrôlent comment le conteneur de servlets va gérer notre JSP. Il en existe trois : **taglib, page, include**.  
Elles sont toujours comprises entre les balises `<%@` et `%>`, et hormis la directive d’inclusion de page qui peut être placée n’importe où, elles sont à placer en tête de page JSP.

### Directive taglib
```java
<%@ taglib uri="maTagLib.tld" prefix="tagExemple" %>
```
Permet d’inclure une bibliothèque, ici une bibliothèque personnalisée maTagLib. Nous reviendrons plus tard dessus.

### Directive page

Elle définit des informations relatives à la page JSP.
Par exemple, pour importer des classes Java :
```java
<%@ page import="java.util.List, java.util.Date"  %>
```

Utile seulement si on utilise du code Java, on va donc s’en passer.

Il existe d’autres options via cette balise page, comme le **contentType**, l’activation de la session, le **pageEncoding**. Toutes ont des valeurs par défaut, on ne s’en servira que rarement.

### Directive include
Une page est rarement constitué d’une seule JSP, c’est courant de découper une page web en plusieurs fragments qui sont ensuite rassemblés dans la page finale à destination de l’utilisateur, ce qui permet de réutiliser certains blocs dans différentes vues, comme le menu par exemple. Pour faire cela, il existe une balise qui inclut le contenu d’un autre fichier (une JSP, un fichier HTML…) dans le fichier courant.

```java
<%@ include file="uneAutreJSP.jsp" %>
```

On ne doit utiliser cette directive que pour du contenu statique, car l’inclusion est réalisée au moment de la compilation donc si le code du fichier change, les répercussions n’auront lieu qu’après une nouvelle compilation. C’est un peu comme **un simple copier-coller d’un fichier à l’autre**.

**Action standard include**
Une autre balise d’inclusion dite “standard” permet d’inclure du contenu “dynamique”. Le contenu sera chargé à l’exécution et non plus à la compilation.

```html
<%-- L'inclusion dynamique d'une page fonctionne par URL relative : --%>
<jsp:include page="page.jsp" />

<%-- Son équivalent en code Java  est : --%>
<% request.getRequestDispatcher( "page.jsp" ).include( request, response ); %>

<%-- Et il est impossible d'inclure une page externe comme ci-dessous :  --%>
<jsp:include page="http://www.siteduzero.com" />
```

L’inconvénient est qu’elle ne prend pas en compte les imports et inclusions faits dans la page réceptice. Par exemple, si on utilise un type List dans une première page, et qu’on compte utiliser une liste dans une seconde page qu’on inclue dans la première page, il faudra importer le type List dans la seconde page.  
Les pages incluses via la balise `<jsp:include .../>` doivent être “indépendantes”, elles doivent pouvoir être compilées séparément, ce qui n’est pas le cas des pages includes via la directive `<%@include … %>`.
- certains serveurs d’applications sont capables de recompiler une page JSP incluant une autre page via la directive d’inclusion, et ainsi éclipser sa principale contrainte.
- pour inclure un même header et un même footer dans toutes les pages de l’application, il est préférable de ne pas utiliser ces techniques d’inclusion mais de spécifier ces portions communes dans le fichier web.xml.
- nous allons découvrir une meilleure technique d’inclusion de pages avec la JSTL.


## La portée des objets
Souvent appelée visibilitée ou **scope** en anglais, elle définit la durée de vie des objets.
Il existe quatre portées différentes :
- **page** (JSP seulement) : les objets dans cette portée sont uniquement accessibles dans la page JSP en question
- **requête** :les objets dans cette portée sont uniquement accessible durant l’existence de la requête en cours
- **session** : objets accessible durant l'existence de la session
- **application** : objets accessible durant l’existence de l’application

Lors du chapitre sur la transmission de données, nous avons créé un objet de portée requête depuis notre servlet puis utilisé depuis notre page JSP. En revanche, il n’est possible de créer et manipuler des objets de portée page que depuis une page JSP, ce n’est pas possible via la servlet.  
**Une session est un objet associé à un utilisateur** en particulier (plus précisément à un navigateur), qui existe pour la durée pendant laquelle un visiteur va utiliser l’application et se termine quand l’utilisateur ferme son navigateur ou est inactif trop longtemps, ou encore se déconnecte du site.  
Il est donc possible de garder en mémoire des données concernant un visiteur d’une requête à l’autre, autrement dit de page en page.  
L’utilisation d’objet ayant une portée application est délicate car ces objets sont accessibles partout, tout le temps et par tout le monde. Afin d’éviter notamment des problèmes de modifications concurrentes, l’initialisation de tels objets doit se faire de préférence dès le chargement de l’application, ne plus toucher à leur contenu, et d'y accéder depuis nos classes et pages uniquement **en lecture seule**.


## Les actions standard

Permet d’accéder à des objets bean depuis une JSP. Cette façon de faire n’est pas la meilleure, elle est déconseillée. Nous en verrons une plus simple et plus propre ensuite.

On connaît déjà l’action standard `<jsp:include>`, on va en voir quatre autres : `<jsp:useBean>`, `<jsp:getProperty>`, `<jsp:setProperty>` et `<jsp:forward>`

### L’action standard useBean
Cette action permet de récupérer un bean.

```html
<%-- L'action suivante récupère un bean de type Coyote et nommé "coyote"
dans la portée requête s'il existe, ou en crée un sinon. --%>
<jsp:useBean id="coyote" class="com.sdzee.beans.Coyote" scope="request" />

<%-- Elle a le même effet que le code Java suivant : --%>
<%
com.sdzee.beans.Coyote coyote = (com.sdzee.beans.Coyote) request.getAttribute( "coyote" );
if ( coyote == null ){
    coyote = new com.sdzee.beans.Coyote();
    request.setAttribute( "coyote", coyote );
}
%>
```

- id est le nom du bean à récupérer, ou le nom du bean à créer
- classe correspond à la classe du bean, obligatoire pour créer un bean, optionnel pour en récupérer un existant
- scope correspond à la portée. si un bean du nom spécifié existe dans ce scope, il est récupéré, sinon un nouveau bean est créé. Si cet attribut n’est pas renseigné, le scope par défaut est la page en cours
- type est un attribut optionnel qui indique le type de déclaration de bean, il doit être une superclasse du bean ou une interface du bean. Il doit être spécifié si class ne l’est pas et vice-versa

Cette action permet de stocker un bean (nouveau ou existant) dans une variable, identifiée par la valeur saisie dans id.

### L’action standard getProperty

permet d’obtenir la valeur d’une propriété d’un bean

### L’action standard setProperty
permet de modifier une propriété du bean utilisé. Il existe quatre façons de faire

### L’action standard forward
permet de faire une redirection vers une autre page. Elle s’effectue côté serveur, comme toutes les actions standard, **il est donc impossible via cette balise de rediriger vers une page extérieure à l’application**. L’action est donc limité aux pages présentes dans le contexte de la servlet ou de la JSP utilisée.

```html
<%-- Le forwarding vers une page de l'application fonctionne par URL relative : --%>
<jsp:forward page="/page.jsp" />

<%-- Son équivalent en code Java  est : --%>
<% request.getRequestDispatcher( "/page.jsp" ).forward( request, response ); %>

<%-- Et il est impossible de rediriger vers un site externe comme ci-dessous :  --%>
<jsp:forward page="http://www.siteduzero.com" />
```

Cela n’implique pas d’aller/retour passant par le navigateur, l’utilisateur ne sait donc pas que sa requête a été redirigé vers une ou plusieurs JSP différentes puisque l’URL dans son navigateur ne change pas. **Le code présent après cette balise dans la page n’est pas exécuté.**

Toutes ces notions sont obsolètes, il existe des meilleures façons de faire.
