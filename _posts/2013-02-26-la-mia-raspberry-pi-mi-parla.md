---
title: La mia Raspberry Pi mi parla
summary: Come far parlare la nostra raspberry pi
categories: Raspberry
layout: post
---
Avete presente la feature figa di Google Translate che permette di pronunciare le parole che vogliamo tradurre ?   
Ecco, ho fatto un piccolo script in bash che data una frase viene mandata a google translate per poi riprodurre automaticamente ciò che abbiamo scritto in forma vocale.

In poche parole come fare parlare la nostra raspberry.   
Lo script è questo:
{% highlight sh lineanchors %}
#!/bin/bash
#By Domenico Luciani aka DLion
# http://dlion.it
# http://about.me/DLion
say()
{
  mplayer -really-quiet -ao alsa "http://translate.google.com/translate_tts?tl=it&q=$*"
}
   
say $*
{% endhighlight %}

L’unica dipendenza è mplayer ed ovviamente una connessione ad internet.   
Una volta salvato come *say.sh* vi basterà digitare qualcosa come:
{% highlight sh lineanchors %}
pi@raspberrypi ~ $ chmod +x ./say.sh
pi@raspberrypi ~ $ ./say.sh DLion e il suo blog, sono fighi.
{% endhighlight %}

Ovviamente dovrete collegare la vostra raspi a qualche speaker.

Saluti, DLion
