# Rapport Projet Mapillary

[Instructions du rapport](https://air.imag.fr/index.php/Projets_2022-2023)

# Sommaire
 - Rappel du sujet/besoin et cahier des charges
 - Technologies employées
 - Architecture technique
 - Réalisations techniques
 - Gestion de projet (méthode, planning prévisionnel et effectif, gestion des risques, rôles des membres ...)
 - Outils (collaboration, CD/CI ...)
 - Métriques logiciels : lignes de code, langages, performance, temps ingénieur (d'après vos journaux), la répartition des lignes de code et des commits en pourcentage entre les membres du projet ...)
 - Conclusion (Retour d'expérience)
 - Glossaire
 - Bibliographie

# Rappel du sujet/besoin et cahier des charges

Mapillary est un service de visualisation d'espaces publics, à l'instar de Google Streetview. Sa particularité vient du fait que les usagers fournissent les photos panoramiques, sur le principe de base des contributions.

Les caméras 360° prennent des photos panoramiques, et Ricoh est un des leaders du marché avec ses caméras grand public Theta. Les photos/vidéos prises par ces caméras peuvent être envoyées à Mapillary. Cependant, il faut géolocaliser ces photographies de manière précise. Pour cela, il est possible de coupler les prises avec les positions relevées au moyen de rovers GNSS RTK relevant des positions centimétriques.

L'objectif du projet est de développer des outils facilitant la contribution des photos/vidéos 360° géolocalisées précisément.

# Technologies employées

 - Rover GNSS RTK
 - Caméra Ricoh Theta

# Architecture technique

![Architecture of the project](images/Architecture.png)

# Réalisations techniques

Nous avons créé une application qui permet de prendre une photographie avec la caméra, de la télécharger depuis la mémoire de cette même caméra, puis de l'envoyer vers les services de Mapillary.

# Gestion de projet



# Outils

Discord, Google Drive, Android Studio

# Métriques logiciels



# Conclusion



# Glossaire

RTK : Real Time Kinetic

# Bibliographie
