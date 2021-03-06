---

copyright:
  year: 2016

---

# Abilitazione dell'autenticazione Google per le applicazioni web
{: #google-auth-web}

Ultimo aggiornamento: 18 luglio 2016
{: .last-updated}

Utilizza Google Sign-In per autenticare gli utenti alla tua applicazione web. Aggiungi la funzionalità di sicurezza {{site.data.keyword.amashort}}. 


## Prima di cominciare
{: #before-you-begin}

È necessario disporre di:
* Un'applicazione web.
* Un'istanza di un'applicazione  {{site.data.keyword.Bluemix_notm}} che è protetta da un servizio {{site.data.keyword.amashort}}. Per ulteriori informazioni su come creare un'applicazione di back-end {{site.data.keyword.Bluemix_notm}}, vedi [Introduzione](index.html).
* L'URI Per il reindirizzamento finale (dopo il completamento del processo di autorizzazione).



## Configurazione di un'applicazione Google per il tuo sito web
Per iniziare a usare Google come un provider di identità, creare un progetto nella [Google Developer Console](https://console.developers.google.com). Parte della creazione di un progetto consiste nell'ottenere un **ID client Google**  e un **Segreto**. Il segreto e l'ID client Google sono identificativi univoci per la tua applicazione utilizzati dall'autenticazione Google e sono necessari per la configurazione del dashboard {{site.data.keyword.amashort}}.


1. Apri la tua applicazione Google nella Google Developer Console. 
3. Aggiungi l'API Google+. 
3. Crea le credenziali utilizzando OAuth. Seleziona applicazione Web nel tipo di applicazione. Immetti l'URI di reindirizzamento {{site.data.keyword.amashort}} nella casella Authorized redirect
URIs. L'URI di autorizzazione di reindirizzamento {{site.data.keyword.amashort}} può essere ottenuto dalla schermata di configurazione Google del dashboard {{site.data.keyword.amashort}} (vedi i seguenti passi). 
6. Salva le modifiche. Prendi nota del segreto applicazione e dell'ID client Google. 


## Configurazione di {{site.data.keyword.amashort}} per l'autenticazione Google
Dopo aver ottenuto il segreto e l'ID client Google, puoi abilitare l'autenticazione Google nel dashboard {{site.data.keyword.amashort}}.

1. Apri la tua applicazione nel dashboard {{site.data.keyword.Bluemix_notm}}.
1. Fai clic sul tile {{site.data.keyword.amashort}}. Il dashboard {{site.data.keyword.amashort}} viene caricato.
1. Fai clic sul pulsante nel pannello Google.
1. Nella sezione **Configure for Web**:   
    * Prendi nota del valore nella casella di testo **Mobile Client Access Redirect URI for Google Developer Console**. Questo è il valore che dovrai aggiungere alla casella **Authorized redirect URIs** in **Restrictions in the Client ID for Web application** di **Google Developers Portal** nel passo 3. 
    * Immetti l'**ID client Google** e il **Segreto client**.
    * Immetti l'URI di reindirizzamento in **Your Web Application Redirect URIs**. Questo valore è per l'URI di reindirizzamento a cui accedere dopo che viene completato il processo di autorizzazione e viene determinato dallo sviluppatore.
1. Fai clic su **Save**.


## Implementazione del flusso dell'autorizzazione {{site.data.keyword.amashort}} utilizzando Google come provider di identità

La variabile di ambiente `VCAP_SERVICES` viene creata automaticamente per ogni istanza del servizio {{site.data.keyword.amashort}} e contiene le proprietà necessarie per il processo di autorizzazione. È formata da un oggetto JSON e può essere visualizzata facendo clic su  **Enviroment Variables**  nel navigator nel lato sinistro della tua applicazione.

Per avviare il processo di autorizzazione:

1. Richiama l'endpoint di autorizzazione (`authorizationEndpoint`) e l'ID client (`clientId`) dalle credenziali del servizio archiviate nella variabile di ambiente `VCAP_SERVICES`. 

    **Nota:** se hai creato il servizio {{site.data.keyword.amashort}} prima dell'aggiunta del supporto web, potresti non avere endpoint di autorizzazione nelle credenziali del servizio. In questo caso utilizza i seguenti endpoint di autorizzazione nella tua regione Bluemix: 


 	Stati Uniti Sud: 
 	```
 	https://mobileclientaccess.ng.bluemix.net/oauth/v2/authorization
 	```
 	Londra: 
 	```
 	https://mobileclientaccess.eu-gb.bluemix.net/oauth/v2/authorization
  	```
  	Sydney: 
  	```
  	https://mobileclientaccess.au-syd.bluemix.net/oauth/v2/authorization
  	```
2. Crea l'URI del server dell'autorizzazione utilizzando `response_type("code")`, `client_id` e `redirect_uri` come parametri di query.
3. Vai dalla tua applicazione web all'URI generato.
  
Il seguente esempio richiama i parametri dalla variabile `VCAP_SERVICES`, creando l'URL e inviando la richiesta di reindirizzamento.
  
```Java
var cfEnv = require("cfenv"); 
app.get("/protected", checkAuthentication, function(req, res, next){ 
 	res.send("Hello from protected endpoint"); 
 }); 

 app.get("/protected", checkAuthentication, function(req, res, next){ 
 	res.send("Hello from protected endpoint"); 
 	function checkAuthentication(req, res, next){ 

	// Controlla se l'utente è autenticato
  if (req.session.userIdentity){ 
		next() 
	} else { 
		// Se non lo è - reindirizza il server dell'autorizzazione
		var mcaCredentials = cfEnv.getAppEnv().services.AdvancedMobileAccess[0].credentials;
		var authorizationEndpoint = mcaCredentials.authorizationEndpoint;
		var clientId = mcaCredentials.clientId;
		var redirectUri = "http://some-server/oauth/callback"; // L'uri di reindirizzamento della tua applicazione web
		var redirectUrl = authorizationEndpoint + "?response_type=code";
		redirectUrl += "&client_id=" + clientId;
		redirectUrl += "&redirect_uri=" + redirectUri;
		res.redirect(redirectUrl);
	} 
} 
```

Tieni presente che il parametro `redirect_uri` rappresenta l'URI di reindirizzamento della tua applicazione web e deve essere uguale al valore definito nel dashboard {{site.data.keyword.amashort}}.
Dopo il reindirizzamento dell'endpoint di autorizzazione l'utente visualizzerà un modulo di accesso da Google. Dopo che l'utente ha consesso le autorizzazioni ad accedere utilizzando l'identità Google, il servizio {{site.data.keyword.amashort}} richiama il tuo URI di reindirizzamento dell'applicazione web fornendo il codice concesso come un parametro di query. 

## Ottenimento dei token
Il passo successivo è quello di ottenere il token di accesso e i token di identità utilizzando il codice concesso ricevuto precedentemente. 

1. Richiama token `tokenEndpoint`, `clientId` e `secret` dalle credenziali del servizio archiviate nella variabile di ambiente `VCAP_SERVICES`. 
  
    **Nota:** se hai creato il servizio {{site.data.keyword.amashort}} prima dell'aggiunta del supporto web, potresti non avere endpoint di autorizzazione nelle credenziali del servizio. In questo caso utilizza i seguenti endpoint di autorizzazione nella tua regione Bluemix: 
 
 	Stati Uniti Sud: 
 	```
 	https://mobileclientaccess.ng.bluemix.net/oauth/v2/token 
  	```
 	Londra: 
  	```
  	https://mobileclientaccess.eu-gb.bluemix.net/oauth/v2/token  
   	```
   	Sydney: 
  	```
   	https://mobileclientaccess.au-syd.bluemix.net/oauth/v2/token 
  	```

2. Invia una richiesta post all'URI del server del token con il tipo di concessione ("authorization_code"), l'id client, l'uri di reindirizzamento e il codice come parametri del modulo. Invia `clientId` e `clientSecret` come credenziali di autenticazione HTTP di base.
 
Il seguente codice richiama i valori necessari e li invia con una richiesta post:
    
```Java
  var cfEnv = require("cfenv");
  var base64url = require("base64url ");
  var request = require('request');

   app.get("/oauth/callback", function(req, res, next){ 
	var mcaCredentials = cfEnv.getAppEnv().services.AdvancedMobileAccess[0].credentials; 
    var tokenEndpoint = mcaCredentials.tokenEndpoint; 
    var formData = { 
		grant_type: "authorization_code",
      client_id: mcaCredentials.clientId,
      redirect_uri: "http://some-server/oauth/callback",   // Il tuo uri di reindirizzamento dell'applicazione web
      code: req.query.code
    }

  request.post({ 
		url: tokenEndpoint, 
    formData: formData 
    }, function (err, response, body){ 
			var parsedBody = JSON.parse(body); 
        req.session.accessToken = parsedBody.access_token; 
        req.session.idToken = parsedBody.id_token; 
        var idTokenComponents = parsedBody.id_token.split("."); // [header, payload, signature] 
      var decodedIdentity= base64url(idTokenComponents[1]);
      req.session.userIdentity = JSON.parse(decodedIdentity)["imf.user"]; 
      res.redirect("/"); 
    }
	).auth(mcaCredentials.clientId, mcaCredentials.secret); 
  }
); 
```

Il parametro `redirect_uri` è l'URI per il reindirizzamento dopo l'esito positivo o negativo dell'autenticazione con Google+ e deve corrispondere a `redirect_uri` dal passo 1.  
   
Assicurati di inviare questa richiesta POST entro 10 minuti, dopo i quali il codice concesso scade. Dopo 10 minuti è necessario un nuovo codice. 

Il corpo della risposta POST contiene il `access_token` e il `id_token` codificati in base64. 

Quando hai ricevuto l'accesso e i token di identità, puoi indicare la sessione web come autenticata e facoltativamente conservare questi token.   


##Utilizzo dell'accesso ottenuto e del token di identità 

Il token di identità contiene informazioni sull'identità dell'utente. Per l'autenticazione Google, il token conterrà tutte le informazioni che l'utente ha deciso di condividere, come il nome completo, l'URL della foto del profilo, e così via.   

Il token di accesso abilita le comunicazioni con le risorse protette dai filtri di autorizzazione {{site.data.keyword.amashort}}. Per ulteriori informazioni, vedi [Protezione delle risorse](protecting-resources.html).

Per effettuare richieste che proteggano le risorse aggiungi un'intestazione dell'autorizzazione alle richieste con la seguente struttura: 

`Authorization=Bearer <accessToken> <idToken>`

####Suggerimenti:
{: tips} 

* Il `accessToken` e il `idToken` devono essere separati da uno spazio vuoto.

* Il `idToken` è facoltativo. Se hai fornito il token di identità, è possibile accedere alla risorsa protetta ma non si riceveranno le informazioni sull'utente autorizzato.  



