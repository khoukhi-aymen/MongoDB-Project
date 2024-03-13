# Base de données gymnass
la base de données « Gestion des salles de sport »dont le schéma relationnel est donné ci‐ dessous :    
VILLES (VILLE)  
SPORTIFS (IDSPORTIF, NOM, PRENOM, SEXE, AGE, IDSPORTIFCONSEILLER*)  
SPORTS (IDSPORT, LIBELLE)  GYMNASES (IDGYMNASE, NOMGYMNASE, ADRESSE, VILLE*, SURFACE)  
ARBITRER (IDSPORTIF*, IDSPORT*)  
ENTRAINER (IDSPORTIFENTRAINEUR*, IDSPORT*) 
JOUER (IDSPORTIF*, IDSPORT*)  
SEANCES (IDGYMNASE*, IDSPORT*, IDSPORTIFENTRAINEUR*, JOUR, HORAIRE, DUREE)


## Installation

1. Cloner le dépôt
2. installer MongoDB
3. installer MongoDBCompass
4. importer la BD gym sur MongoDBCompass

## Utilisation

Exécuter les requettes une par une et voir les résultats


## Licence

Ce projet est sous licence [MIT LINCENSE]. Voir le fichier LICENSE.md pour plus de détails.

## Auteurs

Khoukhi Aymene

## Statut du Projet

Stable
