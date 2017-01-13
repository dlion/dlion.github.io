---
title: Trakt Progress
description: Client da linea di comando per tenere aggiornati i miei progressi su trakt.tv
category: Programming
layout: post
---
Recentemente ho rilasciato sotto licenza MIT un client scritto in Node.js a me molto utile, mi permette di tenere aggiornati i miei progressi su una determinata serie tv o anime; il client in questione mi dice qual è la puntata successiva che dovrò vedere, la percentuale di puntate viste fin'ora e cosa più importante, quando uscirà la prossima puntata.

Per fare ciò il client si basa sulle informazioni prese dal sito [Trakt.tv](http://trakt.tv/), quindi per poterlo usare dovrete effettuare alcune semplici azioni iniziali per configurare il client e renderlo "operativo".

Il client in questione ha una semplice interfaccia visuale che potrete direttamente usare dalla vostra shell, ecco uno screenshot per farvi una idea

![Screenshot](/images/traktprogress1.jpg)

Come potete notare una volta configurato esso prenderà la lista dei vostri anime con puntate in sospeso e ve le mostrerà in una semplice lista che potrete scorrere con le freccette; una volta vista la puntata vi basterà premere **enter** sulla puntata interessata e ripremere enter su **yes** all'interno del messaggio che vi comparirà a video per poter "settare" quella puntata come **vista**.

![Screenshot2](/images/traktprogress2.jpg)


## Installazione

Per installare il client in questione vi basterà digitare dalla vostra shell `sudo npm install traktprogress -g` così da installarlo globalmente all'interno della vostra macchina così da poterlo eseguire semplicemente digitando `traktprogress` da qualsiasi directory.

## Configurazione

* Prima di tutto dovrete crearvi -se non l'avete- un account su [trakt.tv](http://trakt.tv/) appuntandovi il vostro nickname.
* Dopo di ciò dovrete ovviamente cercare e aggiungere gli anime e i telefilm di vostro interesse fino alla puntata fin'ora raggiunta da voi. Ricordate di selezionare le intere stagioni come *viste* così da non avere inconsistenze con le stagioni vecchie. Una volta settato come *viste* vi basterà settare le puntate della stagione in corso come *viste* fin quando non raggiungete quella che v'interessa.
* Fatto ciò dovrete recuperare la vostra personale APIKEY che vi permetterà di prendere i dati dal sito, la trovate [qui](http://trakt.tv/api-docs/authentication) in neretto.
* Ora dovrete criptare la password associata al vostro account tramite l'algoritmo SHA-1, potete benissimo usare questo tool: [SHA-1 Encrypt](http://www.sha1-online.com/)
* Ora non vi resterà che settare tutto all'interno del file di configurazione che trovate in `/usr/lib/node_modules/traktprogress/traktprogress.conf.json`

Se tutto sarà andato per il verso giusto dopo aver digitato `traktprogress` dalla vostra shell avrete la schermata mostrata sopra con la lista dei vostri anime e con le puntate in pending. Ovviamente se raggiungerete il 100% e quindi senza puntate in pending l'anime scomparirà dalla lista lasciando spazio agli anime ancora in pending così da non riempire tutta la lista di anime ormai completati; una volta fatto tutto per uscire non vi rimarrà che premere `q`

Ovviamente il sorgente è disponibile online sul [repo dedicato](https://github.com/dlion/traktprogress) per eventuali segnalazioni di bug, miglioramenti, consigli, etc.

Saluti, DLion.
