# Objectifs et configuration

Nous allons découvrir quelques concepts dans une brève introduction puis les fichiers de configuration clés du projet ainsi que les paramètres importants à modifier pour mettre en place la bibliothèque dans le projet web Java EE.

## C’est sa raison d’être
La JSTL est une bibliothèque, une collection regroupant des balises implémentant des fonctionnalités comme les boucles, les tests conditionnels, le formatage des données, la manipulation de données XML, afin d’éviter l’utilisation de code Java dans les pages JSP.

### Lisibilité du code
Les balises JSTL sont plus lisible que le code Java : une boucle en Java et la même chose avec la JSTL.

```html
<%@ page import="java.util.List, java.util.ArrayList" %>
<%
	List<Integer> list = (ArrayList<Integer>)request.getAttribute("tirage");
	for(int i = 0; i < list.size();i++){
		out.println(list.get(i));
	}
%>
```

```html
<c:forEach var="item" items="${tirage}" >
	<c:out value="${item}" />
</c:forEach>
```

### Moins de code à écrire
La syntaxe est simplifiée et raccourcie en utilisation la JSTL. L’usage des scriptlets (le code Java entouré de <% %>) est fortement déconseillé depuis l’apparition des TagLibs (notamment la JSTL) et des EL.  
Les principaux inconvénients des scriptlets sont les suivants :
- Réutilisation : il est impossible de réutiliser une scriptlet dans une autre page, il faut la dupliquer
- Interface : il est impossible de rendre une scriptlet abstract
- POO : il est impossible dans une scriptlet de tirer parti de l’héritage ou de la composition
-  Debug : si une scriptlet envoie une exception en cours d’exécution, tout s’arrête et l’utilisateur récupère une page blanche
- Tests : on ne peut pas écrire de tests unitaires pour tester les scriptlets
- Maintenance : on passe plus de temps à maintenir un code mélangé, encombré, dupliqué et non testable !

### Pattern MVC
Une JSP sans Java est plus courte et plus lisible. Cela permet aussi de prendre de bonnes habitudes en ne consacrant la vue, en l'occurrence la JSP, qu’à l’affichage. Ne pas déclarer de méthodes dans une JSP, ne pas y modifier de données, ne pas y insérer de traitement métier… Tout cela est recommandé mais la frontière peut paraître mince si on utilise des scriptlets Java, alors qu’avec les tags JSTL, la séparation est plus nette.

### Plusieurs versions
- JSTL 1.0 pour J2EE 3, et un conteneur JSP 1.2 (ex : Tomcat 4)
- JSTL 1.1 pour J2EE4, et un conteneur JSP 2.0 (ex : Tomcat 5.5)
- JSTL 1.2 qui est partie intégrante de Java EE 6, avec un conteneur JSP 2.1 ou 3.0 (ex : Tomcat 6 et 7)

Le changement majeur à retenir est le passage de la 1.0 à la 1.1 est la gestion de la technologie EL.
La version 1.2 apporte des EL “unifiées” et une meilleure intégration du framework JSF, ce qui n’aura aucun impact sur ce cours.


## Configuration

Si nous mettons directement une balise JSTL dans notre JSP, par exemple pour afficher du texte dans une page, cela ne va pas fonctionner car Eclipse ne connaît pas la balise, ce qui est logique car elle est issue d’une bibliothèque (la JSTL). Il faut donc préciser à Éclipse où ce tag est défini, grâce à une directive JSP, destinée à inclure des bibliothèques. Elle s’appelle la directive taglib. Voici donc le code modifié pour inclure la bibliothèque Core :

```html
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
		<title>Test</title>
	</head>
	<body>
		<c:out value="test" />
	</body>
</html>
```

Dans le paramètre **uri** se trouve le lien vers la définition de la bibliothèque. Il y a dans l’arborescence **/jsp/jstl/core** pour la version 1.2. Le lien de la version 1.0 est /jstl/core, il faut faire attention d’avoir le bon lien pour éviter les erreurs.  
Dans le paramètre **prefix** se trouve l’alias qui sera utilisé dans la JSP. Si on met “core” dans le champ prefix de la directive, il faut alors écrire `<core:out>` à la place de `<c:out>`.

La directive n’est pas reconnu par la JSP. C’est Tomcat qui est en cause, car il n’est pas un serveur d’applications Java EE au sens complet du terme. Alors que la JSTL fait parti intégrante de la plateforme Java EE 6, Tomcat 7 n’est pas livré par défaut avec la JSTL. En utilisant un serveur qui respecte les spécifications Java EE, on ne rencontrera pas de problèmes car la JSTL est bien incluse.  
Il faut donc définir où se situe cette bibliothèque et configurer notre projet pour qu’il puisse y accéder. La JSTL contient nativement plusieurs bibliothèques, Core est l’une d’entre elles. On va ajouter l’archive jar de la JSTL entière à notre projet dans le répertoire **lib** de notre **/WEB-INF**.

- La JSTL est composée de cinq bibliothèques de balises standard
- elle permet d’éviter l’utilisation de code Java dans les JSP, de réduire la quantité de code à écrire, et de rendre le code des JSP plus lisible
- sous Tomcat, il faut placer son fichier jar sous /WEB-INF/lib pour qu’elle soit correctement intégrée.
