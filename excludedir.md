# SE3 et seven : problème de remontée des profils itinérants
Samuel GONZALES – Raip de Strasbourg

## Le problème
Certains profils itinerants seven ou win10 pèsent lourd. Ils sont stockés sur le serveur dans l’arborescence /home/profiles/USERNAME.V2 ou .V6 et sont visibles via le partage reseau \\$IPSERVEUR\admhomes\profiles. Ces profils itinerants gonflent lorsque la cle de base de registre
> **HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\ExcludeProfileDirs**

est mal ajustée, car appliquée telle quelle (par exemple via un template, voire un vieux template désactivé) sur un poste win7 ou supérieur elle entraine la remontée sur le serveur d'une partie **lourde** du profil local (cache des applications google, etc...). Pour ce rendre compte de ce problème il suffit de regarder la clé avec regedit.

Par défaut sur SE3 la valeur est fixée à 
> 'Local Settings;Temporary Internet Files;Historique;Temp;Application Data'

Ce qui ne va pas pour du win7 dont la clé a par défaut la valeur suivante :
> AppData\\Local;AppData\\LocalLow;$Recycle.Bin

Ou pour win10 :
> AppData\\Local;AppData\\LocalLow;$Recycle.Bin;OneDrive;Work Folders

Si la clé de registre excludedir est insérée dans le template de base, c'est la catastrophe, les profils itinérants enflent. Les dossiers **AppData\\Local;AppData\\LocalLow** restent donc dans la partie itinérante du profil. De même pour One Drive, installé par défaut sur win10.

## Deux solutions

1 - Première idée : créer une clé spécifique **"excludedirwin7"** et **"excludedirwin10"** mais ce n'est pas possible car le chemin de la clé doit être unique... et comme c'est le même chemin pour chaque version de windows mysql n'est pas d'accord (ou alors il faut modifier la table se3db.corresp).

2 - Deuxième idée : modifier la clé excludedir actuelle en créant une "super clé pour les gouverner tous"!

Sa valeur devrait donc être :

> Local Settings;Temporary Internet Files;Historique;Temp;Application Data;AppData\\Local;AppData\\LocalLow;$Recycle.Bin;OneDrive;Work Folders

Sauf que cette chaîne est formée de plus de 100 caractères qui est la limite de la colonne 'valeur' de la table 'se3db.corresp'. Il faut donc modifier cette valeur de la manière suivante en se connectant en root sur le serveur (via putty par exemple, il suffit ensuite de faire du copier/coller):

*#Accès à mysql*

> mysql -D se3db

*#Info : Lister les noms et types des colonnes*

>mysql> show columns from corresp;

*#Passer de 100 à 150 le nombre de caractères dans la colonne valeur*

>mysql> alter table corresp modify valeur varchar(150) NOT NULL DEFAULT '';

*#Modifier la valeur de la clé excludedir. __Attention il faut absolument les doubles antislashes__*

>mysql> update corresp set valeur = 'Local Settings;Temporary Internet Files;Historique;Temp;Application Data;AppData\\\Local;AppData\\\LocalLow;$Recycle.Bin;OneDrive;Work Folders' where Intitule = 'excludedir' ;

*#Vérifier*

>mysql> select valeur from corresp where Intitule = 'excludedir';

*#quitter*

>mysql> quit

Finalement, on peut appliquer cette clé au template de base pour qu'elle remonte sur les postes pour chaque profil. Et il faut encore supprimer les dossiers qui surchargent le profil itinérant :

> ̀`#rm -rf  /home/profiles/*.V*/AppData/LocalLow

#rm -rf  /home/profiles/*.V*/AppData/Local`

Remarques :

1 - Reste encore à traiter le cas du/des dossiers Geogébra qui apparaissent dans /hom/profile/$USER.V2/GeoGebra $VERSION/

2 - Pour les bienheureux qui n'ont plus xp ou de vista, il est inutile alors de modifier le nombre de caractères autorisés dans la table corresp. La clé sera alors la suivante :

>AppData\\Local;AppData\\LocalLow;$Recycle.Bin;OneDrive;Work Folders

3 - Toutes les remarques sont les bienvenues.


