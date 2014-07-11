---
title: Vedere video con la Raspberry pi dalla riga di comando
description: Come vedere video con la vostra Raspberry comodamente seduti sul divano
category: Raspberry
---
Stasera mi piacerebbe vedere un bel film in tv, uhm, non fanno nulla d’interessante.   
Be' allora opterò per un film, ma ho da fare al computer e il televisore è troppo lontano per attaccare il pc...   
Idea! Perché non usare la raspberry ?

Ah, non ho una tastiera usb a portata di mano ed anche se fosse, non ho voglia di alzarmi per gestire il film. Però a pensarci bene ho settato la mia raspberry pi in modo tale da avere un indirizzo statico, il server ssh attivo e la raspbian customizzata a dovere per non dover collegare una tastiera fisica al dispositivo ma non ho X attivo né configurato; dalla linea di comando entro via ssh e faccio quello che mi pare, ma **come li vedo i film!?**

Questo dubbio mi ha attanagliato per un po’, mi chiedevo:

* Se non ho il server X attivato e/o se non scrivo direttamente dalla raspberry con la tastiera fisica come posso guardarmi un film sul mio televisore senza troppi problemi ?

La risposta è arrivata dopo un paio di _ricerche_.

Se avete una raspbian e volete vedere video con la raspberry pi dalla riga di comando, comodamente dal vostro pc vi basterà usare il software: **omxplayer**   
Questo software già presente sulla raspbian vi permette di farlo.

Collegate all'uscita hdmi il vostro televisore, entrate via ssh e scegliete il video da vedere e digitate: `omxplayer -r -o hdmi <video>`

Vedrete spuntare l'immagine sul vostro schermo, anche se non c'è X attivo, se non avete configurato il DE/WM o qualsiasi altra cosa che riguardi la parte grafica.   
_Io per sicurezza tramite rpi-config ho dato alla gpu più memoria rispetto ai 64 di default così da non esserci problemi in caso di film esosi._

Nulla di complicato, l'opzione `-o` dice al software di redigere l'audio verso l'hdmi, l'opzione `-r` dice al software di adattare il video alla risoluzione dello schermo e in caso vi servissero maggiori info l'opzione `-h` arriva in vostro soccorso. Così potrete godervi i vostri film senza usare roba troppo complicata come [XBMC](http://xbmc.org/) o altri gestori multimediali.

Saluti, Dlion
