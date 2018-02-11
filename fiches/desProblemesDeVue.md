# Des problèmes de vues ?

## Nettoyons notre exemple
Maintenant que nous connaissons la technologie JSP et les EL, nous sommes capable de remplacer le code Java que nous avions écrit dans notre vue par quelque chose de plus propre, lisible et qui suit les recommandations du MVC. Voici où nous en étions après l’introduction d’un bean dans notre exemple :

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

            String parametre = request.getParameter( "auteur" );
            out.println( parametre );
            %>
        </p>
        <p>
            Récupération du bean :
            <%
	    com.sdzee.beans.Coyote notreBean = (com.sdzee.beans.Coyote) request.getAttribute("coyote");
	    out.println( notreBean.getPrenom() );
            out.println( notreBean.getNom() );
            %>
        </p>
    </body>
</html>
```

Pour bien couvrir l’ensemble des méthodes existantes, divisons le travail en deux étapes : avec des scripts et balises JSP, puis avec des EL.

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
            <% String attribut = (String) request.getAttribute("test"); %>
            <%= attribut %>

            <% String parametre = request.getParameter( "auteur" ); %>
            <%= parametre %>
        </p>
        <p>
            Récupération du bean :
            <jsp:useBean id="coyote" class="com.sdzee.beans.Coyote" scope="request" />
            <jsp:getProperty name="coyote" property="prenom" />
            <jsp:getProperty name="coyote" property="nom" />
        </p>
    </body>
</html>
```

On affiche l’attribut et le paramètre via la balise d’expression `<%= %>`. On récupère le bean depuis la requête via la balise `<jsp:useBean>`. On affiche le contenu des propriétés via les balises `<jsp:getProperty>`.

Pour récupérer le bean “coyote”, l’action s’appuie sur l’objet implicite request. Elle cherche un bean nommé coyote dans la requête et en crée un si elle n’en trouve pas. Si on avait précisé session dans l’attribut scope de l’action, elle aurait cherché dans l’objet session. Sans préciser de scope, elle cherche dans l’objet page et en crée un si elle n’en trouve pas.

Maintenant avec des EL

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
            ${test}
            ${param.auteur}
        </p>
        <p>
            Récupération du bean :
            ${coyote.prenom}
            ${coyote.nom}
        </p>
    </body>
</html>
```

Leur simplicité d’utilisation est déconcertante, et notre page ne contient désormais plus une seule ligne de Java !

Nous n’utilisons aucun objet implicite pour accéder aux attributs test et coyote. Avec les actions standard, il est nécessaire de préciser la portée, sinon l’action cherche par défaut dans la portée page.  
Le fonctionnement de la technologie EL est différent. Pour accéder à un objet, présent dans une des quatre portées de l’application, il n’y a pas besoin de spécifier l’objet implicite (la portée) auquel on souhaite accéder. Si on avait écrit `${request.test}` pour accéder à l’objet test de la portée request ou `${request.coyote.prenom}`, ça n’aurait pas fonctionné ! L’objet implicite request ne représente pas la portée request mais l’objet requête en cours d’utilisation.
Le mécanisme de la technologie EL est plus évolué : une expression réalise elle-même un parcours automatique des différentes portées accessibles à la recherche d’un objet portant le nom précisé, de la plus petite à la plus grande. La bonne pratique est donc de ne jamais donner le même nom à des objets existants dans des portées différentes.  
Pour éviter le parcours automatique et cibler directement une portée, il faut utiliser les objets implicites de la technologie EL donnant accès aux attributs existants : **pageScope, requestScope, sessionScope, applicationScope**. Ainsi dans notre exemple précédent, on aurait pu écrire `${requestScope.test}` à la place de `${test}` et cela aurait fonctionné aussi bien. Lors de l’analyse de l’expression EL, le conteneur aurait ainsi reconnu l’objet implicite requestScope et n’aurait pas effectué le parcours des portées : il aurait directement recherché un objet nommé “test” au sein de la portée request uniquement. La bonne pratique veut également que le développeur précise toujours la portée qu’il souhaite cibler.

Pour le paramètre auteur, il n’est pas présent dans une des quatre portées, il est nécessaire d’expliciter l’objet implicite qui permet d’y accéder au sein de l’expression EL. Voilà pourquoi nous avons précisé “param” dans `${param.auteur}`. Si nous avions écrit `${auteur}`, cela n’aurait pas fonctionné car le mécanisme de recherche automatique aurait tenté de trouver un objet “auteur” dans une des quatre portées et n’aurait rien trouvé, car un paramètre de requête est différent d’un attribut de requête. Cela est valable pour tout objet implicite différent des quatre portées : param, header, cookie, paramValues, etc...

## Complétons notre exemple
Les EL ne permettent pas de mettre en place tout ce dont nous aurons besoin dans une vue. Par exemple :
- afficher le contenu d’une liste ou d’un tableau à l’aide d’une boucle
- afficher un texte différent avec une condition (jour pair ou impair par exemple)

Nous ne savons pas encore le faire sans Java.

Pour afficher un objet de type List, il faut réaliser un import avec la directive page, récupérer la liste depuis l’objet requête et afficher son contenu via une boucle for. Cela fonctionne mais avec du code Java, ce que nous voulons éviter.
```html
<p>
    Récupération de la liste :
    <%
    List<Integer> liste = (List<Integer>) request.getAttribute( "liste" );
    for( Integer i : liste ){
        out.println(i + " : ");
    }
    %>
</p>
```

De même, pour utiliser une condition, nous sommes obligé d’utiliser un if/else dans notre JSP pour faire un simple test sur un entier.

```html
<p>
    Récupération du jour du mois :
    <%
    Integer jourDuMois = (Integer) request.getAttribute( "jour" );
    if ( jourDuMois % 2 == 0 ){
        out.println("Jour pair : " + jourDuMois);
    } else {
        out.println("Jour impair : " + jourDuMois);
    }
    %>
</p>
```

Pour ne pas écrire de code Java, il va falloir utiliser le **JSTL, qui est une bibliothèque de balises préconçues** qui permettent, à la manière des balises JSP, de mettre en place les fonctionnalités dont nous avons couramment dans une vue, mais qui ne sont **pas accessibles nativement au sein de la technologie JSP**.

## Le point sur ce qu’il nous manque
### La vue
Nous savons afficher des choses basiques mais dès que notre vue se complexifie un peu, nous ne savons plus faire. Nous allons nous tourner vers le JSTL.

### L’interaction
Nous n’avons fait qu’effleurer la récupération des données envoyées par le client. nous avons développé des vues basiques, couplées à des servlets qui ne font presque rien. Nous allons découvrir que les servlets auront pour objectif de tout contrôler : tout ce qui arrive dans application et ce qui en sortira passera par nos servlets.

### Les données
Nous avons découvert ce qu’est un bean, nous avons une vague idée de comment seront représentées les données au sein du modèle : à chaque entité de données correspondra un bean. Nous allons apprendre à manipuler une base de données depuis notre application, sauvegarder les données de nos beans, enregistrer ce que nous transmet le client… Nous découvrirons que le modèle est en réalité constitué non pas d’une couche mais de deux.
