---
title: ExamProject - Parte 3
description: Il mio progetto d'esame sulla visione artificiale
category: Programming
---
Dopo una breve pausa estiva riprendo a parlare del mio progetto d'esame sulla visione artificiale.   
Oggi andrò a descrivere il funzionamento del programma "Draw", chi non ha mai desiderato di poter **colorare** sullo schermo del proprio computer?

Vediamo il codice così da capire direttamente come funziona il tutto:
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
 
//Librerie necessarie
#include <cv.h>
#include <highgui.h>
#include "../lib/funzioni.h"
 
//Nome della Gui
#define NOME "Draw"
//File di configurazione
#define FILE_CONFIG "../config/config.txt"
 
 
int main(int argc, char *argv[])
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
            CvCapture *frame = cvCaptureFromCAM(web);
            cvSetCaptureProperty(frame,CV_CAP_PROP_FRAME_WIDTH,640);
            cvSetCaptureProperty(frame,CV_CAP_PROP_FRAME_HEIGHT,480);
            //Immagini
            IplImage *img = cvQueryFrame(frame);
            IplImage *hsv = cvCreateImage(cvGetSize(img),8,3);
            IplImage *binary = cvCreateImage(cvGetSize(img),8,1);
            IplImage *buffer = cvCreateImage(cvGetSize(img),8,3);
 
            int step = binary->widthStep/sizeof(uchar);
 
            int i,j,capo=0,XX,YY;
            char tasto;
 
            uchar *target = (uchar*)binary->imageData;
           //Valori
            HSV *low = (HSV*)malloc(sizeof(HSV));
            HSV *high = (HSV*)malloc(sizeof(HSV));
            //Il rettangolo
            Rettangolo *punti = (Rettangolo*)malloc(sizeof(Rettangolo));
 
            CvPoint pos,last;
            //Colore
            CvScalar colore = CV_RGB(191,255,255);
            //Creo la Gui
            cvNamedWindow(NOME,1);
            //Leggo i dati
            leggiConfig(low,high,(char*)FILE_CONFIG);
 
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
 
                punti->xmin = 10000;
                punti->xmax = 0;
                punti->ymin = 10000;
                punti->ymax = 0;
 
                //Rettangolo rosso
                cvRectangle(img,cvPoint(0,0),cvPoint(100,60),CV_RGB(255,0,0),CV_FILLED,8,0);
                //Rettangolo verde
                cvRectangle(img,cvPoint(200,0),cvPoint(300,60),CV_RGB(0,255,0),CV_FILLED,8,0);
                //Rettangolo blu
                cvRectangle(img,cvPoint(400,0),cvPoint(500,60),CV_RGB(0,0,255),CV_FILLED,8,0);
 
                //Controllo l'immagine pixel per pixel
                for(i=0; i < binary->height; i++)
                {
                    for(j=0; j < binary->width; j++)
                    {
                        //Se trovo il colore
                        if(target[i*step+j] == 255)
                        {
                            //Coordinate del mio rettangolo
                            if(j < punti->xmin)
                                punti->xmin = j;
                            if(j > punti->xmax)
                                punti->xmax = j;
                            if(i < punti->ymin)
                                punti->ymin = i;
                            if(i > punti->ymax)
                                punti->ymax = i;
                            //Baricentro del mio rettangolo
                            XX = (punti->xmin+punti->xmax)/2;
                            YY = (punti->ymin+punti->ymax)/2;
 
                            //Se entro dentro il rettangolo rosso
                            if(XX >= 0 && XX <= 100 && YY >= 0 && YY <= 60)
                            {
                                colore = CV_RGB(255,0,0);
                                capo = 1;
                            }
                            //Se entro dentro il rettangolo verde
                            else if((XX >= 200 && XX <= 300) && (YY >= 0 && YY <= 60))
                            {
                                colore = CV_RGB(0,255,0);
                                capo = 1;
                            }
                            //Se entro dentro il rettangolo blu
                            else if((XX >= 400 && XX <= 500) && (YY >= 0 && YY <= 60))
                            {
                                colore = CV_RGB(0,0,255);
                                capo = 1;
                            }
                        }
                    }
 
                }
                //Se ho toccato un rettangolo posso scrivere
                last = cvPoint(XX,YY);
                if(capo == 1)
                {
                    //Creo il cerchio nell'immagine buffer
                    cvCircle(buffer,last,20,colore,-1,CV_AA,0);
                    //Faccio il merge fra il frame e il buffer
                    cvAdd(img,buffer,img,NULL);
                }
                else
                {
                    buffer = cvCreateImage(cvGetSize(img),8,3);
                    //Disegno il cerchio direttamente nel frame in modo da usarlo come puntatore
                   cvCircle(img,last,20,CV_RGB(159,200,120),-1,CV_AA,0);
                }
                //Mostro la gui
                cvShowImage(NOME,img);
                //Prendo il tasto premuto
                tasto = cvWaitKey(15);
 
                //Se premo q esco
                if(tasto == 'q')
                    return 0;
                //Se premo C cancello ogni cosa
                else if(tasto == 'c')
                {
                    cvReleaseImage(&buffer);
                    buffer = cvCreateImage(cvGetSize(img),8,3);
                    capo = 0;
                }
 
 
                img = cvQueryFrame(frame);
            }
            cvReleaseImage(&buffer);
            cvReleaseImage(&img);
            cvReleaseImage(&binary);
            cvReleaseImage(&hsv);
            cvReleaseCapture(&frame);
        }
        else
            puts("webcam not found\n");
    }
    return 0;
}
{% endhighlight %}

In pratica, come già detto, si tratta di un piccolo programmino che permette di "disegnare" sulla gui con un qualsiasi oggetto da noi scelto. (negli esempi precedenti abbiamo usato un pennarello)   
Per realizzare questo programmino mi è venuto in mente un "hack" piuttosto simpatico, il problema principale era che ad ogni frame ciò che disegnavo veniva cancellato/sovrascritto, come risolvere il problema ?

Ho lasciato invariato il frame ed ho creato una immagine "buffer" su cui disegnare e poi unire le due cose facendo un merge delle due immagini così da avere un frame con dentro ciò che ho disegnato per poi far spuntare a video il tutto. Figo, no ? In questo modo avremo il "disegno" sul frame in modo tale da non cancellare il disegno fatto ogni volta che prendiamo un nuovo frame.

Premendo "c" il buffer viene ricreato cancellando il disegno fatto.

Inizialmente il buffer non viene usato, per far vedere il "puntatore", inzierà a scrivere solo quando entreremo all'interno di un rettangolo, un po' come se fosse una tavolozza di colori.

Includiamo le solite librerie necessarie, definiamo i nomi delle gui e facciamo le solite cose, creiamo 3 rettangoli di colori diversi: uno rosso, uno verde e uno blu; tutti pieni. (sì, i colori li ho scelti casualmente, ovviamente se volete voi potete cambiarli) Prendiamo come centro il baricentro del rettangolo. Se entro dentro un qualsiasi rettangolo imposto il colore del puntatore con il colore del rettangolo toccato. Se non ho toccato il rettangolo disegno un cerchio nel frame che verrà cancellato subito dopo, usato principalmente come "puntatore".   
Aspetto 15 millisecondi e se premo il tasto "q" esco, se premo il tasto "c" distruggo il buffer per ricrearlo vuoto e faccio tornare il puntatore. Se esco distruggo tutto liberando la memoria.

Il tutto continua a funzionare disegnando a schermo tanti cerchi consecutivi e del colore del rettangolo toccato, emulando proprio la "pittura".

Il programma termina qui, la maggior parte delle funzioni viste le abbiamo [incontrate precedentemente](/examproject-parte-2/). Quindi ho preferito evitare di essere troppo ridondante e di arrivare alla parte viva del programma tralasciando parti già viste.

![Draw](/public/img/posts/examproject-parte-3.jpg)

Il sorgente è commentato e si può ben notare che circa l'80% delle cose le abbiamo già viste in precedenza.   
Per qualsiasi cosa rivedetevi gli articoli precedenti oppure chiedete pure.

Saluti, DLion
