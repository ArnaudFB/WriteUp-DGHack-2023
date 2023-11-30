# Challenge : A Maritime Journey

## Présentation du challenge

Lorsque l'on arrive sur la page du challenge, nous avons les informations suivantes :
```
Vous faîtes partie d'une compagnie de sauvetage en mer opérant dans la rade de Brest et alentours.

Pendant sa dernière escale, le capitaine de l'un de vos remorqueurs d'intervention, d'assistance et de sauvetage (RIAS)
indique qu'il a obvservé des anomalies sur son GPS lors des dernières missions.

Vous avez été chargé d'investiguer en analysant les données recupérés sur l'ECDIS
(Electronic Chart Display and Information System) du "Sam Brandy".
```

### Premier flag : Unknown frame

```
Pour commencer votre investigation, vous devez retrouver le standard utilisé par les systmes de navigation
dont voici l'une des trames :
$GPRMC,.00,A,4821.984,N,00429.118,W,17.0,170.4,2023,,,A,S*09
Format du Flag: DGHACK{nom_du_standard_numéro} (tout attaché, sans underscore)
```

Une brève recherche Internet nous apprends qu'il s'agit des trames NMEA, à partir de maintenant, je vais utiliser en grande partie les indications disponibles sur se site : http://www.cedricaoun.net/eie/trames%20NMEA183.pdf
On nous parle directement du standard NMEA-0183 , on flag donc avec:
```
DGHACK{NMEA0183}
```

### Deuxième flag : Geographical benchmark

```
Lors de vos analyse, vous avez besoin de retrouver l'une des routes maritimes empruntée par le Sam Brandy.
Retrouver les coordonnées GPS à partir de la trame suivante :
$GPGGA,123519,4815.970,N,0447.340,W,1,08,0.9,545.4,M,46.9M, ,*41
Donnez le résultat au format "degré" "minute" "seconde" (arrondi au centime de seconde).
Format du Flag: DGHACK{DDMMSS.SS[N|S],DDMMSS.SS[W|E]}
```

En utilisant un outil en ligne tel que https://rl.se/gprmc on trouve alors: Position	48,266167°N 4,789000°W
Puis avec un convertisseur en ligne: Position 48° 15' 58.2012" N, 4° 47' 20.4" W

On retire ce qui n'est pas nécessaire et on ajoute des 0 si besoin pour respecter le format du flag.
```
DGHACK{481558.20N,044720.40W}
```

### Troisème flag : Fast Sentence

```
Pour comprendre la cinématique du remorqueur et la cohérence des données de navigation envoyées à l'ECDIS,
vous avez besoin de régénérer vous-même certaines données :
Trouver le moyen de générer la bonne trame VTG émise par le Système GPS sachant que votre navire va à 3.24 noeuds
,que son cap réel est de 34.8° et que son cap vrai magnétique est de 24.2° .Sachant que le mode autonome est utilisé.
Attention, ne mettez pas de 0 devant les métriques (écrivez 34.8 et non pas 034.8). La vitesse en Km/h sera arrondie
au chiffre supérieur sans indiquer de virgule.
Format du Flag: DGHACK{trame_VTG}
```

En réutilisant le site qui détaille les trames NMEA, on trouve déjà : $GPVTG,34.8,T,24.2,M,3.24,N,6,K
De plus, le mode automatique est utilisé, on a ainsi : $GPVTG,34.8,T,24.2,M,3.24,N,6,K,A
Finalement, il faut renseigner le checksum, on peut le faire en ligne grâce à ce site : https://nmeachecksum.eqth.net/
Et on trouve comme trame : $GPVTG,34.8,T,24.2,M,3.24,N,6,K,A*05

```
DGHACK{$GPVTG,34.8,T,24.2,M,3.24,N,6,K,A*05}
```

A noter que si on converti 3,24 noeuds, on trouve 6,00048 km/h arrondi au supérieur, j'avais initialement mis 7 (le côté mathématicien qui ressort), ça et le mode automatique et le checksum, il fallait faire un peu attention pour bien flag.

### Quatrième flag : What a time to be on sea !

```
Il semble que le Sam Brandy ait été identifié le 14/09/2023 loin de sa zone d'opération.
Bien que cela vous semble troublant, vous décidez de reconstituer les évènements grâces aux trames que vous avez récupérés:
$GPGSV,3,1,9,27,65,309,24,31,63,061,10,16,57,203,34,26,55,143,171E
$GPGSV,3,2,9,05,36,257,29,28,32,048,29,08,27,312,34,18,27,109,1940
$GPGSV,3,3,9,09,17,232,2876
Les coordonnées indiquées sont les suivantes :
30°41'24.0"S 133°42'0.0"W
Vous devez retrouver à quelle heure il a été aperçu (GMT+00)
Format du Flag: DGHACK{OCEAN_HH_MM}
```

En se rendant sur le site : https://in-the-sky.org/satmap_worldmap.php?gps=1 aux bonnes coordonnées, en faisant défiler heure par heure et en identifiant les satellites 27, 31, 16 et 26, on trouve en première approximation 10H45. En n'oubliant pas de convertir de GMT-09 à GMT+00 et en testant un peu autour de la valeur trouvé, on flag avec :

```
DGHACK{PACIFIQUE_19_52}
```

### Cinquième flag : Ephemeris Shift

```
Il semblerait que le Sam Brandy ait subi une attaque de type GPS jamming visant à lui faire perdre la connexion.
On considre qu'une perte de connexion se produit dès que le signal est perdu pendant plus d'une dizaines de secondes,
sinon c'est une simple instabilité de connexion.
Retrouver la date et l'heure à laquelle l'attaque à commencé.
Format du Flag: DGHACK{DDMMYYYYhhmmss}
```

Nous avons un fichier GPS_NMEA_dump.txt contenant toutes les informations utiles

En cherchant attentivement, on se rend compte que des données manquent dans certaines trames GPRMC et GPGGA comme ci-contre :

```
$GPRMC,170239.10,V,5000.729904,N,958.348,W,,,2023,09,12,A,N*00
```

Le cap et la vitesse n'apparaissent pas, en cherchant ce pattern dans le document, on trouve la première vraie perte de connexion liée à l'attaque :

```
$GPGGA,170334.50,5000.910034,N,958.609,W,0,00,,,M,0.0,M,,*79
```

On flag avec :

```
DGHACK{12092023170334}
```

### Sixième flag : ID Please

```
Après avoir compris que le Sam Brandy s'est peut-être retrouvé dans une zone exposée au risque de brouillage GPS, il est probable que, d'aprs votre expérience, l'un des navires rencontrés par le Sam Brady ait cherché à cacher ses activités illicites (pêches illégales, contrebande, piraterie, ...). Vous avez récupéré les enregistrements AI captés par le remorqueur, essayez d'identifier un navire qui aurait changer d'identité maritime en cours de route.
Format du Flag: DGHACK{oldDATA_newDATA}
```

Nous avons un fichier AIS_dump.txt contenant toutes les informations utiles.

On commence par traduire avec Python les messages en clair et on se rend compte que seul les MessageType5 nous serons utiles car ils donnent des indications sur les noms de vaisseaux : shipname et les identifiants de vaisseaux : mmsi. On utilise le module pyais de Python pour décoder les messages avec le code suivant : 

```
from pyais import FileReaderStream
def solution():

    output = open("output.txt", "w+")

    for msg in FileReaderStream("AIS_dump.txt"):
        decoded = msg.decode()
        if str(decoded).startswith("MessageType5"):
            output.write(str(decoded))
            output.write("\n")

if __name__ == '__main__':
    solution()
```

Puis on passe sous R pour un peu de traitement de données, on ne conserve que shipname et mmsi dans la dataframe et on fait un tri pour ne conserver que les bateaux dont le mmsi a changé mais qui ont conservé le même nom (et dont le nom apparait seulement 2 fois, sous peine de ne pas pouvoir identifier oldDATA et newDATA). 

```
library(dplyr)

output <- read.table("output.txt", sep = ",")

output <- output[, c(3, 7)]

test <- unique(output)

test <- test %>% group_by(V7) %>% filter( n() == 2 )
```
On trouve le bateau suivant dont le mmsi a clairement changé :
mmsi=227861937, shipname=DELTA YELLOW -> mmsi=883686372, shipname=DELTA YELLOW

On flag avec :

```
DGHACK{227861937_883686372}
```

Malheureusement, il y avait ce bateau : mmsi=368286750, shipname=RESOLUTION -> mmsi=338034319, shipname=RESOLUTION
Mais ce n'était pas la solution attendue
