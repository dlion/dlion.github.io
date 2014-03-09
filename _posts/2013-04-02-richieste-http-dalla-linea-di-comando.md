---
layout: post
title: Richieste HTTP dalla linea di comando
description: introduzione alle cURL
---
Mi è capitato molte volte di dover creare delle richieste [HTTP](http://it.wikipedia.org/wiki/Hypertext_Transfer_Protocol) ad-hoc da inviare a determinate pagine web e/o applicazioni, richieste che posso gestire in modo manuale (ma non troppo) così da avere il pieno controllo di ciò che invio; il tutto fatto dalla mia amata linea di comando...

Ovviamente parlo delle famose [cURL](http://en.wikipedia.org/wiki/CURL).   
Questo strumento davvero eccezionale è praticamente onnipresente in ogni sistema *unix e windows, permette come detto in precedenza di creare ed inviare delle richieste HTTP ad un destinatario.   
Cioè, permettono di ricreare in modo manuale ciò che il browser fa in modo del tutto automatico e nascosto all'utente.   
Vediamo come effettuare una richiesta semplice semplice da console.

`curl articoli.dlion.it/howtousecurl/uso1.php`
{% highlight json %}
{
    "method":"GET",
    "uri":"\/howtousecurl\/uso1.php",
    "headers":
    {
        "host":"articoli.dlion.it",
        "user-agent":"curl\/7.29.0",
        "accept":"*\/*"
    },
    "ip":"152.53.218.150",
    "powered-by":"http:\/\/dlion.it"
}
{% endhighlight %}

Come potete vedere avete appena effettuato una vera e propria richiesta alla pagina che ho appositamente creato che vi restituirà un responso in JSON, in oltre diciamo che di default le curl utilizzano come metodo il metodo GET.

Per cambiare il metodo usato nella richiesta potete usare l’opzione -X in questo modo   
`curl -X POST articoli.dlion.it/howtousecurl/uso1.php` ricevendo come responso
{% highlight json %}
{
    "method":"POST",
    "uri":"\/howtousecurl\/uso1.php",
    "headers":
    {
        "host":"articoli.dlion.it",
        "user-agent":"curl\/7.29.0",
        "accept":"*\/*"
    },
    "ip":"152.53.218.150",
    "powered-by":"http:\/\/dlion.it"
}
{% endhighlight %}

Come potete notare abbiamo appena effettuato una richiesta POST alla stessa pagina.   
Potete usare più metodi come PUT, GET, DELETE,etc.

Ccurl articoli.dlion.it/howtousecurl/uso2.phpon le curl possiamo anche simulare le richieste che effettuiamo dai vari form che incontriamo nelle varie pagine web, ovviamente il tutto direttamente dalla nostra bellissima shell.   
Per esempio, abbiamo questo semplice form e vogliamo accedere senza avere il bisogno di aprire il browser, scrivere user e pass e loggarsi così da automatizzare il tutto.   
Benissimo, dalla shell per "dire" quali sono i parametri basta usare l'opzione `-d` specificando il parametro da inviare.   
`curl articoli.dlion.it/howtousecurl/uso2.php` Ricevendo come responso
{% highlight html linenos %}
<html>
    <head>
        <title>Curl POST Form Data</title>
    </head>
    <body>
        <form action="" method="POST">
            <label for="name">Nome</label>
            <input type="text" name="name"/>
            <label for="pass">Pass</label>
            <input type="password" name="pass"/>
            <input type="submit" value="Accedi"/>
        </form>
    </body>
</html>
{% endhighlight %}

Come potete notare il form una volta "compilato" invierà 1 richiesta POST inviando i 2 parametri name e pass.   
Per accedere vi basterà mettere come user "dlion" e come password "curl".   
`curl -X POST -d "name=dlion" -d "pass=curl" articoli.dlion.it/howtousecurl/uso2.php` oppure   
`curl -X POST -d "name=dlion&pass=curl" articoli.dlion.it/howtousecurl/uso2.php` ricevendo come risultato: `Accesso effettuato` altrimenti `Dati non corretti`

La semplicità con cui le curl permettono certe cose è davvero disarmante perché ci risparmiano davvero molto tempo.   
Se avessimo fatto tutto ciò usando i [socket](http://it.wikipedia.org/wiki/Socket_%28reti%29) avremmo dovuto realizzare richieste ad-hoc sfruttando il protocollo HTTP in modo più che maniacale per ricevere ed inviare i dati corretti.

Le curl ci permettono addirittura di settare cookie, effettuare richieste tamite connessioni sicure, inviare files, scaricare dati,etc.   
Vi basta guardare la [man page](http://www.linuxmanpages.com/man1/curl.1.php) delle curl per capire quanto sia vasto questo tool.

Per non parlare che sono disponibili praticamente per ogni linguaggio conosciuto, volete effettuare una richiesta POST con il php ?   
Niente di più semplice
{% highlight php linenos %}
<?php
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "http://articoli.dlion.it/howtousecurl/uso2.php");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, true);

$data = array('name' => 'dlion',
          'pass' => 'curl');
curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
$output = curl_exec($ch);
curl_close($ch);
echo $output;
?>
{% endhighlight %}

No davvero, le curl sono davvero **ESSENZIALI** in certe cose, uno strumento tanto potente quanto semplice.   
Nei futuri articoli farò largamente uso di tale tool, vi renderete conto da soli di quanto sia fenomenale.

Ovviamente vi consiglio di leggere qualcosa al riguardo vista che questa è stata **solo una infarinatura** (molto, molto, molto infarinatura) di quello che sono le curl.   
Oltre alla solita man page, vi consiglio anche [https://httpkit.com/resources/HTTP-from-the-Command-Line](https://httpkit.com/resources/HTTP-from-the-Command-Line/) o [http://curl.haxx.se/](http://curl.haxx.se/)

Ovviamente per qualsiasi chiarimento, correzione, bonifico bancario,etc. io sono qui.   
Saluti, DLion.
