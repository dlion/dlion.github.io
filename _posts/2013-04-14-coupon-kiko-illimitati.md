---
title: Coupon Kiko illimitati
summary: Diario di un bypassaggio, ottenere infiniti coupon kiko.
categories: Security
layout: post
---
Tranquilli, questo articolo non tratterà né di make-up né di make-down.   
Alcune settimane fa venni contattato dalla mia ragazza su Facebook la quale mi chiese di accedere ad un'app che le permetteva di ricevere un coupon che poteva scambiare con uno smalto, il tutto gratis; l'unico inconveniente ?   
Che poteva prenderne **solo uno**.   
Tutto ciò mi incuriosì... cioè, io metto "mi piace", accedo all'app, inserisco i miei dati, rispondo a qualche domanda e ricevo in cambio un coupon con un codice alfanumerico che dovrò portare in negozio per scambiare gli smalti in questione ?    
Ebbene sì, funzionava proprio così; a quel punto mi balenò in mente una malsana domanda: **Potevo avere più di un coupon ?**

Intanto voglio dire che ho pubblicato solo oggi l'articolo perché tale iniziativa scade proprio oggi, **la mia era una sfida d'intelligenza più che una vera e propria voglia di prendere smalti**. *(Anche perché li avrei comunque dati alla mia ragazza dato che io non sono interessato a certe cose `<3` )*

Un'app di facebook non è altro che un collegamento fra il sito che mette a disposizione l'app e l’utente registrato al social network.   
Quindi l'app può sfruttare delle API che le permettono di interagire con l'utente iscritto a facebook quindi capire se mette mi piace, se è loggato, etc.   
Una volta effettuata tutta la pratica in questione mi veniva restituita una immagine con sopra un codice a barre e sotto di esso un codice alfanumerico simile a questo:
![alphanumerico](/images/coupon-kiko-illimitati.jpg)

Noto che l'immagine in questione ha un indirizzo del genere: `http://www.fbappdev.com/kikoform/data/coupon/p_njkdndfjskf.jpg` quindi vado ad indagare sul sito in questione e...bingo!   
Trovo una pagina web con l'immagine principale dell'app, null'altro.   
Come è giusto che sia controllo il codice html e noto che c'è parecchia roba in javascript allora con il mio fedele [firebug](http://getfirebug.com/) comincio a debuggarmi il codice trovato studiandone il funzionamento notando questo:

{% highlight js lineanchors %}
FB.getLoginStatus(function(response) {
    //console.log('getLoginStatus');
    if (response.status === 'connected') {
      //Autorizzato e connesso
      fb_uid = response.authResponse.userID;
      fb_at = response.authResponse.accessToken;
      fbIsLogged=true;
      //console.log('sono registrato!');
      if(partecipaClicked==true)
        loadQuests(false);
      document.location.reload(true);
    } else if (response.status === 'not_authorized') {
      fbIsLogged=false;
      login();
    } else {
      // the user isn'tlogged in to Facebook.
      fbIsLogged=false;
      login();
    }
  });
{% endhighlight %}   
Cioè se io sono loggato a facebook la pagina in modo automatico si prende il mio *uid* e il mio *altro* richiamando una funzione.   
Se invece non sono loggato mi rimanda al login.

Quasi dimenticavo: **ringrazio gli sviluppatori italiani di tale app che grazie ai loro commenti lasciati sparsi per la pagina mi hanno permesso di capire meglio ciò che succedeva nel codice**. _LOL_

L'UID non è altro che l'User Identificator, cioè un "codice" che identifica ogni utente di facebook, tutti ne hanno uno ed essendo univoci vengono usati per riconoscere gli utenti all'interno del socialnetwork.   
Mentre l'access token è una stringa randomica che dà un accesso sicuro e temporaneo alle API di facebook ed identifica un'app, un user, una pagina,etc.   
Per ulteriori informazioni date una occhiata alla [documentazione ufficiale di facebook](https://developers.facebook.com/docs/concepts/login/access-tokens-and-types/)

Continuo a controllare il codice e noto che sono presenti diverse slide, cioè il sito in questione è formato da slides fatte in jQuery che venivano visualizzate di volta in volta per poi arrivare ad un form che inviava una richiesta ad una determinata pagina php.   
Scorrendo il sorgente noto qualcosa d'interessante, è la parte più importante di tutte 'sto spezzone di sorgente quindi state molto attenti.

{% highlight js lineanchors %}
$('#step4').click(function() {
  if(slider==null) { slider = $('div#formslider').data('jslide') };
  //Qui invia la news letter?
  var check = false;
  check = checkDomanda(4);
  ////Manda i dati via Ajax al form PHP!
  var jsonParameters = creaJson();
  var screenH = $(window).height();
  var screenW = $(window).width();
  $('#wait').css({
    'top': '300px',
    'left':(screenW/2 - 125) + 'px'
  });
  $('#wait').fadeIn();
  $.ajax({
    url: "request.php?method=save",
    type: "POST",
    data: jsonParameters,
    dataType:'json',
    error : function (XHR, textStatus) {
    alert('[FAIL]Impossibile partecipare al Concorso, Riprova più tardi.');
    return;
  }
  ,
  success: function(data, textStatus, jqXHR) {
    $('#wait').fadeOut();
    //console.log('reponse');
    //console.log(data);
    if(typeof(data)==='undefined') {
      alert('[DATA]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    if(typeof(data.response)==='undefined') {
      alert('[RESPONSE]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    if((data.response)!='OK') {
      alert('[RESONSE-OK]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    if(typeof(data.code)==='undefined') {
      alert('[DOCE]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    var codiceInterno = data.code;
    $('#step_counter').removeClass('counter_step1');
    $('#step_counter').removeClass('counter_step2');
    $('#step_counter').removeClass('counter_step3');
    $('#step_counter').addClass('counter_step4');
    if(jsonParameters.js_invio=='email') {
      getCoupon(codiceInterno,'email');
    }
    if(jsonParameters.js_invio=='posta') {
      getCoupon(codiceInterno,'posta');
    }
    slider.slideTo(3);
    FBCanvasSize();
  }
  ....
{% endhighlight %}

Il codice non è affatto complesso, vediamo ciò che fa

{% highlight js %}
$('#step4').click(function() {
{% endhighlight %}   
Intanto tutto quel codice viene eseguito una volta che si clicca sull'elemento con id "step4", semplice jQuery.

Ora notiamo qualcosa d’importante

{% highlight js %}
//Manda i dati via Ajax al form PHP!
var jsonParameters = creaJson();
{% endhighlight %}   
Continuo a ringraziare gli sviluppatori della Kiko per queste chicche *molto esplicite*.   
Cioè quel *jsonParameters* conterrà dei parametri che sicuramente verranno utilizzati per inviare i dati alla pagina php che dovrà elaborarli; teniamola in mente, ci torneremo subito dopo.

Continuiamo  a leggere il codice fino a
{% highlight js lineanchors %}
$.ajax({
    url: "request.php?method=save",
    type: "POST",
    data: jsonParameters,
    dataType:'json',
    error : function (XHR, textStatus) {
    alert('[FAIL]Impossibile partecipare al Concorso, Riprova più tardi.');
    return;
  }
{% endhighlight %}   
Questo è ciò che cercavo, una richiesta [AJAX](https://it.wikipedia.org/wiki/AJAX) alla pagina request.php passando come parametro method=save e come parametri della richiesta POST il contenuto della variabile *jsonParameters* creata con la funzione *creaJson()*.

Se va in errore ritorna un alert altrimenti
{% highlight js lineanchors %}
success: function(data, textStatus, jqXHR) {
    $('#wait').fadeOut();
    //console.log('reponse');
    //console.log(data);
    if(typeof(data)==='undefined') {
      alert('[DATA]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    if(typeof(data.response)==='undefined') {
      alert('[RESPONSE]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    if((data.response)!='OK') {
      alert('[RESONSE-OK]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    if(typeof(data.code)==='undefined') {
      alert('[DOCE]Impossibile partecipare al Concorso, Riprova più tardi.');
      return;
    }
    var codiceInterno = data.code;
    $('#step_counter').removeClass('counter_step1');
    $('#step_counter').removeClass('counter_step2');
    $('#step_counter').removeClass('counter_step3');
    $('#step_counter').addClass('counter_step4');
    if(jsonParameters.js_invio=='email') {
      getCoupon(codiceInterno,'email');
    }
    if(jsonParameters.js_invio=='posta') {
      getCoupon(codiceInterno,'posta');
    }
    slider.slideTo(3);
    FBCanvasSize();
  }
{% endhighlight %}   
Cioè se la risposta della pagina php nominata prima è diverso da *OK* ritornerà errori vari altrimenti verrà inserita all'interno della variabile *codiceInterno* il contenuto del responso del campo code. (indi presumiamo venga restituito un qualche tipo di codice)

Continuiamo e troviamo 2 condizioni.   
Se il campo *js_invio* della variabile *jsonParameters* è uguale ad "email" viene richiamata la funzione *getCoupon* passando come parametri il codice ricevuto ed email, altrimenti se è uguale a "posta" verrà richiamata la stessa con il parametro modificato di conseguenza.

La funzione *getCoupon* ? Quindi c'è una funzione dedicata alla generazione dei coupon ? Uhm,  ma prima direi di riprendere la funzione *creaJson()* così da capire come vengono "sistemati" i dati che vengono inviati.   
Cerco nel codice e la trovo, vediamola:
{% highlight js lineanchors %}
function creaJson() {
  var datiJson = {};
  datiJson.js_nome = $('#nome').val();
  datiJson.js_cognome = $('#cognome').val();
  datiJson.js_email = $('#email').val();
  datiJson.js_nascita = $('#nascita_year').val() + "-" + $('#nascita_month').val() +"-" +$('#nascita_day').val() ;
  datiJson.js_privacy = $('#privacy').attr('checked');
  datiJson.js_privacy2 = $('#privacy2').attr('checked');
  datiJson.js_privacy3 = $('#privacy3').attr('checked');
  datiJson.js_newsletter = $('#newsletter').attr('checked');
  if(datiJson.js_privacy=='checked' || datiJson.js_privacy == true) datiJson.js_privacy = 1;
  if(datiJson.js_privacy2=='checked' || datiJson.js_privacy2 == true) datiJson.js_privacy2 = 1;
  if(datiJson.js_privacy3=='checked' || datiJson.js_privacy3 == true) datiJson.js_privacy3 = 1;
  if(datiJson.js_newsletter=='checked' || datiJson.js_newsletter == true) datiJson.js_newsletter = 1;
  //Domande:
  /* Vecchia versione con Checkbox
  datiJson.js_domanda1 = $('input[name=domanda1]:checked').val();
  datiJson.js_domanda2 = $('input[name=domanda2]:checked').val();
  datiJson.js_domanda3 = $('input[name=domanda3]:checked').val();
  */
  datiJson.js_domanda1 = $('#domanda1').val();
  datiJson.js_domanda2 = $('#domanda2').val();
  datiJson.js_domanda3 = $('#domanda3').val();
  datiJson.js_domanda4 = $('#domanda5').val();
  datiJson.js_invio = $('input[name=domanda4]:checked').val();
  datiJson.js_uid = fb_uid;
  datiJson.data = 'ok';
  return datiJson;
}
{% endhighlight %}   
In pratica viene creata una variabile *datiJson* e formattata per essere trasformata in dati [JSON](https://it.wikipedia.org/wiki/JSON). (Qui ti urge una googlata se non conosci il *JSON*)   
Dopo il '.' sono i relativi campi che vengono riempiti di volta in volta con i valori presi dalla pagina.   
Cioè:

* js_nome = nome
* js_cognome = cognome
* js_email = email
* js_nascita = anno-mese-data
* js_privacy = Sarà 1 se checkata
* js_privacy2 = ↑
* js_privacy3 = ↑
* js_newsletter = ↑

Arrivati qui incontriamo domanda1, domanda2, domanda3, domanda4 che prendono i relativi dati dalle proprie select; guardando nel codice vedo che sono semplici scritte come "nere", "castani", etc. per evitare in futuro di passare dei dati "scorretti".

Andando avanti troviamo

* js_invio = email o posta (ricordate le 2 condizioni precedenti?)
* js_uid
* data = 'ok' (come parametro il campo data avrà "ok")

What ? js_uid ?   
E bene sì, uno dei parametri sarà il nostro **facebook user id**.   
Questo consentirà di verificare se l'utente ha già effettuato l'operazione, consentirà di sapere quale user ha usato l'app, etc.   
Quindi senza di quello ogni tentativo di effettuare una richiesta sarà vana ed ecco perché è possibile prelevare un coupon per account.

Bene, abbiamo capito come vengono e quali dati vengono presi per poi spedirli in formato JSON alla pagina php per poi ricevere come risposta (in caso di successo) un codice X che dovremo passare alla funzione *getCoupon*.

Detto ciò andiamo a vedere come si comporta la funzione getCoupon in merito ai parametri che le vengono passati
{% highlight js lineanchors %}
function getCoupon(codice, mezzo) {
  var myV= $('#my_voucher');
  //Costruisci url...
  var url = siteAddress + 'data/coupon/'+(mezzo=='email'?'v_':'p_')+codice+'.jpg';
  myV.removeClass('print');
  myV.removeClass('online');
  myV.addClass( (mezzo=='email'?'online':'print') );
  $('#print_message').css('display','none');
  $('#web_site').css('display','none');
  $('#print_site').css('display','none');
  var calcoloAltezza = 1230;
  if(mezzo!='email') {
    calcoloAltezza = 990;
    if(url.indexOf('http://')>-1)
      url = url.replace('http://','//');
    if(url.indexOf('https://')>-1)
      url = url.replace('https://','//');
    myV.attr('src','img/tratteggio.png');
    myV.css('background-image', 'url("'+url+'")');
    myV.css('background-size', 'contain');
    $("#forbice").show();
    $('#print_message').css('display','block');
    $('#print_site').css('display','block');
    $('#infotext').html(getInfoText("print"));
  } else {
    calcoloAltezza = 990;
    myV.attr('src',url);
    $("#forbice").hide();
    $('#web_site').css('display','block');
    $('#infotext').html(getInfoText("email"));
  }
  $('#showcase_container').animate({height:calcoloAltezza},1050);
  $('#lastStep').animate({height:calcoloAltezza},1050);
  $('#print_voucher').unbind('click');
  $('#redraw_voucher').unbind('click');
  $('#print_voucher').click(function() {
    var windowObject = window.open('','windowObject','');
    windowObject.document.write('<html><head><title>Kiko cosmetics</title></head><body onload="javascript:window.print();"><p style="text-align:center;"><img src="'+url+'"/></p></body></html>');
    windowObject.document.close();
    w.focus();
  });
  $('#redraw_voucher').click(function(e) {
    e.preventDefault();
    rigeneraCoupon();
  });
  if(UtenteEsistente==false) {
    setTimeout(function() {
      FBCanvasSize_h(true,1400);
    },250);
    setTimeout(
    function() {
      $('#grazie').fadeIn();
      //console.log('mostro.');
    }
    ,4000);
  //setTimeout($('#grazie').fadeIn(),4500);
  }
{% endhighlight %}   
La parte più importante è sicuramente questa:
{% highlight js %}
var url = siteAddress + 'data/coupon/'+(mezzo=='email'?'v_':'p_')+codice+'.jpg';
{% endhighlight %}

Cioè mettiamo nella variabile url una stringa formata da queste variabili:

* *siteAddress* è l'indirizzo del sito
* *codice* è il codice che ci viene restituito

All'interno della stringa vediamo un operatore ternario cioè se il mezzo è "email" allora mette *v_* altrimenti *p_* per poi terminare il tutto con *.jpg*   
Cioè se gli passiamo email e il codice il risultato sarà `http://sito/data/coupon/v_codice.jpg` altrimenti `http://sito/data/coupon/p_codice.jpg`   
Non somigliano molto all'indirizzo dell'immagine che ci era stata data all'inizio ?

Abbiamo scoperto cosa restituisce la funzione *getCoupon*, ottimo; analizziamo altre funzioni utili a capire il comportamento di tale "app".

Scorrendo ancora troviamo
{% highlight js lineanchors %}
function loadQuests(force) {
  //console.log('check quests:'+fb_uid + ", page: "+ kikoPage + 'my Token: ' + fb_at);
  //Verifica se l'utente ha già fatto il form...
  var screenH = $(window).height();
  var screenW = $(window).width();
  $('#wait').css({
    'top': '350px',
    'left':(screenW/2 - 125) + 'px'
  });
  $('#wait').fadeIn();
  $.ajax({
    url: "request.php?method=check",
    type: "POST",
    data: {'user_id':fb_uid,'data':'ok'},
    dataType:'json',
    error : function (XHR, textStatus) {
      $('#wait').fadeOut();
      loadQuestsInner(force);
      return;
    },
    success: function(data, textStatus, jqXHR) {
      $('#wait').fadeOut();
      //console.log('risposta');
      //console.log(data);
      if(typeof(data)==='undefined') {
        loadQuestsInner(force);
        return;
      }
      if(typeof(data.response)==='undefined') {
        loadQuestsInner(force);
        return;
      }
      if(typeof(data.code)==='undefined') {
        loadQuestsInner(force);
        return;
      }
      if((data.response)=='EXISTS' && data.code.length>0) {
        //Vari al pannello 4 ..con il coupon caricato!
        //console.log('codice esista...vado alla fine');
        UtenteEsistente = true;
        loadQuestsInner(force);
        var codiceInterno = data.code;
        partecipaClicked=true;
        $('#slider').show();
        $('#step_counter').removeClass('counter_step1');
        $('#step_counter').removeClass('counter_step2');
        $('#step_counter').removeClass('counter_step3');
        $('#step_counter').addClass('counter_step4');
        try {
          if(data.send=='email') {
            getCoupon(codiceInterno,'email');
          }
          if(data.send=='posta') {
            getCoupon(codiceInterno,'posta');
          }
        } catch (errore) { }
        var slider = null;
        if(slider==null) { slider = $('div#formslider').data('jslide') };
        slider.slideTo(3);
        FBCanvasSize();
        setTimeout(function() {
          $('#avvio_01').hide();
          $('#avvio_04').show();
        },1000);
        return;
      }
    }
  });
{% endhighlight %}   
Questa funzione viene richiamata dall'app per verificare se un determinato utente ha già effettuato in passato delle richieste (sempre tramite f_uid,etc.) vediamola in dettaglio.   
La parte che più ci interessa è sicuramente la richiesta asincrona inviata
{% highlight js lineanchors %}
 $.ajax({
    url: "request.php?method=check",
    type: "POST",
    data: {'user_id':fb_uid,'data':'ok'},
    dataType:'json',
    error : function (XHR, textStatus) {
      $('#wait').fadeOut();
      loadQuestsInner(force);
      return;
    },
{% endhighlight %}   
Come possiamo vedere la richiesta [POST](https://en.wikipedia.org/wiki/POST_%28HTTP%29) viene inviata alla pagina *request.php* passando come parametro *method=check* inviando come dati l'*fb_uid* e "ok".

Se viene ricevuta una risposta di successo accade questo
{% highlight js lineanchors %}
if((data.response)=='EXISTS' && data.code.length>0) {
        //Vari al pannello 4 ..con il coupon caricato!
        //console.log('codice esista...vado alla fine');
        UtenteEsistente = true;
        loadQuestsInner(force);
        var codiceInterno = data.code;
        partecipaClicked=true;
        $('#slider').show();
        $('#step_counter').removeClass('counter_step1');
        $('#step_counter').removeClass('counter_step2');
        $('#step_counter').removeClass('counter_step3');
        $('#step_counter').addClass('counter_step4');
        try {
          if(data.send=='email') {
            getCoupon(codiceInterno,'email');
          }
          if(data.send=='posta') {
            getCoupon(codiceInterno,'posta');
          }
        } catch (errore) { }
        var slider = null;
        if(slider==null) { slider = $('div#formslider').data('jslide') };
        slider.slideTo(3);
        FBCanvasSize();
        setTimeout(function() {
          $('#avvio_01').hide();
          $('#avvio_04').show();
        },1000);
        return;
      }
{% endhighlight %}   
Cioè non viene effettuata nessuna richiesta di inserimento ma bensì viene ricevuto il codice e passato alla funzione *getCoupon* incaricata di formare e mostrare l'immagine nella pagina web.

Un’altra funzione simile è la *rigeneraCoupon* che non fa altro che rigenerare un determinato coupon, simile a quando premiamo lo #step4 vista precedentemente per poi mostrare a schermo il tutto.

Bene, abbiamo capito come funziona quest'app, vediamo di mettere in pratica ciò che abbiamo imparato.

Nel precedente articolo abbiamo visto il funzionamento delle [curl]({{site.url}}/richieste-http-dalla-linea-di-comando/), oggi le metteremo in pratica per effettuare una richiesta alla pagina specificata, vediamo come.

Ricordate com'erano i dati e quali erano ? Non ci resta che inviargli manualmente ciò che l'app si prendeva in automatico
{% highlight sh lineanchors %}
curl --data "js_nome=NomeFalso&\
            js_cognome=CognomeInventato&\
            js_email=indirizzo_falso%40gmail.com&\
            js_nascita=1992-8-6&\
            js_privacy=1&\
            js_privacy2=1&\
            js_privacy3=1&\
            js_domanda1=verdi&\
            js_domanda2=biondi&\
            js_domanda3=normale&\
            js_domanda4=Risposta_random&\
            js_invio=posta&\
            js_uid=facebookuidrandomico&\
            data=ok"\
            "http://www.fbappdev.com/kikoform/request.php?method=save"
{% endhighlight %}

Come responso riceveremo questo:
{% highlight json lineanchors %}
{
    "response":"OK",
    "method":"send",
    "error":"",
    "code":"ODMxZTQ1NDFhYjE2ZjBmOGJlNzE4YTcxNGU0NDNlYjEzNzQ1N2E5MA",
    "coupon":
    {
        "online":"35KO1MLNFAN",
        "sale":"4BFBCCCA83009"
    }
}
{% endhighlight %}

E bene sì, **ce l'abbiamo fatta**.   
Il codice è proprio quello all'interno del campo "code" nel responso, ora non ci resta che andare a "montare" l'url cioè: `https://fbappdev.com/kikoform/data/coupon/p_ODMxZTQ1NDFhYjE2ZjBmOGJlNzE4YTcxNGU0NDNlYjEzNzQ1N2E5MA.jpg`

Abbiamo messo *p_* perché nella richiesta abbiamo specificato "posta" altrimenti avremmo dovuto mettere *v_* e se proviamo a raggiungere l'url riceveremo il nostro bel coupon.

Per ricevere tutti i coupon diversi basterà cambiare il Facebook User ID, funzionano anche in maniera randomica, la pagina genererà il codice in base all'uID che invierete.   
Ovviamente il tutto è perfettamente funzionante perché il codice che viene generato è un codice creato con l'algoritmo dell'app stessa e quindi conforme all'evento in questione.

Penso possa bastare, qualsiasi domanda, chiarimento, dubbio, etc. io sono sempre qui.

Saluti, DLion

---

[English Version](https://domenicoluciani.com/2013/04/14/unlimited-kiko-coupon.html)
