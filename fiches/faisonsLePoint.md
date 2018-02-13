# Faisons le point !

Quelques conseils

## Utilisation de constantes
Pour faciliter la lecture et la modification du code d’une classe, il est recommandé de regrouper sous forme de constantes en début de classe le contenu des attributs de type primitifs au lieu de les écrire en dur au sein du code.
Par exemple, dans la méthode doGet de notre servlet :

```java
/** Stockage du message, du bean et de la liste dans l'objet request */
request.setAttribute( "test", message );
request.setAttribute( "coyote", premierBean );
request.setAttribute( "liste", premiereListe );
request.setAttribute( "jour", jourDuMois );
```

En mettant des constantes :

```java
public class Test extends HttpServlet {
	public static final String ATT_MESSAGE = "test";
	public static final String ATT_BEAN	 = "coyote";
	public static final String ATT_LISTE	 = "liste";
	public static final String ATT_JOUR	 = "jour";
	public static final String VUE	 = "/WEB-INF/test.jsp";
  public void doGet( HttpServletRequest request, HttpServletResponse response )
  throws ServletException, IOException{

  /** Stockage du message, du bean et de la liste dans l'objet request */
	request.setAttribute( ATT_MESSAGE, message );
	request.setAttribute( ATT_BEAN, premierBean );
	request.setAttribute( ATT_LISTE, premiereListe );
	request.setAttribute( ATT_JOUR, jourDuMois );
    }
  }
```

L’intérêt est que toutes les données utilisées en dur au sein de la classe sont accessibles en un coup d’oeil. En nommant intelligemment les constantes, on peut savoir sans parcourir le code à quoi elles correspondent, par exemple en mettant ATT en préfixe des noms des attributs de requête et en nommant VUE le chemin vers la JSP. Si on souhaite modifier une de ces données, il ne sera pas nécessaire de parcourir le code.


## Inclure automatiquement la JSTL Core à toutes les JSP
Pour utiliser les balises de la bibliothèques Core dans nos JSP, il faut inclure la directive **include** en tête de chaque JSP.
```html
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

Afin d’éviter de dupliquer cette ligne dans l’intégralité des vues, il est possible de rendre cette inclusion automatique dans le fichier **web.xml**.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
	<jsp-config>
		<jsp-property-group>
			<url-pattern>*.jsp</url-pattern>
			<include-prelude>/WEB-INF/taglibs.jsp</include-prelude>
		</jsp-property-group>
	</jsp-config>
```

Le fonctionnement est très simple, `<jsp-property-group>` contient deux balises :
- `<url-pattern>` qui permet de spécifier à quels fichiers appliquer l’inclusion automatique, ici tous les fichiers JSP
- `<include-prelude>` qui précise l’emplacement du fichier à inclure en tête de chacune des pages, ici le fichier taglib.jsp.

Il ne reste plus qu’à créer un fichier taglibs.jsp sous le répertoire /WEB-INF et y placer la directive taglib.


```html
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

Ce système est équivalent à une inclusion statique, en d’autres termes à une directive `<%@ include file="/WEB-INF/taglibs.jsp" %>` placée en tête de chaque JSP.

Il est possible d’inclure automatiquement un fichier en fin de page en utilisant `<include-coda>` à la place de `<include-prelude>`.

## Formater proprement et automatiquement notre code avec Eclipse
On peut utiliser un style de formatage par défaut (window > preferences > java > code style > formatter), en créer un ou en importer un. Une fois qu’il est appliqué, il suffit de faire CTRL + MAJ + F dans une page pour la formater.

Eclipse peut automatiser le formatage à chaque sauvegarde afin de conserver un formatage parfait (window > preferences > java > editor > save actions), il faut cocher “Perform the selected actions on save” et “format source code”.
