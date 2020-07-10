# monitoring
Projet de monitoring d'un parc d'applications

## le principe

Les applications sont souvent liées les unes aux autres et lorsqu'une application tombe, il se produit souvent une réaction en chaîne.
Le but est alors de monitorer ces applications et de pouvoir détecter en un instant l'étendue de la panne. 
Un autre but est aussi d'atténuer l'effet cascade et ne pas rester dans le flou lors d'un incident.


## l'application en elle-même

Il s'agit d'une simple application spring boot qui lance à intervalles très réguliers des pings d'une ou plusieurs applications.

### ping de vie de l'application

Le ping de vie est un point d'entrée de toute application qu'on souhaite monitorer. Il permet de définir les applications qui sont UP ou DOWN

Dans la majorité des cas , il s'agit d'une url dans le controller rest parent qui renvoie un simple objet qui définit l'application dans son environnement.


Typiquement l'objet suivant reflète la clé du ping :
{
  "nom": 'nomApplication',
 "environnement": 'environnement(dev, integ, recette, prod)'
  "version": 'versionApplication'
}

Sur l'application, une interface permettra une nouvelle url à monitorer. il s'agit que de cela car c'est l'appel ensuite qui déterminera tout seul pour quelle application et environnement il s'agit.

il permet de :
* détecter qu'une application tombe ou est en cours de livraison
* définir quel application est placé sur l'url
* d'avoir les différents environnement
* les versions déployés
* une timeline de livraison des applications sur les environnements

## connexion entre applications

une application communique avec d'autres applications par plusieurs moyens à sa disposition

### ping de connexion

lorsqu'une application est up, cela ne veut pas dire que l'application est opérationnelle. si elle ne peut pas atteindre une référentielle, transmettre en temps réel l'application. il peut y avoir une perte de transfert de données.

le ping de connexion est le premier point d'entée entre deux applications. lorsque l'on établit un lien entre application, il est important de commencer par le test de connexion à travers un ping. le ping de connexion permet de faire un premier pas simple pour tester la configuration pour que l'appel se passe bien. et ensuite, il permet de créer le futur ping de connexion

encore une fois, il s'agit de rajouter dans le controller rest parent une url qui renvoit un liste des connexions entres applications et envirronnement.
l'application fera une boucle d'avoir des différents pings pour chaque links qu'elle possède

voici l'objet qui reflète le ping des connexions d'une application:
{
  "nom": 'nomApplication',
  "environnement": 'environnement(dev, integ, recette, prod)'
  "version": 'versionApplication',
  "applicationOk" :[
    {
      "nom: 'nomApplicationAppele',
     "environnement: 'environnement(dev, integ, recette, prod) Appele'
      "version: 'versionApplicationAppele',
      "url : 'urlAppele'
    }
  ],
  "applicationEchec" : [
    {
      "nom": 'nomApplicationAppele',
      "url" : 'urlAppele'
    }
  ]
}

sur l'application,
une interface permettra d'ajouter cette nouvelle url à monitorer. il s'agit que de cela car c'est l'appel ensuite qui déterminera tout seul quelles applications sont liés et monitorés 

il permet de :
* détecter les liens en echecs et d'avoir un état des lieux 
* définir quelles applications sont liées avec une application
* d'avoir les couples applications/environnement liées entre elles
* les versions déployés et les pouvoir détecter les soucis d'assemblage et ainsi liées des versions
* coordonnéer les livraisons de version entre application (pouvoir liéer deux versions pour obliger leur cohérence dans les environnements)


## monitoring à partir de notre application

cela se base sur trois type de couleurs:
* verte : application OK
* orange : application coupée et maitrisé
* violet : processus de livraison en cours
* rouge : application down non maitrisé
un filtre peut être ajouter pour voir ou ne pas voir les différents couleurs

### écran de vie

l'écran de vie permet d'avoir une vue globale de l'état des applications.

sa vue par défaut lorsque l'on arrive sur l'écran, c'est la liste des applications hors services  (qui regroupe les applications down ou éteintes) quelque soit l'environnement . 

ensuite, il possède plusieurs onglets.
un onglet represente un environnement, il se base sur la liste des environnements récupérer par les différents pings. 
cela nous permet d'avoir l'ensemble des applications d'un envirronement (detection application manquante)


### écran de links

l'écran de links permet d'avoir le modele spaghetti du parc applicatif

sa vue par défaut lorsque l'on arrive sur l'écran, c'est la liste des links hors services  (qui regroupe les applications down ou éteintes) quelque soit l'environnement.

ensuite, il possède plusieurs onglets.
un onglet represente un environnement, il se base sur la liste des environnements récupérer par les différents pings. 
cela nous permet d'avoir une vue spaghetti (ou vu mode reseau scnf) de l'état des liens entre applications

deux filtres de la vue permettront de limiter le reseaux à l'appelant ou/et l'appelé.



### timeline

une timeline permet de voir l'historique des applications et des environements au cours du temps et de voir les plannifications de livraison
cela permet aussi d'avoir les nuances de couleurs et donc des évenements 
cela peut servir de point de travail pour la coordination d'une livraison (cas d'adhérence entre application) et détecter les incohérences de livraisons

### statistique 

un dashboard de continuité d'application qui permet par exemple de voir dans un temps imparti :
 * compteur de down pour une/des application(s)
 * des périodicitées horaire/semaine/mois de down (exemple: une application avec un traitement lourd impacte entre 14h-15h  chaque vendredi les temps réponses d'un link)

## actions dans l'application

### déclarer une application

une interface permet d'enregister une application avec un url du ping de vie. ainsi qu'une liste d'email de contact pour les alertes

### déclarer une application qui communique avec d'autres applications

une interface permet d'avoir un url de ping de connexion

### déclarer/ plannifier une livraison

une interface permet de définir/plannifier une livraison d'une application sur un environnement pour une version
ainsi cela permet de ne pas être rouge, mais d'etre violet sur le monitoring. ignorant donc la partie alerte.

un mail est lancé aux contacts mais aussi aux contacts des applications qui ont un link avec cette application dans cette environnement

### déclarer une prise en charge

lorsqu'une application est down, un mail est envoyés non seulement aux contacts de l'application mais aussi aux contacts des applications qui ont un link avec cette application dans cette environnement.

l'équipe peut venir dire que cela est pris en compte par l'équipe et qu'une analyse est en cours, un autre mail préviendra qu'une action est bien en cours, atténuant l'effet alerte

cela permet de passer de rouge à orange sur l'écran.

le mail d'envoi aux contacts de l'applicaton contient un bouton qui permet directement d'aller dire que c'est bien lu et pris en compte (qui fera l'appel rest qui faut pour valider la prise en compte (bonus email de la personne qui a dit qu'il avait pris en compte=> affichage !)

### déclarer une résolution

suite à l'analyse ou directement, lorsque l'on sait resoudre le problème, en déclarant une résolution cela veut dire que l'application va revenir à la normale.

le mail d'envoi aux contacts de l'applicaton contient un bouton qui permet directement d'aller dire que c'est en cours de UP

lors du passage ente orange-> vert et rouge -> vert par la sonde du ping, un mail est bien sur envoyés aux contacts (appli et link) pour prévenir du retour à la normale

### critère down

afin de limiter les alertes , il peut être nécessaire de définir un critère de down, est ce que l'application est up 24/7
alors on peut ajouter une/des periodes periode à tester qu'on ajoute à la déclaration de l'application

une application est down par un seul appel ping, est ce que l'on considère que c'est une interruption ? un compteur de down peut être ajouté, ainsi on définit un seuil d'acceptance de down. si on ping toutes les 1 minutes , on peut autorisé le fait d'etre down 3* 1 min. ains on ne ne lance les alertes qu'après 3 min de down.




