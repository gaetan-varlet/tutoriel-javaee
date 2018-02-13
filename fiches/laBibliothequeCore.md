# La bibliothèque Core

## Les variables et expressions
Avant cela, il faut copier la directive JSP nécessaire à l’utilisation de la bibliothèque Core, qui doit être présente sur chacune des pages du projet utilisant les balises JSTL. Nous verrons dans un prochain chapitre comment  il est possible de ne plus se soucier de cette commande.

```html
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

### Affichage d’une expression
La balise est `<c:out value=”” />`. Le seul attribut obligatoire est value. Il peut contenir une chaîne de caractères simple ou une expression EL.
```html
<c:out value="test" /> <%-- Affiche test --%>
<c:out value="${ 'a' < 'b' }" /> <%-- Affiche true --%>
```

Il y a deux attributs optionnels :
- **default** : permet d’afficher une valeur par défaut si le contenu de l’expression est vide
- **escapeXml** : permet de remplacer les caractères de scripts <,>,”,’ et & par leur équivalents en html. Option activée par défaut.

Pourquoi utiliser une balise pour afficher simplement du texte ? Car l’attribut escapeXml permet d’échapper automatiquement les caractères spéciaux de nos textes (voir l’avertissement concernant les failles XSS).

Quelques exemples de l’attribut **default** :

```html
<%-- Cette balise affichera le mot 'test' si le bean n'existe pas : --%>
<c:out value="${bean}">
    test
</c:out>

<%-- Elle peut également s'écrire sous cette forme : --%>
<c:out value="${bean}" default="test" />

<%-- Et il est interdit d'écrire : --%>
<c:out value="${bean}" default="test">
    une autre chaine
</c:out>
```

Un exemple avec l'attribut **escapeXml** :
```html
<%-- Sans préciser d'attribut escapeXml : --%>
<c:out value="<p>Je suis un 'paragraphe'.</p>" />

<%-- La balise affichera : --%>
&lt;p&gt;Je suis un &#039;paragraphe&#039;.&lt;/p&gt;

<%-- Et en précisant l'attribut à false :--%>
<c:out value="<p>Je suis un 'paragraphe'.</p>" escapeXml="false" />

<%-- La balise affichera : --%>
<p>Je suis un 'paragraphe'.</p>
```

L’activation par défaut de l’option escapeXml empêche l’interprétation de ce qui est affiché par le navigateur en modifiant les éléments de code HTML présents dans le contenu traitée. Il faut prendre l’habitude d’utiliser ce tag JSTL pour afficher des variables notamment quand elles sont saisies par l’utilisateur.

```html
<%-- Mauvais exemple --%>
<input type="text" name="donnee" value="${donneeSaisieParUnUtilisateur}" />

<%-- Bon exemple --%>
<input type="text" name="donnee" value="<c:out value="${donneeSaisieParUnUtilisateur}"/>" />
```

Les données récupérées depuis un formulaire sont potentiellement dangereuses puisqu’elles permettent des attaques de type XSS ou d’injection de code. L’utilisation du tag `<c:out>` permet d’échapper les caractères spéciaux responsables de cette faille, et ainsi prévenir tout risque à ce niveau. Nous verrons cela en détail dans le chapitre sur les formulaires.

### Gestion d’une variable

Avant de parler variable, parlons **portée**. Il existe quatre portées :
- **la page** : les objets créés avec la portée page ne sont accessibles que depuis cette page, et une fois la réponse retournée au navigateur, ces données ne sont plus accessibles
- **la requête** : ces objets ne sont accessibles que depuis les pages qui traitent cette même requête. Si la requête est transmise à une autre page, les données sont conservées, mais sont perdues en cas de redirection
- **la session**
- **l’application** : c’est comme une variable globale accessible partout

La balise utilisée pour la création d’une variable est `<c:set>`, avec les attributs **var** (nom de la variable), **value** (valeur) et **scope** (portée de la variable).

```html
<%-- Cette balise met l'expression "Salut les zéros !"
dans l'attribut "message" de la requête : --%>
<c:set var="message" value="Salut les zéros !" scope="request" />

<%-- Et est l'équivalent du scriplet Java suivant : --%>
<% request.setAttribute( "message", "Salut les zéros !" ); %>
```

Pour afficher le contenu de la variable :
```html
<%-- Affiche l'expression contenue dans la variable "message" de la requête --%>
<c:out value="${requestScope.message}" />
```

La modification d’une variable s’effectue de la même manière que sa création. Le code suivant créera une variable “maVariable” si elle n’existe pas déjà et initialisera son contenu à 12.

```html
<%-- L'attribut scope n'est pas obligatoire.
Rappelez-vous, le scope par défaut est dans ce cas la page,
puisque c'est le premier dans la liste des scopes parcourus --%>
<c:set var="maVariable" value="12" />
```

On peut aussi initialiser une variable en utilisant le corps de la balise plutôt que l’attribut value.
```html
<c:set var="maVariable"> 12 </c:set>
```

Il est possible d’imbriquer d’autres balises dans le corps de cette balise. Par exemple pour initialiser la valeur d’une variable depuis une valeur lue dans un paramètre de l’URL.
```html
<c:set var="locale" scope="session">
   <c:out value="${param.lang}" default="FR"/>
</c:set>
```

Modifier les propriétés d’un objets
On sait pour l’instant définir une variable de n’importe quel type, qui défini par l’expression que l’on écrit dans l’attribut value du tag <c:set>.
```html
<%-- Crée un objet de type String --%>
<c:set scope="session" var="description" value="Je suis une loutre." />

<%-- Crée un objet du type du bean ici spécifié dans l'attribut 'value'--%>
<c:set scope="session" var="tonBean" value="${monBean}" />
```

Pour modifier les propriétés du bean, il nous manque deux attributs, **target** (qui contient le nom de l’objet dont la propriété sera modifiée) et **property** (qui contient le nom de la propriété qui sera modifiée).
```html
<!-- Définir ou modifier la propriété 'prenom' du bean 'coyote' -->
<c:set target="${coyote}" property="prenom" value="Wile E."/>

<!-- Définir ou modifier la propriété 'prenom' du bean 'coyote'
via le corps de la balise -->
<c:set target="${coyote}" property="prenom">
   Wile E.
</c:set>

<!-- Passer à null la valeur de la propriété 'prenom' du bean 'coyote' -->
<c:set target="${coyote}" property="prenom" value="${null}" />
```

Mettre un null dans l’attribut value pour un bean fait passer la valeur de la propriété à null. Dans une map, cette action supprime l’entrée de la map, ce qui revient au même que la balise suivante sur la suppression.

Pour supprimer une variable, il y a la balise `<c:remove>`, avec un seul attribut obligatoire var. Par défaut, c’est le scope page qui sera parcouru si l’attribut scope n’est pas explicité.
```html
<%-- Supprime la variable "maVariable" de la session --%>
<c:remove var="maVariable" scope="session" />
```


## Les conditions

Les conditions simples, correspondent au bloc if() du langage Java. Le seul attribut obligatoire est **test**.
```html
<c:if test="${ 12 > 7 }" var="maVariable" scope="session">
    Ce test est vrai.
</c:if>
```

ici, le corps de la balise est une simple chaîne de caractères qui ne sera affichée dans la page finale que si la condition est vraie, si l’expression contenue dans l'attribut test renvoie true.
**var** et **scope** ont le même rôle que dans la balise `<c:set>`, le résultat du test sera stocké dans la variable et le scope défini. L’intérêt réside dans le stockage de tests coûteux pour accéder simplement à des variables de scope au lieu de refaire le test.

Les conditions multiples correspondent en Java au if()/else if() ou au bloc switch().
```html
<c:choose>
    <c:when test="${expression}">Action ou texte.</c:when>
    ...
    <c:otherwise>Autre action ou texte.</c:otherwise>
</c:choose>
```

La balise `<c:choose>` ne peut contenir aucun attribut et son corps ne peut contenir qu’une ou plusieurs balises `<c:when>` et une ou zéro balise `<c:otherwise>`.  
La balise `<c:when>` ne peut exister qu’à l’intérieur d’une balise `<c:choose>`, équivalent du mot-clé case en Java dans un bloc switch(). Comme `<c:if>`, elle doit se voir définir un attribut test contenant la condition. A l’intérieur d’un `<c:choose>`, un seul `<c:when>` verra son corps évalué, les conditions étant mutuellement exclusives.  
La balise `<c:otherwise>` ne peut également exister qu’à l’intérieur d’une balise `<c:choose>` et après la dernière balise `<c:when>`. Elle est l’équivalent du mot clé default en Java dans un bloc switch(). Elle ne peut contenir aucun attribut et son corps ne sera évalué que si aucune des conditions la précèdent dans le bloc n’est vérifié.


## Les boucles

Il y a deux types de boucles `<c:forEach>` pour parcourir une collection et `<c:forTokens>` pour parcourir une chaîne de caractères.

Exemple de boucle “classique” de boucle for en Java
```html
<%-- Boucle calculant le cube des entiers de 0 à 7 et les affichant
dans un tableau HTML --%>
<table>
  <tr>
    <th>Valeur</th>
    <th>Cube</th>
  </tr>

<%
int[] cube= new int[8];
/* Boucle calculant et affichant le cube des entiers de 0 à 7 */
for(int i = 0 ; i < 8 ; i++)
{
  cube[i] = i * i * i;
  out.println("<tr><td>" + i  + "</td> <td>" + cube[i] + "</td></tr>");
}
%>

</table>
```

La même chose avec la JSTL :
```html
<%-- Boucle calculant le cube des entiers de 0 à 7
et les affichant dans un tableau HTML --%>
<table>
  <tr>
    <th>Valeur</th>
    <th>Cube</th>
  </tr>

<c:forEach var="i" begin="0" end="7" step="1">
  <tr>
    <td><c:out value="${i}"/></td>
    <td><c:out value="${i * i * i}"/></td>
  </tr>
</c:forEach>

</table>
```

Le second code est beaucoup plus clair, les balises JSTL s’intègrent bien au formatage HTML, ce qui rend la compréhension du code beaucoup plus simple à comprendre.
Les attributs :
- begin : valeur de début de notre compteur (valeur de “i” dans la boucle Java)
- end : valeur de fin de notre compteur. La valeur dans end est parcouru, cela correspond donc à une comparaison de type <=
- step : pas d’incrémentation, qui vaut 1 par défaut si on ne le spécifie pas
- var : attribut non obligatoire, mais la valeur du compteur ne sera pas accessible comme c’est le cas ici

Voici le rendu HTML :

| Valeur | Cube  |
| :---:  | :---: |
| 0      | 0     |
| 1      | 1     |
| 2      | 8     |
| 3      | 27    |
| 4      | 64    |
| 5      | 125   |
| 6      | 216   |
| 7      | 343   |

**Itération sur une collection**  
Les itérations sur les collections sont utilisées dans la création de pages web. La syntaxe utilisée pour parcourir une collection est similaire à celle d’une boucle simple sauf que l’attribut **items** est requis, il indique la collection à parcourir. Voici un exemple ou l’on souhaite afficher des news sur une page web.

```html
<%@ page import="java.util.*" %>
<%
  /* Création de la liste et des données */
  List<Map<String, String>> maListe = new ArrayList<Map<String, String>>();
  Map<String, String> news = new HashMap<String, String>();
  news.put("titre", "Titre de ma première news");
  news.put("contenu", "corps de ma première news");
  maListe.add(news);
  news = new HashMap<String, String>();
  news.put("titre", "Titre de ma seconde news");
  news.put("contenu", "corps de ma seconde news");
  maListe.add(news);
  pageContext.setAttribute("maListe", maListe);
%>

<c:forEach items="${maListe}" var="news">
<div class="news">
  <div class="titreNews">
      <c:out value="${news['titre']}" />
  </div>
  <div class="corpsNews">
      <c:out value="${news['contenu']}" />
  </div>
</div>
</c:forEach>
```

L’attribut **var** indique le nom de la variable qui sera liée à l’élément courant de la collection parcourue par la boucle.
Cela donne le code HTML suivant :
```html
<div class="news">
  <div class="titreNews">
      Titre de ma première news
  </div>
  <div class="corpsNews">
      corps de ma première news
  </div>
</div>

<div class="news">
  <div class="titreNews">
      Titre de ma seconde news
  </div>
  <div class="corpsNews">
      corps de ma seconde news
  </div>
</div>
```

Il est possible d’utiliser les attributs d’une boucle simple comme dans l’exemple suivant pour n’afficher que les 10 premières news :
```html
<c:forEach items="${maListe}" var="news" begin="0" end="9">
   ...
</c:forEach>
```

Si les attributs begin et end dépassent le contenu réel de la collection, la boucle s’arrêtera automatiquement lorsque le parcours de la liste sera terminé.

Il est possible d’itérer directement sur le résultat d’une requête SQL mais nous ne le verrons pas ce n’est pas souhaitable de faire des requêtes SQL depuis une JSP.

**L’attribut varStatus**  
Il est utilisable sur tout type d’itérations. Il crée un variable de scope qui permet de stocker un objet LoopTagStatus qui définit un ensemble de propriétés définissant l’état courant d’une itération :


| Propriété | Description |
| :---:  | :---: |
| begin | la valeur de l’attribut begin
| end | la valeur de l’attribut end
|step | la valeur de l’attribut step
| first | booléen précisant si l’itération courante est la première
| last | booléen précisant si l’itération courante est la dernière
| count | compteur d’itérations (commence à 1)
| index | index d’itérations (commence à 0)
| current | élément courant de la collection parcourue

Par exemple, dans l’exemple précédent avec l’attribut varStatus :
```html
<c:forEach items="${maListe}" var="news" varStatus="status">
<div class="news">
  News n° <c:out value="${status.count}"/> :
  <div class="titreNews">
      <c:out value="${news['titre']}" />
  </div>
  <div class="corpsNews">
      <c:out value="${news['contenu']}" />
  </div>
</div>
</c:forEach>
```

L’objet créé par cet attribut varStatus n’est visible que dans le corps de la boucle, tout comme l’attribut var.

**Itération sur une chaîne de caractères**  
Une variante de `<c:forEach>` existe spécialement dédiée aux chaînes de caractères : `<c:forTokens>`. La syntaxe est très proche, un seul attribut apparaît : **delims**, où l’on spécifie les uns à la suite des autres les caractères qui serviront de séparateurs dans la chaîne que la boucle parcourra. Tous les autres attributs vus précédemment peuvent également s’appliquer.

```html
<p>
<%-- Affiche les différentes sous-chaînes séparées par une virgule
ou un point-virgule --%>
<c:forTokens var="sousChaine" items="salut; je suis un,gros;zéro+!" delims=";,+">
  ${sousChaine}<br/>
</c:forTokens>
</p>
```

Le rendu HTML :
```html
<p>
  salut<br/>
   je suis un<br/>
  gros<br/>
  zero<br/>
  !<br/>
</p>
```

**Ce que la JSTL ne permet pas (encore) de faire**  
En Java, on peut sortir d’une boucle avec **break** ou **continue**. Cela n’est pas possible dans la JSTL. On peut contourner le problème en utilisant des conditions <c:if> de manière plus ou moins efficace. On peut aussi dans le cas de collections déporter le travail de la boucle dans une classe Java. Il ne faut pas trop en demander à la vue pour ne pas bafouer MVC.


## Les liens

La balise `<c:url>` a pour objectif de générer des URL. On peut se demander à quoi ça sert, car en HTML, pour créer un lien, une simple balise `<a>` suffit.
```html
<a href="url">lien</a>
```

Pourquoi utiliser la balise `<c:url>` ?
Car une adresse n’est pas qu’une simple chaîne de caractères, elle est soumise à plusieurs contraintes. Voici les trois fonctionnalités associées à la balise que l’on va détailler :
1. ajouter le nom de contexte aux URL absolues
2. réécrire l’adresse pour la gestion des sessions (si les cookies sont désactivés ou absents, par exemple)
3. encoder les noms et contenus des paramètres de l’URL

L’attribut value contient l’adresse, et l’attribut var permet de stocker le résultat dans une variable. Exemple :
```html
<%-- Génère une url simple, positionnée dans un lien HTML --%>
<a href="<c:url value="test.jsp" />">lien</a>

<%-- Génère une url et la stocke dans la variable lien --%>
<c:url value="test.jsp" var="lien" />
```

### 1. L’ajout du contexte
Lorsque l’URL est absolue (fait référence à la racine de l’application et commence par /), le contexte de l’application sera par défaut ajouté en début d’adresse.
Avec les adresses relatives, pas de soucis car elles ne font pas référence au contexte, et pointent vers le répertoire courant. Mais pour les adresses absolues, il serait nécessaire d’écrire en dur le contexte de l’application sans cette fonctionnalité et tout tout changer ensuite lors du passage du développement en production.

```html
<%-- L'url absolue ainsi générée --%>
<c:url value="/test.jsp" />

<%-- Sera rendue ainsi dans la page web finale si le contextPath est "Test" --%>
/Test/test.jsp

<%-- Et une url relative ainsi générée --%>
<c:url value="test.jsp" />

<%-- Ne sera pas modifiée lors du rendu --%>
test.jsp
```

### 2. Gestion des sessions
Si le conteneur JSP détecte un cookie stockant l’identifiant de session dans le navigateur, alors aucune modification ne sera apportée à l’URL. Si le cookie est absent, les URL générées via `<c:url>` seront réécrites pour intégrer l’identifiant de session en question.

```html
<%-- L'url ainsi générée --%>
<c:url value="test.jsp" />

<%-- Sera rendue ainsi dans la page web finale,
si le cookie est présent --%>
test.jsp

<%-- Et sera rendue sous cette forme si le cookie est absent --%>
test.jsp;jsessionid=A6B57CE08012FB431D
```

### 3. Encodage
Avec la balise `<c:url>`, les paramètres que l’on souhaite passer à cette URL seront encodés (caractères spéciaux transformés en leurs codes HTML). Seulement les paramètres (noms et contenu) seront encodés, le reste de l’URL ne sera pas modifié. Le but est d’assurer la compatibilité avec l’action standard d’inclusion `<jsp:include>` qui ne sait pas gérer une URL encodée.
Pour transmettre proprement des paramètres à une URL, une balise existe `<c:param>`, qui ne peut exister que dans le corps des balises `<c:url>`, `<c:import>` ou `<c:redirect>`.

```html
<c:url value="/monSiteWeb/countZeros.jsp">
  <c:param name="nbZeros" value="${countZerosBean.nbZeros}"/>
  <c:param name="date" value="22/06/2010"/>
</c:url>
```

L’attribut name contient le nom du paramètre et value son contenue. C’est cette balise qui se charge de l’encodage

```html
<%-- Une URL générée de cette manière --%>
<a href="<c:url value="/monSiteWeb/test.jsp">
  <c:param name="date" value="22/06/2010"/>
  <c:param name="donnees" value="des données contenant des c@r#ct%res bi&a**es..."/>
</c:url>">Lien HTML</a>

<%-- Sera rendue ainsi dans la page web finale --%>
<a href="/test/monSiteWeb/test.jsp?date=22%2f06%2f2010&donnees=des+donn%e9es+contenant+des+c%40r%23ct%25res+bi%26a**es...">Lien HTML</a>
```

Les caractères & séparant les différents paramètres n’ont pas été modifiés en leur code HTML &amp, car ils font partie intégrante de l’URL.

**Pour résumer**  

```html
<%-- L'url avec paramètres ainsi générée --%>
<c:url value="/monSiteWeb/countZeros.jsp">
  <c:param name="nbZeros" value="123"/>
  <c:param name="date" value="22/06/2010"/>
</c:url>

<%-- Sera rendue ainsi dans la page web finale,
si le cookie est présent et le contexte est Test --%>
/Test/monSiteWeb/countZeros.jsp?nbZeros=123&date=22%2f06%2f2010

<%-- Et sera rendue sous cette forme si le cookie est absent --%>
/Test/monSiteWeb/countZeros.jsp;jsessionid=A6B57CE08012FB431D?nbZeros=123&date=22%2f06%2f2010
```

On observe :
- la mise en place de paramètres via `<c:param>`
- l’ajout automatique du contexte en début d’URL absolue
- l’encodage automatique des paramètres
- le non-encodage de l’URL
- l’ajout automatique de l’identifiant de session

### Redirection

La balise `<c:redirect>` est utilisée pour envoyer un message de redirection HTTP au navigateur de l’utilisateur. Elle ressemble à l’action `<jsp:forward>` mais il y a une grosse différence : elle entraîne un changement d’URL dans le navigateur de l’utilisateur contrairement à `<jsp:forward>`, qui se fait côté serveur, alors que la redirection est effectué par le navigateur. Le forwarding est donc limité aux pages présentes dans le contexte de la servlet utilisée alors que la redirection, exécutée côté client, peut rediriger vers n’importe quelle page web.
Le forwarding est plus performant (pas d’aller-retour passant par le navigateur de l’utilisateur) mais il est moins flexible que la redirection. Le forwarding  impose certaines contraintes car l’utilisateur n’est pas au courant que sa requête a été redirigée vers une ou plusieurs servlets ou JSP différentes puisque l’URL qui est affichée dans son navigateur ne change pas. Ceci peut poser des problèmes si l’utilisateur rafraîchit sa page puisque cela appellera à nouveau la servlet d’origine. Avec le forwarding, le code présent après l’instruction n’est pas exécuté.
Il est plus simple pour débuter d’utiliser la redirection via la balise `<c:redirect>`.

```html
<%-- Forwarding avec l'action standard JSP --%>
<jsp:forward page="/monSiteWeb/erreur.jsp">

<%-- Redirection avec la balise redirect --%>
<c:redirect url="http://www.siteduzero.com"/>


<%-- Les attributs valables pour <c:url/> le sont aussi pour la redirection.
Ici par exemple, l'utilisation de paramètres --%>
<c:redirect url="http://www.siteduzero.com">
  <c:param name="mascotte" value="zozor"/>
  <c:param name="langue" value="fr"/>
</c:redirect>

<%-- Redirigera vers --%>
http://www.siteduzero.com?mascotte=zozor&langue=fr
```

### Imports
La balise `<c:import>` est en quelque sorte un équivalent à `<jsp:include>` qui propose plus d’options et pallie ainsi les manques de l’inclusion standard.
L’attribut url est le seul obligatoire, désignant le fichier à importer.

```html
<%-- Copie le contenu du fichier ciblé dans la page actuelle --%>
<c:import url="exemple.html"/>
```

La fonction d’import permet d’utiliser une variable pour stocker le flux récupéré, via l’attribut **varReader***, ce qui permet d’effectuer des traitements sur les pages importées. Nous verrons cela en utilisant la bibliothèque xml de la JSTL, notamment pour lire et parser des fichiers XML.

```html
<%-- Copie le contenu d'un fichier xml dans une variable (fileReader),
puis parse le flux récupéré dans une autre variable (doc). --%>
<c:import url="test.xml" varReader="fileReader">
  <x:parse var="doc" doc="${fileReader}" />
</c:import>
```

Comme pour la redirection, <c:import> permet d’inclure des pages extérieures au contexte de notre servlet ou à notre serveur, contrairement à l’action standard JSP. Quelques exemples :

```html
<%-- Inclusion d'une page avec l'action standard JSP. --%>
<jsp:include page="index.html" />

<%-- Importer une page distante dans une variable
Le scope par défaut est ici page si non précisé. --%>
<c:import url="http://www.siteduzero.com/zozor/biographie.html" var="bio" scope="page"/>

<%-- Les attributs valables pour <c:url/> le sont aussi pour la redirection.
Ici par exemple, l'utilisation de paramètres --%>
<c:import url="footer.jsp">
  <c:param name="design" value="bleu"/>
</c:import>
```

### Les autres bibliothèques de la JSTL
Il existe cinq bibliothèques en JSTL.
- fmt : destinée au formatage et au parsage des données. Permet notamment la localisation de l’affichage
- fn : destinée au traitement de chaînes de caractères
- sql : destinée à l’interaction avec une base de données. A utiliser plutôt en test, car le code ayant trait au stockage des données doit être dans le modèle ou dans une couche dédiée, pour suivre MVC à la lettre.
- xml : destinée au traitement de fichiers et données XML. Trouve difficilement sa place dans une application MVC, ces traitements ayant souvent leur place dans des objets du modèle et pas dans la vue.
