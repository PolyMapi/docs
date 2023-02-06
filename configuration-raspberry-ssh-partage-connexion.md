Prérequis :
 - Logiciels : VSCode avec l'extension "Remote SSH de Microsoft", Raspberry Pi Imager (testé sur la version 1.7.3)
 - Matériel : Un Raspberry, un téléphone permettant de réaliser un partage de connexion

Générer sur l'ordinateur une clé SSH avec la commande `ssh-keygen -t ed25519`

Insérer la micro-SD du Raspberry dans le lecteur de l'ordinateur.

Démarrer le partage de connexion. Connecter l'ordinateur au partage. Noter le SSID et le mot de passe. S'assurer qu'il soit actif tout au long de la manoeuvre.

Ouvrir Raspberry PI Manager
Dans système d'exploitation/Choisissez l'OS, prendre "Raspberry PI OS (32-bit)"
Dans le menu engrennage qui vient d'appraître, remplir les champs suivants :
 - Set hostname –> Laisser désactivé
 - Enable SSH -> Allow public authentication only -> Activer, et rentrer le contenu de la clé publique
 - Set username and password -> Activer, et rentrer les identifiants de votre choix
 - Configure wireless LAN -> Activer, et rentrer le SSID et le mot de passe du partage de connexion
 - Wireless LAN country -> Pays dans lequel vous le réalisez (nous avons mis FR)
 - Set locale settings -> Laisser désactivé
 - Persistent settings -> Activez/Désactivez selon votre choix. (Note : La télémétrie implique l'envoi de données, ce qui peut être un coût supplémentaire lors d'un partage de connexion)
Dans Stockage, choisir la Micro SD
Cliquer sur Ecrire, et confirmez l'écrasement des données

Une fois la procédure terminée, éjecter la micro-SD et la réinserrer dans le Raspberry, puis allumer ce dernier.
Lancer VSCode
Cliquer sur le petit bouton vert en bas à gauche de l'écran
Dans la liste des actions affichée en haut de l'écran, se connecter en SSH
Indiquer la connexion à nomdutilisateurchoisi@raspberrypi.
Une fois connecté, le bouton vert en bas à gauche indiquera la connexion en SSH.
Une fenêtre s'ouvira, permettant de travailler sur le Raspberry et d'accéder à ses fichiers.