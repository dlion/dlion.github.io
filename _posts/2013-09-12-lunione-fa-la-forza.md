---
title: L'unione fa la forza
layout: post
description: Introduzione a Git
---
Mi ero ripromesso di non fare i soliti HowTo, le solite guide o le solite cose che si possono trovare praticamente ovunque ma oggi farò uno strappo a questa regola.   
Nel *2013*, ancora c'è gente che è all'oscuro di cosa sia **git**, perché esiste **github** e come funzioni il mondo oltre il loro **IDE DevC++** datato 1900; avete capito bene, esistono sviluppatori, gente che studia informatica, gente più o meno pratica che non ha mai sentito parlare di git, **inammissibile**.   
Ebbene sì, oggi spiegherò in modo più o meno esaustivo come funziona git e del perché esiste tale strumento.

Git in breve è un software, scritto da [Linus Torvalds](http://it.wikipedia.org/wiki/Linus_Torvalds) che permette la gestione di un progetto in modo distribuito.

## Cosa Significa ?
Pensate al kernel [Linux](http://it.wikipedia.org/wiki/Linux), il Kernel Linux è un agglomerato di centinaia di cartelle che contengono centinaia di file con dentro milioni di righe di codice; il tutto gestito da milioni di sviluppatori nel mondo che devono implementare, correggere e migliorare ciò che si trovano davanti, tutto questo in diversi momenti della giornata.

Non spingiamoci troppo oltre; immaginate di dover sviluppare un progetto in team con degli amici o con dei colleghi, immaginate di dover creare un progetto che prende molto tempo e che dovreste gestire per lunghi periodi, immaginate una qualsiasi cosa che richieda modifiche mantenendo la possibilità di tenere aggiornato chiunque stia "guardando" il progetto in questione.

**Attenzione**, qui non si parla solo di sviluppo per programmatori, può essere anche il solo l'editare un articolo, correggere un esercizio ad un amico, segnarsi appunti, etc.

Tutto questo e molto altro è **Git**

## Cosa posso farci ?
Vediamo in breve cosa è possibile fare con git:

* Pensate alla possibilità di creare una funzione e di scrivere di vostro pugno ciò che avete fatto e come funziona, cioè git vi dà la possibilità di "commentare" il vostro operato di volta in volta creando una vera e propria storyline del vostro progetto.
* Pensate alla possibilità di creare più "rami" del progetto a cui lavorate permettendovi di lavorare su più "copie" del progetto permettendovi di creare nuove funzionalità senza alterare il progetto principale.
* Pensate alla possibilità di dividervi i compiti all'interno del team lavorando a "parti" di progetto differenti contemporaneamente e di poterle unire alla fine dello sviluppo nel progetto finale.
* Pensate alla possibilità di essere sempre aggiornati sullo sviluppo del progetto, di sapere *ESATTAMENTE* quali righe sono state modificate, cosa è stato tolto e cosa è stato aggiunto il tutto correlato da commenti, il tutto fatto da voi o dai vostri collaboratori.
* Pensate alla possibilità di ritornare indietro ad una particolare modifica del codice, non una modifica fatta oggi, fatta ieri o fatta 2 giorni fa, una modifica anche fatta 3 mesi fa da un vostro collaboratore.
* Pensate alla possibilità di conoscere in ogni momento quale file avete modificato, quale file dev'essere "commentato" e cosa avete fatto fin'ora rispetto al progetto iniziale.
* Pensate alla possibilità di fare TUTTO in locale e solo dopo che sarete certi del vostro operato metterlo a "disposizione" degli altri membri del team.

Incredibile, non credete ? Una volta che avrete cominciato ad usarlo *non potrete più farne a meno*.

## Come funziona ?
Per usare git ci sono due principali metodi:

1. La **Shell**
2. La **GUI**

Il primo metodo è quello di usare la shell, ostica inizialmente ma indispensabile alla fine, il secondo metodo è quello di usare un programma che sfrutta l'interfaccia grafica automatizzando ciò che andrete a scrivere da Shell.

Vediamo una breve introduzione da shell per usare git:   
Ovviamente non vi dirò come installare git, [Google è vostro amico](http://lmgtfy.com/?q=installare+git).


Per iniziare ad usare git dovrete indicare la cartella "base" del vostro progetto, cioè dove risiederà il vostro progetto.   
Quindi creiamo la cartella con il nome "pippo": `mkdir pippo`, dopo di ciò entriamo all'interno della cartella ed inizializziamo il progetto con `git init`.   
Fatto ciò all'interno della cartella verrà creata una directory chiamata `.git` questa conterrà tutto il necessario per permettere a git di funzionare e di fare le sue piccole magie, ovviamente non toccatela, sarà git stesso a metterci roba, modificarla,etc.

Ora creiamo un file e chiamiamolo `README.md`, editiamolo e scriviamoci dentro salvando le modifiche.   
Dopo di ciò dobbiamo dire a git che abbiamo aggiunto un file nuovo al nostro progetto con il comando `git add README.md` e commentiamo il nostro operato o "committiamo" le nostre modifiche con `git commit -m "Ho creato il file README.md"`.

Ecco fatto, nella nostra shell cui verrà restituito un feedback con i file che abbiamo "cambiato".   
Ora proviamo a modificare il nostro README aggiungendo una riga al nostro file e dopo digitiamo `git diff README.md`, a schermo avremo qualcosa di simile:
{% highlight sh %}
diff --git a/README.md b/README.md
index 121bce1..c0e4839 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@
Ciao io sono un file README
+Ho appena Aggiunto una nuova riga
{% endhighlight %}

Questo ci mostra ciò che abbiamo modificato all'interno del file, cioè il file rispetto al progetto corrente cosa ha di diverso.   
Le modifiche ci vanno bene ? Aggiungiamo la nuova versione del file e commentiamola con `git add README.md ; git commit -m "Ho aggiunto una nuova riga"`.

Se sentiamo il bisogno di sapere ciò che abbiamo fatto nel corso del tempo ci basta digitare `git log` per vedere tutti i commit che abbiamo effettuato, cioè ogni nostra "aggiunta/modifica" fatta al progetto con la data, l'ora e chi ha fatto quella determinata modifica.

Ora facciamo qualcosa di più complesso, il **branch**, abbiamo bisogno di aggiungere "una funzionalità" al nostro progetto ma di lasciare il progetto originale invariato così da poterlo aggiornare mentre noi sviluppiamo tale feature.

Creiamo un "ramo" o "branch" del progetto chiamandolo "sviluppo" con `git branch sviluppo`, a questo punto vediamo i nostri rami con `git branch`; ne vedremo due: il **master** cioè il ramo principale e **sviluppo**; vedremo un `*` accanto al branch in cui saremo posizionati.   
Il nostro interesse è sviluppare nuove funzioni lasciando invariato il tutto così dobbiamo "switchare" o spostarci sul nuovo ramo con `git checkout sviluppo` ridigitando `git branch` vedremo che questa volta l'asterisco sarà accanto a `sviluppo`; cominciamo ad apportare le nostre modifiche.

Creiamo un file nuovo chiamandolo "SECONDO.md", scriviamoci dentro e cancelliamo la prima riga di README.md sostituendola con un'altra scritta, alla fine aggiungiamo e committiamo il tutto con `git add README.md SECONDO.md ; git commit -am "Ho fatto roba varia in sviluppo"`; fatto ciò verra restituito il feedback di ciò che abbiamo fatto; come abbiamo visto possiamo aggiungere e commentare più file contemporaneamente così da "raggruppare" le varie modifiche, infatti l'opzione `-a` sta per "all", dopo di ciò switchamo di nuovo nel ramo principale per vedere cosa è successo con `git checkout master` e **TADAN!** il tutto è rimasto invariato, nessun SECONDO.md, nessuna modifica al file, niente di niente.

Però ora vogliamo aggiungere le nostre modifiche al ramo principale perché siamo sicuri che ciò che abbiamo fatto funzioni e che debba essere messo, così effettuiamo un "merge" dei due rami con `git merge sviluppo`, il vostro terminale vi restituirà qualcosa come:
{% highlight sh %}
Updating f0a407c..02bc2c1
Fast-forward
README.md  | 2 +-
SECONDO.md | 1 +
2 files changed, 2 insertions(+), 1 deletion(-)
create mode 100644 SECONDO.md
{% endhighlight %}
e per "magia" tutto ciò che si trovava nel ramo sviluppo è stato "unito" al ramo principale.

Questa era la cosa dal punto di vista della shell, sicuramente molto utile dal punto di vista didattico ma come già detto non è l'unica via. Esistono parecchi programmi che consentono di usare git attraverso una interfaccia grafica, tra questi ci sono:

* [SmartGit] (http://www.syntevo.com/smartgithg/)
* [SourceTree] (http://www.sourcetreeapp.com/)
* [GitHub] (http://github.com/)
* [Code School] (http://try.github.io/levels/1/challenges/1)

Davvero molto ben fatti e vale la pena provarli.

## Cosa è GitHub ?

* Pensate ad un social network che vi permette di usare git e di tenere tutto online.
* Pensate ad un social network che vi consente di dare visibilità ai vostri progetti open source.
* Pensate di trovare un bel progetto open source e di volerlo migliorare dando il **vostro contributo**.
* Pensate ad una comunità che vi aiuti a mantenere, sviluppare e testare il vostro progetto rendendolo addirittura *famoso*.

Questo e molto altro è [GitHub](http://github.com/).

GitHub è uno strumento **FONDAMENTALE** per chi vuole visibilità nel mondo *OpenSource* o per chi vuole semplicemente mettere i propri progetti alla portata di tutti.   
Vi chiederete "ma a che pro mettere i nostri progetti a disposizione di tutti?"   

Due parole: "**Open Source**".   
Pensate ad un programma che usate spesso, un programma con cui vi trovate bene e che vi piace però notate che manca qualcosa, che qualcosa è fatta male o che potrebbe essere migliorata in qualche modo, pensate alla possibilità di "prendere" questo programma, **modificare quello che volete**; aggiungendo ciò che vi sembra migliore, **migliorando** ciò che vi sembra poco adatto, togliendo ciò che ritenete inutile, pensate alla possibilità di inviare queste modifiche all'autore del programma, pensate alla possibilità di migliorare ciò che già c'era prima e alla possibilità di far spuntare il vostro nome fra i "collaboratori" rendendo il programma in questione non solo come lo volete voi ma migliore per tutti.   
Questo è Open Source e questo è quello che vi permette di fare GitHub.

GitHub vi mette a disposizione infiniti "spazi" dove poter caricare i vostri progetti, tutto gratis con l'unica condizione che tali progetti **devono essere pubblici**, se volete privacy dovrete pagare o usare qualcosa di diverso.   
Dopo aver "pubblicato" il vostro progetto **CHIUNQUE** potrà "forkarlo" cioè "copiarlo" nel proprio spazio ed apportare le modifiche che più  aggrada alla propria copia del progetto in questione, vi potrà segnalare dei bug ed inviarvi le sue modifiche (pull request) che potrete "unire" al vostro programma migliorandolo. Ovviamente voi potrete fare lo stesso con tutti i progetti degli altri utenti.

Oltre ad essere un ottimo strumento avrete a disposizione un posto dove poter "conservare" i vostri progetti e farli vedere, un po' come un curriculum online di ciò che avete fatto; molte aziende lo richiedono.   
Questo è il mio, per farvi un esempio: [https://github.com/DLion](https://github.com/DLion), ovviamente Github non è solo questo, c'è molto altro, non vi resta che scoprirlo da soli!

## Un repository privato
Abbiamo detto che GitHub offre gratuitamente **SOLO** repository pubblici dove poter inserire i propri progetti, aggiornarli,etc.   
Per i progetti privati io personalmente uso [BitBucket](https://bitbucket.org/) perché mette a disposizione infiniti repository privati.   
Cioè permette le stesse cose che fa GitHub bene o male ma fornisce gratuitamente anche "spazi" o "repository" non visibili da chiunque ma solo dai vostri collaboratori o da voi. Ma come potete ben capire non fornisce la stessa visibilità che fornisce GitHub.

Abbiamo parlato di GitHub, di BitBucket ma non abbiamo detto che **CHIUNQUE** può avere il proprio git all'interno di un qulsiasi server, facciamo l'esempio di quello aziendale quindi raggiungibile solo dal team di sviluppo, magari supervisionato da un project manager che visionerà i vari commit,etc. un tool che all'interno di una azienda non può e non deve mancare.   
Pensate alla possibilità di scrivere codice in modo asincrono senza perdere tempo a capire cosa e come è stato modificato un certo sorgente, un certo file o una certa cartella del progetto. Pensate alla possibilità di segnalare errori, creare rami di sviluppo ed aggiungere membri del team in un progetto funzionante. Pensate alla possibilità di tenere aggiornati tutti gli utenti di tale progetto contemporaneamente e di avere tutte le modifiche da più persone senza la minima difficoltà.

## Guide
Ovviamente questa è solo una breve introduzione a ciò che è realmente git, cosa consente di fare e perché usarlo, vi consiglio delle guide utili:

* [http://git-scm.com/book/it/](http://git-scm.com/book/it/)
* [http://rogerdudler.github.io/git-guide/index.it.html](http://rogerdudler.github.io/git-guide/index.it.html)
* [http://www.linux.it/~rubini/docs/git/](http://www.linux.it/~rubini/docs/git/)

## Conclusione
In conclusione, come avete fatto a vivere senza questo strumento **ESSENZIALE** per tutto questo tempo ?   
Davvero, mi sono sentito in obbligo di dover fare un simile articolo, è davvero "ridicolo" che al giorno d'oggi se ne parli davvero poco all'interno di mondi come le università, i LUG o altre associazioni di questo genere. Io stesso sono stupito della poca popolarità attribuita a git.   
Spero in un futuro non troppo lontano di vedere spopolare questo strumento anche in zone "limitrofe" come la mia città. (la mia come chi lo sa, magari anche le vostre)

Spero che questa introduzione vi sia servita per incuriosirvi, per darvi una idea e per farvi conoscere questo eccezionale strumento che è Git.   
Saluti, DLion.
