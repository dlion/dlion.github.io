---
title: ExamProject Parte 0
description: Il mio progetto d'esame sulla visione artificiale
category: Programming
---
Oggi vi voglio descrivere il mio progetto d’esame per la maturità che ho sostenuto lo scorso anno sulla visione artificiale.   
Mi ha fruttato un bel 30. (il massimo)   
La possibilità di far “vedere” alle macchine, la possibilità di dare degli occhi alle nostre piccole creature artificiali e permettergli di effettuare delle azioni in base a ciò che vedono, non vi sembra sia interessante ?

Il progetto è suddiviso in vari software da me scritti che sfruttano le famosissime librerie di Computer Vision, [OpenCV](http://it.wikipedia.org/wiki/OpenCV).   
Per il momento non vi farò un corso di come usare tali librerie, se in futuro farò qualcosa di simile è perché ho tempo per farlo; per il resto potete documentarvi sul [sito ufficiale](http://opencv.willowgarage.com/wiki/) che vi aiuta molto.

I software che andremo a vedere sono 5, software scritti in C in parecchi mesi di lavoro e testati su [Slackware](http://it.wikipedia.org/wiki/Slackware).   
Le componenti essenziali per far funzionare il tutto sono:

* Ovviamente le librerie OpenCV installate sulla macchina
* Una webcam, possibilmente con una buona risoluzione
* A volte potrebbero servirvi dei colori, tipo degli evidenziatori
* Un luogo ben illuminato
* Linux

Non avevo idea di come chiamare il progetto quindi lasciai **ExamProject** (alla faccia della banalità, eh?)   
Vediamo in generale di cosa si tratta il progetto in questione:

1. Il primo software l’ho chiamato **"calibra"**, questo software è la base per poter usare i prossimi due software.   
Questo programma ci consente di _trackare un colore a nostra scelta_, che sia di un verdino chiaro, che sia un rosa scuro, questo programma ci permetterà di scegliere una determinata colorazione e di seguirla all’interno dell’immagine presa dalla webcam.   
In poche parole il computer riesce a capire dove sta quel colore all’interno dell’immagine e di seguirla per tutto il ciclo di vita del programma.   
Questo primo programma dovrà essere eseguito prima di tutti gli altri per poter memorizzare il colore nelle condizioni di luce in cui ci troviamo in quel preciso momento.
2. Il secondo software che ho scritto l’ho chiamato **"Bubbles"**, è un gioco da me ideato con vari livelli di difficoltà crescenti.   
Lo scopo del gioco è scoppiare tutte le palline rosse senza toccare quelle blu.   
Il tutto con il solo ausilio di un colore da noi scelto in precedenza.   
Usare l’oggetto colorato come se fosse un joystick. (tipo la wii?)   
Non fatevi ingannare dalla semplicità, è stato piuttosto arduo scriverlo.
3. Il terzo software l’ho chiamato **"Draw"**, usa lo stesso criterio del color tracking dei 2 software precedenti, permettendoci di colorare sullo schermo.   
Esattamente, usare il nostro oggetto colorato come se fosse un pennello e basta premere un tasto per cancellare la "lavagna".
4. Il quarto software l’ho chiamato **"Head"** è un software che utilizza gli _Haar Cascade File_ per riconoscere il volto di una persona all’interno dello schermo. (Fantasia portami via)   
Ho sviluppato anche un modo per identificare anche gli occhi, il naso e la bocca usando il _ROI_ (Region Of Interest) insieme ai Cascade, che però non ho voluto usare per motivi più o meno di tempo.
5. Il quinto software l’ho chiamato **"Movement"**, in pratica si tratta di un software di monitoraggio che offre 2 modalità:   
La prima modalità permette di scattare la foto a qualcosa e di monitorarla.   
Se quell’immagine cambia anche solo di un pixel il software in questione creerà un video, un video contenente *SOLO* i cambiamenti.   
Al termine del software avrete un avi (quindi un video compresso) con la data e l’ora. (utilizzando anche il supporto ad ffmpeg)   
La seconda modalità consente di registrare i movimenti in un determinato streaming.   
Cioè, lasciate la webcam accesa ovunque ed essa registrerà solo i movimenti rilasciandovi un video alla fine.   
Dimenticatevi ore e ore di filmato privo di nulla, con questo software potrete registrare giornate intere ed alla fine avere solo i cambiamenti che sono stati rilevati.   

Più o meno la descrizione generale è questa, spero che vi abbia incuriositi.   
Negli articoli a seguire spiegherò nel dettaglio i software da me ideati e scritti, magari fornendo anche documentazione aggiuntiva, esempi ed immagini.   
Ovviamente accetto consigli di ogni genere da chiunque abbia voglia di aiutarmi.


Saluti, DLion
