.. _bonnespratiques:

=================
Bonnes Pratiques
=================

-----------
Principes
-----------

Que vous souhaitiez développer une application web ou embarquée dans un smartphone, vous avez sans doute le souci de garantir son bon fonctionnement et sa disponibilité. Voici quelques conseils destinés à vous faire utiliser notre plateforme de manière optimale.

------------------------------------
Accès direct aux données protégées
------------------------------------

Comme vous l'avez sans doute remarqué, certaines de nos données ne sont utilisables que si vous disposez d'un compte vous y autorisant. Dès lors que vous implémentez ces informations dans votre code, il vous appartient de veiller à leur sécurité, leur non-divulgation et leur facilité d'emploi tout au long de leur cycle de vie (mise à jour, suppression...)

En Javascript/Ajax
-------------------

Une des particularité du Javascript est d'être chargé et de s'exécuter côté client. Si vous intégrez vos identifiants dans votre code javascript, ils seront donc forcément accessibles à l'utilisateur de votre site. Employer ruses et contournements ne fera que rendre plus difficile leur récupération, mais à coup sûr ils apparaîtront tôt ou tard dans les requêtes que vous allez émettre vers nos services. Pour tout développement en Javascript/Ajax, il faut donc utiliser les technique de `Proxyfication`_ des requêtes décrites plus bas. 

En PHP, Python, Java
---------------------

Pas de problèmes avec ces languages qui s'exécutent côté serveur. Il suffit d'implémenter les exemples_ proposés.

Sur plateforme mobile
----------------------

Les développements effectués sur plateforme mobile sont relativement sûrs dans la mesure où ils sont compilés et sont donc distribués à vos utilisateurs sous une forme qui rend extrêmement difficile la récupération des informations qui s'y trouvent. Par contre, il vous faut considérer un aspect pratique lié au cycle de vie de vos identifiant. Pour peu que vous changiez de mot de passe, ou même de compte, comment va se passer la diffusion du nouveau couple login/mot de passe sur les terminaux de vos utilisateurs ? Devront-ils faire une mise à jour ? Pourrez-vous provoquer cette mise à jour obligatoire ? Cette réflexion devra avoir été menée avant de diffuser votre solution. 

Performances
--------------

Une autre question importante est celle des performances de notre plateforme. Elle n'est pas dimensionnée pour tenir la même charge que www.google.fr. Si votre application a du succès, si un tweet la propulse vers des sommets de popularité très rapidement, êtes vous sûrs que nos infrastructures tiendront la charge et permettront à vos utilisateurs d'avoir une bonne expérience avec votre application ? 

Conclusion
--------------

Pour chacune des situations ci-dessus, il vous faut veiller à la mise en oeuvre des pratiques conseillées. Le plus simple est parfois de récupérer directement la donnée.  


-----------------------------------
Récupération des données
-----------------------------------

Vos droits d'accès vous permettent d'accéder à la donnée et de la stocker sur vos serveurs. Leur fréquence de récupération doit être adaptée à leur fréquence de mise à jour. Une fois intégrée à votre infrastructure, la résolution des problèmes évoqués précédemment est beaucoup plus simple :

* Sur plateforme mobile, les utilisateurs accèdent directement à votre version des données. Si vos login/mot de passe change, cela affecte votre script de récupération des données uniquement et non l'application déployée chez les clients, qui de son côté peut utiliser un système d'authentification/autorisation qui vous est propre. 

* Pour les performances : la récupération ponctuelle, même si elle est fréquente, affecte peu les performances de notre plateforme. Pour peu que votre infrastructure soit dimensionnée pour absorber la charge générée par votre application, vous n'aurez pas de problème à craindre de notre côté en vous y prenant ainsi.

-----------------------------------
Proxyfication
-----------------------------------

Que les données soient installées au sein de votre infrastructure ou que votre application accède directement aux données sur la nôtre, il est des cas d'utilisation ou le recours à une proxyfication va néanmoins être indispensable. C'est surtout le cas en environnement Javascript avec lequel vous avez deux obstacles :

* L'impossiblité de cacher les login/mot de passe à transmettre pour accéder aux données. Vous pouvez néanmoins avoir mis en oeuvre, après avoir récupéré les données, un mécanisme d'authentification qui est propre à vos utilisateurs, auquel cas vous réglez une partie du problème

* L'accès à un domaine tiers en Ajax. Ceci est strictement interdit par "Same Origine Policy" qui impose au contenu appelé par une requête Ajax d'être situé sur le même domaine et le même port que la page principale dans laquelle se situe la requête. Cette règle ne vaut pas pour les images, ce qui explique que l'on puisse utiliser les flux WMS externes directement, mais vaut pour tous nos autres types de services : WFS, JSON ou KML. 

Pour contourner ces restrictions, vous pouvez employer l'astuce de la proxyfication. Le principe est de transmettre les requêtes AJAX à un script situé sur votre propre serveur (donc mêmes domaine et port que la page principale), qui transmettra les requêtes au service externe, sans être soumis aux mêmes contraintes d'un client web. Cela poeut également vous servir pour distinguer les comptes d'accès à votre serveur de celui utilisé pour vous connecter à notre infrastructure. 


     CLIENT WEB ------> SCRIPT PROXY ------> SERVICE EXTERNE
	  user:toto.........user:my_account......>
	  
	  
Implémenter un script de proxyfication n'est pas très compliqué, mais il faut respecter certaines règles. Ne pas le laisser ouvert à tous vents par exemple, car sinon n'importe qui pourrait venir s'en servir pour naviguer sur internet de manière anonyme. Ne pas le laisser faire n'importe quoi non plus, afin qu'un utilisateur ne puisse pas envoyer n'importe quelle requête à votre serveur. 

Implémentation en Python
---------------------------

Le script que nous allons utiliser ici est fourni avec OpenLayers sous le nom de proxy.cgi.

Il s'agit d'un script écrit en Python, parfaitement adapté au WFS. Pour l'utiliser, il faut le déposer dans un répertoire exécutable du serveur web (généralement le cgi-bin), en régler les droits pour le rendre exécutable (chmod a+x proxy.cgi par exemple) et indiquer à OpenLayers sa présence pour qu'il l'utilise (car OpenLayers est malin mais pas devin...) avec la directive :

.. code-block:: javascript

  OpenLayers.ProxyHost = "/cgi-bin/proxy.cgi?url=";
  
Pour toute information complémentaire concernant OpenLayers et un Proxy, veuillez vous référer à la page de FAQ http://trac.osgeo.org/openlayers/wiki/FrequentlyAskedQuestions#ProxyHost


.. code-block:: python

	#!/usr/local/bin/python


	"""This is a blind proxy that we use to get around browser
	restrictions that prevent the Javascript from loading pages not on the
	same server as the Javascript.  This has several problems: it's less
	efficient, it might break some sites, and it's a security risk because
	people can use this proxy to browse the web and possibly do bad stuff
	with it.  It only loads pages via http and https, but it can load any
	content type. It supports GET and POST requests."""

	import urllib2
	import cgi
	import sys, os

	# Designed to prevent Open Proxy type stuff.
	# replace 'my_target_server' by the external domain you are aiming to
	allowedHosts = ['localhost','my_target_server']

	method = os.environ["REQUEST_METHOD"]

	if method == "POST":
	    qs = os.environ["QUERY_STRING"]
	    d = cgi.parse_qs(qs)
	
		# checks if a url parameter exists in the POST request. If not, go to hell.
	    if d.has_key("url"):
	        url = d["url"][0]
	    else:
	        url = "http://www.openlayers.org"
	else:
	    fs = cgi.FieldStorage()
		# checks if a url parameter exists in the GET request. If not, go to hell.
	    url = fs.getvalue('url', "http://www.openlayers.org")

	try:
	    host = url.split("/")[2]
	
		# reply with HTTP 502 code if the host is not allowed
	    if allowedHosts and not host in allowedHosts:
	        print "Status: 502 Bad Gateway"
	        print "Content-Type: text/plain"
	        print
	        print "This proxy does not allow you to access that location (%s)." % (host,)
	        print
	        print os.environ
	    # checks if the request is a http or https request  
	    elif url.startswith("http://") or url.startswith("https://"):
    
	        if method == "POST":
	            length = int(os.environ["CONTENT_LENGTH"])
	            headers = {"Content-Type": os.environ["CONTENT_TYPE"]}
	            body = sys.stdin.read(length)
	            r = urllib2.Request(url, body, headers)
	            y = urllib2.urlopen(r)
	        else:
	            y = urllib2.urlopen(url)
        
	        # print content type header
	        i = y.info()
	        if i.has_key("Content-Type"):
	            print "Content-Type: %s" % (i["Content-Type"])
	        else:
	            print "Content-Type: text/plain"
	        print
        
	        print y.read()
        
	        y.close()
	    else:
	        print "Content-Type: text/plain"
	        print
	        print "Illegal request."

	except Exception, E:
	    print "Status: 500 Unexpected Error"
	    print "Content-Type: text/plain"
	    print 
	    print "Some unexpected error occurred. Error text was:", E
	
	

Ce script PHP fait la même chose : 

.. code-block:: php

    <?php
		/*
		License: LGPL as per: http://www.gnu.org/copyleft/lesser.html
		$Id: proxy.php 3650 2007-11-28 00:26:06Z rdewit $
		$Name$
		*/

		////////////////////////////////////////////////////////////////////////////////
		// Description:
		// Script to redirect the request http://host/proxy.php?url=http://someUrl
		// to http://someUrl .
		//
		// This script can be used to circumvent javascript's security requirements
		// which prevent a URL from an external web site being called.
		//
		// Author: Nedjo Rogers
		////////////////////////////////////////////////////////////////////////////////

		// define alowed hosts
		$aAllowedDomains = array('localhost','my_target_server')

		// read in the variables

		if(array_key_exists('HTTP_SERVERURL', $_SERVER)){
			$onlineresource=$_SERVER['HTTP_SERVERURL'];
		}else{
			$onlineresource=$_REQUEST['url'];
		}
		$parsed = parse_url($onlineresource);
		$host = @$parsed["host"];
		$path = @$parsed["path"] . "?" . @$parsed["query"];
		if(empty($host)) {
			$host = "localhost";
		}

		if(is_array($aAllowedDomains)) {
			if(!in_array($host, $aAllowedDomains)) {
				die("le domaine '$host' n'est pas autorisé. contactez l'administrateur.");
			}
		}

		$port = @$parsed['port'];
		if(empty($port)){
			$port="80";
		}
		$contenttype = @$_REQUEST['contenttype'];
		if(empty($contenttype)) {
			$contenttype = "text/html; charset=ISO-8859-1";
		}
		$data = @$GLOBALS["HTTP_RAW_POST_DATA"];
		// define content type
		header("Content-type: " . $contenttype);

		if(empty($data)) {
			$result = send_request();
		}
		else {
			// post XML
			$posting = new HTTP_Client($host, $port, $data);
			$posting->set_path($path);
			echo $result = $posting->send_request();
		}

		// strip leading text from result and output result
		$len=strlen($result);
		$pos = strpos($result, "<");
		if($pos > 1) {
			$result = substr($result, $pos, $len);
		}
		//$result = str_replace("xlink:","",$result);
		echo $result;

		// define class with functions to open socket and post XML
		// from http://www.phpbuilder.com/annotate/message.php3?id=1013274 by Richard Hundt

		class HTTP_Client {
			var $host;
			var $path;
			var $port;
			var $data;
			var $socket;
			var $errno;
			var $errstr;
			var $timeout;
			var $buf;
			var $result;
			var $agent_name = "MyAgent";
			//Constructor, timeout 30s
			function HTTP_Client($host, $port, $data, $timeout = 30) {
				$this->host = $host;
				$this->port = $port;
				$this->data = $data;
				$this->timeout = $timeout;
			}

			//Opens a connection
			function connect() {
				$this->socket = fsockopen($this->host,
				$this->port,
				$this->errno,
				$this->errstr,
				$this->timeout
			);
			if(!$this->socket)
				return false;
			else
				return true;
			}

			//Set the path
			function set_path($path) {
				$this->path = $path;
			}

			//Send request and clean up
			function send_request() {
				if(!$this->connect()) {
					return false;
				}
				else {
					$this->result = $this->request($this->data);
					return $this->result;
				}
			}

			function request($data) {
				$this->buf = "";
				fwrite($this->socket,
				"POST $this->path HTTP/1.0\r\n".
				"Host:$this->host\r\n".
				"Basic: ".base64_encode("guillaume:catch22")."\r\n".
				"User-Agent: $this->agent_name\r\n".
				"Content-Type: application/xml\r\n".
				"Content-Length: ".strlen($data).
				"\r\n".
				"\r\n".$data.
				"\r\n"
			);

			while(!feof($this->socket))
				$this->buf .= fgets($this->socket, 2048);
				$this->close();
				return $this->buf;
			}


			function close() {
				fclose($this->socket);
			}
		}



		function send_request() {
			global $onlineresource;
			$ch = curl_init();
			$timeout = 5; // set to zero for no timeout

			// fix to allow HTTPS connections with incorrect certificates
			curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
			curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 1);

			//curl_setopt($ch, CURLOPT_USERPWD, 'guillaume:catch22');
			//curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);

			curl_setopt($ch, CURLOPT_URL,$onlineresource);
			curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
			curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, $timeout);
			curl_setopt($ch, CURLOPT_ENCODING , "gzip, deflate");

			if( ! $file_contents = curl_exec($ch)){
				trigger_error(curl_error($ch));
			}
			curl_close($ch);
			$lines = array();
			$lines = explode("\n", $file_contents);
			if(!($response = $lines)) {
				echo "Unable to retrieve file '$service_request'";
			}
			$response = implode("",$response);
			return utf8_decode($response);
		}
	?> 


---------------------
Gestion des horaires
---------------------

Historiquement, les données publiées sur data.grandlyon.com présentent des horaires de manière hétérogènes:

* dans l’attribut ‘openinghours’ au format `openinghours <https://schema.org/openingHours>`_, s’il existe dans la table
* dans l’attribut ‘openinghoursspecification’ au format `openinghoursspecification <https://schema.org/OpeningHoursSpecification>`_, s’il existe dans la table

Afin d’uniformiser et de simplifier la prise en compte des horaires, la Métropole a ajouté un nouvel attribut ‘horaires’ au format `openinghours OSM <https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification>`_

   La Métropole encourage à passer au format `openinghours OSM <https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification>`_ dès que possible.

La Métropole présente des horaires sur l’année N et l’année N+1. La définition du statut ouvert/fermé au moment actuel doit être faite au niveau du client réutilisateur.

Documentation
-------------

Voici les éléments de documentation concernant le format openinghours OSM:

* `version française <https://wiki.openstreetmap.org/wiki/FR:Key:opening_hours>`_
* `spécification anglaise <https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification>`_ (plus complète)
* `outil d’évaluation <https://openingh.openstreetmap.de/evaluation_tool/?lng=fr>`_
* `outil de saisie et d’évaluation <https://projets.pavie.info/yohours/>`_

Usage Métropole par rapport à la documentation
----------------------------------------------

Afin de simplifier la prise en compte des horaires, la Métropole n’exploite pas toutes les possibilités du format `openinghours OSM <https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification>`_

Présentation des horaires
~~~~~~~~~~~~~~~~~~~~~~~~~

Les horaires présentés sont découpés en fragments, séparés par des points-virgules. Chaque fragment représente les horaires d’une période ou d’un jour donné. La Métropole présente toujours les fragments dans le même ordre selon leur type:

* horaires “par défaut”
* horaires des plages de dates spécifiques (par ex, plages de vacances ou période de fermeture), présentées chronologiquement
* horaires des dates spécifiques (par ex, jours fériés ou journée de travaux), présentées chronologiquement

   Conformément au format, chaque fragment prend la priorité sur les fragments précédents. Ainsi, les horaires d’un jour férié sont une exception aux horaires par défaut et sont donc prioritaires dans la prise en compte.

Ci-dessous une reformulation des éléments avec des exemples.

Le premier fragment d’horaire correspond aux horaires par défaut (par ex, ``Mo-Fr 09:00-12:00,14:00-17:00, Sa 09:00-17:00``). Il ne comporte pas de date ou de plage de date. Il correspond en valeurs aux horaires présentés dans l’attribut openinghours.

Dans l’ordre de dates croissante, sont ensuite présentées les plages des dates spécifiques (par ex, ``2023 Apr 01-2023 Oct 30 Mo-Fr 08:30-12:00,13:30-18:00, Sa 08:30-18:30``) 

Puis suivent les jours spécifiques, classés également par date croissante ``2023 Apr 10 closed "Lundi de Pâques"; 2023 May 01 closed "1er mai"``

Tout fragment peut être commenté. Les noms des jours fériés et des vacances scolaires sont automatiquement indiqués en commentaire en fin de fragment.

Simplification dans la prise en compte
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Le format `openinghours OSM <https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification>`_ permet beaucoup de variété dans les saisies des horaires. La Métropole a limité son utilisation afin de faciliter la réutilisation et standardiser sa présentation.

La Métropole est restée dans une description simple des horaires, comme dans les exemples du chapitre précédent:

   <plages de date>|<date> <horaires> <comment>

Les mots-clés utilisés sont limités à: ‘24/7’ ou ‘24/7 open’ (pas de ‘24/7 closed’ qui se traduira par une absence d’horaires: ‘horaires’ = NULL)

Les éléments suivants **ne** sont **pas** utilisés et feront l’objet d’une information s’ils le sont:

* les <rule_modifier> ‘off’ et ‘unknown’ ne sont pas utilisés
* si le <selector_sequence> est ‘24/7’, le <rule_modifier> ‘closed’ n’est pas utilisé. L’horaire ne sera pas présenté
* pour l’élément <timespan>, seul le <time> est utilisé (pas de <extended_time>, <variable_time> ou <event>)
* pour l’élément <weekday_selector>, seul le <weekday_sequence_> est utilisé (pas de <holiday_sequence>; les mots-clés ‘PH’, ‘SH’, ‘easter’ sont remplacés par des plages datées)
* pour l’élément <weekday_range>, le <day_offset> n’est pas utilisé
* l’élément <week_selector> n’est pas utilisé
* pour l’élément <monthday_range>, le <date_offset> n’est pas utilisé
* pour l’élément <date_from>, le <variable_date> n’est pas utilisé
* pour l’élément <year_selector>, seul <year_range> = <year> est utilisé et l’année est toujours présenté avec au moins un mois

Cas les plus complexes
~~~~~~~~~~~~~~~~~~~~~~

Quelques cas font apparaitre un <wday>[<nth_entry>] (par ex, Tu[1] ou Sa[-1] dans les données mairies)

Il est possible (quoique très rare) qu’une même plage de dates apparaissent plusieurs fois, en raison d’une règle de substitution à une règle précédente:

``2023 Feb 04-2023 Feb 19 Mo-Fr 08:30-12:30,13:30-16:45 "Vacances d'Hiver"; 2023 Feb 04-2023 Feb 19 Tu[1] 08:30-12:00,14:30-16:45 "Vacances d'Hiver"``

Le cas présenté ci-dessus n’est plus d’actualité, mais peut exister

Cette complexité peut entraîner des différences entre l’attribut ‘horaires’ et l’attribut ‘openinghoursspecification’. Ces horaires sont trop complexes pour être traités avec le process actuel de la Métropole. Ils sont alors simplifiés. Par ex, la valeur suivante ``Sa 09:00-11:45; Sa[-1] closed`` sera retranscrite en ``Sa 09:00-11:45`` dans l’attribut ‘openinghoursspecification’ des données de la Métropole.

   La Métropole encourage à passer au format `openinghours OSM <https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification>`_ dès que possible.

Exemples d’horaires
-------------------

déchèteries
~~~~~~~~~~~

=========== ================ =============== ============================================================= ========== ========
identifiant nom              type_decheterie horaires                                                      date_debut date_fin
=========== ================ =============== ============================================================= ========== ========
1           Caluire-et-Cuire Classique       Lu-Ve 09:00-12:00,14:00-17:00, Sa 09:00-17:00, Di 09:00-12:00
1           Caluire-et-Cuire Classique       Lu-Ve 08:30-12:00,13:30-18:00, Sa 08:30-18:30, Di 09:00-12:00 04-01      10-30
1           Caluire-et-Cuire Classique                                                                     Fériés
1           Caluire-et-Cuire Classique       09:00-12:00,14:00-16:00                                       12-24
1           Caluire-et-Cuire Classique       09:00-12:00,14:00-16:00                                       12-31
20          Lyon 5           Fluviale        Sa 10:00-16:00
20          Lyon 5           Fluviale        Sa 09:00-18:00                                                04-01      10-30
21          Lyon 1           Mobile          Ve 14:00-20:00                                                2022-09-01
21          Lyon 1           Mobile          Ve 14:00-20:00                                                2022-10-07
21          Lyon 1           Mobile          Ve 14:00-20:00                                                2022-11-04
21          Lyon 1           Mobile          Ve 14:00-20:00                                                2022-12-02
=========== ================ =============== ============================================================= ========== ========

résultat en openinghours OSM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pour la déchèterie 1 (Caluire)

``Mo-Fr 09:00-12:00,14:00-17:00, Sa 09:00-17:00, Su 09:00-12:00; 2023 Apr 01-2023 Oct 30 Mo-Fr 08:30-12:00,13:30-18:00, Sa 08:30-18:30, Su 09:00-12:00; 2024 Apr 01-2024 Oct 30 Mo-Fr 08:30-12:00,13:30-18:00, Sa 08:30-18:30, Su 09:00-12:00; 2023 Apr 10 closed "Lundi de Pâques"; 2023 May 01 closed "1er mai"; 2023 May 08 closed "8 mai"; 2023 May 18 closed "Ascension"; 2023 May 29 closed "Lundi de Pentecôte"; 2023 Jul 14 closed "14 juillet"; 2023 Aug 15 closed "Assomption"; 2023 Nov 01 closed "Toussaint"; 2023 Nov 11 closed "11 novembre"; 2023 Dec 24 Su 09:00-12:00,14:00-16:00; 2023 Dec 25 closed "Jour de Noël"; 2023 Dec 31 Su 09:00-12:00,14:00-16:00; 2024 Jan 01 closed "1er janvier"; 2024 Apr 01 closed "Lundi de Pâques"; 2024 May 01 closed "1er mai"; 2024 May 08 closed "8 mai"; 2024 May 09 closed "Ascension"; 2024 May 20 closed "Lundi de Pentecôte"; 2024 Jul 14 closed "14 juillet"; 2024 Aug 15 closed "Assomption"; 2024 Nov 01 closed "Toussaint"; 2024 Nov 11 closed "11 novembre"; 2024 Dec 24 Tu 09:00-12:00,14:00-16:00; 2024 Dec 25 closed "Jour de Noël"; 2024 Dec 31 Tu 09:00-12:00,14:00-16:00``

où

le premier fragment constitue les openingshours

``Mo-Fr 09:00-12:00,14:00-17:00, Sa 09:00-17:00, Su 09:00-12:00``

suivent les plages de dates, classées par date croissante

``2023 Apr 01-2023 Oct 30 Mo-Fr 08:30-12:00,13:30-18:00, Sa 08:30-18:30, Su 09:00-12:00; 2024 Apr 01-2024 Oct 30 Mo-Fr 08:30-12:00,13:30-18:00, Sa 08:30-18:30, Su 09:00-12:00``

suivent les jours spécifiques, classés par date croissante

``2023 Apr 10 closed "Lundi de Pâques"; 2023 May 01 closed "1er mai"; 2023 May 08 closed "8 mai"; 2023 May 18 closed "Ascension"; 2023 May 29 closed "Lundi de Pentecôte"; 2023 Jul 14 closed "14 juillet"; 2023 Aug 15 closed "Assomption"; 2023 Nov 01 closed "Toussaint"; 2023 Nov 11 closed "11 novembre"; 2023 Dec 24 Su 09:00-12:00,14:00-16:00; 2023 Dec 25 closed "Jour de Noël"; 2023 Dec 31 Su 09:00-12:00,14:00-16:00; 2024 Jan 01 closed "1er janvier"; 2024 Apr 01 closed "Lundi de Pâques"; 2024 May 01 closed "1er mai"; 2024 May 08 closed "8 mai"; 2024 May 09 closed "Ascension"; 2024 May 20 closed "Lundi de Pentecôte"; 2024 Jul 14 closed "14 juillet"; 2024 Aug 15 closed "Assomption"; 2024 Nov 01 closed "Toussaint"; 2024 Nov 11 closed "11 novembre"; 2024 Dec 24 Tu 09:00-12:00,14:00-16:00; 2024 Dec 25 closed "Jour de Noël"; 2024 Dec 31 Tu 09:00-12:00,14:00-16:00``

à noter la présence des horaires du 24/12 et du 31/12 classés parmi les jours fériés

mairies
~~~~~~~

=========== ========================== ========================================================================================================= ======================== ========
identifiant nom                        horaires                                                                                                  date_debut               date_fin
=========== ========================== ========================================================================================================= ======================== ========
S1324       Mairie d'Albigny-Sur-Saône Lu-Fr 08:30-12:30, Lu,Me 13:30-17:30, Ma,Je 08:30-12:30 "sur rendez-vous uniquement", Sa 09:00-12:00
S1324       Mairie d'Albigny-Sur-Saône Lu-Fr 08:30-12:30, Lu,Me 13:30-17:30, Ma,Je 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30      Vacances de la Toussaint
S1324       Mairie d'Albigny-Sur-Saône Lu-Fr 08:30-12:30, Lu,Me 13:30-17:30, Ma,Je 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30      Vacances de Noël
S1324       Mairie d'Albigny-Sur-Saône Lu-Fr 08:30-12:30, Lu,Me 13:30-17:30, Ma,Je 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30      Vacances d'Hiver
S1324       Mairie d'Albigny-Sur-Saône Lu-Fr 08:30-12:30, Lu,Me 13:30-17:30, Ma,Je 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30      Vacances de Printemps
S1324       Mairie d'Albigny-Sur-Saône                                                                                                           Fériés
S1326       Mairie de Bron             Lu-Ve 08:30-12:00, Lu,Me,Je,Ve 13:30-17:00, Ma 13:30-18:30
S1326       Mairie de Bron                                                                                                                       Fériés
S1376       Mairie de La Mulatière     Lu-Ve 08:45-12:30, Ma,Je,Ve 13:30-17:00, Me 13:30-16:30, Sa[1,3] 09:00-11:45 "uniquement sur rendez-vous"
S1376       Mairie de La Mulatière                                                                                                               Fériés
S1392       Mairie de Montanay         Lu-Ve 08:30-12:00, Lu,Ma,Je,Ve 15:00-17:00, Sa 09:00-11:45, Sa[-1] closed
S1392       Mairie de Montanay                                                                                                                   Fériés
S1392       Mairie de Montanay                                                                                                                   Vacances
=========== ========================== ========================================================================================================= ======================== ========

résultat en openinghours OSM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pour la mairie S1324 (Albigny)

``Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Sa 09:00-12:00; 2023 Feb 04-2023 Feb 19 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances d'Hiver"; 2023 Apr 08-2023 Apr 23 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Printemps"; 2023 Oct 21-2023 Nov 05 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de la Toussaint"; 2023 Dec 23-2024 Jan 07 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Noël"; 2024 Feb 17-2024 Mar 03 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances d'Hiver"; 2024 Apr 13-2024 Apr 28 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Printemps"; 2024 Oct 19-2024 Nov 03 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de la Toussaint"; 2024 Dec 21-2025 Jan 05 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Noël"; 2023 Apr 10 closed "Lundi de Pâques"; 2023 May 01 closed "1er mai"; 2023 May 08 closed "8 mai"; 2023 May 18 closed "Ascension"; 2023 May 29 closed "Lundi de Pentecôte"; 2023 Jul 14 closed "14 juillet"; 2023 Aug 15 closed "Assomption"; 2023 Nov 01 closed "Toussaint"; 2023 Nov 11 closed "11 novembre"; 2023 Dec 25 closed "Jour de Noël"; 2024 Jan 01 closed "1er janvier"; 2024 Apr 01 closed "Lundi de Pâques"; 2024 May 01 closed "1er mai"; 2024 May 08 closed "8 mai"; 2024 May 09 closed "Ascension"; 2024 May 20 closed "Lundi de Pentecôte"; 2024 Jul 14 closed "14 juillet"; 2024 Aug 15 closed "Assomption"; 2024 Nov 01 closed "Toussaint"; 2024 Nov 11 closed "11 novembre"; 2024 Dec 25 closed "Jour de Noël"``

où

le premier fragment constitue les openingshours

``Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Sa 09:00-12:00``

suivent les plages de dates, classées par date croissante

``2023 Feb 04-2023 Feb 19 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances d'Hiver"; 2023 Apr 08-2023 Apr 23 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Printemps"; 2023 Oct 21-2023 Nov 05 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de la Toussaint"; 2023 Dec 23-2024 Jan 07 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Noël"; 2024 Feb 17-2024 Mar 03 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances d'Hiver"; 2024 Apr 13-2024 Apr 28 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Printemps"; 2024 Oct 19-2024 Nov 03 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de la Toussaint"; 2024 Dec 21-2025 Jan 05 Mo-Fr 08:30-12:30, Mo,We 13:30-17:30, Tu,Th 08:30-12:30 "sur rendez-vous uniquement", Fr 13:30-16:30 "Vacances de Noël"``

suivent les jours spécifiques, classés par date croissante

``2023 Apr 10 closed "Lundi de Pâques"; 2023 May 01 closed "1er mai"; 2023 May 08 closed "8 mai"; 2023 May 18 closed "Ascension"; 2023 May 29 closed "Lundi de Pentecôte"; 2023 Jul 14 closed "14 juillet"; 2023 Aug 15 closed "Assomption"; 2023 Nov 01 closed "Toussaint"; 2023 Nov 11 closed "11 novembre"; 2023 Dec 25 closed "Jour de Noël"; 2024 Jan 01 closed "1er janvier"; 2024 Apr 01 closed "Lundi de Pâques"; 2024 May 01 closed "1er mai"; 2024 May 08 closed "8 mai"; 2024 May 09 closed "Ascension"; 2024 May 20 closed "Lundi de Pentecôte"; 2024 Jul 14 closed "14 juillet"; 2024 Aug 15 closed "Assomption"; 2024 Nov 01 closed "Toussaint"; 2024 Nov 11 closed "11 novembre"; 2024 Dec 25 closed "Jour de Noël"``

------------
Synthèse
------------

Comme nous l'avons vu, il y a différentes stratégies à utiliser selon les flux que vous utilisez et la plateforme pour laquelle vous développez. Faire du WMS dans une application web sera plus simple que traiter du WFS volumineux dans une application iPhone. On peut cependant distinguer les approches les plus intéressantes :

* Pour de l'image simple, sans authentification, utilisez un flux direct vers notre plateforme
* pour les gros volumes texte (WFS, JSON...) récupérez la donnée à intervalles réguliers et servez là depuis votre serveur. Ca peut aussi vous éviter le recours à un script proxy.
* Pour les applications nomades sur Smartphone, privilégiez l'autonomie de l'application par rapport aux modalités d'accès aux données. Récupérez les données, et implémentez un service listant les données disponibles, de sorte que vous pourrez intégrer de nouvelles couches de données à votre application sans la mettre à jour. 



	 
