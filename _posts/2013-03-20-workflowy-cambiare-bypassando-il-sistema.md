---
title: Workflowy, cambiare bypassando il sistema
description: Diario di un bypassamento
category: Security
layout: post
---
Mi è stato consigliato da poco un servizio davvero niente male, si tratta di [WorkFlowy](https://workflowy.com/).   
Tale servizio permette di stilare una fantastica todo list in grado di "seguire" il vostro pensiero stilando gerarchicamente tutti i contenuti.

Ecco alcuni esempi:
![uno]({{site.image_url}}/workflowy-cambiare-bypassando-il-sistema.jpg)

![due]({{site.image_url}}/workflowy-cambiare-bypassando-il-sistema-1.jpg)


Come si può notare ha molte funzionalità e girovagando un po' mi accorgo di alcune impostazioni che posso cambiare
![impo]({{site.image_url}}/workflowy-cambiare-bypassando-il-sistema-2.jpg)

Figa, posso cambiare il tema e il font, ottimo quel bianco era troppo "Bianco".   
Quindi mi accinsi a cambiare il tema e vidi che c'era un tema "hacker": nero, verde; abbastanza figo!   
Dato che avevo voglia di provare tentai di settare tale tema al mio profilo quando mi bloccai.

Non riuscivo a settarlo, per farlo avrei dovuto pagare.   
**‘SPÈ, cosachecosa!?**   
_Pagare per settare un font o un tema ed abilitare 4 cagate tipo dropbox et simila ?_   
Allora mi balzò in mente una piccola idea: Non posso bypassare 'sta procedura e settarmi il tema da me ?   
Allora cominciai con lo scrutare il sorgente della pagina.   
Come prima cosa notai subito l'inclusione del css di default   
`media/versioned/20130318210837/themes/desktop.default.css`   
quindi tentai una scemenza e cambiai "default" in "hacker".

Funzionò, mi spuntò il css del tema che volevo e con firebug o con lo strumento "ispeziona" di firefox cambiai la stringa e il tema in questione cambiò diventando *super pr0 l337 h4x0r*.   
Ma tutto ciò funzionava solo in locale, se io avessi cambiato o aggiornato la pagina il tutto sarebbe tornato normale, non mi andava affatto così continuai a guardare il sorgente e notai questo:
{% highlight js linenos %}
var THEME_OPTIONS = [{
  "pretty_name": "Default",
  "font": "default",
  "type": "theme",
  "name": "default",
  "is_free": true
},
{
  "pretty_name": "Dark",
  "font": "lucidagrande",
  "type": "theme",
  "name": "dark",
  "is_free": false
},
{
  "pretty_name": "Wood",
  "font": "courier",
  "type": "theme",
  "name": "wood",
  "is_free": false
},
{
  "pretty_name": "Steel",
  "font": "helvetica_light",
  "type": "theme",
  "name": "steel",
  "is_free": false
},
{
  "pretty_name": "Minimal",
  "font": "helvetica_light",
  "type": "theme",
  "name": "light",
  "is_free": false
},
{
  "pretty_name": "Space",
  "font": "helvetica_light",
  "type": "theme",
  "name": "space",
  "is_free": false
},
{
  "pretty_name": "Hacker",
  "font": "andale",
  "type": "theme",
  "name": "hacker",
  "is_free": true
}];

var FONT_OPTIONS = [{
  "pretty_name": "Sans-serif",
  "font_styles": "font-family:'Helvetica Neue',Arial, Sans-serif;font-weight:normal;",
  "type": "font",
  "name": "default",
  "is_free": true
},
{
  "pretty_name": "Serif",
  "font_styles": "font-family:Times;font-weight:normal;",
  "type": "font",
  "name": "times",
  "is_free": true
},
{
  "pretty_name": "Light",
  "font_styles": "font-family:'Helvetica Neue', Arial, Sans-serif;font-weight:300;",
  "type": "font",
  "name": "helvetica_light",
  "is_free": false
},
{
  "pretty_name": "Typewriter",
  "font_styles": "font-family:Courier, Monospace;font-weight:normal;",
  "type": "font",
  "name": "courier",
  "is_free": false
},
{
  "pretty_name": "Terminal",
  "font_styles": "font-family:\"Andale Mono\", Monospace;font-weight:normal;",
  "type": "font",
  "name": "andale",
  "is_free": false
},
{
  "pretty_name": "Interface",
  "font_styles": "font-family:\"Lucida Grande\", \"Lucida Sans Unicode\";font-weight:normal;",
  "type": "font",
  "name": "lucidagrande",
  "is_free": false
}];
{% endhighlight %}

In poche parole il sito mi stava dicendo quali erano i font/temi e come si chiamavano, ottimo!

Come secondo passo ho utilizzato un utile plugin di firefox che mi permette di "sniffare" le richieste POST/GET che inviavo al server.   
Il plugin in questione è [Live HTTP Headers](https://addons.mozilla.org/it/firefox/addon/live-http-headers/).

Potevo cambiare solo un misero font nelle impostazioni, provai con quello ed ecco cosa ottenni come risultato:   
![risult]({{site.image_url}}/workflowy-cambiare-bypassando-il-sistema-3.jpg)

Cioè per settare il font inviava solo un parametro "font" con attributo "serif" alla pagina *change_settings*

A quel punto pensai che i temi venissero trattati allo stesso modo, ma il dubbio che mi sorse fu: c'è un controllo per vedere se sono un utente pro o meno ?   
In caso negativo riesco altrimenti fallisco… e **tombola!**   
Invio come parametro theme=hacker e la pagina mi ritorna per "magia" `{"success": true}`, ricarico la pagina e...

####TADAN!

![Success]({{site.image_url}}/workflowy-cambiare-bypassando-il-sistema-4.jpg)

Figo no ?

Ovviamente per ora non mi interessa fare altro con questo strumento, in futuro chi lo sa.   
Ecco un esempio lampante di controllo errato delle richieste in entrata, sarebbe bastato controllare che {utente} fosse {pro} per bloccare questo "bypassaggio".   
Non ho messo in pratica tecniche particolari o azionato tool segreti, il bug è principalmente fra la tastiera e la sedia. (come sempre, del resto)

Saluti, DLion

---

[English Version](https://domenicoluciani.com/2013/03/20/how-to-change-theme-workflowy.html)
