---
title: SmagenBot
description: Bot che sfrutta le API di telegram scritto in Nodejs
category: Programming
layout: post
---

Un po' di tempo fa [Telegram](https://core.telegram.org/bots) mise a
disposizione degli sviluppatori delle API per la creazione e la gestione di bots
che appoggiandosi al servizio consentivano di interfacciarsi allo stesso consentendo
ai bots di fare determinate azioni ricevendo comandi dagli utenti.

Le API sono molto semplici da usare, il tutto consiste in pochi passaggi, in primis
quello di contattare il bot padre per farsi dare un token, dopo di che è possibile
usare gli endpoint messi a disposizione da telegram per poter far compiere azioni
al bot.

Così decisi di creare il mio bot personale, **SmagenBot**.

Un semplice bot scritto in Nodejs basato su plugins, basta avviare il bot e man mano
aggiungere i vari plugins che con il tempo andrete creando per le vostre esigenze
attivandoli con il comando `/nomeplugin`.   
Questo sistema permette al bot di rimanere sempre online e di poterlo migliorare
con nuove funzioni semplicemente aggiungendo un file all'interno della cartella
appositamente creata.

Le funzionalità del bot in questione non sono ancora molte, forse la più utile è
proprio quella di poter settare chi il bot dovrà "servire" così da non rischiare
che altri utenti possano usarlo, usa nodejs quindi sfrutta le peculiarità async
del linguaggio ed è molto leggero perché i plugins li esegue sul momento anziché
caricarli tutti in memoria.

Il template dei plugins è molto intuitivo, basta semplicemente creare una funzione
exec con cui ritornerete una callback che conterrà due array contenenti nel primo
parametro un oggetto con i vari campi e nel secondo array il tipo di dato tornato;
che siano oggetti di photos, text, etc.   
Per maggiori informazioni basta vedere gli esempi contenuti nella cartella `plugins`

Il repo lo trovate qui: [smagenBot](https://github.com/dlion/smagenBot)

Per ora ci sono pochi plugins o almeno, quelli che uso io sono privati proprio perché
ho creato smagenBot per un uso personale.

![smagenBot](https://camo.githubusercontent.com/9a41999bf648a82ef806b235fa1ed5a8a2ede779/687474703a2f2f692e696d6775722e636f6d2f5a4d324d7a4b612e706e67)

Se volete suggerire nuovi plugins, migliorare il codice, segnalare bug, sentitevi
liberi di fare PR ed aprire issues.

Saluti, DLion.

---

[English Version](//domenicoluciani.com/past/programming/node/2015/07/14/smagenbot.html)
