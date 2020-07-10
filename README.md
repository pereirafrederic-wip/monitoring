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
* jaune : uniquement pour une application instable ( ping appli OK, au moins un ping link Down ).
* orange : application/links coupée et maitrisé.
* violet : uniquement pour application avec un processus de livraison en cours.
* rouge : application/links down non maitrisé.
un filtre peut être ajouter pour voir ou ne pas voir les différents couleurs.

### écran de vie

L'écran de vie permet d'avoir une vue globale de l'état des applications.

Sa vue par défaut lorsque l'on arrive sur l'écran, c'est la liste des applications hors services  (qui regroupe les applications down ou éteintes ou instable) quelque soit l'environnement . 


Ensuite, il possède plusieurs onglets:
Un onglet represente un environnement, il se base sur la liste des environnements récupérer par les différents pings. 
Cela nous permet d'avoir l'ensemble des applications d'un envirronement (detection application manquante).


### écran de links

L'écran de links permet d'avoir le modele spaghetti du parc applicatif.

Sa vue par défaut lorsque l'on arrive sur l'écran, c'est la liste des links hors services  (qui regroupe les links down ou éteintes) quelque soit l'environnement.

Ensuite, il possède plusieurs onglets:
Un onglet represente un environnement, il se base sur la liste des environnements récupérer par les différents pings. 
Cela nous permet d'avoir une vue spaghetti (ou vu mode reseau scnf) de l'état des liens entre applications d'un envirronement.

Deux filtres de la vue permettront de limiter le reseaux à l'appelant ou/et l'appelé.



### timeline

Une timeline permet de voir l'historique des applications et des environements au cours du temps et de voir les plannifications de livraison.
Cela permet aussi d'avoir les nuances de couleurs et donc des évenements .
Cela peut servir de point de travail pour la coordination d'une livraison (cas d'adhérence entre application) et détecter les incohérences de livraisons.

### statistique 

Un dashboard de continuité d'application qui permet par exemple de voir dans un temps imparti :
 * compteur de down pour une/des application(s).
 * des périodicitées horaire/semaine/mois de down (exemple: une application avec un traitement lourd impacte entre 14h-15h  chaque vendredi les temps réponses d'un link).

## actions dans l'application

### déclarer une application

Une interface permet d'enregister une application avec un url du ping de vie. ainsi qu'une liste d'email de contact pour les alertes.

### déclarer une application qui communique avec d'autres applications

Une interface permet d'ajouter l'url de ping de links d'une application .

### déclarer/ plannifier une livraison

Une interface permet de définir/plannifier une livraison d'une application sur un environnement pour une version.
ainsi cela permet de ne pas être rouge, mais d'etre violet sur le monitoring. ignorant donc la partie alerte.

un mail est lancé aux contacts mais aussi aux contacts des applications qui ont un link avec cette application dans cet environnement.

### déclarer une prise en charge

Lorsqu'une application est down, un mail est envoyés non seulement aux contacts de l'application mais aussi aux contacts des applications qui ont un link avec cette application dans cette environnement.

L'équipe peut venir dire que cela est pris en compte par l'équipe et qu'une analyse est en cours, un autre mail préviendra qu'une action est bien en cours, atténuant l'effet alerte

Cela permet de passer de rouge à orange sur l'écran.

Le mail d'envoi aux contacts de l'applicaton contient un bouton qui permet directement d'aller dire que c'est bien lu et pris en compte (qui fera l'appel rest qui faut pour valider la prise en compte (bonus email de la personne qui a dit qu'il avait pris en compte=> affichage !).

### déclarer une résolution

Suite à l'analyse ou directement, lorsque l'on sait resoudre le problème, en déclarant une résolution cela veut dire que l'application va revenir à la normale.

Le mail d'envoi aux contacts de l'applicaton contient un bouton qui permet directement d'aller dire que c'est en cours de UP.

Lors du passage ente orange-> vert et rouge -> vert par la sonde du ping, un mail est bien sur envoyés aux contacts (appli et link) pour prévenir du retour à la normale.

### critère down

Afin de limiter les alertes , il peut être nécessaire de définir un critère de down, est ce que l'application est up 24/7.
Alors on peut ajouter une/des periodes periode à tester qu'on ajoute à la déclaration de l'application

Une application est down par un seul appel ping, est ce que l'on considère que c'est une interruption ? un compteur de down peut être ajouté, ainsi on définit un seuil d'acceptance de down. si on ping toutes les 1 minutes , on peut autorisé le fait d'etre down 3* 1 min. ains on ne ne lance les alertes qu'après 3 min de down.




