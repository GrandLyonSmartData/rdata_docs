.. _authentification:

Authentification
=================

Principes
-------------------

Certaines des données publiées par les services Data nécessitent une autorisation pour pouvoir y accéder par API (pour les charger dans un SIG comme QGIS par exemple). 
Afin d'en obtenir une, vous devez ouvrir un compte sur GrandLyon Connect https://moncompte.grandlyon.com/login/ pour vous connecter sur la plateforme data et ensuite définir votre mot de passe pour la plateforme data en utilisant la fonction 'changer mon mot de passe' sur la page profil de votre compte https://data.grandlyon.com/onegeo-login/fr/profile/.

Attention les mots de passe de votre compte GrandLyon Connect et celui que vous définissez pour la plateforme peuvent et devraient être différents.

Une fois ces opération réalisées, vous aurez un identifiant (généralement l'adresse email utilisée lors de la création du compte) et un mot de passe. Ceux-ci vous sont personnels, et leur utilisation dans le contexte du développement d'application pour les tiers doit donc être fait avec certaines précautions. 

Mais voyons d'abord comment utiliser ces éléments d'identification pour accéder à des données protégées. 

La méthode d'authentification utilisée est le `Basic Auth HTTP <http://fr.wikipedia.org/wiki/Authentification_HTTP#M.C3.A9thode_Basic>`_ Lors de l'accès à chacun des types de services, il est donc nécessaire d'utiliser le header HTTP nommé 'Authorization' dans lequel seront insérés login et mot de passe, séparés par deux points (":") et encodé en `base64 <http://fr.wikipedia.org/wiki/Base64>`_. Le header ressemble alors à ceci :

::

  Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
 
Mais ne vous y méprenez pas. L'encodage en base64 n'est pas un cryptage ! Le procédé est réversible et on peut donc retrouver les valeurs encodés très facilement. Couplé à un flux HTTPS, qui crypte les informations transmises par et vers le serveur, ils ne sont pas récupérables, mais écrits tels quels dans le code ils le sont. 


Exemple avec cURL et WGET
--------------------------

L'utilisation du header authorization avec cURL est très simple. Imaginons un utilisateur doté des identifiants suivants :

* login : demo
* password : demo4dev

L'instruction cURL à utiliser pour accéder à la donnée "demo.demovelov" sur le service data serait alors :

::

    cURL -u demo:demo4dev curl https://data.grandlyon.com/fr/datapusher/ws/rdata/tcl_sytral.tcllignebus_2_0_0/all.json?compact=false

sauf erreur, vous devriez alors recevoir un flux json. 

L'instruction WGET à utiliser est comparable : 

:: 

    wget --http-user=demo --http-password=demo4dev https://data.grandlyon.com/fr/datapusher/ws/rdata/tcl_sytral.tcllignebus_2_0_0/all.json?compact=false
 

Exemples avec PHP ou Python
---------------------------

Que ce soit en PHP ou en Python, les librairies utilisées pour émettre des requêtes HTTP intègrent toutes les possibilité de déclarer le Header Authorization.

Pour Python et urllib2 nous aurons :

.. code-block:: python

    import urllib2, base64
    
    # set basic information
    username = 'demo'
    password = 'demo4dev'
    url = 'https://data.grandlyon.com/fr/datapusher/ws/rdata/tcl_sytral.tcllignebus_2_0_0/all.json?compact=false'
    
    # prepare the request Object
    request = urllib2.Request(url)
    
    # encode the username / password couple into a single base64 string
    base64string = base64.encodestring('%s:%s' % (username, password)))
    
    # then add this string into the Authorization header
    request.add_header("Authorization", "Basic %s" % base64string)
    
    # and open the url
    result = urllib2.urlopen(request)
    
    # then handle the result the way you like

En PHP, nous utiliserons la librairie cURL intégrée :

.. code-block:: php

    <?php

    // set basic information
    $username='demo';
    $password='demo4dev';
    $URL='https://data.grandlyon.com/fr/datapusher/ws/rdata/tcl_sytral.tcllignebus_2_0_0/all.json?compact=false';
    
    // instantiate a new cUrl object
    $ch = curl_init();
    
    // Everything is an option with PHPCurl !
    curl_setopt($ch, CURLOPT_URL,$URL);
    
    // set RETURNTRANSFER to true to be able to handle the result
    curl_setopt($ch, CURLOPT_RETURNTRANSFER,1);
    
    // set this option for enabling Basic Auth HTTP rules
    curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
    
    // the previous setting will help here to encode the username/password into the correct format
    curl_setopt($ch, CURLOPT_USERPWD, "$username:$password");
    
    // and lift off...
    $result=curl_exec ($ch);
    
    // then handle the result the way you like
    
    ?>

