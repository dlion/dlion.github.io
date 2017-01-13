---
title: amatApiKiss
description: Un approccio fin troppo KISS per servire API
category: programming
layout: post
---
Di recente l'azienda municipalizzata auto trasporti di Palermo (AMAT) ha rilasciato degli open datas con le informazioni sui mezzi, gli orari, le rotte, l'azienda e molto altro così mi sono subito attivato per creare un piccolo tool per mettere a disposizione di chiunque voglia delle API REST per servire tali dati ed effettuare ricerche mirate.

Di proposito ho voluto usare un approccio KISS per puro divertimento, ciò significa che il sistema da me proposto non è assolutamente da prendere in considerazione per progetti a lungo termine né dovrebbe essere usato per avere una soluzione efficiente al problema. L'ho realizzato soprattutto per gioco.

## Specifiche

Ho deciso di realizzare il tutto usando Node.js perché penso si presti bene alla necessità di leggere e/o manipolare files in formato
json.

Per gestire le richieste e creare delle API REST ho deciso di usare [Restify](http://restify.com/); ho preferito restify ad hapi
o ad express perché mi consente di poter avere un sistema completamente customizzabile di routes che non mi vincoli troppo (vedi hapi) e
che allo stesso tempo si limiti a fornirmi delle funzionalità basilari per la gestione delle routes senza altre cose in mezzo.
(vedi express).
 
La peculiarità più interessante è senza ombra di dubbio quella di poter avere a disposizione delle routes dinamiche, infatti basterà
semplicemente aggiungere un nuovo file json alla directory `db` per avere una nuova route con nome il nome del file aggiunto. Per
esempio, se avete nella cartella `db` 3 files: `tracks.txt.json`, `shapes.txt.json` e `maps.txt.json` avrete 3 routes create in modo
dinamico chiamate `/tracks`, `/shapes` e `/maps`.
 
La parte dell'interazione con gli open data l'ho lasciata a [lowdb](https://github.com/typicode/lowdb), un semplice modulo che si limita
a darmi una mano nella lettura/scrittrura di files json e allo stesso tempo fornirmi delle API per la ricerca di elementi all'interno di
essi.
 
## Installazione

Clonate il repo che trovate qui: [amatAPIkiss](https://github.com/dlion/amatAPIkiss) digitando `git clone https://github.com/dlion/amatAPIkiss`,
entrate nella directory ed installate le dipendenze digitando `npm install`.
 
## Conversione
La prima cosa che bisogna fare in un primo uso è convertire gli open datas (organizzati come fossero csv) in files json così da poterli
usare con lowdb; per fare ciò ho creato all'interno della cartella `db` uno script che sfruttando [csvjson](https://github.com/pradeep-mishra/csvjson)
permette di convertire i files nella directory `db` in uno schema simile
 

{% highlight js lineanchors %}
{ "nomefile": [ { ... }, { ... }, { ... } ] }
{% endhighlight %}

ritrovandovi poi dei files *.txt.json (i nostri open datas convertiti).
 
## Avvio
 
Per avviare il tutto vi basterà digitare `node index` all'interno della root della directory. Di default il server sarà reperibile
all'indirizzo `127.0.0.1:3000`.
 
## Ricerca
 
Ho implementato un semplice sistema di ricerca basato sul sistema chiave/valore dei file json fornitomi. Per esempio, volete cercare
all'interno del file shapes tutti gli oggetti con `shape_id` uguale a `1010` ? Vi basterà visitare l'indirizzo
`127.0.0.1:3000/shapes/shape_id/1010` che darà in output i risultati trovati. In poche parole il tutto si riduce a:
`127.0.0.1:3000/<db>/<key>/<value>`.

