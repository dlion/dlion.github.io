---
title: ExamProject parte 1
description: Il mio progetto d'esame sulla visione artificiale
category: Programming
---
Torniamo a parlare del mio progetto d’esame per la maturità.   
Nella parte precedente abbiamo visto una piccola libreria home-made per semplificarmi il lavoro.   
Quest'oggi parleremo del software che ci permetterà di **trackare un qualsiasi colore** e allo stesso tempo di dire agli altri software quale colore abbiamo scelto.

Il programma in questione si trova dentro la cartella config, cioè il *calibra.c*
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
 
#include <cv.h>
#include <highgui.h>
#include "../lib/funzioni.h"
//File di configurazione
#define FILE_CONFIG "config.txt"
//Nomi delle GUI
#define NORMAL "Calibra"
#define BINARY "Calibra - Binaria"
 
int main(int argc,char *argv[])
{
    int web;
    //Controllo se i parametri sono corretti
    if(argc != 2)
        printf("usage: %s <mode>\n0 - integrate webcam\n1 - external webcam",argv[0]);
    else
    {
        web = atoi(argv[1]);
 
        if(web >= 0 && web <= 1)
        {
            //Inizializzo la webcam
            CvCapture *capt = cvCaptureFromCAM(web);
            //Setto le proprietà della webcam a 640x480
            cvSetCaptureProperty(capt,CV_CAP_PROP_FRAME_WIDTH,640);
            cvSetCaptureProperty(capt,CV_CAP_PROP_FRAME_HEIGHT,480);
            //Prendo il primo frame dalla webcam e lo salvo
            IplImage *imm = cvQueryFrame(capt);
            //Creo immagini
            IplImage *hsv = cvCreateImage(cvGetSize(imm),8,3);
            IplImage *binary = cvCreateImage(cvGetSize(imm),8,1);
 
            //Variabili varie
            int i,j,step = binary->widthStep/sizeof(uchar);
            uchar *target = (uchar*)binary->imageData;
            char tasto;
            //Alloco spazio per i valori HSV
            HSV *low = (HSV*)malloc(sizeof(HSV));
            HSV *high = (HSV*)malloc(sizeof(HSV));
            //Alloco spazio per il rettangolo
            Rettangolo *punti = (Rettangolo*)malloc(sizeof(Rettangolo));
            //Leggo i dati dal file di configurazione
            leggiConfig(low,high,(char*)FILE_CONFIG);
            //Creo le GUI
            cvNamedWindow(NORMAL,CV_WINDOW_AUTOSIZE);
            cvNamedWindow(BINARY,CV_WINDOW_AUTOSIZE);
            //Ciclo while che si occuperà di prendere i frame
            while(imm)
            {
                //Creo le trackbar
                cvCreateTrackbar("HMIN",NORMAL,&low->H,255,NULL);
                cvCreateTrackbar("SMIN",NORMAL,&low->S,255,NULL);
                cvCreateTrackbar("VMIN",NORMAL,&low->V,255,NULL);
 
                cvCreateTrackbar("HMAX",NORMAL,&high->H,255,NULL);
                cvCreateTrackbar("SMAX",NORMAL,&high->S,255,NULL);
                cvCreateTrackbar("VMAX",NORMAL,&high->V,255,NULL);
               //Ruoto l'immagine
                cvFlip(imm,imm,1);
                //Converto in hsv
                cvCvtColor(imm,hsv,CV_BGR2HSV);
                //cerco il colore
                cvInRangeS(hsv,cvScalar(low->H,low->S,low->V),cvScalar(high->H,high->S,high->V),binary);
                //Riduco i disturbi
                riduciNoise(binary,binary);
                //Resetto il rettangolo
                *punti = {10000,0,10000,0};
 
                //Controllo pixel per pixel
                for(i=0; i < binary->height; i++)
                {
                    for(j=0; j < binary->width; j++)
                    {
                        if(target[i*step+j] == 255)
                        {
                            //I punti del nostro rettangolo
                            if(j < punti->xmin)
                                punti->xmin = j;
                            if(j > punti->xmax)
                                punti->xmax = j;
                            if(i < punti->ymin)
                                punti->ymin = i;
                            if(i > punti->ymax)
                                punti->ymax = i;
                        }
                    }
                }
                //Creo il rettangolo
                cvRectangle(imm,cvPoint(punti->xmin,punti->ymin),cvPoint(punti->xmax,punti->ymax),CV_RGB(255,0,0),2,8,0);
               //Creo il baricentro
                cvCircle(imm,cvPoint((punti->xmin+punti->xmax)/2,(punti->ymin+punti->ymax)/2),2,CV_RGB(0,0,255),-1,CV_AA,0);
                //Mostro le GUI
                cvShowImage(NORMAL,imm);
                cvShowImage(BINARY,binary);
                //Aspetto 15 millisecondi
                tasto = cvWaitKey(15);
 
                switch(tasto)
                {
                    //Se premo 'q' esco
                    case 'q':
                        return 2;
                        break;
                    case 's':
                        //Se premo 's' salvo i dati che mi servono
                        scriviConfig(low,high,(char*)FILE_CONFIG);
                        break;
                }
                //Prendo un altro frame
                imm = cvQueryFrame(capt);
            }
            //Libero tutto ciò che ho allocato in precedenza
            cvReleaseImage(&imm);
            cvReleaseImage(&binary);
            cvReleaseImage(&hsv);
            cvReleaseCapture(&capt);
        }
        else
            puts("webcam not found");
    }
 
    return 0;
}
{% endhighlight %}

* Partiamo includendo le librerie necessarie al corretto funzionamento delle OpenCV cioè *cv.h*, *highgui.h* e poi la libreria *funzioni.h* vista in precedenza.
* Definiamo dove trovare il file di configurazione, il nome della gui dove mostreremo l'immagine del frame preso dalla webcam e il nome della gui dove mostreremo il risultato in binario dei valori presi.
* Controllo se ho inserito i parametri giusti in modo tale da sapere quale webcam usare.
* Inizializzo la webcam con la funzione *cvCaptureFromCAM* specificando come parametro _0 per la webcam interna_ e _1 per la webcam esterna_.
* Setto le proprietà della webcam utilizzando le dimensioni **640×480**
* Prendo il primo frame e lo salvo in *imm*. (In questo momento imm conterrà un frame estrapolato dalla nostra webcam)
* Creo un'immagine **hsv** dove salveremo l'immagine convertita in hsv di dimensioni pari a quelle contenute in imm.
* Creo un'immagine dove salveremo l'immagine binaria che troveremo.
* Prendiamo lo step dell'immagine binaria; il campo *widthStep* indica il numero di bytes tra l'inizio di ogni riga di pixel.
* Dopo abbiamo *target*, cioè i pixel veri e propri dell'immagine.
* Abbiamo però bisogno di castare come `unsigned char` la variabile perché di suo è definita come `char*` e la maggior parte delle volte tratteremo immagini `unsigned char`.
* Definiamo una variabile tasto che useremo per prendere il tasto premuto dall’utente.
* Allochiamo 2 puntatori alla struttura HSV e un puntatore alla struttura *Rettangolo* viste precedentemente.
* Prendiamo i valori dal file di configurazione e li mettiamo rispettivamente in low e high.
* Dopo creiamo 2 gui.
* Ora entriamo nella parte viva del programma, cioè nel while che avrà il compito di ripetersi finché imm non ritorna **false**.
* Creiamo delle *trackbar* con cui potremmo interagire e vedere il cambiamento in tempo reale frame per frame.   
Le trackbar saranno 6, ognuna di essi gestirà il valore minimo e il valore massimo di H,S,V che vorremmo analizzare.   
I valori vanno da 0 a 255 e le trackbar appariranno nella gui **NORMAL**.
* Ruotiamo il frame in modo tale da averla specchiata a noi.
* Convertiamo l'immagine da **BGR ad HSV** ed inseriamo l'immagine convertita in hsv.
* La funzione *cvInRangeS* è una funzione di **filtraggio**.
* Attraverso i valori di massimo e minimo che gli vengono passati lei come risultato darà un'immagine binaria con **pixel bianchi per identificare la zona trovata e pixel neri per tutto il resto**.
* Riduco i disturbi nell'immagine binaria (_puntini bianchi che possono sfasare il processo di tracking_)
* Qui inizia la ricerca vera e propria, abbiamo la nostra immagine binaria con i suoi pixel bianchi, dobbiamo solo trovarli così da identificare dove si trova il colore da noi cercato all'interno di tale immagine.
* Per far questo facciamo una **ricerca su ogni pixel che compone l'immagine**:
* Se troviamo un _pixel bianco_ (255) salviamo i punti dei vertici del nostro rettangolo.
* Con la funzione *cvRectangle* disegniamo il nostro rettangolo in *imm* (il nostro frame) di colore rosso.   
Così da vedere nel frame della webcam il nostro rettangolo come se fosse presente all'interno della stanza.
* Con la funzione *cvCircle* disegniamo il baricentro del nostro rettangolo di blu.
* Dopo mostriamo le gui con il nostro frame (su cui abbiamo disegnato) e l'immagine binaria trovata.
* La funzione *cvWaitKey* aspetta _15 ms_ e ci permette di controllare se abbiamo premuto un tasto, infatti con lo switch controlliamo:
* Se premiamo il tasto 'q' il nostro programma **termina**.
* Se premiamo il tasto 's' scriviamo i dati presi sul file di configurazione in modo che tutti i software che leggano da quel file usino esclusivamente i valori prelevati.
* Alla fine prendiamo un altro frame e torniamo all'inizio del ciclo.
* In caso di uscita **deallochiamo il tutto**.

Ecco uno screenshot: (scusate la faccia di cazzo ma non ero in me, capitemi.)   
![calibra]({{site.image_url}}/examproject-parte-1.jpg)

Come si può vedere abbiamo le trackbar e abbiamo il nostro rettangolino rosso solo ed esclusivamente intorno all'area interessata, cioè quella di colore fucsia.   
Nella gui con l'immagine binaria possiamo vedere come venga preso solo il colore da noi impostato precedentemente segnando i pixel di bianco e mettendo di nero tutto il resto.   
Il rettangolino seguirà quel colore per tutto lo schermo, segnando anche il baricentro.   
Dopo aver salvato la nostra configurazione verrà scritto un file **config.txt** che conterrà i valori per identificare il colore.   
**Ricordate bene che i programmi che utilizzano tale file non funzioneranno correttamente se non sono stati presi valori DAVVERO ottimali o se il file in questione non viene creato, quindi il programma calibra dovrà essere il primo programma ad essere eseguito, chiamatela appunto "calibrazione"**.

Spero di essere stato il più chiaro possibile, se avete domande non esitate a chiedere, se avete suggerimenti non esitate a propormeli, va be' avete capito.


Saluti, DLion
