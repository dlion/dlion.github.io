---
title: Interfacciare il nunchuck con la raspberry pi
description: Come interfacciare il nunchuck della wii con la raspberry pi attraverso le porte GPIO
category: Raspberry
---
Girovagando per la città sono incappato in un nunchuck a basso costo, una volta portato a casa l'ho connesso alla mia raspberry interfacciandomi con esso attraverso le porte GPIO della stessa.

# Di cosa abbiamo bisogno

* Breadboard
* nunchuck
* raspberry
* jumper

# Modello di nunchuck

Il modello da me acquistato non è quello originale Nintendo ma bensì una "copia" chiamata funchuck che viene a costare anche meno di 5€ ormai.   
![funchuck]({{site.image_url}}/nunchuck-chinese.jpg)

## Qual è la differenza fra i due ?

I pin principali sono 4:

1. DATA
2. IN
3. CLOCK
4. GND

Quello originale ne avrà uno in più chiamato `PRESENZA` che è pressocché inutile quindi non sarà un problema.

# Schema
```sh
| 1   3 |
| 6 5 4 |
|_------_|
```
1. DATA
3. IN (3.3v)
4. CLOCK
5. PRESENZA (Praticamente inutile)
6. GND

## Schema di collegamento

![schema]({{site.image_url}}/nunchuck-sketch-raspberry.jpg)

Come potete notare il tutto è collegato direttamente quindi non avrete bisogno di connettori particolari, saldare fili o altre cose più "pratiche".

1. PIN 3.3v --(red)--> IN
2. PIN 2 (0 SDA) --(blue)--> DATA
3. PIN 3 (1 SCL) --(green)--> CLOCK
4. PIN GND (0v) --(black)--> GND

# Sulla raspi

Una volta collegato il tutto dovremo configurare la nostra raspi così da poterci interfacciare con essa al nostro nuovo device.   
Confido in una vostra precedente installazione delle librerie [wiringPi](http://wiringpi.com/) così da non aver problemi durante tutto il processo.

1. Carichiamo il modulo i2c tramite wiringPi con il comando `sudo gpio load i2c`
2. Verifichiamo che la nostra raspberry veda con successo il nostro device con il comando `sudo gpio i2cd`, come output avremo un risultato simile   
```sh
0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:         -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- UU -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- 52 -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

`UU` sta ad indicare che quella locazione è momentaneamente occupata, non ci interessa.   
Il `52` in basso è l'elemento cruciale, sta ad indicare che il nostro device viene visto e che si trova all'indirizzo `0x52`, prendiamone nota.

A quel punto ho realizzato un sorgente in C compilabile tranquillamente con `gcc <source.c> -o <binary>` per poi eseguirlo con `sudo ./<binary>`, lo trovate [qui](https://github.com/dlion/Raspi/blob/master/nunchuck.c)

```c
/*
 * The MIT License (MIT)
 * Copyright (c) 2014 Domenico Luciani http://dlion.it domenicoleoneluciani@gmail.com
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 
 * The above copyright notice and this permission notice shall be included in all
 * copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, 
 * INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <linux/i2c.h>
#include <linux/i2c-dev.h>
#include <fcntl.h>
#include <sys/ioctl.h>

#define DEVICE "/dev/i2c-1"
#define ADDRESS 0x52
#define LEGGI 0
#define SCRIVI 1

int comunica(char*, int, int);

int main(void) 
{
    char buffer_read[6], //Buffer ricezione byte
         buffer_init[] = { 0xF0, 0x55 }, //Sequenza di init
         buffer_stop[] = { 0x00 }; //Sequenza di stop
  
    int file; //File handler
    int z, c; //Pulsanti
    int responso; //Responso della comunicazione

    responso = comunica(buffer_init, 2, SCRIVI);
    if(responso == 0)
        puts("Init avvenuto con successo");
    else
    {
        printf("Errore nella comunicazione %d\n", responso);
        exit(responso);
    }
      
    while(responso == 0)
    {
        //Stop
        responso = comunica(buffer_stop, 1, SCRIVI);
        if(responso != 0)
        {
            printf("Errore nella comunicazione %d\n", responso);
            exit(responso);
        }

        //Leggo
        responso = comunica(buffer_read, 6, LEGGI);
        if(responso != 0)
        {
            printf("Errore nella comunicazione %d\n", responso);
            exit(responso);
        }

        // Pulsanti
        z = buffer_read[5] & 0x01;
        c = (buffer_read[5] >> 1) & 0x01;
    
        //Asse X
        buffer_read[2] <<= 2;
        buffer_read[2] |= ((buffer_read[5] >> 2) & 0x03);
        //Asse Y                
        buffer_read[3] <<= 2;
        buffer_read[3] |= ((buffer_read[5] >> 6) & 0x03);

        printf("Analog X: %d Analog Y: %d Asse-X: %d Asse-Y: %d Asse-Z: %d ", buffer_read[0], buffer_read[1], buffer_read[2], buffer_read[3], buffer_read[4]);

        printf("Pulsante Z: ");
        (z == 1) ? printf("non premuto ") : printf("premuto ");
    
        printf("Pulsante C: ");
        (c == 1) ? printf("non premuto\n\f") : printf("premuto\n\f");
    
        usleep(200000);
    }
    
    return 0;
}

int comunica(char *buffer, int ndati, int mod)
{
    int file;

    if((file = open(DEVICE, O_RDWR)) < 0)
        return -1;

    if(ioctl(file, I2C_SLAVE, ADDRESS) < 0)
        return -2;

    if(mod == SCRIVI)
    {
        if(write(file, buffer, ndati) != ndati)
            return -3;
    }
    else if(mod == LEGGI)
    {
        if(read(file, buffer, ndati) != ndati)
            return -3;
    }
    else
        return -4;
  
    close(file);

    return 0;
}
```

Attenzione, se avete una raspberry rev1 anziché di una rev2 dovrete cambiare `/dev/i2c-1` con `/dev/i2c-0`

Il nunchuck utilizza il protocollo [l²C](http://it.wikipedia.org/wiki/I%C2%B2C) un protocollo master/slave e comunica a 400KHz; per poter usarlo dobbiamo fornire l'address ricavato prima (`0x52`) e per poter `comunicare` con il nostro nunchuck bisogna innanzitutto inviare una sequenza di byte di init. Questa sequenza nei device originali è `0x40 0x00` mentre negli altri (come nel mio) è `0xF0 0x55`  e a quel punto dire al nostro dispositivo che vogliamo leggere e non ci rimarrà che salvare e decodificare i 6 byte che il device ci "sputerà" fuori per poi inviare uno stop `0x00` ad ogni ciclo.    

In output riceveremo le informazioni che ci servivano qui: Posizione X/Y dell'analogico; se abbiamo premuto i pulsanti C/Z e le coordinate degli assi X/Y/Z dell'accelerometro.
<script type="text/javascript" src="https://asciinema.org/a/8812.js" id="asciicast-8812" async ></script>

Questa potrebbe essere solo la punta dell'iceberg, pensate alle enormi possibilità che offre tale device su un qualcosa come la raspi; poter comandare droni, macchinine o qualsiasi altra cosa con il solo movimento del polso o del pollice; le possibilità sono davvero tante.

Spero di avere presto il tempo necessario per continuare la cosa e realizzare qualcosa di simpatico.

Saluti, DLion
