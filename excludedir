#Accès à mysql
mysql -D se3db
#Info : Lister les noms et types des colonnes
show columns from corresp;
#Passer de 100 à 120 le nombre de caractères dans la colonne valeur
alter table corresp modify valeur varchar(120) NOT NULL DEFAULT '';
#Modifier la valeur de la clé excludedir
update corresp set valeur = 'Local Settings;Temporary Internet Files;Historique;Temp;Application Data;AppData\\Local;AppData\\LocalLow' where Intitule = 'excludedir' ;
#Vérifier 
select valeur from corresp where Intitule = 'excludedir';
#quitter 
quit
