---
layout: post
title:  "Welcome to Jekyll!"
date:   2013-12-02 02:34:57
tags: jekyll update
---

You'll find this post in your `_posts` directory - edit this post and re-build (or run with the `-w` switch) to see your changes!
To add new posts, simply add a file in the `_posts` directory that follows the convention: YYYY-MM-DD-name-of-post.ext.

Jekyll also offers powerful support for code snippets:

{% highlight ruby linenos %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

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

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
