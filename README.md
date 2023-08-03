# Calcolo distanza scuole d'infanzia da centri urbani
## Dati disponibili
### confini comuni d'Italia da ISTAT
i dati si trovano a [questa pagina](https://www.istat.it/it/archivio/222527)<br/>
I dati sono utilizzati sono quelli dei [confini non generalizzati](https://www.istat.it/storage/cartografia/confini_amministrativi/non_generalizzati/2023/Limiti01012023.zip), in particolare quello dei confini comunali (**Com1012023_WGS84.shp**)

### elenco delle scuole da Ministero Istruzione
Da questa [risorsa](https://dati.istruzione.it/opendata/opendata/catalogo/elements1/?area=Scuole) sono presi i dati delle anagrafiche delle:
- scuole statali
- scuole paritarie
- scuole statali di Valle d'Aosta, Provincia Autonoma di Trento e Provincia Autonoma di Bolzano
- scuole paritarie di Valle d'Aosta, Provincia Autonoma di Trento e Provincia Autonoma di Bolzano<br/>

note:
* i dati sono privi di coordinate geografiche

### elenco centri abitati da OpenStreetMap
A causa del fatto che in Italia esistono diversi comuni sparsi e del fatto di avere un punto significativo che indica il baricentro sociale di un luogo, si è optato per scaricare i dati dei centi urbani da OpenStreetMap selezionando i valori di [city](https://wiki.openstreetmap.org/wiki/Tag%3Aplace%3Dcity), [sub-urb](https://wiki.openstreetmap.org/wiki/Tag%3Aplace%3Dsuburb), [town](https://wiki.openstreetmap.org/wiki/Tag%3Aplace%3Dtown) e [village](https://wiki.openstreetmap.org/wiki/Tag%3Aplace%3Dvillage) dal tag [place](https://wiki.openstreetmap.org/wiki/Map_features#Place) <br/>
 si possono ottenere anche attraverso questa [query alle overpass-api](https://overpass-turbo.eu/s/1yku)<br/>
<br/>
note
 - la categorizzazione è delegata all'interpretazione di chi ha inserito i dati
 - il punto che rappresenta il toponimo è a discrezione di chi ha inserito i dati: in alcuni casi è il "baricentro sociale", in altri il centroide del confine comunale, in altri la piazza principale o il municipo o il campanile ...
 - ci sono diversi casi in cui è stata creata un'area. In tal caso si sceglie di utilizzare un punto significativo all'interno dell'area sulla base della [funzione di geopandas](https://geopandas.org/en/stable/docs/reference/api/geopandas.GeoSeries.representative_point.html)
   
###  grafo stradale di openstreetmap
si è preso il file pbf che si trova su [geofabrik](https://download.geofabrik.de/europe/italy-latest.osm.pbf) 

## Calcolo delle distanze
Il calcolo delle distanze avviene tramite una istanza [GraphHoper](https://www.graphhopper.com/) da eseguire sul proprio computer.<br/>
L'installazione prevede l'installazione di java.<br/>
Il software si scarica con queste istruzioni:
```bash
wget https://repo1.maven.org/maven2/com/graphhopper/graphhopper-web/7.0/graphhopper-web-7.0.jar https://raw.githubusercontent.com/graphhopper/graphhopper/7.x/config-example.yml https://download.geofabrik.de/europe/italy-latest.osm.pbf
```
l'esecuzione invece con questa istruzione
```
java -D"dw.graphhopper.datareader.file=italy-latest.osm.pbf" -jar graphhopper*.jar server config-example.yml
``````
L'istruzione *-D"dw.graphhopper.datareader.file=italy-latest.osm.pbf"* può essere ignorata qualora venga modificato il file *config-example.yml* con il giusto riferimento al file.

## Geocoding
Per arricchire le informazioni sulle posizioni delle scuole d'infanzia viene utilizzato il servizio di [geocoding di ESRI ArcGIS](https://developers.arcgis.com/documentation/mapping-apis-and-services/geocoding/geocode-addresses/) sempre attraverso [geopandas](https://geopandas.org/en/stable/docs/reference/api/geopandas.tools.geocode.html)<br/> 
Il geocoding comunque spesso restituisce valori che vanno verificati i corretti.<br/>
Alcuni consigli per effettura delle verifiche<br/>:
- il punto si trova all'interno del comune interessato?
- il punto si trova vicino alla via dichiarata? (controllo con uso di reverse-geocoding?)<br/>
Ulteriori verifiche possono essere fatte con ricerche mirate online e attraverso la verifica visita attraverso strumenti come Google Street View<br/>
<br/>note<br/>
- fare molta attenzione al numero di chiamate al secondo

# Procedura
Lo script jupiter notebook [calcolo_distanze_centri_abitati_scuole_infanzia.ipynb](calcolo_distanze_centri_abitati_scuole_infanzia.ipynb) carica prima i dati cercando nella sottodirectory "data".<br/>
Da qui fa una serie di operazioni di pulizia/integrazione dei dati che archivia sempre in "data" in modo che, ad un secondo avvio dello script, non debba rifare quelle operazioni.<br/>
La sezione che si occupa del calcolo delle distanze dalle scuole di infanzia ai centri abitati filtra per una provincia (allo stato attuale è stata inserita Bergamo).<br/>
Da qui filtra quindi i dati delle scuole e dei confini comunali della provincia scelta.<br/>
Il calcolo delle distanze avviene prendendo poi un comune alla volta, individuando i centri abitati al suo interno e quelli dei comuni confinanti (questo per ridurre i tempi di attesa dei calcoli) - sono quindi *esclusi* i comuni confinanti di altra provincia o regione.<br/>
Allo stesso modo individua le scuole d'infanzia (statali o paritarie) contenute nei comuni individuati e avvia le chiamate per il calcolo delle distanze interrogando l'instanza GraphHoper avviata sul proprio computer.<br/>
Le distanze sono archiviate in metri ed i tempi di pecorrenza in millesecondi.<br/>
La tabella viene generata in "data" con il nome di "distanze_scuoleinfanzia_centriabitati_*Provincia*.csv" dove *Provincia* corrisponde al nome della provincia scelta.<br/>
Nel file vengono poi riportati:
- nome della scuola
- indirizzo
- coordinate scuola
- codice identificativo della scuola
- comune
- codice comune istat
- nome del centro abitato
- comune dove si trova il centro abitato
- codice istat del comune
- coordinate del centro abitato
- tempo di percorrenza dalla scuola al centro abitato in millesecondi utilizzando l'auto
- distanza percorsa dalla scuola al centro abitato in metri utilizzando l'auto
