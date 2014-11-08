---
title: BugHunting ERSU, quanto sono al sicuro i nostri dati?
description: Bug Hunting sul portale dell'ERSU, quanto sono sicuri i nostri dati? Resoconto di una caccia spietata al bug.
category: Security
layout: post
---
E.R.S.U. - Ente Regionale per il Diritto allo Studio Universitario; è questo il nome dell'ente che si occupa di ricevere le richieste da parte degli studenti riguardo alle borse di studio che dovranno essere erogate ai suddetti vincitori fornendo agevolazioni importanti agli studenti meritevoli.

Le richieste generalmente sono tantissime contando tutti gli studenti dell'università che si ritengono idonei a ricevere la borsa di studio e contengono generalmente una fotocopia della _carta di identità_, _l'isee_, _l'iseu_, _l'ispeu_, _nuclei familiari_ e tante altre informazioni private.
Cosa pensate possa accadere se tali informazioni potessero essere consultate da **chiunque**?

In particolare oggi parlerò dell'ERSU della città di **Palermo**, dove vivo e dove frequento la facoltà di _Scienze Informatiche_ proprio all'[Università degli studi di Palermo](http://portale.unipa.it/ "UNIPA"); raccontando alcuni aneddoti sulla alquanto discutibile sicurezza usata per gestire determinati enti.

## Invio del modulo di partecipazione

Ogni studente universitario è obbligato a mandare un file pdf contenente tutti i dati utili per poter effettuare la domanda di partecipazione alla richiesta per la borsa di studio; dati quali fotocopia avanti/dietro della carta di identità, dichiarazione ISEEU, nucleo familiare, ISPEU, Codice Fiscale, Firma più altre eventuali informazioni che potrebbero servire per lo scopo sopra descritto; una vera e propria miniera di informazioni riguardo ad uno studente e al suo nucleo familiare; tale pdf dev'essere grande meno di 2Mb.
La procedura in questione funziona egregiamente e BAM, il documento è online!

## WHAT?

Vado a visualizzare il documento in questione e noto che il file in questione è stato rinominato in un formato piuttosto preciso: `<numerodipratica>_ERSU_2014.pdf` all'interno della cartella `2014/Doc/`.   
A quel punto la curiosità ebbe il sopravvento: **e se cambiassi il numero di pratica?**

![OMG]({{site.image_url}}/bughunting-ersu-1.gif)

Esattamente quello che state pensando, avevo appena aperto un pdf di un altro studente **con dentro tutti i suoi relativi dati PRIVATI**, ciò stava a significare che lo stesso criterio era stato adottato per tutti gli studenti che avevano fatto domanda, ripeto: **TUTTI**.

Per conferma mi sono realizzato un piccolo script in node.js che sfruttava il PoC in questione che chiamai [xaviERSU](https://github.com/DLion/xaviERSU "xaviERSU").

![Xavier]({{site.image_url}}/bughunting-ersu-2.png)

> Trovali, trovali tutti

Script che mi permetteva, iterando i numeri di pratica scannerizzando gli url di capire, se un determinato pdf era presente o meno all'interno di quella cartella tramite il messaggio di stato che il server restituiva. (**evitando quindi di scaricare o aprire documenti con dati non miei**, voglio sottolinearlo, eh )

Pensate ai possibili utilizzi se tali informazioni fossero cadute in mano a gente senza scrupoli, crawler vari, etc.   
Furti di identità, annullamento della privacy, vendita di informazioni che con tutta probabilità avrebbero lucrato su quella quantità così gargantuesca di dati attraverso data mining e tanto altro...

Cioè, riassumendo potevo scaricare e quindi guardare tutti i pdf inviati da tutti gli studenti dell'anno 2014 senza alcuna restrizione e senza alcuna autenticazione, **inconcepibile**.

## Segnalazione

Il mio primo pensiero ovviamente è stato quello di segnalare tale "mancanza" agli addetti, così contattai l'ersu tramite l'account twitter [@ersupalermo](https://twitter.com/ersupalermo) che cordialmente mi invitò a contattare il responsabile.   
2 giorni dopo l'invio dell'email il suddetto mi rispose che era stato risolto il problema e che se avessi trovato altro sarebbe stati felici di risolverlo. Fantastico!

## Dalle stelle alle stalle

Fiducioso andai a controllare il modulo in questione e notai che il nome era cambiato, `doc2014_<hashMD5>.pdf` all'interno della stessa cartella.

![what]({{site.image_url}}/bughunting-ersu-3.gif)

Ebbene sì, gli "addetti ai lavori" si sono limitati al cambiare SOLO il nome ai file hashandolo tramite MD5 rendendo univoco il documento (rendendo futile l'iterazione adottata prima), perfetto, ora è più difficile identificare il nome dei file ma non impossibile.
L'hash sarà sicuramente formato dal numero di pratica -che permette di identificare univocamente il documento- più qualche SALT particolare così da rendere difficile il decrypt di tale stringa che se scoperta (rainbow tables, brute force attack, dictionary attack) farebbe cadere la "protezione" adottata dagli sviluppatori chiamati in causa dall'ente in questione, il danno che ne conseguirebbe sarebbe troppo grande per lasciare come unica protezione il semplicissimo e banalissimo rename dei file usando un algoritmo di hashing ormai praticamente obsoleto.

## Occhio non vede...

Ormai atterrito dalla notizia precedente cercai di indagare meglio e notai subito che per accedere al modulo di autocertificazione creato dinamicamente (che aveva lo stesso problema di quello precedente essendo basato sempre sul numero di pratica, risolto allo stesso modo) prima di redirezionarmi al file in questione veniva fatta una richiesta alla pagina `ultimaPagina.php?idPratica=<IDPRATICA>`, cioè veniva effettuata una query al db cercando l'id della pratica e di conseguenza tutti i dati annessi a quella pratica riportando il tutto sul pdf.   
uhm...... `ultimaPagina.php?idPratica='`

> You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''''' at line 1

![comeon]({{site.image_url}}/bughunting-ersu-4.gif)

Avete capito bene, è bastato inviare un apice per mettere in ginocchio il sistema in questione rendendo possibile eventuali attacchi [Blind SQL Injection](http://en.wikipedia.org/wiki/SQL_injection) con cui è LETTERALMENTE possibile dumpare tutto il database con le giuste query.   
Se prima erano in pericolo "solo" i dati di tutti gli studenti che avevano fatto ricorso ora sono in pericolo TUTTI i dati presenti nel loro database, davvero inaccettabile.

## Riflessioni

Ovviamente ho segnalato tutti i problemi riscontrati e ho suggerito persino le possibili soluzioni; non ho mai ricevuto una risposta in proposito ma ora ricontrollando noto che il problema della Blind SQL Inj è stato risolto.

Questo mi fa capire che chiunque abbia creato il portale non si sia minimamente preoccupato di gestire gli input provenienti dall'utente né tanto meno di usare tecnologie adatte ad evitare ciò. (l'uso di ORM attraverso engine come PDO è ormai uno standard nel 2014, eh; usarli aiuta a risolvere l'85% di 'sti problemi)

Si tratta di tenere al sicuro milioni di dati sensibili, dati di milioni di studenti e famiglie.   
Come è possibile che ai giorni nostri debbano capitare situazioni del genere su sistemi tanto sensibili ? Da quanto erano lì i bug ?

Io non ho continuato a cercare eventuali bug, che potrebbero essere sfuggiti agli autori della piattaforma, ho fatto il mio dovere da buon "cittadino", cioè segnalare le falle ed aiutare a rendere un sistema così importante un po' più protetto; aiutare che anche i miei dati siano un po' più protetti; fatelo anche voi.

Ovviamente tanto di capello a quei signori che abbastanza velocemente mi hanno ascoltato in parte e che si sono messi a disposizione per riparare al danno, non tutti i mali vengono per nuocere e il mettersi a disposizione è già un primo passo.

Nei vari siti amministrativi che ho visitato ho sempre notato CARENZA di sicurezza, carenza che agli occhi di un qualsiasi utente possono essere anche non notate o addirittura ignorate; carenze che non passano inosservate a chi invece, ha un po' di cultura in materia. Cosa potrebbe succedere se tali informazioni cadessero in mano a persone che non la pensano esattamente come me ? Quali potrebbero essere le conseguenze ? Perché nessuno se ne preoccupa davvero ?

Viviamo in una era dove la privacy è importante; ogni giorno nascono sistemi creati appositamente per tutelare la privacy dei cittadini e quindi degli utenti e mi ribolle il sangue al sol pensiero che i miei dati invece vengano buttati lì, su un server qualsiasi con policy di sicurezza praticamente nulla come se fossero roba inutile, alla mercé di tutti; in questo mondo così materialista **io sono i miei dati** e se trattano male i miei dati è come se trattassero male me.

Saluti, DLion
