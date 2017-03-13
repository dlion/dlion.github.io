---
title: Earthquake hackathon
summary: Creare uno stretto legame tra nuove tecnologie informatiche (sistemi distribuiti, Cloud, serverless, ecc) e le scienze della terra connesse alla prevenzione del rischio (sismico, idrogeologico e ambientale), allo scopo di incrementare il livello di salvaguardia della popolazione.
categories: Personal, Hackaton, Programming, Events
layout: post
---

![earthquakehackathon1](/images/earthquakehackathon_1.jpg)

Giorno 11 Marzo 2017 si è svolto presso la sede di [Mikamai](https://www.mikamai.com/) a Milano un hackathon davvero interessante a cui ho partecipato, sponsorizzato da Amazon Web Services e Cloud Alliance. Il tema principale erano i terremoti, il tutto usando tecnologie IoT && Serverless seguiti da un team di geologi qualificati.

![earthquakehackathon2](/images/earthquakehackathon_2.jpg)

Il programma della giornata prevedeva:

* 9:30 registrazione
* 10:00 presentazione dell'hackathon
* 10:30 coffee break offerto da Amazon Web Services con creazione team
* 11:00 sviluppo
* 13:30 pranzo offerto da Amazon Web Services
* 14:30 sviluppo
* 17:30 presentazione dei progetti da parte dei team
* 18:30 premiazione (con premi offerti da Amazon Web Services e Cloud Alliance)

## Presentazione, coffee break e formazione dei gruppi

![earthquakehackathon3](/images/earthquakehackathon_3.jpg)

La presentazione e l'apertura dell'hackathon ha chiarito fin da subito molti aspetti interessanti, in loco erano presenti diversi geologi che hanno spiegato in breve come funziona un terremoto, quali [onde generava](https://it.wikipedia.org/wiki/Onde_sismiche) e quali strumenti potevano essere usati per analizzarli.   
Una volta finita la presentazione iniziale c'è stato un piccolo coffee break in cui dovevamo conoscerci e formare i teams con cui intraprendere la sfida in questione; riuscendo a formare in fine 3 team da 2 sviluppatori e 1 geologo ciascuno.

## I sensori a nostra disposizione

Ad ogni team è stato dato un set di sensori e devices per affrontare la sfida, i sensori a disposizione erano 3:

* Geofono
* Sensore di vibrazioni
* Accelerometro

I devices a nostra disposizione erano principalmente 2:

* Arduino
* Raspberry Pi

## Idee

Una volta formato il team è giunto il momento cruciale: trovare l'idea.   
La linea guida dataci era più o meno questa:

> Costruire una rete di dispositivi low cost che in tempo reale trasmettano al sistema (su cloud) dati sui terremoti, permettendo di calcolare con anticipo l'arrivo delle onde distruttive nelle zone prossime all'epicentro.

Il sistema in questione quindi doveva prevedere il raccoglimento dei dati dai sensori per poi mandarli in cloud e calcolare (grazie all'aiuto dei geologi dal punto di vista teorico e di AWS dal punto di vista computazionale) in anticipo le onde del sisma.

![earthquakehackathon6](/images/earthquakehackathon_6.jpg)

![earthquakehackathon7](/images/earthquakehackathon_7.jpg)

![earthquakehackathon8](/images/earthquakehackathon_8.jpg)

## Verso la fine

I teams dopo un primo step di competizione son finiti per collaborare fra di loro, unendo le idee cercando di venire a capo alle problematiche sorte durante la giornata, come interfacciarsi ai dispositivi, quali soluzioni adottare per avere un sistema il più possibile realtime così da permettere di allertare in tempo tutte le persone coinvolte e i servizi da usare per calcolare il tutto.

## Riflessione

Purtroppo al giorno d'oggi ci è impossibile prevedere i terremoti, se si riuscisse a ricavare un pattern si riuscirebbe a salvare centinaia di vite all'anno, il problema maggiore è la mancanza di dati in merito. Al giorno d'oggi è quasi impossibile trovare dei dati utili per l'analisi dei terremoti dal punto di vista fisico parlando del "durante", cosa non vera per i dati post-terremoto come le vittime, i danni e altri dati di natura puramente numerica. Dati molto utili ma che comunque non permettono un'analisi "vera" di quello che è il fenomeno vero e proprio. Non conosco i motivi di questa "segretezza" né li concepisco, avere dei dati analogici dell'evento dall'inizio alla fine consentirebbe probabilmente di fare dei passi in avanti in questo ambito magari consentendo in futuro di permettere di salvare delle vite. Avere un grosso dataset di dati ci consentirebbe di fare analisi approfondite, magari usando tecniche di machine learning per capire in qualche modo come sfruttare tale analisi per fini efficaci, utili ad evitare tragedie (o quantomeno ci si può provare).

## Conclusioni

Una esperienza davvero positiva sotto ogni punto di vista, l'evento ha permesso a diversi professionisti e non di collaborare e imparare l'uno dall'altro, chi era esperto di cloud e AWS ma a secco di IoT ha avuto modo di vedere come usare le porte GPIO della Raspberry, al contrario chi era pratico di IoT ha potuto vedere come configurare e mettere online servizi AWS e per l'invio dei dati.

![earthquakehackathon4](/images/earthquakehackathon_4.jpg)

In premio i teams hanno ricevuto dei Kindle e le Raspberry Pi su cui hanno lavorato durante l'evento

![raspberrypi](/images/earthquakehackathon_5.jpg)

Ovviamente vi invito a consultare il sito ufficiale per [ulteriori info](http://www.earthquakehackathon.it/), a presto!