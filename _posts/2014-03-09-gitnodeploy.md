---
title: GitNoDeploy
date: 2014-03-09 11:00:00
summary: Il deploy non è mai stato così semplice!
categories: Programming
layout: post
---
Dovendo lavorare molto in locale uno delle parti più noiose è il dover caricare i file aggiornati su ogni server su cui dovrà girare quella determinata applicazione.    
Pensai subito ad una soluzione rapida ed efficiente per il deploying dei miei progetti all'interno dei server a mia disposizione in modo del tutto automatico.

Così scrissi un piccolo script in node.js che mi dava la capacità di deployare qualsiasi software all'interno di qualsiasi server io volessi con un comando.

Nacque **GitNoDeploy**.

## Cosa è ?

Come già detto GitNoDeploy è un piccolo script in node.js che permette di effettuare un deploy dei propri file su tutti i server in cui gira.

## Come funziona ?

Il suo utilizzo è semplice.   
Si carica lo script sul server, si configura tramite l'apposito file di configurazione e si lascia in ascolto sulla porta 9000; per fare ciò basta modificare il file di configurazione lasciato di default indicando:

* L'url del repo da cui aggiornare il codice.
* La path della cartella da aggiornare.
* La porta di ascolto con l'eventualità di sopprimere qualsiasi messaggio a video.
* Se vogliamo possiamo pure impostare un comando da far eseguire allo script una volta effettuato l'aggiornamento.

Io uso moltissimo github così decisi di affidarmi alle sue WebHooks; una volta creato il repo si va nelle impostazioni dello stesso e troviamo le "WebHooks", all'interno di questa sezione possiamo aggiungere a nostro piacimento l'url a cui inviare le richieste; ovviamente metteremo l'url su cui gira GitNoDeploy seguito dalla porta (es: http://servermio:9000) settando come Payload version il `application/vnd.github.v3+json` e come opzione `Just the push event` così da dire a Github di inviare l'avviso di aggiornamento al server indicato, sulla porta indicata in formato json ad ogni push (aggiornamento) sul repo; fatto ciò non vi resta che salvare.

Da quel preciso istante ad ogni push effettuato su quel repo comporterà una richiesta inviata a GitNoDeploy che aggiornerà il repo da voi indicato per poi - se indicato - eseguire un comando.

Tutto ciò può essere fatto su tutti i server che volete, gli caricate lo script, lo eseguite ed indicate a github a quale url inviare le richieste ed il gioco è fatto, lo script aggiornerà le directory all'interno dei server con l'ultimo commit disponibile sul vostro repo, niente di più efficiente e gratuito.

Per ottimizzare le cose io uso anche [forever.js](https://github.com/nodejitsu/forever) che mi permette di demonizzare un qualsiasi script in node.js e quindi di metterlo in background consentendomi di salvare log di ogni genere.

Il codice ovviamente è reperibile su Github al seguente indirizzo: [GitNoDeploy](https://github.com/dlion/GitNoDeploy)

Come al solito, se avete suggerimenti, critiche, dubbi, bug da segnalare ed altro fate un fischio.

Saluti, DLion.
