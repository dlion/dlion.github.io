---
title: Node Mobile TIM
description: Modulo per visualizzare i dati della propria promozione TIM direttamente da shell
category: Programming
layout: post
---

Ultimamente non ho una connessione a casa e sono quindi costretto a destreggiarmi con quei pochi Gb che la promozione che ho attivato sul mio telefono mi fornisce; mi capita spesso di fare il tethering tramite usb che su slackware viene reso abbastanza semplice e comodo:

* Si inserisce il cavo usb
* Si abilita il tethering dal cellulare
* si diventa root tramite `su`
* e si dà il comando `dhcpcd usb0`

Da quel momento si è perfettamente connessi alla rete del proprio cellulare.

## Il problema

Uno degli inconvenienti di ciò è proprio quello di dover fare meno operazioni possibili per poter preservare la quantità di dati a disposizione, (nel mio caso solo 1Gb) come per esempio visitare facebook all'[indirizzo mobile](http://m.facebook.com) idem per [twitter](http://m.twitter.com) e per tutti gli altri siti; così dentro di me è scattato un vero e proprio bisogno di sapere spesso quanti Mb mi rimanessero; andare sul sito della [tim](http://tim.it) ogni volta significava sprecare altri preziosissimi Kb fra il caricamento delle immagini, js vari, etc. cosa che non potevo assolutamente permettere, così mi venne in mente di sviluppare un piccolo script che si loggava e mi prendeva i dati senza dover fare richieste inutili con una perdita maggiore di dati; e così fu.

## MobileTIM

Ho sviluppato in node.js [un modulo](https://github.com/dlion/mobiletim) che facesse ciò, una volta connessi con il vostro telefono -non ho sviluppato un sistema di autenticazione perché usando la vostra connessione l'autenticazione è automatica- vi basta eseguirlo per farvi spuntare tutti i vostri dati sulla vostra shell.

## Installazione

Potete installare il modulo da npm tramite il comando `sudo npm mobiletim -g`

o potete clonare il repo di github se non lo volete installare tramite il comando `git clone https://github.com/dlion/mobiletim`

## Uso

Per usarlo, una volta installato da npm vi basterà digitare `mobiletim` e attendere il risultato

o se avete clonato il repo vi basterà digitare `node index.js`.

## Screenshot

![screenshot](http://i.imgur.com/eUdZb89.jpg)

## Conclusione

Al solito per qualsiasi suggerimento, bug, etc. potete scriverlo qui sotto o aprire una issue su github all'indirizzo più sopra.

Saluti, DLion.

