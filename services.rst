Les services offerts par GrandLyon Data
=============================================

La plateforme Data vous permet d'accéder à différents services de consultation des données OpenData. Les services sont de deux types :
des services de visualisation qui vous permettent d'afficher des images (cartes) et des services de téléchargement qui vous permettent d'accéder directement à la donnée, selon des modalités et des formats variés.

Lors de la mise en oeuvre d'une application cartographique, on ne peut pas traiter toutes les données de la même manière. Il y a en effet des données qui habillent (fond de carte par ex.), d'autres qui renseignent sur le contexte (points d'intérêt divers : mairie, gare...) et d'autres enfin qui portent l'information importante, celle que l'on souhaite vraiment mettre en avant et exploiter dans l'application (disponibilité des vélos, localisation des bus en temps réel...).

Le fond de plan sera toujours proposé au format image, non interactif, et très souvent issu d'un système de cache tuilé permettant la récupération rapide sous forme de petites tuiles (256 x 256 pixels) de son contenu. Ce sera toujours la couche la plus basse dans l'application, celle en dessous de toutes les autres, pour des raisons évidentes de visibilité.

Les informations cruciales, importantes, porteuses de la valeur ajoutée de l'application seront les plus interactives possibles, pour que d'un survol une info-bulle donne accès à l'essentiel, qu'un clic ouvre une fiche complète, qu'un changement de style (taille, couleur) dynamique permette de souligner la sélection, l'association, la relation avec un autre élément. Pour ce faire il faut donc utiliser un service de téléchargement, seul à même de transmettre les données brutes permettant la mise en oeuvre d'une couche vectorielle dans l'application cartographique ou de donner accès à tous les attributs (informations associées à l'objet géographique) directement.

Pour les autres couches de données il faut faire des arbitrages. On ne peut tout charger en vectoriel pour des raisons de lisibilité (trop de points/lignes qui clignotent, grossissent ou s'agitent en même temps rendent la carte inutilisable) et de performance (chaque point est inscrit dans le DOM de la page. Plusieurs milliers de points deviennent très lourds à gérer pour le navigateur). Donc on transige. On utilise les services de visualisation (WMS, format image) pour les données dont l'emplacement l'information la plus importante (par ex : stationnement handicapé, il n'y a rien à mettre dans une info-bulle, l'important est que la place soit là où elle est, et il suffit donc de l'afficher) ou dont l'emprise spatiale a un intérêt particulier (contour d'un parc, d'une zone règlementée...). Et on choisit le format vectoriel pour quelques informations certes secondaires par rapport à l'objet principal de l'application, mais dont les caractérisques sont importantes à connaître (stations vélos, événement...)



Service WMS
-----------

Le service WMS est le service de visualisation par excellence. Il sert à "voir" la donnée géographique avec une mise en forme prédéfinie (couleurs, styles, symboles...). C'est le service à privilégier pour intégrer des jeux de données au format image.

Il est accessible à partir de l'URL

https://data.grandlyon.com/geoserver/wms/[nom_du_service]

Généralement on fait appel à son opération GetCapabilities pour connaître son contenu (liste des couches) :

https://data.grandlyon.com/geoserver/wms?SERVICE=WMS&REQUEST=GetCapabilities&VERSION=1.3.0

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez le contenu à l'aide d'une requête GetMap de ce type :

https://data.grandlyon.com/geoserver/wms?LAYERS=adr_voie_lieu.adrcommune&FORMAT=image%2Fpng&EXCEPTIONS=application%2Fxml&TRANSPARENT=TRUE&VERSION=1.1.1&SERVICE=WMS&REQUEST=GetMap&STYLES=&SRS=EPSG%3A4171&BBOX=4.7,45.6,5,45.9&WIDTH=720&HEIGHT=780

.. image:: https://data.grandlyon.com/geoserver/wms?LAYERS=adr_voie_lieu.adrcommune&FORMAT=image%2Fpng&EXCEPTIONS=application%2Fxml&TRANSPARENT=TRUE&VERSION=1.1.1&SERVICE=WMS&REQUEST=GetMap&STYLES=&SRS=EPSG%3A4171&BBOX=4.7,45.6,5,45.9&WIDTH=720&HEIGHT=780
   :alt: GrandLyon Data : le service WMS
   :class: floatingflask

Ca peut vous sembler un peu compliqué et fastidieux... Mais les librairies cartographiques sont là pour vous aider. Leaflet et OpenLayers implémentent des classes WMS qui feront tout ça pour vous, en adaptant les requêtes selon les manipulations faites sur la carte. Il vous suffit généralement de renseigner l'URL de base du service et d'indiquer le nom de la couche que vous souhaitez utiliser comme l'illustrent nos :ref:`exemples`.

Service WFS
-----------

Les services WFS permettent de récupérer la donnée brute telle qu'elle est enregistrée dans la base de données. Il s'agit d'un service de téléchargement, même si ce téléchargement (récupération de la donnée) n'est pas forcément complet, c'est-à-dire qu'on peut l'effectuer sur une partie seulement du territoire, notamment pour les données les plus volumineuses.

Il est accessible à partir de l'URL

https://data.grandlyon.com/geoserver/wfs/[nom_du_service]

Généralement on fait appel à son opération GetCapabilities pour connaître son contenu (liste des couches) :

https://data.grandlyon.com/geoserver/wfs?SERVICE=WFS&REQUEST=GetCapabilities&VERSION=1.1.0

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez le contenu à l'aide d'une requête GetFeatures de ce type :

https://data.grandlyon.com/geoserver/wfs?SERVICE=WFS&REQUEST=GetFeature&typename=pvo_patrimoine_voirie.pvotronconwebcriter&VERSION=1.1.0

Mais là encore, rassurez-vous, les librairies cartographiques disposent des classes nécessaires à une utilisation simple de ce type de service.

Le format généralement utilisé en WFS est le GML (Geographic Markup Language) qui est un dérivé du XML. Toutefois, ce format n'est pas forcément le plus simple à utiliser dans le cadre d'une application web. Aussi, tout en utilisant le WFS, il est possible de recevoir un flux au format GeoJSON en ajoutant le paramètre OUTPUTFORMAT=geojson :

https://data.grandlyon.com/geoserver/wfs?SERVICE=WFS&REQUEST=GetFeature&typename=pvo_patrimoine_voirie.pvotronconwebcriter&VERSION=1.1.0&OUTPUTFORMAT=geojson

Service API Features
--------------------

Le service API Features est similaire au service WFS, mais offre une interface de programmation plus moderne.

La racine du service se trouve ici : https://data.grandlyon.com/geoserver/ogc/features/v1/

Une définition OpenAPI est disponible ici : https://data.grandlyon.com/geoserver/ogc/features/v1/openapi?f=text%2Fhtml

La reqûete suivante permet de lister les jeux de données disponbiles : https://data.grandlyon.com/geoserver/ogc/features/v1/collections

Le format généralement utilisé avec l'API Features est le GeoJSON, mais il est modifiable à travers le paramètre "f" (par exemple, f=application/gml+xml;version=3.2). Voici un exemple d'URL pour récupérer un jeu de données :

https://data.grandlyon.com/geoserver/ogc/features/v1/collections/metropole-de-lyon:pvo_patrimoine_voirie.pvotronconwebcriter/items?f=application%2Fjson&limit=50

Davantage d'informations sur ce standard peuvent être trouvées ici https://ogcapi.ogc.org/features/ et ici https://opengeospatial.github.io/e-learning/ogcapi-features/text/basic-index.html

Service WCS
-----------

Les services WCS (Web Coverage Service) permettent de récupérer directement les données brutes des couches raster (comme les orthophotos et les MNT). Le terme Coverage (couverture) correspond au jeu de données raster.
Il s'agit donc d'un service de téléchargement dans lequel il est possible de filtrer le jeu de données à récupérer sur une partie seulement du territoire.

Il est accessible à partir de l'URL :

https://data.grandlyon.com/geoserver/wcs

De même que pour les WMS ou le WFS, on fait appel à son opération GetCapabilities pour connaître son contenu (liste des couches disponibles ) :

https://data.grandlyon.com/geoserver/wcs?service=WCS&request=GetCapabilities&version=1.1.0

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez la description détaillée à l'aide d'une requête DescribeCoverage de ce type :

https://data.grandlyon.com/geoserver/wcs?service=WCS&request=DescribeCoverage&version=1.1.0&identifiers=grandlyon:MNT2012_Altitude_10m_CC46

Les informations retournées ne concernent plus que les couches spécifiées dans le paramètre identifiers (ici MNT2012_Altitude_10m_CC46) et sont un peu plus détaillées que dans le GetCapabilities.

Enfin, pour obtenir la couverture souhaitée, on utilise une requête GetCoverage de ce type :

https://data.grandlyon.com/geoserver/wcs?SERVICE=WCS&VERSION=1.0.0&REQUEST=GetCoverage&FORMAT=GeoTIFF&COVERAGE=grandlyon:MNT2015_Ombrage_2m&BBOX=1836243.96544679999351501,5162352.9513221001252532,1842093.96544679999351501,5168132.9513221001252532&CRS=EPSG:3946&RESPONSE_CRS=EPSG:3946&WIDTH=585&HEIGHT=578

Encore une fois, c'est un service standardisé et les librairies cartographiques disposent des classes nécessaires à une utilisation simple de ce type de service.

Service CSW
-----------

Les services CSW (Catalog Services for the Web) permettent d'interagir avec le catalogue de métadonnées de GrandLyon Data.

Ils recouvrent 2 grands types d'usage : la consultation et l'édition des métadonnées. Dans le cas présent, seules les fonctionnalités de consultation sont concernées puisqu'il n'y a pas lieu de mettre à jour le catalogue de la plateforme GrandLyon Data.
Les requêtes CSW vont ainsi permettre de rechercher des données et d'accéder à la fiche descriptive détaillée d'une donnée.

Comme pour les services précédemment décrit, la découverte du service se fait via le GetCapabilities :

https://data.grandlyon.com/geonetwork/srv/fre/csw?SERVICE=CSW&request=GetCapabilities&service=CSW&version=2.0.2

Pour effectuer une recherche, on utilise l'opération GetRecords, dans laquelle on peut spécifier des critères de recherche. Par exemple :

https://data.grandlyon.com/geonetwork/srv/fre/csw?SERVICE=CSW&request=GetRecords&service=CSW&version=2.0.2&resultType=results&OUTPUTSCHEMA=http://www.opengis.net/cat/csw/2.0.2&ELEMENTSETNAME=brief%20&CONSTRAINTLANGUAGE=CQL_TEXT&typeNames=csw:Record&maxRecords=1000&constraint_language_version=1.0.0

Notez le paramètre ELEMENTSETNAME qui permet de choisir le type d'élements retournés (brief, summary ou full). L'utilisation de startPosition et maxRecords permet de gérer la pagination pour ne pas charger d'un coup les plus de 500 fiches. Les critères de recherche peuvent être renseignés soit avec CQL, soit avec OGC FE (Filter Encoding).

L'opération GetRecordById permet d'accéder à une métadonnée à partir de son identifiant, donc d'obtenir le contenu détaillée pour une fiche précise :

https://data.grandlyon.com/geonetwork/srv/fre/csw?SERVICE=CSW&request=GetRecordById&service=CSW&version=2.0.2&resultType=results&OUTPUTSCHEMA=http://www.opengis.net/cat/csw/2.0.2&ELEMENTSETNAME=full%20&id=3e6cd8af-5adb-4d9c-8638-f22db9b121fd

L'utilisation de ce service n'est pas simple au premier abord mais il est très performant et permet de retrouver toutes les fonctionnalités de recherche et de consultation disponibles sur le catalogue de la plateforme afin de les intégrer dans un client externe. Enfin, c'est un service standard et diverses documentations beaucoup plus détaillées sur le CSW sont facilement accessibles sur le web.


Services REST (en JSON)
-----------------------

Pour accéder aux données sous forme alphanumérique (par opposition aux services cartographiques), notre infrastructure dispose de services JSON permettant une navigation facile et rapide entre les différents jeux de données mis à disposition.

Le point d'entrée de chaque service est construit sur le pattern suivant : 

``https://data.grandlyon.com/fr/datapusher/ws/<service>/all.json``

Les services actuellement disponibles sont "grandlyon" et "rdata" :

``https://data.grandlyon.com/fr/datapusher/ws/grandlyon/all.json``

et

``https://data.grandlyon.com/fr/datapusher/ws/rdata/all.json``

Ces documents listent l'ensemble des tables disponibles en consultation/téléchargement. Certaines peuvent avoir un accès restreint en fonction de vos droits.

**Exemple de résultat** : 

:: 
  
  {
      
      results: [{
      
         table_schema: "abr_arbres_alignement",
         
         href: "https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre.json",
         
         table_name: "abrarbre"
      
      },{
         
         table_schema: "adr_voie_lieu",
         
         href: "https://data.grandlyon.com/fr/datapusher/ws/grandlyon/adr_voie_lieu.adradresse.json",
         
         table_name: "adradresse"

      },{
      
         ...
         
      }]

   }

A chaque table est associée une URL de la forme : 

``https://data.grandlyon.com/fr/datapusher/ws/<service>/<table_schema>.<table_name>.json``

De lien en lien, vous pouvez alors naviguer vers la description des tables.

*Exemple* : https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre.json

::

   {
      
      requested_table: "abr_arbres_alignement.abrarbre",
      
      nb_records: 92216,
      
      database_href: "https://data.grandlyon.com/fr/datapusher/ws/grandlyon/all.json",
      
      nb_results: 26,
      
      results: [{
      
         is_pk: false,
         
         column_type: "varchar",
         
         precision: 50,
         
         is_nullable: "YES",
         
         href: "https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json",
         
         column_name: "essencefrancais"
      
      },{
         
         is_pk: false,
         
         column_type: "int4",
         
         precision: 32,
         
         is_nullable: "YES",
         
         href: "https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre/circonference_cm.json",
         
         column_name: "circonference_cm"
      
      },{
      
         ...
         
      }]

   }

Liste des champs affichés :

* **is_pk**: est-ce l’identifiant de la couche 

* **column_type**: type de champ (numérique, texte, etc.)

* **precision**: longueur du champ

* **is_nullable**: peut il y avoir des valeurs nulles ?

* **href**: valeurs distinctes possible de l’attribut ciblé 

* **column_name**: nom du champ

L'url contenue dans href permet de consulter les différentes valeurs présentes dans un champ particulier (par ex. les essences des arbres de la métropole).

*Exemple* : https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json

::

   {
      
      fields: [
         
         "essencefrancais"
      
      ],
      
      nb_results: 401,
      
      values: [
         
         "Magnolia à grandes fleurs",
        
         "Erable rouge 'Schlesingeri'",
         
         "Arbre puant des Chinois",
         
         "Chène rouge d'Espagne",
         
         "Frêne d'Amérique",
         
         "Orme champêtre",
         
         "Chêne pédonculé fastigié, Chêne pyramidal",
         
         ...
      
      ]
   
   }

Ce dernier mode dispose d'options particulières :

* **compact** : si false, décrit la valeur pour chacun des enregistrements, sinon liste les différentes valeurs trouvées dans la table. True par défaut.

* **maxfeatures** : indique le nombre maximal d'enregistrement à faire remonter par le service. 1000 par défaut.

* **start** : indique l'index de départ, afin de pouvoir paginer les résultats. 1 par défaut.

On peut ainsi demander au service les essences de 50 arbres à partir du 100e dans la base :

https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json?compact=false&maxfeatures=50&start=101


On peut également accéder à la totalité du contenu de la table (ou paginer ce contenu) en utilisant une URL du type :

https://data.grandlyon.com/fr/datapusher/ws/rdata/jcd_jcdecaux.jcdvelov/all.json?compact=false

pour consulter l'intégralité des enregistrements. 

Il faut noter que sur l'appel de all.json (affichage de tous les champs), seul le mode compact est disponible. 

Le nombre d’objet renvoyé par défaut est fixé à 1000 pour des raisons de performances. Il est possible d’outrepasser ce retour grâce au paramètre « maxfeatures ».

*Exemple* : 
https://data.grandlyon.com/fr/datapusher/ws/grandlyon/gip_proprete.gipdecheterie/all.json?maxfeatures=10

Il est également possible de filtrer les objets renvoyés selon une valeur d'attribut avec une url de la forme : 

``https://data.grandlyon.com/fr/datapusher/ws/<service>/<table_schema>.<table_name>/all.json?field=<attribut>&value=<valeur>``

*Exemple* : 
https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre/all.json?field=essencefrancais&value=Marronnier%20de%20Virginie

all.json contient aussi des informations supplémentaires liées à la pagination, à savoir des liens vers les pages précédentes et suivantes sous la forme d'une URL reprenant la valeur de maxfeatures utilisée  pour la page en cours et modifiant la valeur du paramètre "start" en fonction de la page en cours. 

*Exemple* : 
https://data.grandlyon.com/fr/datapusher/ws/grandlyon/gip_proprete.gipdecheterie_3_0_0/all.csv?maxfeatures=5&start=10

Cette URL retourne les enregistrements 10 à 15 de la couche déchetterie.

Les services REST-JSON sont ainsi particulièrement adaptés à la constitution de listes de valeurs, de tableaux et de grilles paginés, d'interface de navigation au sein des données.

Ensuite, à la mode Django, paramètres personnalisés sous la forme `champs__opérateur`, avec champ = nom du champ cible et opérateur à choisir parmi eq, gt, gte, lt, lte, in

Exemple : https://data.grandlyon.com/fr/datapusher/ws/grandlyon/abr_arbres_alignement.abrarbre/all.json?codeinsee__eq=69116&commune__in=LIMONEST,BRON&dateplantation__gte=2009-03-01&gid__in=6795,6798

Les opérateurs:
* `eq` : (equal) égalité
* `gt` : (greater than) strictement plus grand que
* `gte` : (greater than or equal) plus grand ou égal
* `lt` : (lesser than) strictement plus petit que
* `lte` : (lesser than or equal) plus petit ou égal
* `in` : (in) dans la liste, les éléments de la liste sont séparés par des virgules.


Services REST (en CSV)
----------------------

*Exemple* :
https://data.grandlyon.com/fr/datapusher/ws/grandlyon/gip_proprete.gipdecheterie/all.csv?maxfeatures=5&start=10

De la même façon que l'on requête le service JSON, on peut demander un extrait CSV en remplaçant l'extension ".json" de l'URL par ".csv".

Il est possible de remplacer le séparateur décimal en ajoutant 'ds=,' ou 'ds=.' dans la requête.

Le séparateur de colonne peut aussi être changé en utilisant l'option "separator=;" par exemple.

Un paramètre supplémentaire "geometry=on" (off par défaut) ajoute une colonne "WKT" contenant la géométrie de l'objet au format [WKT](https://fr.wikipedia.org/wiki/Well-known_text)

Export Shapefile
----------------
L'export shapefile est utilisable depuis le service WFS par l'utilisation du format SHAPEFILE (par exemple : https://data.grandlyon.com/geoserver/sytral/ows?SERVICE=WFS&VERSION=2.0.0&request=GetFeature&typename=sytral:tcl_sytral.tcllignebus_2_0_0&outputFormat=SHAPE-ZIP&SRSNAME=EPSG:4171&format_options=CHARSET:UTF-8&sortBy=gid). Cela renvoie alors à l'utilisateur un zip contenant un shapefile (SHP + SHX + DBF) de la couche. 


Service WMTS (Orthophotographies)
---------------------------------

Des services WMTS/WMS supplémentaires existent diffusant des flux d'orthophotographies.

Ils sont accessibles ici:

* https://imagerie.data.grandlyon.com/all/wmts?service=WMTS&request=getcapabilities
* https://imagerie.data.grandlyon.com/2154/wmts?service=WMTS&request=getcapabilities
* https://imagerie.data.grandlyon.com/3857/wmts?service=WMTS&request=getcapabilities
* https://imagerie.data.grandlyon.com/3946/wmts?service=WMTS&request=getcapabilities

Ces flux disposent d'un cache et sont à privilégier par rapport aux flux WMS disponibles sur https://data.grandlyon.com/geoserver/wms

Services OpenMapTiles
---------------------

Ce service propose des tuiles OpenMaptiles à utiliser pour des fonds de carte.

Client de démonstration: https://openmaptiles.data.grandlyon.com/data/v3/#8.37/45.796/4.592

Ces tuiles sont mises à jour de façon hebdomadaires en utilisant les données OpenStreetMap.

https://openmaptiles.data.grandlyon.com/data/v3/1/1/0.pbf

Ces tuiles peuvent être utilisées par les principaux frameworks SIG web (MaplibreGL, Leaflet,...) par exemple : https://openmaptiles.org/docs/website/maplibre-gl-js/



Services KML
------------
Le GrandLyon publie ses données au format KML. Les données sont accessibles via le service WFS et le format *kml* à l'url suivante : 
https://data.grandlyon.com/geoserver/ows?SERVICE=WFS&VERSION=2.0.0&request=GetFeature&typename=[organisation:][schema].[name]&outputFormat=kml&SRSNAME=EPSG:4171&sortBy=gid

*Exemple* : https://data.grandlyon.com/geoserver/ows?SERVICE=WFS&VERSION=2.0.0&request=GetFeature&typename=sytral:tcl_sytral.tcllignebus_2_0_0&outputFormat=kml&SRSNAME=EPSG:4171&sortBy=gid


Services MVT
------------

Les jeux de données vectoriels sont disponibles au format MVT via le protocole WMTS en choisissant le format *application/vnd.mapbox-vector-tile* (Mapbox Vector Tile)[https://docs.mapbox.com/vector-tiles/specification/]

https://data.grandlyon.com/geoserver/gwc/service/wmts?LAYERS=sytral:tcl_sytral.tcllignebus_2_0_0&SERVICE=WMTS&VERSION=1.0.0&REQUEST=GetTile&layer=sytral:tcl_sytral.tcllignebus_2_0_0&TILEMATRIX=EPSG:900913:10&TILEMATRIXSET=EPSG:900913&FORMAT=application/vnd.mapbox-vector-tile&TILECOL=525&TILEROW=365

Ce format est comparable au WFS mais est tuilé et les geométries sont simplifiées. Le but est d'être beaucoup plus rapide que le WFS en permettant en plus d'être mis en cache. En sortie on obtient une tuile encodée en utilisant le format (PBF)[https://fr.wikipedia.org/wiki/Protocol_Buffers]  (protobuf) (équivalent plus compact du JSON)

Ces tuiles peuvent être utilisées par les clients web comme MapboxGL, MapLibre ou OpenLayers.

QGis peut aussi lire ces tuiles en utilisant le plugin "Vector Tiles Reader". QGIS 3.20 gère le MVT.


Géocodeur Photon
---------------

Ce service permet d'effectuer des géocodages (conversion d'une adresse postale ou nom de lieu en coordonnées géographiques, `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=208bis%20rue%20garibaldi,%20lyon>`_) et des géocodages inversés (conversion de coordonnées géographiques en adresse postale ou nom de lieu, `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/reverse?lon=4.8459162&lat=45.6615222&limit=10&radius=0.1>`_).

Il est propulsé par l'outil libre Photon (cf. https://github.com/komoot/photon#photon-api) et exploite:

* les adresses de la Base d'Adresses Nationale (`BAN <https://adresse.data.gouv.fr>`_), sur le territoire de la Métropole de Lyon et les départements 01-38-69
* des points d'intérêt sur le territoire de la Métropole de Lyon, publiés au travers des données de `Data Grand Lyon <https://data.grandlyon.com>`_ (voir données concernées dans la section 'Points d'intérêt')

L'instance de photon (photon-bal) exposée par la Métropole de Lyon est disponible sur le endpoint

https://download.data.grandlyon.com/geocoding/photon-bal/

Le lien pour réaliser une requête de géocodage est le suivant (remplacer les .. par le lieu à géocoder):

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=...

Exemples :

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=lyon 
https://download.data.grandlyon.com/geocoding/photon-bal/api?q=%22Rue%20garibaldi%22

Les résultats sont retournés selon le type d'objets OSM (Open Street Map) et comportent des tags OSM (osm_key) portant des valeurs de tag (osm_value):

================= ======= =========== ====================== ==========================================================================================================
type d'objet      tag OSM valeur tag  afficher plutôt        exemple
================= ======= =========== ====================== ==========================================================================================================
Adresse           place   house       housenumer + street    `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=37%20rue%20du%20Repos,%20Lyon>`_
Voie              highway street      street                 `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=rue%20du%20Repos,%20Lyon>`_
Commune           place   city        city                   `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=Villeurbanne>`_

Pour les Points d'intérêt, voir le chapitre plus bas pour les tags spécifiques à ces données

A noter que la valeur de tag est également reprise dans l'attribut 'type' de la réponse de l'API.

Code INSEE de la commune
~~~~~~~~~~~~~~~~~~~~~~~~

Nativement, Photon ne renvoie que le code postal de la commune. Dans le fragment 'properties' de la réponse, le tag 'extra' -> 'insee' a été ajouté et contient le code insee de la commune. Ce tag 'insee' est une chaîne de caractères sur 5 caractères, afin de gérer l'Ain (01).

Dans l'exemple ci-dessous, la réponse de l'API contient extra":{"insee":"69383","metropole":"true"}

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=208bis%20rue%20garibaldi,%20lyon

Adresses Métropole/hors de la Métropole
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

L'instance photon permet d'interroger des adresses de la Métropole de Lyon et des départements limitrophes (01,38,69)

Afin de différencier ces deux types d'adresses (Métropole ou hors Métropole), un tag 'extra'->'metropole' a été ajouté dans le fragment 'properties' de la réponse. Ce tag 'metropole' est un booléen, qui prend la valeur 'true' (adresse Métropole) ou n'est pas présent (adresse hors Métropole)

Dans l'exemple suivant, sur la Métropole, la réponse de l'API contient "extra":{"insee":"69383","metropole":"true"}

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=208bis%20rue%20garibaldi,%20lyon

Dans l'exemple suivant hors Métropole, la réponse API ne contient pas le tag 'métropole' mais "extra":{"insee":"69264"}

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=316%20rue%20montesquieu,%20villefranche

Points d'intérêt
~~~~~~~~~~~~~~~~

L'instance photon contient majoritairement des adresses. Un besoin était de pouvoir localiser des points d'intérêt sur le territoire métropolitain (cela ne concerne donc pas les communes hors Métropole). Par exemple, pouvoir se localiser sur le parc Blandan.

En complément des adresses, des points d'intérêt ont donc été ajoutés, disposant de tags OSM (Open Street Map) spécifiques.

La recherche d'un point d'intérêt fait remonter des résultats avec des tags différents:

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=parc%20blandan

Selon le type du point d'intérêt, le résultat comporte un tag (osm_key) et une valeur de tag (osm_value) spécifiques, qui correspondent à ceux utilisés dans le cadre de l'usage de données Open Street Map (OSM). Voici ci-dessous la liste des types de points d'intérêt exploités, la source de ces données sur le portail data, les tags OSM associés et leurs valeurs:

========================================================================================================== ======= =============== ======================================================================================================================================
Point d'intérêt                                                                                            tag OSM valeur tag      exemple
========================================================================================================== ======= =============== ======================================================================================================================================
`Parcs et jardins <https://data.grandlyon.com/portail/fr/jeux-de-donnees/parcs-jardins-metropole-lyon>`_   leisure park            `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=parc%20blandan>`_
`Mairies <https://data.grandlyon.com/portail/fr/jeux-de-donnees/mairies-metropole-lyon-point-interet-v2>`_ amenity townhall        `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=mairie%20de%20bron>`_
`Maisons de la Métropole <https://data.grandlyon.com/portail/fr/jeux-de-donnees/maisons-metropole-lyon>`_  amenity social_facility `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=maison%20de%20la%20m%C3%A9tropole%20-%20givors>`_
`Piscines <https://data.grandlyon.com/portail/fr/jeux-de-donnees/piscines-metropole-lyon-point-interet>`_  leisure sports_centre   `exemple <https://download.data.grandlyon.com/geocoding/photon-bal/api?q=centre%20nautique%20tony>`_
========================================================================================================== ======= =============== ======================================================================================================================================

Concernant les points d'intérêt, le nom principal du point est contenu dans l'attribut 'name', mais il peut être utile d'afficher l'adresse et/ou la commune selon l'application.

Pour différencier les adresses des points d'intérêt, le tag 'extra' a été enrichi avec un tag 'extra'->'espace_public'. Ce tag 'espace_public' est un booléen, qui prend la valeur 'true' (si l'objet retourné est un point d'intérêt) ou n'est pas présent (si c'est une adresse ou une voie).

Oullins-Pierre-Bénite
~~~~~~~~~~~~~~~~~~~~~

Depuis le 1er janvier 2024, les communes d'Oullins et Pierre-Bénite ont fusionné pour former Oullins-Pierre-Bénite. Hors, ces communes présentent une centaine d'adresses en doublon, ce qui pose un souci de filtre des résultats.

Pour permettre le filtre des résultats de la requête localisés sur 'Oullins-Pierre-Bénite', un tag 'district' a été ajouté dans le fragment 'properties'. Il contient le nom de l'ancienne commune ('Oullins' ou 'Pierre-Bénite') et permet de filtrer les résultats. Le tag 'postcode' permet également de différencier les deux anciennes communes (69600 pour Oullins, 69310 pour Pierre-Bénite)

Par exemple:

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=14%20boulevard%20de%20l%27europe,%20pierre-b%C3%A9nite

contient le tag "district":"Pierre-Bénite" et le tag "postcode":"69310"

Biais de localisation
~~~~~~~~~~~~~~~~~~~~~

L'extension du volume des adresses nécessitera peut-être de centrer la recherche sur une zone (par ex, une recherche centrée sur Lyon pour prioriser les adresses sur la Métropole). Pour ce faire, il est possible d'utiliser le biais de localisation

La documentation Komoot décrit l'utilisation du biais de localisation (Location bias):

https://github.com/komoot/photon/?tab=readme-ov-file#search-with-location-bias

L''utilisation du biais permet de prioriser les adresses par rapport à leur proximité aux coordonnées fournies

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=rue%20lamartine

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=rue%20lamartine&lat=45.75&lon=4.84

Filtre par tags
~~~~~~~~~~~~~~~

Avec photon, il est possible de faire une recherche sur les tags OSM spécifiques. Par ex, je recherche la "Rue du Repos, Lyon" et je souhaite que seule la rue me soit retournée et pas les points d'adresse, je peux alors compléter ma requête avec le paramètre osm_tag:

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=rue%20du%20repos,%20lyon&osm_tag=highway:street

Il est donc possible d'interroger les points d'intérêt en utilisant les tags. Par ex, les résultats 'townhall' sur Bron me retourne les points d'intérêt qui répondent à ce tag:

https://download.data.grandlyon.com/geocoding/photon-bal/api?q=bron&osm_tag=amenity:townhall

Voir à ce titre la documentation de photon:

https://github.com/komoot/photon/?tab=readme-ov-file#filter-results-by-tags-and-values

A ce jour, photon ne permet pas de filtrer sur les extra tags. Cette évolution a été demandée par la communauté d'utilisateurs à Komoot.


Statistiques liées à un jeu de données
--------------------------------------

Pour interroger les statistiques, les requêtes prennent la forme suivante :
`https://data.grandlyon.com/statistiques/dataset?start=2023-02-5&uuid=4d59a6fd-f99f-47af-a0d3-8f21082a45fa&layername=sytral:tcl_sytral.tcllignebus_2_0_0&granularity=month&end=2024-02-5`

::

   [
     {"date": "2019-04-08", "service":"wms", "count":362},
     {"date": "2019-04-08", "service":"wfs", "count":123},
     {"date": "2019-04-08", "service":"ws", "count":12},
     {"date": "2019-04-08", "service":"kml", "count":2},
     {"date": "2019-04-15", "service":"wms", "count":364},
     {"date": "2019-04-15", "service":"wfs", "count":125},
     {"date": "2019-04-15", "service":"ws", "count":10},
     {"date": "2019-04-15", "service":"kml", "count":4},
     ...
     {"date": "2020-03-30", "service":"wms", "count":462},
     {"date": "2020-03-30", "service":"wfs", "count":223},
     {"date": "2020-03-30", "service":"ws", "count":22},
     {"date": "2020-03-30", "service":"kml", "count":50},
     {"date": "2020-04-06", "service":"wms", "count":202},
     {"date": "2020-04-06", "service":"wfs", "count":113},
     {"date": "2020-04-06", "service":"ws", "count":22},
     {"date": "2020-04-06", "service":"kml", "count":7},
   ]


Les paramètres demandés (tous insensibles à la casse):

* UUID l'uuid du jeu de données
* layername nom de la couche de donnée
* start : format YYYY-MM-DD la date de début, si la granularité est la semaine ou le mois et que la date est "dans" la semaine/mois, je prends le début de la semaine/mois
* end : format YYYY-MM-DD la date de fin, si la granularité est la semaine ou le mois et que la date est "dans" la semaine/mois, je prends la fin de semaine/mois
* granularity : day, week, year.

Pour la réponse :

Tableau de dictionnaire : un dictionnaire par élément de granularité (jour, semaine, mois). Le dictionnaire contient:

* date: format YYYY-MM-DD la date de début de l'élément de granularité
* count: le nombre de fois que la ressource a été vue, indépendamment du service (WMS/WFS...)
