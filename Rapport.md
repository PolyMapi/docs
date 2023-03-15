# Rapport Projet Mapillary

[Instructions du rapport](https://air.imag.fr/index.php/Projets_2022-2023)

<img align="left" src="images/Logo_polytech.png" alt="logo Polytech" height="100">
<img align="right" src="images/logo-lig.png" alt="logo LIG" height="100">
<p align="center">
  <img  height="100" src="images/Logo_UGA.png" alt="logo UGA" height="100">
</p>

## Sommaire
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

## Rappel du sujet

  Mapillary est un service de visualisation d'espaces publics, à l'instar de Google Streetview. Ce service est basé sur les contributions de ses utilisateurs. Ainsi, quand un usager téléverse une photographie (à 360 degrés ou non), Mapillary traitera l’image de différentes manières, par exemple en floutant les visages/plaques d'immatriculation, puis la téléversera à son tour sur OpenStreetMap. Notre projet vise à développer des outils facilitant la contribution de photos/vidéos 360°, géolocalisées au centimètre près.

  La Cinématique temps réel (ou RTK, Real Time Kinematic) est une technique de positionnement par satellite qui permet de géolocaliser des composants au centimètre près.
Le but de notre projet est de créer un système permettant de relier ensemble une caméra 360° ainsi qu’un module RTK pour automatiser le processus de prise et téléversement de photographie 360° sur les services de Mapillary.


## Technologies employées

  Nous avons utilisé un certain nombre de technologies différentes au cours du projet.

#### Réseau Centipede RTK

  Le réseau Centipede RTK est composé d’un ensemble de bases Centipede. Chacune connaît sa localisation GPS exacte au centimètre près, et demande à courts intervalles réguliers sa position aux satellites, afin de déterminer la différence entre sa position réelle et sa position supposée par les satellites. Cette différence est matérialisée sous la forme de données de correction.

  Afin de pouvoir profiter des fonctionnalités du réseau Centipede RTK, il faut utiliser un client “n-trip”. Après le choix de la base, le client reçoit les données de correction de la zone. Les bases ont une portée limitée d’environ 50 kilomètres, il faut donc s’assurer de rester dans leur zone pour obtenir des données de correction pertinentes. Bien sûr, plus le client s’éloigne du centre de la base, moins les corrections données lui sont pertinentes, même s’il reste dans la portée des 50 kilomètres.

#### Module Ublox ZED-F9P/Carte GNSS RTK Click

  Le module ZED-F9P est un récepteur GNSS permettant d’obtenir des coordonnées GPS très précises. Il est composé d’une antenne GNSS, qui reçoit les signaux GPS, GLONASS, Galileo, et BeiDou, et est intégré à une carte GNSS RTK, qui possède des connecteurs I2C qui leur permettent de transmettre des données à d’autres composants électroniques. Le module fonctionne de la manière suivante : Il reçoit des coordonnées GPS imprécises depuis son antenne, et les rectifie en se basant sur les données de correction de la base Centipede sélectionnée fournies par le client n-trip, qu’il reçoit depuis les connecteurs de la carte GNSS. Il renvoie ensuite dans ses connecteurs les coordonnées corrigées sous forme de paquets RTCM.

  La carte GNSS RTK permet de faire de la connectique (alimentation, antenne, connecteurs UART, etc.), afin de pouvoir lier et utiliser le module.

#### Carte ESP32

  La carte ESP32 est utilisée à des fins de communication. Quand elle est mise en route, elle charge le programme qui y est inscrit. Le but de cette carte est de faire communiquer le client n-trip, qui communique par Bluetooth, et le module ZED-F9P, qui communique par I2C. Autrement dit, faire transiter les données de correction et les coordonnées GPS corrigées. Dans le cadre de ce projet, il s’agit du seul composant électronique nécessitant un paramétrage et une programmation pour fonctionner.

#### Caméra Ricoh Theta SC

  La caméra Ricoh Theta SC est une caméra permettant de prendre des photos et des vidéos à 360° et qui fournit une API gérée par un serveur HTTP interne permettant de créer des applications exploitant les fonctionnalités de la caméra.

#### Mapillary

  Mapillary est un outil qui extrait des données cartographiques depuis des images. Il utilise la vision par ordinateur ainsi que des algorithmes d’apprentissage automatique pour identifier des caractéristiques de la photo, telles que les visages et plaques d'immatriculation (à des fins de floutage), les panneaux de signalisation, les façades de bâtiment, etc. Une fois l’image traitée, elle est téléversée sur OpenStreetMap.

  Le service fournit également une API, qui permet de programmer une application utilisant certaines des fonctionnalités proposées par Mapillary. Nous souhaitions nous servir de l’API pour téléverser les photos corrigées à partir d’une application mobile.

## Architecture technique

![Project architecture](images/Project-architecture.png)

Architecture du projet

  L’architecture que nous avons adoptée relie les deux composants électroniques à l’application Android. La caméra 360° est capable de se connecter en Wi-Fi, tandis que la carte ESP32 utilise le Bluetooth. La carte GNSS RTK, surmontée du module ZED-F9P, est reliée à l’ESP32 en Wi-Fi, tandis que le module reçoit des coordonnées GPS. Finalement, les photographies 360° aux coordonnées corrigées seront téléversées sur Mapillary en se connectant au service en Wi-Fi.

## Gestion de projet

#### Méthode

  Pour notre projet, nous avons utilisé la méthodologie agile Scrum, qui est l'une des approches les plus populaires de la méthodologie agile. La première étape que nous avons suivie a été la planification du sprint, qui est une période de deux semaines pendant laquelle nous avons travaillé sur une partie spécifique de notre projet. Nous avons évalué ce que nous pouvions réaliser pendant cette période et nous avons déterminé les tâches que chaque membre de l'équipe devait accomplir. Puis, chaque jour, nous faisions une réunion quotidienne de 10 à 15 minutes à la manière d’un Daily Scrum. Au cours de ces réunions, nous avons discuté de l'avancement de notre travail, des obstacles rencontrés et des tâches que nous avions prévues pour la journée. Enfin, à chaque fin de sprint, nous avons présenté notre travail accompli au cours du sprint et discuté des résultats entre nous et avec notre tuteur Nicolas Palix que nous voyions tous les lundis. Cela nous permettait d’examiner ce qui avait bien fonctionné pendant le sprint et ce qui pouvait être amélioré pour le suivant.

  La méthodologie agile Scrum nous a été très utile pour collaborer. En effet, cela nous a permis de travailler en équipe pour résoudre les problèmes et atteindre les objectifs de notre projet. De plus, cette gestion de projet agile nous a aidé à gagner en flexibilité car nous avons pu adapter les plans et les tâches en fonction de l’évolution du projet, des changements de priorités et des imprévus. Notre équipe a aussi pu obtenir des résultats rapides et réguliers car la méthodologie agile permettait de valider notre travail et de nous assurer qu'on était sur la bonne voie pour atteindre les objectifs du projet. La communication régulière et la collaboration au sein de l'équipe ont également été améliorées grâce à cette méthodologie qui nous a semblé très innovante pour la gestion de projet.


#### Rôles

  Les rôles de notre projet ont été répartis de la façon suivante :

 - Gergely Fodor était notre chef de projet et responsable base de données ;
 - Tom Kacha était le Scrum Master ainsi que le responsable caméra ;
 - Baptiste Jardin était le responsable GPS ;
 - Samuel Conjard était le responsable gestion de données.

#### Outils de gestion de projet

  Nous avons créé un Trello afin de mettre en place la méthodologie Agile susdite. À chaque tâche à effectuer, nous l’ajoutions au trello puis nous la déplacions lorsque cette dernière était terminée.

![Trello picture](images/Trello.png)

Gestion de projet sur Trello


#### Planning

  Nous avons enfin créé un diagramme de Gantt afin de pouvoir comparer le planning prévisionnel et le planning effectif.

![Previsional schedule](images/Provisional_schedule.png)

Planning prévisionnel

![Effective_schedule](images/Effective_schedule.png)

Planning effectif

## Outils

Nous avons utilisé plusieurs outils afin de mener à bien notre projet. Tout d’abord, afin de bien collaborer, nous avons mis en place un serveur discord afin de pouvoir plus facilement communiquer. Aussi, ce serveur discord était relié à notre dépôt Git, ce qui permettait d’afficher directement depuis discord les nouveaux commits et d’être notifié en temps réel. Nous avons aussi utilisé Google Drive afin de pouvoir partager des documents tels que des schémas ou encore des rapports ou présentations. Nous avons également utilisé Canva afin de réaliser notre poster en Anglais et notre flyer.

Comme mentionné ci-dessus, nous avons créé un trello afin de nous aider à mettre en place une méthodologie agile.

Enfin, nous avons utilisé Android Studio comme IDE afin de développer notre application android. Nous avons développé notre application en Java et Android Studio mettait à disposition un mode débogage avec un téléphone portable android, ainsi qu’un émulateur permettant de faire tourner un système Android virtuel sur un ordinateur.

## Métriques logiciels

Temps ingénieur total : 45 jours x 7 h/jours x 4 = 1260h

#### Controller :

![Metrique controller](images/metrique_controller.png)

Nombre de commits : 43

#### SourceTHETA : 

![Metrique sourceRTK](images/metrique_sourceRTK.png)

Nombre de commits : 7

#### SourceRTK :

![Metrique sourceTHETA](images/metrique_sourceTHETA.png)

Nombre de commits : 4

#### Doc :

Nombre de commits : ???

