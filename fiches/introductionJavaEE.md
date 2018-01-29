# Introduction au Java EE

## Java EE n'est pas Java
Le terme « Java » fait bien évidemment référence à un langage, mais également à une plate-forme : son nom complet est « Java SE » pour Java Standard Edition.  
Le terme « Java EE » signifie Java Enterprise Edition. Il fait quant à lui référence à une extension de la plate-forme standard. Autrement dit, la plate-forme Java EE est construite sur le langage Java et la plate-forme Java SE, et elle y ajoute un grand nombre de bibliothèques remplissant tout un tas de fonctionnalités que la plate-forme standard ne remplit pas d'origine. L'objectif majeur de Java EE est de faciliter le développement d'applications web robustes et distribuées, déployées et exécutées sur un serveur d'applications.

## Consulter un site web
C’est un simple échange entre un client (le navigateur sur l’ordinateur) et un serveur (machine sur laquelle est hébergé le site), via le protocole HTTP :
l’utilisateur saisit une URL dans le navigateur
le navigateur envoie une requête HTTP au serveur pour lui demander la page correspondante
le serveur reçoit cette requête, l’interprète et génère une page web qu’il va renvoyer au client par le biais d’une réponse HTTP
le navigateur reçoit via cette réponse la page web finale qu’il affiche alors à l’utilisateur

Ce qu’il faut retenir :
les données sont échangées entre le client et le serveur via le protocole HTTP
le client ne comprend que les langages de présentation de l’information qui sont le HTML, le CSS et le JavaScript
les pages sont générées sur le serveur de manière dynamique, à partir du code source du site.
