---
title: ExamProject - Parte 5
summary: Il mio progetto d'esame sulla visione artificiale
categories: Programming
layout: post
---
Un altro -ed ultimo- post sul mio progetto d'esame sulla visione artificiale.   
Questa volta vi parlerò di "Movement", un software in grado di monitorare una determinata zona.   
La peculiarità di questo software è che è in grado di registrare SOLO ciò che è davvero importante all'interno dello scenario, quindi solo quando c'è del movimento all'interno della zona da monitorare.   

Esistono due modalità d'uso:

* La versione statica che permette di fotografare un determinato scenario e di registrare quando si verificano delle modifiche rispetto all'originale.
* La versione dinamica che permette di confrontare un frame dall'altro funzionando praticamente da sensore di movimento.

Ecco il sorgente

{% highlight c lineanchors %}
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
#include <time.h>
#include "../lib/funzioni.h"
//Nome della gui
#define NOME "Control"
int main(int argc,char **argv)
{
    if(argc != 2)
    {
        printf("usage: %s <mode>\n0 - integrate webcam\n1 - external webcam\n",argv[0]);
        exit(-1);
    }
    else
    {
        int web=atoi(argv[1]);
        if(web >= 0 && web <= 1)
        {
            CvCapture *cam = cvCaptureFromCAM(web);
            cvSetCaptureProperty(cam,CV_CAP_PROP_FRAME_WIDTH,640);
            cvSetCaptureProperty(cam,CV_CAP_PROP_FRAME_HEIGHT,480);
            IplImage *img = cvQueryFrame(cam);
            IplImage *copia = cvCreateImage(cvGetSize(img),8,3);
            IplImage *prima = NULL;
            IplImage *binary = cvCreateImage(cvGetSize(img),8,1);
            IplImage *ris = cvCreateImage(cvGetSize(img),8,3);
            cvNamedWindow(NOME,1);
            //Variabili per prendere l'orario e la data correnti
            time_t tempo;
            struct tm *timeobj;
            time(&tempo);
            timeobj = localtime(&tempo);
            char nome[25];
            long int num=0;
            //Funzione per inserire i dati del tempo in nome
            strftime(nome,24,"%H-%M-%S_%F.avi",timeobj);
            //Creo il writer che si occuperà di scrivere i vari frame presi come video compresso in formato divx
            CvVideoWriter *video = cvCreateVideoWriter(nome,CV_FOURCC('D','I','V','X'),15,cvSize(640,480),1);
            //Inizializzo i font
            CvFont scritta,info;
            cvInitFont(&scritta,CV_FONT_HERSHEY_SIMPLEX,1.0,1.0,0,5,CV_AA);
            cvInitFont(&info,CV_FONT_HERSHEY_SIMPLEX,.6,.6,0,1,6);
            char tasto;
            int i,j,trovato=0,scelta,step = binary->widthStep/sizeof(uchar);
            uchar *target = (uchar*)binary->imageData;
            //Scelta fra dinamica e statica
            do
            {
                printf("-- Scelta modalita' --\n1)Dinamica -- Se ci saranno variazioni tra un frame e l'altro\n2)Statica -- Se ci sono variazioni fra un determinato frame e il frame corrente\nScelta: ");
                scanf("%1d",&scelta);
            }while(scelta < 1 || scelta > 2);
            while(img)
            {
                //Ruoto l'immagine
                cvFlip(img,img,1);
                //Prendo le informazioni sul tempo
                time(&tempo);
                timeobj = localtime(&tempo);
                strftime(nome,24,"%H:%M:%S %F",timeobj);
                //Scrivo le info a schermo
                cvPutText(img,nome,cvPoint(415,475),&info,CV_RGB(0,255,255));
                //Copio il frame
                cvCopy(img,copia);
                riduciNoise(img,img);
                //Dinamica
                if(scelta == 1)
                {
                    //Se è il primo frame preso
                    if(prima == NULL)
                    {
                        prima = cvCreateImage(cvGetSize(img),8,3);
                        //Copio img in prima
                        cvCopy(img,prima);
                    }
                    else
                    {
                        //Se non è il primo frame controllo se ci sono differenze
                        cvAbsDiff(img,prima,ris);
                        //Da colore a grigia
                        cvCvtColor(ris,binary,CV_BGR2GRAY);
                        //Il threshold dell'immagine
                        cvThreshold(binary,binary,62,255,CV_THRESH_BINARY);
                        riduciNoise(binary,binary);
                        cvCopy(img,prima);
                    }
                }
                //Statica
                else
                {
                    //Se ho preso il frame da monitorare
                    if(prima != NULL)
                    {
                        cvAbsDiff(img,prima,ris);
                        cvCvtColor(ris,binary,CV_BGR2GRAY);
                        cvThreshold(binary,binary,62,255,CV_THRESH_BINARY);
                        riduciNoise(binary,binary);
                    }
                }
                //Controllo l'immagine pixel per pixel
                for(i=0; i < binary->height && trovato != 1; i++)
                {
                    for(j=0; j < binary->width && trovato != 1; j++)
                    {
                        if(target[i*step+j] == 255)
                        trovato = 1;
                    }
                }
                //Se trovo un cambiamento
                if(trovato)
                {
                    num++;
                    //Inserisco "REC O" nell'immagine
                    cvPutText(copia,"REC",cvPoint(10,25),&scritta,CV_RGB(255,0,0));
                    cvCircle(copia,cvPoint(100,15),5,CV_RGB(255,0,0),20,8);
                    //Salvo il frame trovato
                    cvWriteFrame(video,copia);
                    trovato = 0;
                }
                //Mostro l'immagine
                cvShowImage(NOME,copia);
                tasto = cvWaitKey(15);
                if(tasto == 'q')
                    break;
                //Se premo v salvo il frame da monitorare
                else if(tasto == 'v' && scelta == 2)
                {
                    prima = cvCreateImage(cvGetSize(img),8,3);
                    cvCopy(img,prima);
                }
                img = cvQueryFrame(cam);
            }
            //Se ho preso dei frame
            if(num != 0)
            {
                //Scrivo il video
                cvReleaseVideoWriter(&video);
                printf("Video %s salvato\n",nome);
            }
        }
        else
            puts("webcam not found");
    }
    return 0;
}
{% endhighlight %}

Se il software trova delle differenze mostra in alto `REC` seguito da un cerchio rosso iniziando a salvare tutti i frame che differiscono da quello originale/precedente.
Una volta chiuso il programma prima di terminare salva tutti i frame facendone un video che salverà nella sua directory con nome uguale alla data corrente e l'ora così avendo a disposizione un video con **SOLO** i cambiamenti evitando video di parecchi GB.

## Parte statica

Abbiamo una parte da monitorare.   
Posizioniamo la webcam verso la parte da controllare (assicurandoci di uscire dalla vista della webcam) e premiamo `v`; da quel momento qualsiasi cosa passerà dalla zona monitorizzata o se la webcam vedrà delle differenze rispetto al frame preso da noi verrà registrata.

## Parte dinamica

La parte dinamica funziona in una modalità simile alla precedente con la differenza che noi non avviamo la registrazione.   
Il programma una volta avviato prenderà vari frame ed ogni frame verrà confrontato con il precedente, se trova una differenza fra i 2 frame salva il frame con la differenza.    
Alla fine crea un video dei frame presi esattamente come la modalità precedente.   
In ogni frame verrà salvato ora e data.

## Source
Inizialmente includiamo le solite librerie, inizializziamo la webcam e creiamo immagini che ci serviranno successivamente; creiamo la gui e tramite la funzione `localtime` ci ricaviamo l'ora e la data corrente per poi salvare il tutto nella variabile `nome`.   
Ci creiamo il writer passando come parametri il nome del file `o-r-a_d-a-t-a.avi`, la codifica (usiamo la divx per una maggiore compressione), i frame per secondo, la grandezza del video e se è un video a colori o meno.   

Inizializziamo le scritte e prendiamo i dati dell'immagine; facciamo scegliere la modalità, giriamo l'immagine, aggiungiamo al frame l'orario e la data e copiamo il frame in una variabile `copia` che ci servirà per il video in uscita, non dovrà essere modificata.   
Riduciamo i disturbi nel frame ed entriamo nel ciclo; se abbiamo scelto la modalità automatica controlliamo che quello corrente sia il primo frame; se è il primo frame creiamo una immagine temporanea dove copieremo questo primo frame.   
Se non è il primo frame controlliamo le differenze fra il frame corrente e il precedente con la funzione `cvAbsDiff` passando tramite parametri il frame corrente, quello precedente e un'immagine dove metteremo il risultato; convertiamo l'immagine in bianco e nero ed effettuiamo un `threshold` dell'immagine per poi ridurre eventuali noise ricavando una immagine binaria pulita copiando il frame nuovo nella variabile temporanea, aggiornandola.   
Nel caso avessimo scelto la modalità statica controlliamo se abbiamo l'immagine temporanea che conterrà il frame da monitorare; In caso positivo facciamo esattamente la stessa cosa di prima senza copiare il frame ovviamente.   
Controlliamo l'immagine binaria in cerca di differenze, se ci sono pixel bianchi ci sono differenze scrivendo a schermo `REC` così da avvertire che stiamo registrando e con `cvWriteFrame` memorizziamo il frame appena preso.   
Mostriamo il tutto aspettando 15ms; se premiamo il tasto `v` e abbiamo scelto la modalità statica creiamo l'immagine temporanea e copiamo il frame dentro di essa per poi prendere un altro frame e tornare all'inizio del ciclo da cui una volta usciti potremo vedere il risultato usando un banalissimo video player come VLC.

Il progetto termina qui, al solito per qualsiasi dubbio, segnalazioni, etc. sapete dove trovarmi.   
Al solito il progetto è [online](https://github.com/dlion/ExamProject).

Saluti, DLion.
