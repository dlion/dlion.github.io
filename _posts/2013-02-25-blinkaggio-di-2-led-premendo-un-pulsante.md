---
title: Blinkaggio di 2 led premendo un pulsante
description: Far blinkare 2 led grazie alla RaspBerry
category: Raspberry
layout: post
---
Come vi avevo accennato dato che possiedo una Raspi passo quel poco tempo libero a farci cavolate per incrementare la mia conoscenza di elettronica e della raspi stessa.  
Ieri mi annoiavo ed ho provato a fare qualcosa di semplice, cioè, ho collegato alla raspi un pushbutton e 2 led; quando premo il pulsantino la raspi fa accendere i led.

Ovviamente è solo un esercizio minchione per cominciare, non aspettatevi molto da me nel campo dell’elettronica perché sono una pippa.  
Non vi dirò di prendere né colla vinilica né carta igienica, Muciaccia mi scuserà  
Prima di cominciare vi dico cosa ho usato:

* 1 breadboard
* 1 resistenza da 10Kohm
* 2 resistenze da 300Ohm
* Jumper vari per collegare il circuito
* 1 push button
* 2 LED
* 1 RaspBerry Pi
* 1 Barra di plutonio

Questa era la parte “hardware”, ora vediamo quella software, cioè quel programmino che una volta avviato prenderà il segnale dal pulsantino ed invierà il segnale verso i led.   
In poche parole sarà lui a “comandare” i led quando effettueremo una determinata azione.

{% highlight c linenos %}
/*
  Blink 2 Led with push button
  By Domenico Luciani aka DLion
*/
 
//Using wiringPi library
#include <wiringPi.h>
#include <stdio.h>
#include <stdlib.h>
 
int main (void)
{
    //Using pin 8 to read input (pin 3 on the raspi)
    int pin8_in = 8;
    //Using pin 0 and 1 to write output (pin 11 and 12 on the raspi)
    int pin0_out = 0;
    int pin1_out = 1;
     //Starting Setup
     if(wiringPiSetup() == -1)
        exit(1);
    //Set pin 8 to input
    pinMode(pin8_in,INPUT);
    //Set pin 0 and 1 to output
    pinMode(pin0_out,OUTPUT);
    pinMode(pin1_out,OUTPUT);
 
    while(1)
    {
        //Push button pressed
        if(digitalRead(pin8_in) == 0)
        {
            //LED 0 ON
            digitalWrite(pin0_out,1);
            //Wait 50 ms
            delay(50);
            //LED 1 ON
            digitalWrite(pin1_out,1);
            delay(50);
            //LED 0 OFF
            digitalWrite(pin0_out,0);
            delay(50);
            //LED 1 OFF
            digitalWrite(pin1_out,0);
            delay(50);
        }
        //Wait 100 ms
        delay(100);
    }
    return 0;
}
{% endhighlight %}

Come vedete il sorgente è scritto in C ed uso l’abbastanza famosa libreria [wiringPi](https://projects.drogon.net/raspberry-pi/wiringpi/) che permette di usare le porte GPIO della Raspi come se fosse un arduino interfacciandosi con l’hardware descritto sopra.   
Il codice è commentato quindi non credo ci vogliano molte spiegazioni, in caso vi basta dare una occhiata alla documentazione della libreria sopra citata per eventuali dubbi.   
Detto ciò vediamo come realizzare il tutto.     
Da questa immagine vediamo come sono i pin della RaspBerry Pi e come sono visti dalla libreria WiringPi   
![GPIO](/images/blinkaggio-di-2-led-premendo-un-pulsante.jpg)

Come potete vedere abbiamo 3 sezioni per pin.     
La prima sezione è come viene visto il tutto dalle librerie WiringPi, la seconda per ora lasciamola stare e la terza è il nome dei pin.

In questo articolo i pin che ci interesseranno saranno:

* il pin 1 (3.3v)
* il pin 3 (SDA0)
* il pin 6 (0V)
* il pin 11 (GPIO0)
* il pin 12 (GPIO1)

Il primo pin ci servirà a dare corrente al pushbutton.   
Il pin 3 ci servirà per leggere il segnale del pulsantino, quando premiamo il pulsantino lui lo saprà.   
Il pin 6 è il ground, la messa a terra, ci servirà per chiudere il circuito. (Il negativo, diciamo)   
Il pin 11 e il pin 12 sono i pin che controlleranno i LED inviando un segnale opportuno per farli accendere o spegnere.   

## **Attenzione**   

### **usate il pin 1, cioè quello più a sinistra perché quello a destra è il pin 2, cioè quello che genera 5V e il pin 3 in entrata non può gestire 5V friggendovi il mondo.**

Nulla di difficile, no ? Ora non vi resta che collegare la board in questo modo:

* V3.3 -> 10Kohm -> Pin 3 -> PushButton -> 0V

Come primo passaggio abbiamo collegato l’alimentazione ad una resistenza di 10Kohm collegata poi al jumper che va nel pin 3 e in serie abbiamo collegato il positivo del pushbutton per poi collegare il negativo del pushbutton al ground.

Dopo di ciò non ci resta che collegare i led:

* pin 11 -> 330Ohm -> + LED -> – LED -> 0V
* pin 12 -> 330Ohm -> +LED -> -LED -> 0V

Una immagine a cazzo tanto per farvi vedere quanto sono impedito   
![Immagine a cazzo](/images/blinkaggio-di-2-led-premendo-un-pulsante-1.jpg)

Come avete potuto notare dal sorgente prima postato all’interno di esso non definisco i pin come 11,12,3,etc.   
Ma devo usare lo standard di wiringPi quindi 8 per la lettura, 0 ed 1 per la scrittura.

Dopo di ciò non vi resta che compilare il sorgente C ed avviarlo   
{% highlight sh linenos %}
gcc nomesorgente.c -o blink -lwiringPi
sudo ./blink
{% endhighlight %}

Ecco qui 2 video di scarsa qualità che ho girato tanto per farvi perdere tempo   
[https://vimeo.com/60373916](https://vimeo.com/60373916)   
[https://vimeo.com/60373914](https://vimeo.com/60373914)   
Per qualsiasi domanda io sono qui.

Saluti, DLion

---

[English Version](https://domenicoluciani.com/2013/02/25/2-leds-blink.html)
