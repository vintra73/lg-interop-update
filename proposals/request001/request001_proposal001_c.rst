**state**: inWorking


[ID_AUTH_REST_02] Direct Trust con materiale crittografico su REST e con unicità del token/messaggio
============================================================================================

Il seguente profilo estende il profilo ID_AUTH_REST_01. Comunicazione
tra fruitore ed erogatore che assicuri a livello di messaggio:

-  accesso del soggetto fruitore, quale organizzazione o unità
   organizzativa fruitore, o entrambe le parti

-  la difesa dalle minacce derivanti dagli attacchi: Replay Attack
   quando il JWT o il messaggio NON DEVONO essere riprocessati.

Descrizione
-------------

Il presente profilo declina l’utilizzo di:

-  JSON Web Token (JWT) definita dall’:rfc:`7519`

-  JSON Web Signature (JWS) definita dall’:rfc:`7515`

L'erogatore e il fruitore DEVONO utilizzare la Piattaforma Digitale 
Nazionale Dati per l’interoperabilità di cui al comma 2 dell'articolo 
50-ter del CAD per la costituzione del trust, tramite il materiale crittografico 
depositato applicando i profili di emissione dei voucher previsti dalla stessa.

La costituzione del trust tra fruitore ed erogatore PUÒ essere realizzata al di fuori della Piattaforma Digitale Nazionale Dati per l’interoperabilità, attraverso l'utilizzo di materiale crittografico basato su certificati X.509, solo nel caso in cui il fruitore non possa accreditarsi alla stessa e comunque entro 12 mesi dal superamento di tale impedimento l'erogatore e fruitore DEVONO aggiornare le modalità di costituzione del trust assicurando lo stesso per il tramite della Piattaforma Digitale Nazionale Dati per l’interoperabilità.

In quanto segue si declina il presente pattern in assenza della Piattaforma Digitale 
Nazionale Dati per l’interoperabilità, si rimanda alle "Linee Guida sull’infrastruttura tecnologica della Piattaforma Digitale Nazionale Dati per l’interoperabilità dei sistemi informativi e delle basi di dati" adottate ai sensi dell’articolo 50-ter, comma 2 del CAD l'esplicitazione delle modalità da applicare in presenza della Piattaforma Digitale Nazionale Dati per l’interoperabilità.

Il fruitore inoltra un messaggio all’erogatore includendo o
referenziando il certificato X.509 e una porzione significativa del
messaggio firmata.

L’erogatore, ricevuto il messaggio, verifica il certificato X.509 e
valida la porzione firmata del messaggio, inclusa la corrispondenza del
destinatario e l’intervallo di validità della firma. 

L’erogatore verifica inoltre l’univocità dell’identificativo ricevuto
nel JWT.

Se la verifica e la validazione sono superate, l’erogatore elabora la
richiesta e produce la relativa risposta.

.. mermaid::

     sequenceDiagram
     
      activate Fruitore
      activate Erogatore
      Fruitore->>+Erogatore: 1. Request()
      Erogatore-->>Fruitore: 2. Response
      deactivate Erogatore
      deactivate Fruitore

*Figura 6 - Accesso del Fruitore*

Regole di processamento
-----------------------

La creazione ed il processamento dei JWT DEVE rispettare
le buone prassi di sicurezza indicate in :rfc:`8725`.

**A: Richiesta**

1. Il fruitore predispone il payload del messaggio (ad esempio un
   oggetto JSON)

2. Il fruitore, o il soggetto individuato dal trust definito tra erogatore 
   e fruitore, costruisce il JWT popolando:

   a. il Jose Header con almeno i parameter:

      i.   alg con l’algoritmo di firma, vedi :rfc:`8725`

      ii.  typ uguale a JWT

      iii. una o più delle seguenti opzioni per referenziare il
           certificato X.509:

-  x5u (X.509 URL)

-  x5c (X.509 Certificate Chain)

-  x5t#S256 (X.509 Certificate SHA-256 Thumbprint)

   b. il payload del JWT coi claim rappresentativi degli elementi chiave
      del messaggio, **contenente almeno**:

      iv.  i riferimenti temporali di emissione e scadenza: :code:`iat`, :code:`exp`.
           Se il flusso richiede di verificare l’istante di prima
           validità del token, si può usare il claim :code:`nbf`.

      v.   il riferimento dell’erogatore in :code:`aud`;

      vi.  un identificativo univoco del token jti. Se utile alla logica
           applicativa l’identificativo può essere anche collegato al
           messaggio.

3. il fruitore, o il soggetto individuato dal trust definito tra erogatore 
   e fruitore firma il token adottando la JWS Compact Serialization

4. il fruitore posiziona il JWT nell’ :httpheader:`Authorization`

5. Il fruitore spedisce il messaggio all’erogatore.

**B: Risposta**

6.  L’erogatore decodifica il JWT presente in :httpheader:`Authorization`
    secondo le indicazioni contenute in :rfc:`7515#section-5.2`,
    le buone prassi indicate in :rfc:`8725`
    e valida i claim contenuti nel JOSE Header, in particolare verifica:

    c. il contenuto dei claim iat ed exp;

    d. la corrispondenza tra se stesso e il claim aud;

    e. l’univocità del claim jti

7.  L’erogatore recupera il certificato X.509 referenziato nel JOSE
    Header
    facendo attenzione alle indicazioni contenute in :rfc:`8725#section-3.10`

8.  L’erogatore verifica il certificato secondo i criteri del trust

9.  L’erogatore valida la firma verificando l’elemento Signature del JWT

10. L’erogatore garantisce l’accesso al fruitore

11. Se le azioni da 6 a 10 hanno avuto esito positivo, il messaggio
    viene elaborato e viene restituito il risultato del servizio
    richiamato.

Note:

-  In merito agli algoritmi da utilizzare si fa riferimento alle 
   Linee Guida sulla sicurezza, emanate dall'Agenzia per l'Italia Digitale
   ai sensi dell'articolo 71 del decreto legislativo 7 marzo 2005, n. 82 (Codice dell'Amministrazione Digitale).

-  Un meccanismo simile può essere utilizzato specularmente per
   l’erogatore.

Esempio
-----------

Di seguito è riportato un tracciato del messaggio inoltrato dal fruitore
all’interfaccia di servizio dell’erogatore.

Esempio porzione pacchetto HTTP.

.. code-block:: http

   GET https://api.erogatore.example/rest/service/v1/hello/echo/Ciao HTTP/1.1
   Accept: application/json
   Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5c.vz8...

Esempio porzione JWT

.. code-block:: python

   # *header*
   {
	   "alg": "ES256",
	   "typ": "JWT",
	   "x5c": [
		   "MIICyzCCAbOgAwIBAgIEC..."
		   ]
   }
   
   # *payload*
   {
	   "aud": "https://api.erogatore.example/rest/service/v1/hello/echo"
	   "iat": 1516239022,
	   "nbf": 1516239022,
	   "exp": 1516239024,
           "iss": "https://api.fruitore.example",
           "sub": "https://api.fruitore.example",
	   "jti": "065259e8-8696-44d1-84c5-d3ce04c2f40d"
   }

Gli elementi presenti nel tracciato rispettano le seguenti scelte
implementative e includono:

-  l’intervallo temporale di validità, in modo che il JWT possa essere
   usato solo tra gli istanti :code:`nbf` ed :code:`exp`;

-  indica l’istante iat di emissione del JWT. Se le parti possono
   accordarsi nel considerarlo come l’istante iniziale di validità del
   token, :rfc:`7519` non assegna a questo claim nessun ruolo specifico
   nella validazione, a differenza di :code:`nbf`;

-  il riferimento al firmatario del token nel claim aggiuntivo :code:`iss`, che deve essere
   raccordato con il contenuto del certificato;

-  il riferimento al fruitore nel claim aggiuntivo :code:`sub`;

-  il destinatario del JWT, che DEVE sempre essere validato;

-  contenuto della certificate chain X.509 (:code:`x5c`)

-  algoritmi di firma e digest (:code:`alg`).

Le parti, in base alle proprie esigenze, individuano gli specifici algoritmi 
secondo quanto indicato nelle Linee Guida sulla sicurezza, emanate dall'Agenzia per l'Italia Digitale 
ai sensi dell'articolo 71 del decreto legislativo 7 marzo 2005, n. 82 (Codice dell'Amministrazione Digitale).
