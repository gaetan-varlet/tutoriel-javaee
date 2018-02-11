# La technologie JSP (2/2)

## Expression Language
L’Expression Language est souvent raccourci en EL. Elles permettent de **se passer des actions standards d’utilisation des beans**. Les expressions EL permettent via une syntaxe très épurée d’effectuer des tests basiques sur des expressions, et de manipuler simplement des objets et attributs dans une page sans utiliser de code ou de script Java. La syntaxe est la suivante : `${ expression } `
C’est ce qui est situé entre les accolades qui va être interprété.

### La réalisation de tests
On peut effectuer des tests à l’intérieur d’une expression. grâce à des opérateurs arithmétiques pour les nombres (+ - * / %), logiques pour les booléens (&& || !) et relationnels (== ou eq, != ou ne, < ou lt, > ou gt, <= ou le, >= ou ge).

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>Test des expressions EL</title>
    </head>
    <body>
    <p>
        <!-- Logiques sur des booléens -->
        ${ true && true } <br /> <!-- Affiche true -->
        ${ true && false } <br /> <!-- Affiche false -->
        ${ !true || false } <br /> <!-- Affiche false -->

        <!-- Calculs arithmétiques -->
        ${ 10 / 4 } <br /> <!-- Affiche 2.5 -->
        ${ 10 mod 4 } <br /> <!-- Affiche le reste de la division entière, soit 2 -->
        ${ 10 % 4 } <br /> <!-- Affiche le reste de la division entière, soit 2 -->
        ${ 6 * 7 } <br /> <!-- Affiche 42 -->
        ${ 63 - 8 } <br /> <!-- Affiche 55 -->
        ${ 12 / -8 } <br /> <!-- Affiche -1.5 -->
        ${ 7 / 0 } <br /> <!-- Affiche Infinity -->

        <!-- Compare les caractères 'a' et 'b'. Le caractère 'a' étant bien situé avant le caractère 'b' dans l'alphabet ASCII, cette EL affiche true. -->
        ${ 'a' < 'b' } <br />  

        <!-- Compare les chaînes 'hip' et 'hit'. Puisque 'p' < 't', cette EL affiche false. -->
        ${ 'hip' gt 'hit' } <br />

        <!-- Compare les caractères 'a' et 'b', puis les chaînes 'hip' et 'hit'. Puisque le premier test renvoie true et le second false, le résultat est false. -->
        ${ 'a' < 'b' && 'hip' gt 'hit' } <br />

        <!-- Compare le résultat d'un calcul à une valeur fixe. Ici, 6 x 7 vaut 42 et non pas 48, le résultat est false. -->
        ${ 6 * 7 == 48 } <br />
    </p>
    </body>
</html>
```

Pour les objets String, on peut utiliser des double quotes (guillemets) ou aussi des simple quotes (apostrophes).  
Pour réaliser l’égalité d’objets de type non standard (autre que String ou Integer par exemple) via une EL, il faudra réimplémenter les méthodes equals() ou compareTo().

Il existe aussi deux autres types de tests fréquemment utilisés :
- les conditions ternaires, de la forme `test ? si oui : sinon`
- les vérification si vide ou null, grâce à l’opérateur `empty`

```html
<!-- Vérifications si vide ou null -->
${ empty 'test' } <!-- La chaîne testée n'est pas vide, le résultat est false -->
${ empty '' } <!-- La chaîne testée est vide, le résultat est true -->
${ !empty '' } <!-- La chaîne testée est vide, le résultat est false -->

<!-- Conditions ternaires -->
${ true ? 'vrai' : 'faux' } <!-- Le booléen testé vaut true, vrai est affiché -->
${ 'a' > 'b' ? 'oui'  : 'non' } <!-- Le résultat de la comparaison vaut false, non est affiché -->
${ empty 'test' ? 'vide' : 'non  vide'  } <!-- La chaîne testée n'est pas vide, non vide est affiché -->
```

### La manipulation d’objets
Le véritable intérêt des expressions EL est que l’on peut manipuler des objets et leur appliquer tous ces tests, notamment les beans. On peut accéder aux propriétés d’un bean par exemple.
Pour afficher la propriété prénom du bean nommé coyote, il suffit d’écrire dans la JSP :
```html
${ coyote.prenom }
```

coyote est le nom du bean, prenom est un champ privé du bean (accessible par getPrenom()) et l’opérateur point permet de séparer le bean visé de sa propriété.
Le navigateur affiche alors : “While E.”
Le code java mis en oeuvre dans les coulisses lors de l’interprétation de l’EL est :

```java
Coyote bean = (Coyote) pageContext.findAttribute( "coyote" );
if ( bean != null ) {
    String prenom = bean.getPrenom();
    if ( prenom != null ) {
        out.print( prenom );
    }
}
```

L’expression EL simplifie l’écriture de manière frappante.
Ci-dessous quelques exemples :

```html
<!-- Syntaxe conseillée pour récupérer la propriété 'prenom' du bean 'coyote'. -->
${ coyote.prenom }

<!-- Syntaxe correcte, car il est possible d'expliciter la méthode d'accès à la propriété. Préférez toutefois la notation précédente. -->
${ coyote.getPrenom() }

<!-- Syntaxe erronée : la première lettre de la propriété doit être une minuscule. -->
${ coyote.Prenom }
```

Les EL permet de manipuler les beans au sein de tests :

```html
<!-- Comparaison d'égalité entre la propriété prenom et la chaîne "Jean-Paul" -->
${ coyote.prenom == "Jean-Paul" }

<!-- Vérification si la propriété prenom est vide ou nulle -->
${ empty coyote.prenom }

<!-- Condition ternaire qui affiche la propriété prénom si elle n'est ni vide ni nulle, et la chaîne "Veuillez préciser un prénom" sinon -->
${ !empty coyote.prenom ? coyote.prenom : "Veuillez préciser un prénom" }
```

En outre, **les expressions EL sont protégées contre un éventuel retour null** :
```html
<!-- La scriptlet suivante affiche "null" si la propriété "prenom" n'a pas été initialisée,
et provoque une erreur à la compilation si l'objet "coyote" n'a pas été initialisé : -->
<%= coyote.getPrenom() %>

<!-- L'action suivante affiche "null" si la propriété "prenom" n'a pas été initialisée,
et provoque une erreur à l'exécution si l'objet "coyote" n'a pas été initialisé : -->
<jsp:getProperty name="coyote" property="prenom" />

<!-- L'expression EL suivante n'affiche rien si la propriété "prenom" n'a pas été initialisée,
et n'affiche rien si l'objet "coyote" n'a pas été initialisé : -->
${ coyote.prenom }
```

Au delà des beans, il est possible d’utiliser les collections au sens large : java.util.List, java.util.Set, java.util.Map, etc…

Exemple avec une **liste** :
```html
<!-- Les quatre syntaxes suivantes retournent le deuxième élément de la liste de légumes  -->
${ legumes.get(1) }<br />
${ legumes[1] }<br />
${ legumes['1'] }<br />
${ legumes["1"] }<br />
```

Le code ci-dessous illustre l’exemple d’un **tableau**. Les trois notations avec les crochets fonctionnent de la même manière avec un tableau qu’avec une liste.  
Il est impossible d’utiliser l’opérateur point pour accéder à un élément d’une liste ou d’un tableau, une exception sera renvoyée.

```html
<!-- Les trois syntaxes suivantes retournent le troisième élément du tableau  -->
${ animaux[2] }<br />
${ animaux['2'] }<br />
${ animaux["2"] }<br />
```

Exemple d'une Map :
```html
<!-- Les quatre syntaxes suivantes retournent la valeur associée à la clé "cookies" de la Map de desserts  -->
${ desserts.cookies }<br />
${ desserts.get("cookies") }<br />
${ desserts['cookies'] }<br />
${ desserts["cookies"] }<br />

<%
/* Création d'une chaîne nommée "element" et contenant le mot "cookies" */
String element = "cookies";
request.setAttribute("element",element);
%>
<!-- Il est également possible d'utiliser un objet au lieu d'initialiser la clé souhaitée directement dans l'expression -->
${ desserts[element] }<br />
```

La notation avec l’opérateur point fonctionne dans une Map. On peut aussi mettre un attribut de requête en guise de clé afin d’accéder à la valeur de la Map. Il ne faut pas entourer de quotes l’objet “élément”, sinon cela signifierait que l’on cherche à accéder à la valeur associée à la clé “élément”, ce qui n’est pas le comportement souhaité.  
Les notations avec l’opérateur point et les crochets fonctionnent tous les deux dans une Map mais il est recommandé de réserver la notation avec l’opérateur point pour les propriétés de bean et que l’on utilise les crochets pour accéder au contenu d’une Map, ce qui donne plus de lisibilité au code.
D’ailleurs pour accéder au propriétés d’une bean, il est possible d’utiliser les crochets :

```html
${bean[“propriété”]}
```

### Désactiver l’évaluation des expressions EL
Le format utilisé par les expressions EL, ${...}, n’était défini dans les premières versions de la technologie JSP. Afin d’assurer la rétrocompatibilité avec le code, il est possible d’empêcher l’interprétation des expressions EL :
- au cas par cas, grâce à la directive page `<%@ page isELIgnored ="true" %>`. Avec true, les EL seront ignorées et apparaîtront en temps que simple chaîne de caractères. Avec false, elles seront interprétées.
- sur tout ou partie des pages grâce à une section à ajouter dans le fichier web.xml, grâce à l’option `<el-ignored>`
```xml
<jsp-config>
    <jsp-property-group>
        <url-pattern>*.jsp</url-pattern>
        <el-ignored>true</el-ignored>
    </jsp-property-group>
</jsp-config>
```

Le champ `<url-pattern>` permet de définir sur quelles pages appliquer ce comportement. `*.jsp` signifie que toutes les pages de l’application seront impactées par cette configuration.

Par défaut, la valeur de l’option isELIgnored va dépendre de la version de l’API servlet utilisée par l’application. Si la version est supérieure ou égale à 2.4, les expressions EL seront évaluées par défaut. Si la version est inférieure à la 2.4, il est possible que les expressions EL soient ignorées par défaut, pour assurer la rétrocompatibilité.  
Pour savoir si une application peut utiliser les expressions EL, il faut s’assurer de la version de l’API servlet supportée par le conteneur qui fait tourner l’application. Par exemple, Tomcat 7 implémente la version 3.0, notre conteneur est donc capable de gérer les expressions EL. Il faut ensuite s’assurer que notre application est déclarée correctement pour utiliser cette version, dans la balise `<web-app>` du fichier web.xml.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
  version="3.0">

    ...

</web-app>
```

La mise en place de ces attributs implique que notre application va se baser sur la version 3.0 de l’API servlet.  
Notre conteneur le supporte et notre application est correctement configurée, les expressions EL seront donc interprétées par défaut.
Si on change de serveur, il est possible que la version supportée par le conteneur soit différente. Si par exemple, notre application tourne sur un Tomcat 6, alors la version maximum de l’API servlet supportée est 2.5. Il faut donc modifier la configuration du fichier web.xml en conséquence.


## Les objets implicites
Il en existe de deux types : ceux mis à disposition via la technologie JSP, et ceux mis à disposition via la technologie EL.

### Les objets de la technologie JSP

```html
<p>
    <%
    String attribut = (String) request.getAttribute("test");
    out.println( attribut );
    %>
</p>
```

Dans cette exemple tirée d’une JSP, nous utilisons l’objet *out* sans l’instancier, de même que l’objet request.
**Dans les coulisses, le conteneur se charge d’instancier ou de récupérer les objets out et request lorsqu’il traduit la JSP en servlet.** Ces objets sont donc dits implicites car on n’a pas besoin de les déclarer de manière explicite.
Le conteneur met à notre disposition toute une série d’objets implicites, tous accessibles directement depuis les pages JSP.


Identifiant | Type de l’objet | Description
--- | --- | ---
pageContext | PageContext | Il fournit des infos utiles relatives au contexte d’exécution. Il permet notamment d’accéder aux attributs présents dans les différentes portées de l’application. Il contient aussi une référence vers tous les objets implicites suivants.
application | ServletContext | Il permet depuis une page JSP d’obtenir ou de modifier des informations relatives à l’application dans laquelle elle est exécutée.
session | HttpSession | Il représente une session associée à un client. Il est utilisé pour lire ou placer des objets dans la session de l’utilisateur courant.
request | HttpServletRequest | Il représente la requête faite par le client. Il est généralement utilisé pour accéder aux paramètres et aux attributs de la requête, ainsi qu’à ses en-têtes.
response | HttpServletResponse | Il représente la réponse qui va être envoyée au client. Il est généralement utilisé pour définir le Content-Type de la réponse, lui ajouter des en-têtes ou encore rediriger le client.
exception | Throwable | Il est uniquement accessible dans les pages d’erreurs JSP. Il représente l’exception qui a conduit à la page d’erreur en question.
out | JspWriter | Il représente le contenu de la réponse qui va être envoyé au client. Il est utilisé pour écrire dans le corps de la réponse.
config | ServletConfig | Il permet depuis une page JSP d’obtenir les éventuels paramètres d’initialisation disponibles.
page | objet this | Il est l’équivalent de la référence this et représente la page JSP courante. Il est déconseillé de l’utiliser, pour des raisons de dégradations de performances notamment.

Ces neufs objets peuvent être utilisés à travers le code Java que nous écrivons dans les JSP. Cependant, comme nous ne voulons plus écrire de code Java dans nos JSP, nous allons voir les objets de la technologie EL.

### Les objets de la technologie EL
La technologie EL permet de profiter des objets implicites sans écrire de code Java.  
Le concept est sensiblement le même que pour les objets implicites JSP : il s’agit d’objets gérés automatiquement par le conteneur lors de l’évaluation des expressions EL, auxquels on peut accéder sans les déclarer.


Catégorie | Identifiant | Description
--- | --- | ---
JSP | pageContext | Objet contenant des informations sur l’environnement du serveur.
Portées | pageScope | Une Map qui associe les noms et valeurs des attributs ayant pour portée la page.
| requestScope | Une Map qui associe les noms et valeurs des attributs ayant pour portée la requête.
| sessionScope | Une Map qui associe les noms et valeurs des attributs ayant pour portée la session.
| applicationScope | Une Map qui associe les noms et valeurs des attributs ayant pour portée l’application.
Paramètres de requête | param | Une Map qui associe les noms et valeurs des paramètres de la requête.
| paramValues | Une Map qui associe les noms et multiples valeurs des paramètres de la requête sous forme de tableaux de String.
En-têtes de requête | header | Une Map qui associe les noms et valeurs des paramètres des en-têtes HTTP.
| headerValues | Une Map qui associe les noms et multiples valeurs des paramètres des en-têtes HTTP sous forme de tableaux de String.
Cookies | cookie | Une Map qui associe les noms et les instances des cookies.
Paramètres d’initialisation | initParam | Une Map qui associe les données contenues dans les champs `<param-name>` et `<param-value>` de la section `<init-param>` du fichier web.xml.

Le seul objet implicite commun entre les JSP et les expressions EL est le pageContext, nous y reviendrons dans le chapitre suivant.
A part pageContext, tous les objets implicites de la technologie EL sont des Map. Les Maps sont des objets qui peuvent se représenter dans un tableau à deux colonnes :
- la première colonne contient les clés qui doivent être uniques
- la seconde contient les valeurs qui peuvent être associées à plusieurs clés
Chaque ligne du tableau ne peut contenir qu’une clé et une valeur.
Les expressions EL permettent d’accéder aux Maps comme nous l’avons vu précédemment.

```java
<%
	String paramLangue = request.getParameter("langue");
	out.println( "Langue : " + paramLangue );
	%>
	<br />
	<%
	String paramArticle = request.getParameter("article");
	out.println( "Article : " + paramArticle );
	%>
```

En rentrant l’URL http://localhost:8080/test/test_obj_impl.jsp?langue=fr&article=782, on va afficher dans le navigateur :
```
Langue : fr
Article : 782
```

En utilisant les expressions EL, on peut arriver au même résultat plus simplement :
```html
Langue : ${ param.langue }
<br />
Article : ${ param.article }
```

S’il y a plusieurs valeurs associées à un même paramètre, il faut utiliser l’objet implicite paramValues. L’objet **param** est une `Map<String,String>`, alors que **paramValues** est une `Map<String,String[]>`.On peut avoir plusieurs valeurs pour un seul paramètre en le précisant plusieurs dans l’URL.

```http
http://localhost:8080/test/test_obj_impl.jsp?langue=fr&article=782&langue=zh
```

Si on continue à utiliser `${param.langue}`, on ne pourra afficher que la première valeur qui est retournée par défaut. Pour afficher la seconde valeur, il faut utiliser l’objet paramValues : `${paramValues.langue[1]}`.  
Si on essaie d’accéder à un élément non défini, par exemple le quatrième élément de langue alors qu’il y en a que deux, l’expression EL détectera une valeur nulle et n’affichera rien.  
On doit obligatoirement cibler un élément du tableau, car si on écrit `${paramValues.langue}`, l’expression EL affichera la référence Java contenant le tableau.

Il existe d’autres cas impliquant plusieurs valeurs pour un même paramètre, par exemple en HTML, un select à choix multiples (liste déroulante). Dans ce cas, l’objet paramValues est nécessaire pour récupérer la liste des valeurs associées au paramètre.

Pour ce qui est de l’objet implicite **headerValues**, son utilité est plus discutable. Les headers sont la plupart du temps séparées par des points-virgules et concaténées dans une seule String, rendant l’emploi de cet objet inutile. La Map header est souvent suffisante.

Les objets implicites EL sont des raccourcis qui rendent l’accès aux différents concepts liés à HTTP extrêmement pratiques !
