# La JavaBean

Souvent raccourci en bean, le JavaBean est un composant réutilisable, défini dans les spécifications du langage Java, il n’est pas spécifique au Java EE.
Tout objet conforme à ces quelques règles peut être appelé un bean.


## Objectifs
C’est un simple objet Java qui suit certaines contraintes et représente généralement des données du monde réel.
- **les propriétés** : un bean est **paramétrable**, on appelle propriétés les champs non publics présents dans un bean. Les propriétés permettent de paramétrer le bean en y stockant des données
- **la sérialisation** : un bean doit pouvoir être **persistant**. La sérialisation est un processus qui permet de sauvegarder l’état d’un bean et donne la possibilité de le restaurer par la suite. Ce mécanisme permet une persistance des données.
- **la réutilisation** : un bean est **réutilisable**. Il contient que des données ou du code métier, il n’a pas de lien direct avec la couche de présentation, et peut également être distant de la couche d’accès aux données (voir modèle DAO)
- **l’introspection** : un bean est conçu pour être **paramétrable de manière dynamique**. L’introspection est un processus qui permet de connaître le contenu d’un composant (attributs, méthodes, événements (non concerné en Java EE)) de manière dynamique.

Le caractère réutilisable d’un bean permet de minimiser les duplications de logiques dans une application.


## Structure

Un bean :
- doit être une classe publique
- doit avoir au moins un constructeur par défaut, public et sans paramètres
- peut implémenter l’interface Serializable, il devient ainsi persistant et son état peut être sauvegardé
- ne doit pas avoir de champs publics
- peut définir des propriétés (des champs non publics), qui doivent être accessibles via des méthodes publiques getter et setter, suivant des règles de nommage

Exemple d’un bean :
```java
/* Cet objet est une classe publique */
public class MonBean{
	/* Cet objet ne possède aucun constructeur, Java lui assigne donc un constructeur par défaut public et sans paramètre.  */

	/* Les champs de l'objet ne sont pas publics (ce sont donc des propriétés) */
	private String proprieteNumero1;
	private int proprieteNumero2;

	/* Les propriétés de l'objet sont accessibles via des getters et setters publics */
	public String getProprieteNumero1() {
		return this.proprieteNumero1;
	}

	public int getProprieteNumero2() {
		return this.proprieteNumero2;
	}

	public void setProprieteNumero1( String proprieteNumero1 ) {
		this.proprieteNumero1 = proprieteNumero1;
	}

	public void setProprieteNumero2( int proprieteNumero2 ) {
		this.proprieteNumero2 = proprieteNumero2;
	}

	/* Cet objet suit donc bien la structure énoncée : c'est un bean ! */
}
```

## Mise en place

Création d’un bean dans le dossier src, dans un package beans.

**Configuration du projet sous Eclipse**
Pour rendre nos objets accessibles à notre application, les classes compilés à partir de nos fichiers sources doivent être dans un dossier “classes” sous le répertoire /WEB-INF. Voir schéma d’un application web dans la première partie.  
Par défaut, Eclipse envoie automatiquement les classes compilés dans un dossier nommé “build”. Il faut changer cela en modifiant le Build Path de notre application, en cliquant sur Configure Build Path sur le projet puis en changeant le chemin dans l’onglet source :
monAppli/WebContent/WEB-INF/classes
Nous pouvons maintenant manipuler nos beans directement depuis nos servlets et JSP.

**Mise en service dans l’application**
Nous pouvons créer un bean dans la méthode doGet() de la servlet :
```java
/* Création du bean */
Coyote premierBean = new Coyote();
/* Initialisation de ses propriétés */
premierBean.setNom( "Coyote" );
premierBean.setPrenom( "Wile E." );

/* Stockage du message et du bean dans l'objet request */
request.setAttribute( "coyote", premierBean );
```

et l’afficher dans la JSP :

```html
<p>
		 Récupération du bean :
		 <%
com.sdzee.beans.Coyote notreBean = (com.sdzee.beans.Coyote) request.getAttribute("coyote");
out.println( notreBean.getPrenom() );
		 out.println( notreBean.getNom() );
		 %>
</p>
```

Nous allons maintenant apprendre à utiliser un bean depuis une JSP sans utiliser de code Java.
