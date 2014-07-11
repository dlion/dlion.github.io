---
title: Aggiornare la Raspberry Pi
description: Aggiornare la vostra Raspberry Pi
category: Raspberry
layout: post
---
Ok un post sulla raspberry pi ci voleva dopo tutto 'sto tempo.   
Dato che lo sto facendo in questo preciso istante voglio descrivervi la procedura per aggiornare la vostra raspberry pi che effettuo io.

Ora sto usando la distro raspbian, cioè la vecchia e cara debian ottimizzata per la raspberry.   
Intanto cosa ci serve per cominciare ?

* Una raspberry pi, genius!
* Dentro c’è raspbian, right ?
* Una connessione ad internet, sì ok sto prendendo tempo
* Ringraziare gli dei/spiriti/occhicchesia perché vada tutto a buon fine.

Intanto come è di giusto vi raccomando di **fare un backup** dei vostri dati importanti in caso di errori vari. (tanto lo so che non lo farete) Il procedimento può bloccarsi, possono mancare dipendenze, si può rompere qualcosa nel vostro sistema (un po' come in qualsiasi distro) quindi tenete gli occhi ben aperti.

* Iniziamo con l'aggiornare la definizione dei pacchetti in nostro possesso dando un bel `sudo apt-get update`
* Una volta scaricato il tutto date un bel `sudo apt-get dist-upgrade`
* Dopo di che, io di solito accedo al menù contestuale messo a disposizione da raspian con il comando `sudo raspi-config`
* Scendo in basso e premo invio su "update"
* una volta completato premo su "finish"
* riavvio.
* Se è andato tutto bene diamo il comando `sudo -s` per avere i privilegi adatti
* digitiamo `wget http://goo.gl/1BOfJ -O /usr/bin/rpi-update` per scaricare la versione aggiornata di [rpi-update](https://raw.github.com/Hexxeh/rpi-update/master/rpi-update) che permette di aggiornare il firmware. - AGGIORNAMENTO: Ora rpi-update è presente nel repo di debian quindi per installarlo non dovrete far altro che digitare `sudo apt-get install rpi-update`
* digitiamo `chmod +x /usr/bin/rpi-update` per renderlo eseguibile.
* Poi sempre da root digitate `rpi-update`

Una volta finito questo procedimento (verifichiate che sia stato fatto tutto in modo corretto e senza errori) riavviate pure e godetevi la vostra raspberry pi aggiornata.

Saluti, DLion
