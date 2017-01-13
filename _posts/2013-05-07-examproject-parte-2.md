---
title: ExamProject parte 2
description: Il mio progetto d'esame sulla visione artificiale.
category: Programming
layout: post
---
Alcuni articoli fa vi descrissi la parte 1 del mio progetto d'esame, oggi volevo continuare illustrandovi in modo più o meno dettagliato la seconda parte di tale progetto.   
Il programma che analizzeremo ora non è altro che un giochino che permette tramite la tecnica del color tracking di poter "toccare" le bolle su schermo facendole scoppiare.   
Il "gioco" in questione è totalmente sviluppato usando il C e le librerie OpenCV, ha vari livelli di difficoltà, permette la gestione del punteggio e tanto altro.


Incominciamo con il premettere che il tutto è stato fatto durante il mio progetto d’esame (sì lo so, sono ridondante ma è giusto ripeterlo) ergo non aspettatevi cose mirabolanti anche perché il tempo scarseggiava e il caffè stava per finire.

Vediamo il codice
{% highlight c linenos %}
/*
# This file is part of Computer Vision Exam Project
#
# Copyright(c) 2012 Domenico Luciani
# domenicoleoneluciani@gmail.com
#
#
# This file may be licensed under the terms of of the
# GNU General Public License Version 3 (the ``GPL'').
#
# Software distributed under the License is distributed
# on an ``AS IS'' basis, WITHOUT WARRANTY OF ANY KIND, either
# express or implied. See the GPL for the specific language
# governing rights and limitations.
#
# You should have received a copy of the GPL along with this
# program. If not, go to http://www.gnu.org/licenses/gpl.html
# or write to the Free Software Foundation, Inc.,
# 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.
*/
 
//Librerie necessarie al funzionamento
#include <cv.h>
#include <highgui.h>
#include <unistd.h>
#include <pthread.h>
#include <time.h>
//Libreria con funzioni mie
#include "../lib/funzioni.h"
 
//Nome della Gui
#define NOME "Bubbles"
//File di configurazione
#define FILE_CONFIG "../config/config.txt"
//Numero massimo di bolle da creare
#define NUM_MAX_BOLLE 50
//File del punteggio
#define CLASSIFICA "punteggio.txt"
//Punteggio per definire i vari livelli
#define SECONDO_LIVELLO 100
#define TERZO_LIVELLO 500
#define QUARTO_LIVELLO 1000
//Variabile globale per il punteggio.
int punt = 0;
//variabile per indicare se ho preso il nemico
int preso = 0;
//Struttura della bolla
typedef struct
{
    //Mutex
    pthread_mutex_t mutex;
    //Coordinate e passo
    int X,Y,passo;
    //Se e' un nemico o meno
    int chi,web;
}Bolla;
 
//Funzione per dare nuovi valori alla bolla passata
void nuoviValori(Bolla *dato)
{
    dato->X = rand() % (500 - 0 + 1) + 0;
    dato->Y = 1;
    dato->passo = 1;
}
 
 
//Funzione per aggiornare con un ritardo la posizione delle bolle
void *up(void *var)
{
    Bolla *val = (Bolla*)var;
 
    //Inizio del ciclo che modifica i valori
    while(1)
    {
            //Aspetto dei millisecondi randomici
            usleep((rand() % (30000 - 800 + 1) + 800));
            //Blocco il mutex su questo thread
            pthread_mutex_lock(&val->mutex);
            if(preso != 1)
            {
                //Se la bolla arriva alla fine dello schermo ritorna su con nuovi valori
                if(val->Y >= 475 )
                {
                    //Se non sono un nemico diminuisco il punteggio
                    if(val->chi != 1)
                        punt -= 10;
                    else
                        punt += 10;
 
                    nuoviValori(val);
                }
                else
                    //Faccio camminare la bolla
                    val->Y += val->passo;
                //Sblocco il mutex
            }
            pthread_mutex_unlock(&val->mutex);
    }
    //Distruggo il mutex alla fine del thread
    pthread_mutex_destroy(&val->mutex);
}
 
//Funzione che prende il frame ed esegue tutte le altre operazioni a schermo
void *video(void *stato)
{
    //Che webcam usare
    int *web = (int*)stato;
    //Indico da quale sorgente prendere i frame
    CvCapture *frame = cvCaptureFromCAM(*web);
    cvSetCaptureProperty(frame,CV_CAP_PROP_FRAME_WIDTH,640);
    cvSetCaptureProperty(frame,CV_CAP_PROP_FRAME_HEIGHT,480);
    //Inizializzo le immagini
    IplImage *img = cvQueryFrame(frame);
    IplImage *hsv = cvCreateImage(cvGetSize(img),8,3);
    IplImage *binary = cvCreateImage(cvGetSize(img),8,1);
 
    //Inizializzo le bolle
    Bolla *val[NUM_MAX_BOLLE];
 
    //File dei punteggi
    FILE *filetto;
    //Stringhe varie
    char tasto,messaggio[20];
    //Livello
    int livello=1;
    //Vari indici
    int k,i,j;
    //Numero di bolle fin'ora create
    int num=1;
    //Iteratori
    int champion,ciclo_in=0,ciclo_out=0;
 
    //Step per l'immagine
    int step = binary->widthStep/sizeof(uchar);
 
    uchar *target = (uchar*)binary->imageData;
    //Alloco spazio per prendere i valori HSV da usare
    HSV *low = (HSV*)malloc(sizeof(HSV));
    HSV *high = (HSV*)malloc(sizeof(HSV));
    //creo NUM_MAX_BOLLE thread
    pthread_t th[NUM_MAX_BOLLE];
    //Inizializzo le scritte
    CvFont scritta,avviso;
    //Imposto lo stile del font
    cvInitFont(&scritta,CV_FONT_HERSHEY_SIMPLEX,.6,.6,0,1,6);
    cvInitFont(&avviso,CV_FONT_HERSHEY_SIMPLEX,1.0,1.0,0,5,CV_AA);
    //Creo una Gui chiamata NOME
    cvNamedWindow(NOME,1);
    //Inizializzo il seme
    srand((unsigned)time(NULL));
 
    for(k=0; k < NUM_MAX_BOLLE; k++)
        val[k] = (Bolla*)malloc(sizeof(Bolla));
    //Prima Bolla
    pthread_mutex_init(&val[0]->mutex,NULL);
    nuoviValori(val[0]);
    val[0]->chi = 0;
    //Creo il thread
    pthread_create(&th[0],NULL,&up,(void*)val[0]);
 
    //Prendo i dati dal file di config
    leggiConfig(low,high,(char*)FILE_CONFIG);
 
    filetto=fopen(CLASSIFICA,"r");
    if(filetto == NULL)
    {
        fclose(filetto);
        filetto = fopen(CLASSIFICA,"a");
        champion = 0;
    }
    else
        fscanf(filetto,"%d",&champion);
 
    fclose(filetto);
 
    //Prende i frame
    while(img)
    {
        //Ruoto l'immagine
        cvFlip(img,img,1);
        //Converto l'immagine da RGB a HSV
        cvCvtColor(img,hsv,CV_BGR2HSV);
        //cerco il mio colore
        cvInRangeS(hsv,cvScalar(low->H,low->S,low->V),cvScalar(high->H,high->S,high->V),binary);
        //Riduco i disturbi
        riduciNoise(binary,binary);
        //Ciclo per far comparire la scritta START
        if(ciclo_in >= 0 && ciclo_in <= 9)
        {
            cvPutText(img,"START",cvPoint((binary->width/2)-50,binary->height/2),&avviso,CV_RGB(255,50,60));
            ciclo_in++;
        }
 
        //Cerco in tutta l'immagine pixel per pixel
        //Altezza
        for(i=0; i < binary->height; i++)
        {
            //Larghezza
            for(j=0; j < binary->width; j++)
            {
                //Se trovo il colore bianco
                if(target[i*step+j] == 255)
                {
                    //Controllo per tutte le bolle che ho fin'ora
                    for(k=0; k < num; k++)
                    {
                        //Se tocco una bolla
                        if(j >= val[k]->X && j <= val[k]->X+10 && i >= val[k]->Y && i <= val[k]->Y+10)
                        {
                            //Se tocco un nemico
                            if(val[k]->chi == 1)
                                preso = 1;
                            else
                            {
 
                                pthread_mutex_lock(&val[k]->mutex);
                               if(preso != 1)
                               {
                                   //Temporary add to play pop (Only for unix)
                                   system("/usr/bin/play -q pop.wav 2> /dev/null");
                                   punt += 10;
                                    //Scrivo +10 sopra la bolla
                                    cvPutText(img,"+10",cvPoint(val[k]->X,val[k]->Y-10),&scritta,CV_RGB(0,0,255));
                                    nuoviValori(val[k]);
                               }
                                    pthread_mutex_unlock(&val[k]->mutex);
                            }
                        }
 
                    }
 
                }
 
            }
        }
 
        //Se Tocco un nemico esco
        if(preso == 1)
        {
           if(ciclo_out >= 0 && ciclo_out <= 50)
           {
 
                //Visualizzo messaggi vari
                cvPutText(img,"GAME OVER",cvPoint((binary->width/2)-50,binary->height/2),&avviso,CV_RGB(0,0,255));
                sprintf(messaggio,"Punteggio: %d ",punt);
                cvPutText(img,messaggio,cvPoint((binary->width/2)-110,(binary->height/2)+80),&avviso,CV_RGB(0,0,255));
 
                sprintf(messaggio,"Livello: %d ",livello);
                cvPutText(img,messaggio,cvPoint(img->width-100,20),&scritta,CV_RGB(255,255,0));
                sprintf(messaggio,"Record: %d ",champion);
                cvPutText(img,messaggio,cvPoint(img->width-150,img->height-5),&scritta,CV_RGB(255,255,0));
                ciclo_out++;
           }
           if(punt > champion)
           {
               puts("NUOVO RECORD!");
               filetto = fopen(CLASSIFICA,"w");
               fprintf(filetto,"%d",punt);
               fclose(filetto);
               puts("Record salvato!");
               champion = punt;
           }
 
           if(ciclo_out == 50)
                break;
        }
        else
        {
            //Per ogni bolla
            for(k=0; k < num; k++)
            {
                //Disegno un nemico
                if(val[k]->chi == 1)
                    cvCircle(img,cvPoint(val[k]->X,val[k]->Y),10,CV_RGB(0,0,255),20,8);
                else
                    //Disegno una bolla
                    cvCircle(img,cvPoint(val[k]->X,val[k]->Y),10,CV_RGB(255,0,0),20,8);
            }
            sprintf(messaggio,"Punteggio: %d ",punt);
            cvPutText(img,messaggio,cvPoint(20,20),&scritta,CV_RGB(255,255,0));
            sprintf(messaggio,"Livello: %d ",livello);
            cvPutText(img,messaggio,cvPoint(img->width-100,20),&scritta,CV_RGB(255,255,0));
            sprintf(messaggio,"Record: %d ",champion);
            cvPutText(img,messaggio,cvPoint(img->width-150,img->height-5),&scritta,CV_RGB(255,255,0));
 
            //Se aumenta il punteggio aumento il livello
            switch(punt)
            {
                case 100:
                    if(livello == 1)
                    {
                        //Creo altre bolle incrementando la difficoltà
                        for(k=num; k < 10; k++)
                        {
                            pthread_mutex_init(&val[k]->mutex,NULL);
                            nuoviValori(val[k]);
                            //Creo nemici
                            if(k <= 2)
                                val[k]->chi = 1;
                            else
                                val[k]->chi = 0;
 
                            pthread_create(&th[k],NULL,&up,(void*)val[k]);
                        }
                        num = 10;
                        livello++;
                    }
                    break;
                case 500:
                    if(livello == 2)
                    {
                        for(k=num; k < 20; k++)
                        {
                            pthread_mutex_init(&val[k]->mutex,NULL);
                            nuoviValori(val[k]);
                            if(k <= 15)
                                val[k]->chi = 1;
                            else
                                val[k]->chi = 0;
 
                            pthread_create(&th[k],NULL,&up,(void*)val[k]);
                        }
                        num = 20;
                        livello++;
                    }
                    break;
                case 1000:
                    if(livello == 3)
                    {
                        for(k=num; k < 30; k++)
                        {
                            pthread_mutex_init(&val[k]->mutex,NULL);
                            nuoviValori(val[k]);
                            if(k <= 25)
                                val[k]->chi = 1;
                            else
                                val[k]->chi = 0;
 
                            pthread_create(&th[k],NULL,&up,(void*)val[k]);
                        }
                        num = 30;
                        livello++;
                    }
                    break;
                case 1500:
                    if(livello == 4)
                    {
                        for(k=num; k < 40; k++)
                        {
                            pthread_mutex_init(&val[k]->mutex,NULL);
                            nuoviValori(val[k]);
                            if(k <= 35)
                                val[k]->chi = 1;
                            else
                                val[k]->chi = 0;
 
                            pthread_create(&th[k],NULL,&up,(void*)val[k]);
                        }
                        num = 40;
                        livello++;
                    }
                    break;
                case 2000:
                    if(livello == 5)
                    {
                        for(k=num; k < 49; k++)
                        {
                            pthread_mutex_init(&val[k]->mutex,NULL);
                            nuoviValori(val[k]);
                            if(k <= 45)
                                val[k]->chi = 1;
                            else
                                val[k]->chi = 0;
 
                            pthread_create(&th[k],NULL,&up,(void*)val[k]);
                        }
                        num = 49;
                        livello++;
                    }
                    break;
            }
        }
 
        cvShowImage(NOME,img);
 
        //Aspetto il tasto
        tasto = cvWaitKey(15);
        //Se premo q esco
        if(tasto == 'q')
            break;
        //Prendo l'altro frame
        img = cvQueryFrame(frame);
    }
 
    //Distruggo tutti i mutex
    for(k=0; k < num; k++)
        pthread_mutex_destroy(&val[k]->mutex);
 
    //Distruggo tutto
    cvReleaseImage(&img);
    cvReleaseCapture(&frame);
}
 
//Main
int main(int argc,char *argv[])
{
    //thread
    pthread_t th1;
    int web;
 
    if(argc != 2)
        printf("usage: %s <mode>\n0 - integrate webcam\n1 - external webcam\n",argv[0]);
    else
    {
        web = atoi(argv[1]);
        if(web >= 0 && web <= 1)
        {
            printf( "Bubbles Game v. 1.0\n"
                    "Progettato e realizzato da Domenico Luciani\n"
                    "Classe 5B Informatica Abacus\n"
                    "dell'Istituto Tecnico Industriale Vittorio Emanuele III\n"
                    "Passare 0 come parametro per usare la webcam integrata\n"
                    "Passare 1 come parametro per usare la webcam esterna\n"
                    "Legenda:\n"
                    "1 - Ogni pallina rossa presa fa aumentare il punteggio di 10 punti\n"
                    "2 - Se viene presa una pallina blu si perde istantaneamente\n"
                    "3 - Ogni pallina rossa persa fa diminuire il punteggio di 10 punti\n"
                    "4 - All'aumentare del livello aumenta anche la difficoltà\n"
                    "Buon divertimento\n");
            //Creo un nuovo thread
            pthread_create(&th1,NULL,&video,(void*)&web);
            //Il processo aspetta che lo streaming video smetta di esistere
            pthread_join(th1,NULL);
        }
        else
            puts("webcam not found");
    }
    return 0;
}
{% endhighlight %}


Il primo problema che dovetti affrontare fu quello di far scendere le bolle ad una velocità più o meno "normale" e allo stesso tempo prendere ogni singolo frame dalla webcam ed identificare il colore così da far funzionare il tutto.   
Risolvetti questo problema con l'uso dei [thread](http://it.wikipedia.org/wiki/Thread_%28informatica%29) e della programmazione concorrente.

L'uso del "video" l'ho lasciato al thread principale mentre la gestione delle bolle l'ho fatta gestire da un thread a parte così da "separare" le due cose ma allo stesso tempo unirle perché se avessi fatto gestire il tutto in un unico ciclo le bolle sarebbero scese alla stessa velocità con cui prendevo i frame e/o viceversa ovviamente il tutto gestito da opportuni mutex.

Definiamo il nome della gui, dove si trova il file di configurazione, il numero massimo di bolle da creare, il file dei punteggi, definiamo i valori per passare da un livello all'altro. Dichiariamo delle variabili globali necessarie per il punteggio e per dire se abbiamo toccato una bolla "nemica".   
Creiamo una struttura *Bolla* che conterrà tutte le informazioni di ogni bolla **come se fosse un oggetto** con i vari attributi, questa struttura servirà ai thread per scambiarsi fra di loro varie risorse.

A questo punto vediamo la funzione *nuoviValori* che servirà ad inizializzare la bolla passata come parametro facendo ritornare la bolla all'inizio in una posizione pseudo randomica.

Ora troviamo la funzione up. Questa funzione verrà eseguita in un **thread a parte** e gestirà le posizioni delle bolle. Dentro ad un ciclo while infinito aspetta dei millisecondi a caso, blocchiamo il mutex. Se non abbiamo preso un nemico controlliamo che la bolla non abbia raggiunto la fine della gui, se la bolla nemica raggiunge la fine incrementiamo il punteggio di +10 altrimenti lo decrementiamo di -10. Se la bolla non ha raggiunto la fine della gui incrementiamo la sua coordinata y e le diamo nuovi valori.   
Sblocchiamo il mutex.

Ora vediamo la funzione video, questa funzione è come se fosse la nostra **funzione principale**.   
Dichiariamo le bolle e le variabili necessarie. Facciamo esattamente le stesse cose che abbiamo fatto con il [programma di calibrazione](/examproject-parte-1/) per quanto riguarda il color tracking.   
Le funzioni *cvInitFont* inizializzano lo stile da usare per scrivere su un'immagine.   
Alloco le Bolle e creo una prima bolla per poi creare il thread corrispondente passando la prima bolla. Leggo qual è il punteggio massimo fatto fin'ora. Entro nel ciclo while che si occupa di prendere i frame e gestire la stampa a video. Effettuo le solite operazioni sul frame. Creo un piccolo ciclo per far comparire a video la scritta "START".   
**Cerco in tutta l'immagine** e se trovo il colore che cercavo faccio un ciclo verificando che io tocchi una delle bolle che fin'ora ho creato.   
Se la tocco e vedo che è un nemico allora **perdo**, altrimenti blocco il mutex di quella determinata bolla, avvio il suono (lo scoppio della bolla) ed aumento di 10 il punteggio, scrivo "+10" sopra la bolla e do nuovi valori alla bolla sbloccando poi alla fine di tutto il mutex. Se durante il ciclo ho toccato un nemico faccio un ciclo di pochi secondi dove faccio spuntare a video la scritta "GAME OVER", il punteggio, il livello e il record. Se supero il record scrivo il nuovo record sul file del punteggio. Se non ho perso faccio disegnare su schermo ogni bolla che fin'ora ho creato, il livello, il record e il punteggio.

**Ora vediamo la gestione del livello**.   
Se arrivo a 100, 500, 1000, 1500 aumento il livello aumentando il numero di bolle complessive (normali/nemiche) dopo mostro il tutto. **Se premo 'q' esco e libero tutto**.

Ora abbiamo il nostro main vero e proprio che si occuperà di creare il primo thread e aspetterà che termini il suo ciclo.


Un giochino piuttosto semplice nella sua praticità ma è la prima cosa che mi è venuta in mente; e farla in C non è stato uno scherzetto dal punto di vista ideativo.   
Mostra tranquillamente come è possibile usare il color tracking per interagire con delle bolle disegnate su schermo. Questo procedimento è possibile applicarlo ad immagini e ad altre figure.   
La cosa che più conta è che la webcam riesca a **trackare per bene** il colore altrimenti avrà dei problemi a rilevare se tocchiamo o meno le bolle. (come è normale che sia)

Ecco uno screenshot
![trackcolore](/images/examproject-parte-2.png)

Sì lo so, avrei potuto gestire meglio il tutto, il codice magari è scritto male e ci sono schifezze in giro; mea culpa ma avevo davvero poco tempo.   
In ogni caso la parte interessante è stata proprio quella di interfacciare i miei movimenti con quelli creati dal computer, assemblare il tutto attraverso il multi-threading e rendere piacevole il "gioco" senza troppe problematiche.

Come sempre per qualsiasi domanda e/o chiarimento io sono qui.   
Presto continuerò con le altre parti del progetto, stay tuned!

Saluti, DLion.
