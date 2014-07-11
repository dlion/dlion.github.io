---
title: Guida alla programmazione in Linux
description: GaPiL, Linux and beyond
category: Linux
---
Io uso giornalmente Linux e molto spesso essendo un programmatore ho bisogno di interagire con quest'ultimo ad un livello più "basso", un livello che i normali utenti vedono solo con il binocolo tramite [GUI](https://en.wikipedia.org/wiki/Graphical_user_interface) sbrilluccicose o animazioni flashose.

La programmazione mi permette di toccare con mano la parte viva del sistema, permettendomi di sfruttare le caratteristiche del mio sistema a mio piacimento.   
Per far ciò dovrei conoscere ogni singola funzione per ogni singolo utilizzo, no ?   
Direi che ciò è quasi impossibile, come potrei mai memorizzare centinaia di funzioni per poter plasmare il mondo in cui mi trovo ? Qui ci viene in soccorso il progetto **GaPiL**.

Si tratta di un progetto tutto italiano, un progetto in continua evoluzione ed espansione; [Simone Piccardi](http://piccardi.gnulinux.it/), autore del progetto GaPiL descrive così il suo progetto:
> GaPiL nasce dalla mia convinzione profonda che la "filosofia" che ispira il software libero si applichi anche ad altri campi che non siano necessariamente quelli della scrittura di programmi per computer. In particolare ritengo che possa assumere una grande rilevanza in ambiti come quelli dell'educazione e della formazione.   
Ma se trovare della buona documentazione libera, specie per quanto riguarda i programmi che girano sul sistema GNU/Linux, è ormai relativamente facile, la produzione di buoni testi didattici che insegnino a programmare in questo sistema è ancora molto limitata, soprattutto se li si cercano in lingua italiana.   
GaPiL è un tentativo di scrivere un manuale di programmazione di sistema in ambiente di tipo Unix, con una particolare attenzione alle caratteristiche specifiche delle interfacce fornite dal kernel Linux. Per questo motivo si parla di Linux e non di GNU/Linux.   
Nonostante questa specificità, essendo la gran parte delle funzioni di sistema standardizzate, la guida dovrebbe risultare utile anche facendo riferimenti ad altri sistemi di tipo Unix come i vari *BSD; in ogni caso si sono sottolineate esplicitamente le caratteristiche specifiche di Linux.   
Benché buona parte della trattazione delle funzioni di libreria sia del tutto identica, facendo riferimento a standard generali come POSIX, si è comunque prestata particolare attenzione alle funzioni delle GNU libc, che sono la versione più usata delle librerie del C, senza dimenticare, ove note, di citare le differenze con possibili alternative come le libc5 o le uclibc.   
L'obiettivo resta comunque quello di riuscire a produrre un testo, rilasciato sotto GNU FDL, che possa servire a chi si accosta per la prima volta alla programmazione avanzata e di sistema su un kernel Linux, con la speranza di poter un giorno raggiungere la qualità dei lavori del compianto R. W. Stevens.

In poche parole il progetto GaPiL non è altro che un enorme manuale del sistema operativo Linux per i programmatori del kernel.   
Contiene al suo interno descrizioni di funzioni utili, funzioni che ogni giorno noi utenti linuxari usiamo sulla nostra shell, che i programmi che usiamo usano tacitamente, su cui è basato l'intero sistema.   
GaPiL è disponibile anche online così da poterlo consultare senza il bisogno di scaricarlo.

Certo, magari non è l'unica fonte di sapere; certo, magari non sostituirà le [man pages](http://it.wikipedia.org/wiki/Man_%28Unix%29) o altri progetti più grandi ma è comunque un buon inizio per qualcosa di più ampio.

Il link al progetto è il seguente: [GaPiL](http://gapil.gnulinux.it/)

Per esempio, voglio usare in un mio programma la funzione kill per terminare un dato processo, ecco uno spezzone del capitolo dedicato a questa funzione:
> quando si vuole inviare un segnale generico ad un processo occorre utilizzare la apposita system call, questa può essere chiamata attraverso la funzione kill, il cui prototipo è:   

{% highlight c %}
#include <sys/types.h>    
#include <signal.h>   
int kill(pid_t pid, int sig)
{% endhighlight %}

> Invia il segnale sig al processo specificato con pid.   
La funzione restituisce 0 in caso di successo e -1 in caso di errore nel qual caso errno assumerà uno dei valori:   
EINVAL Il segnale specificato non esiste.   
ESRCH Il processo selezionato non esiste.   
EPERM Non si hanno privilegi sufficienti ad inviare il segnale.   
Lo standard POSIX prevede che il valore 0 per sig sia usato per specificare il segnale nullo. Se la funzione viene chiamata con questo valore non viene inviato nessun segnale, ma viene eseguito il controllo degli errori, in tal caso si otterrà un errore EPERM se non si hanno i permessi necessari ed un errore ESRCH se il processo specificato non esiste. Si tenga conto però che il sistema ricicla i pid (come accennato in sez. 3.2.1) per cui l'esistenza di un processo non significa che esso sia realmente quello a cui si intendeva mandare il segnale.   
Il valore dell'argomento pid specifica il processo (o i processi) di destinazione a cui il segnale deve essere inviato e può assumere i valori riportati in tab. 9.4.

Cosa ne pensate di tale progetto ?   
Io lo trovo più che utile, sia dal punto di vista didattico che "utilistico" e spero che anche voi come me possiate apprezzare lo sforzo e la voglia di creare qualcosa di utile per tutti.

Saluti, DLion.
