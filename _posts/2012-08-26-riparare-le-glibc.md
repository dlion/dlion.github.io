---
title: Riparare le glibc su Slackware
description: Riparare in modo sicuro le glibc
category: Linux
layout: post
---
Su Linux esistono tante librerie/pacchetti importanti che permettono il corretto funzionamento del sistema, alcune fra queste sono le glibc.   
Può capitare malauguratamente che certe librerie durante un aggiornamento, una modifica o qualsiasi altra cosa vengano danneggiate.   
(Mi è capitato proprio ieri armeggiando con la nuova versione delle librerie multilib, ho risolto il problema e quindi mi è venuto in mente di scrivere qualcosa al riguardo)

Una volta che le librerie vengano "rotte" sarete impossibilati dal fare **QUALSIASI COSA**, dal semplice `ls` al comando più complesso, la console non vi sarà più da aiuto.   
A questo punto il cosiddetto **FORMATTONE** pensate sia più che dovuto ma a che serve avere una distro linux (Slackware, e che distro!) se ad ogni minimo problema dovete formattare?   
(Non vi ricorda un certo OS di nostra conoscenza ? No ? Vi do una mano, finisce con dows e inizia con Win)

Bene, vediamo il procedimento per riuscire nell’impresa di riparare il tutto e far tornare funzionante la nostra bella distro.

## Download del mirror
Prima di tutto dovrete scaricarvi l'intero mirror della versione della vostra distro altrimenti usare il dvd d'installazione se l'avete.   
Io per esempio avevo il dvd della current a 64 bit che mi sono masterizzato poche settimane fa (ho usato un dvd riscrivibile così da aggiornarlo una volta ogni tanto)   
Potete trovare le iso delle varie versioni qui: (http://ftp.heanet.ie/mirrors/ftp.slackware.com/pub/slackware/) oltre ai vari mirror che potete benissimo scaricare sul vostro sistema.   
A questo punto che avete il cd o la copia della slackware su un supporto esterno/interno vi serve una distro da cui "sistemare", ci viene in aiuto la versione live della distro **Salix**, una derivata della Slackware che mette a disposizione i suoi tool d'installazione che ci saranno utili al fine di riparare il danno; ho scaricato la versione LIVE, quella con XFCE, leggera ed ottima, la trovate qui: (http://www.salixos.org/wiki/index.php/Download_Xfce)

## Procedura di boot con la live
Una volta scaricata ed installata su un cd/pendrive/quellochevoletevoi potete riavviare il tutto. (Dovrete riavviare manualmente con il bottoncino d'accensione dato che il comando "reboot" non vi funzionerà)   
Una volta riavviato il computer dite al vostro bios di far partire prima il cd/pendrive e dopo tutto il resto, in questo modo la distro live viene avviata e caricata, vi verrà chiesto di premere continuamente `CTRL-D`, fatelo senza problemi, è normale e se tutto funziona perfettamente vi ritrovate con la distro attiva, connessa ad internet e che vi vede tutti i dispositivi.

## Dentro la live
Avviate la console e digitate: `su` mettendo come password `live` , ora sarete **root**, spostatevi (comando `cd`) in `/mnt/` qui trovate le cartelle `hd` e `cdrom`, digitate `fdisk -l` che vi mostrerà tutti i dispositivi a cui potrete accedere.   
Il mio sistema Linux era `/dev/sda4` mentre il cdrom `/dev/sr0` a quel punto dovremmo montare il tutto con `mount /dev/sr0 cdrom/` (Abbiamo montato il dvd) e `mount /dev/sda4 hd/` (Abbiamo montato la partizione contenente la nostra Slackware danneggiata) a quel punto digitate questo comando: `rm -rf /hd/lib64/incoming`; **ovviamente a `lib64` dovrete sostituire lib se avete una 32 bit, io ho una 64bit.** Poi sempre da console date `ROOT=/mnt/hd` **state attenti allo slash finale**, **non deve esserci**, poi date `export ROOT`.   
Abbiamo alterato la nostra radice che punterà alla radice della nostra partizione, in poche parole `/mnt/hd` sarà il nostro nuovo `/`

## Ripariamo
Adesso inizia la parte più delicata, la sostituzione dei pacchetti danneggiati, digitate:
{% highlight sh linenos %}
upgradepkg --reinstall --install-new cdrom/slackware64/a/glibc-solibs-*.txz
upgradepkg --reinstall --install-new cdrom/slackware64/a/glibc-zoneinfo-*.txz
upgradepkg --reinstall --install-new cdrom/slackware64/l/glibc-*.txz
{% endhighlight %}

Ovviamente `cdrom/slackware64` è la directory dove risiede il mirror della mia distro installata/scaricata su dvd.

Se tutto sarà andato a buon fine le librerie saranno stato reinstallate e dovrebbero funzionare, per verificarlo date il comando `chroot hd/ /bin/sh -l`, se funziona sarete dentro la vostra Slackware, anche se non sembrerà voi starete armeggiando con la vostra distro come se foste all'interno di essa normalmente, quindi se dando il comando "ls" tutto funziona significa che avrete risolto il problema.   
Io ho voluto esagerare e ho dato un bel `slackpkg upgrade-all` così da aggiornare la mia slackware da fuori così da finire per bene l'aggiornamento che avevo interrotto e in caso aggiornare le librerie vecchie installate (avendo una current avevo le libgc più vecchie di quelle disponibili), uscite con `exit` e dopo riavviate la live Salix; rimuovete il cdrom/penna usb e fate partire la vostra distro.

Dopo aver riavviato la mia distro ha ripreso a funzionare, vi scrivo proprio da lì.   
Spero vi sia stata utile.
Saluti, DLion
