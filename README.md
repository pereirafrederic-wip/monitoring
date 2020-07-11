# monitoring

Projet de monitoring d'un parc d'applications

## le principe

Les applications sont souvent liées les unes aux autres et lorsqu'une application tombe, il se produit souvent une réaction en chaîne.
Le but est alors de monitorer ces applications et de pouvoir détecter en un instant l'impact de l'incident. 
Un autre but est aussi d'atténuer l'effet cascade et ne pas rester dans le flou lors d'un incident.

## l'application en elle-même

Il s'agit d'une simple application spring boot qui lance à intervalles très réguliers des pings d'une ou plusieurs applications.

### ping de vie de l'application

Le ping de vie est un point d'entrée de toute application qu'on souhaite monitorer. Il permet de définir l'état des applications.
Dans la majorité des cas , il s'agit d'une URL dans le controller rest parent qui renvoie un simple objet et qui définit l'application dans son environnement.

il faut faire les étapes suivantes:
 * créer un point d'entrée de ping , exemple /ping
 * déclarer dans l'application de monitoring l'url : url-application/ping


retour fait à l'application de monitoring  :

```json
{
  "nom": "nomApplication",
  "environnement": "environnement(dev, integ, recette, prod)",
  "version": "versionApplication"
}
```

Sur l'application, une interface permettra d'ajouter une nouvelle url applicative à monitorer. L'URL détermienra à elle seule l'application et l'environnement qu'on souhaite atteindre.

Ce processus permet :
* d'avoir une cartographie applicative à jour.
* de détecter qu'une application est UP, ou DOWN.
* de lister les différents environnements.
* de connaître les versions déployés.
* une timeline de livraison des applications sur les environnements.

## connexion entre applications

Une application communique avec d'autres applications par plusieurs moyens à sa disposition.

### ping de links

Lorsqu'une application est up, cela ne veut pas dire que l'application est opérationnelle. si elle ne peut pas atteindre par exemple référentielle ou transmettre en temps réel à une autre application. cela peut representer une perte de données, un blocage etc...

Lorsque deux applications doivent communiquer entre elle (app1 appelle app2), il convient de faire les étapes suivantes : 

* app1 doit créer son point d'entrée pour l'application de monitoring , par exemple /ping-links.
* app2 doit créer son point d'entrée pour le ping-links de l'app1 , par exemple /app1/ping.
* app1 doit appeler /app1/ping  lorsque l'on appelle /ping-links et determiner si le lien DOWN est considéré comme bloquant/critique.
* app1 doit rajouter dans l'application de monitoring, l'url : url-app1/ping-links.

note: bien sur, il revient aux applications de déterminer les bons test de connexions entre 2 applications (sécurisé, url , etc...).

Retour fait à l'application de monitoring par app1 :
```json
{
  "nom": "nomApplication",
  "environnement": "environnement(dev, integ, recette, prod)",
  "version": "versionApplication",
  "applicationOk" :[
    {
      "nom": "nomApplicationAppele",
     "environnement": "environnement(dev, integ, recette, prod) Appele",
      "version": "versionApplicationAppele",
      "url" : "urlAppele"
    }
  ],
  "applicationEchec" : [
    {
      "url" : "urlAppele",
      "bloquant" :"cela necessite t il de passer en jaune (boolean)"
    }
  ]
}
```
Sur l'application,
une interface permettra d'ajouter cette nouvelle url ping-links à monitorer.  c'est l'appel ensuite qui déterminera tout seul quelles applications sont liées et monitorées. 

Il permet de :
* détecter les links en echecs et d'avoir un état des lieux 
* définir quelles applications sont liées avec une application
* d'avoir les couples applications/environnement liées entre elles (détecter/éviter le cross environnement)
* les versions déployés et les pouvoir détecter les soucis d'assemblage et ainsi liées des versions
* coordonnéer les livraisons de version entre application (pouvoir liéer deux versions pour obliger leur cohérence dans les environnements)


## monitoring à partir de notre application

Cela se base sur trois type de couleurs:
* verte : application/links OK.
* bleu : uniquement pour application avec un processus de livraison en cours.
* jaune : uniquement pour une application instable ( ping appli OK, au moins un ping link Down critique/bloquant).
* orange : application/links coupée et maitrisé.
* rouge : application/links down non maitrisé.

Un filtre peut être ajouté pour voir ou ne pas voir les différents couleurs.

### écran de vie

L'écran de vie permet d'avoir une vue globale de l'état des applications.

La vue par défaut lorsque l'on arrive sur l'écran, est la liste des applications hors service  (ce qui regroupe les applications down ou éteintes ou instables) quel que soit l'environnement . 

Ensuite, il possède plusieurs onglets:
Un onglet represente un environnement, il se base sur la liste des environnements récupérés par les différents pings. 
Cela nous permet d'avoir l'ensemble des applications d'un environement (detection application manquante).


### écran de links

L'écran de links permet d'avoir le modele spaghetti du parc applicatif.

La vue par défaut lorsque l'on arrive sur l'écran, est la liste des links hors services  (qui regroupe les links down ou éteints) quel que soit l'environnement.

Ensuite, il possède plusieurs onglets:
Un onglet represente un environnement, il se base sur la liste des environnements récupérés par les différents pings. 
Cela nous permet d'avoir une vue spaghetti (ou vue mode réseau SNCF) de l'état des liens entre applications d'un environnement.

Deux filtres de la vue permettront de limiter l'affichage du monitoring à l'appelant ou/et l'appelé.



### timeline

Une timeline permet de voir l'historique des applications et des environements au cours du temps et de voir les planifications de livraison.
Cela permet aussi d'avoir les nuances de couleurs et donc des événements .
Cela peut servir de point de travail pour la coordination d'une livraison (cas d'adhérence entre application) et détecter les incohérences de livraisons.

### statistique 

Un dashboard de continuité d'application qui permet par exemple de voir dans un temps imparti :
 * compteur de DOWN pour une/des application(s).
 * des périodicités horaire/semaine/mois de DOWN (exemple: une application avec un traitement lourd impacte entre 14h-15h  chaque vendredi les temps réponses d'un link).

## actions dans l'application

### déclarer une application

Une interface permet d'enregister une application avec une URL du ping de vie. ainsi qu'une liste d'emails de contact pour les alertes.

### déclarer une application qui communique avec d'autres applications

Une interface permet d'ajouter l'url de ping de links d'une application .

### déclarer/ plannifier une livraison

Une interface permet de définir/planifier une livraison d'une application sur un environnement pour une version.
Cela permet de ne pas être rouge, mais d'etre violet sur le monitoring. ignorant donc la partie alerte.

Un mail est lancé aux contacts mais aussi aux contacts des applications qui ont un link avec cette application dans cet environnement.

### déclarer une prise en charge

Lorsqu'une application est DOWN, un mail est envoyé non seulement aux contacts de l'application mais aussi aux contacts des applications qui ont un link avec cette application dans cet environnement.

L'équipe peut venir dire que cela est pris en compte par l'équipe et qu'une analyse est en cours, un autre mail préviendra qu'une action est bien en cours, atténuant l'effet alerte

Cela permet de passer de rouge à orange sur l'écran (prise en charge par l'équipe ou blocage volontaire d'une fonctionnalité).

Le mail d'envoi aux contacts de l'applicaton contient un bouton qui permet directement d'aller dire que c'est bien lu et pris en compte (qui fera l'appel rest qui faut pour valider la prise en compte (bonus email de la personne qui a dit qu'il avait pris en compte=> affichage !).

### déclarer une résolution

Suite à l'analyse ou directement, lorsque l'on sait resoudre le problème, en déclarant une résolution cela veut dire que l'application va revenir à la normale.

Le mail d'envoi aux contacts de l'applicaton contient un bouton qui permet directement d'aller dire que c'est en cours de UP.

Lors du passage ente orange-> vert et rouge -> vert par la sonde du ping, un mail est bien sûr envoyé aux contacts (appli et link) pour prévenir du retour à la normale.

### critère down

Afin de limiter les alertes , il peut être nécessaire de définir un critère de DOWN, en limitant par exemple l'analyse sur une plage horaire (ouverture de l'application seulement entre 8h et 0h par exemple).
Alors on peut ajouter une/des périodes à tester qu'on ajoute à la déclaration de l'application

Définition d'un seuil d'acceptance de nombre de DOWN.
Si on ping toutes les minutes, on peut par exemple faire apsser au ROUGE l'indicateur à partir du troisième ping en échec. On n'alerte ainsi qu'après le seuil d'acceptance de trois minutes (micro coupure) soit dépassé.
On peut aussi décdier de stopper ponctuellement le monitoing pendant une heure par exemple.


# différents façon de collecter

la présentation ci dessus est une présentation d'un mode superviseur où c'est l'application qui va checker les sondes de vies etc....


l'avantage ,c'est que cela demande presque rien à developper, aucune nouvelle dépendance, ni même de besoin de monter en compétence
l'inconvénient , c'est que l'application de monitoring doit pouvoir appeler tous le monde 
+ l'application de monitoring devient une vraie application à part entière



il existe plusieurs autres façons de faire :
 
 
 ## api rest 
 
le ping de l'application peut être remplacé par un poing d'entrée rest dans l'application de monitoring
les applications à intervalle regulier appelle ce service pour signaler qu'il est toujours en vie
 si tu n'as plus de signal , c'est que ton application est down
 
 le ping de links serait pensé dans le sens inverse, si tu n'arrive pas à communiquer avec une application,
 tu appelle le service rest en signalant qui tu es et qui tu n'arrives pas à joindre 
 
 l'avantage ,c'est que cela demande presque rien à developper, aucune nouvelle dépendance, ni même de besoin de monter en compétence
 l'inconvénient , c'est que les applications doivent pouvoir joindre l'application de monitoring
 et ne pas joindre l'application de monotoring ne veut pas dire que l'application est down et tu n'arriveras pas à signaler un souci de links
  
 
 ## file MQ  / kafka
 
 le principe peut etre le même avec des files MQ , au lieu d'appeler un service, on envoi un message dans une file MQ 
 que l'application de monitoring viendra lire et agreger 
 
  l'avantage , c'est que c'est asynchrone
 l'inconvénient ,c'est que les applications doivent pouvoir joindre la file de message
 et ne pas joindre la file de message ne veut pas dire que l'application est down et tu n'arriveras pas à signaler un souci de links
 + il faut ajouter cette fonctionnalité à toutes les applications et savoir le mettre en place
 
## suite kibana/elastic search   idatha

le principe est logger sur un modele simple et léger les 2 informations. 
ainsi en utilisant le bon filtre on peut avoir le monitoring et suivre en temps réelle 

 l'avantage , pas d'application et la puissance de ses services. les nombreux graphiques et possibilités.
 l'inconvénient , faut faire le necessaire pour logger sur ces plateformes. beaucoup de logs et aucune intelligence que pourrait avoir notre application de monitoring
 
