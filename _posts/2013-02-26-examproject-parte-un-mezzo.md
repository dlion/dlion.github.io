---
title: ExamProject parte ½
summary: Il mio progetto d'esame sulla visione artificiale
categories: Programming
layout: post
---
Nell'articolo precedente vi ho introdotto il mio progetto d'esame sulla visione artificiale.   
Inizialmente mi è d'obbligo parlarvi di una piccolissima libreria che ho scritto per facilitarmi alcune cose piuttosto ridondanti, vedremo nel dettaglio le sue caratteristiche introducendo alcuni concetti base come l'HSV o il ROI.

Parto dicendo che potete trovare il progetto al seguente link: [https://github.com/dlion/ExamProject](https://github.com/dlion/ExamProject)   
Come potete ben notare ho creato un repository contenete tutti i sorgenti divisi per cartelle.   
Vi consiglio di clonare il repository in locale e compilarlo lì piuttosto che provare ogni singolo programma separatamente.

La libreria si trova nella directory *lib* dentro la quale potrete trovare l'header con i prototipi delle funzioni e il sorgente delle stesse.

- Header File
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
 
#include "funzioni.c"
 
/*
* -Funzione per leggere il file di configurazione-
* I primi 2 parametri indicano dove salvare i valori presi.
* I valori presi vengono salvato su 2 strutture apposite.
* Il terzo parametro è il nome del file da cui leggere.
*/
void leggiConfig(HSV*,HSV*,char*);
 
/*
* -Funzione per scrivere sul file di configurazione-
* Stessi parametri di prima.
*/
void scriviConfig(HSV*,HSV*,char*);
 
/*
* -Funzione per diminuire la dimensione di un'immagine-
*  Il primo parametro indica l'immagine.
*  Il secondo parametro è il valore in percentuale di riduzione.
*  Viene ritornata l'immagine modificata.
*/
IplImage* diminuisci(IplImage*,int);
 
/*
* -Funzione per ridurre i disturbi in un'immagine-
*  Il primo parametro indica l'immagine sorgente.
*  Il secondo parametro indica l'immagine di destinazione.
*/
void riduciNoise(IplImage*,IplImage*);
 
/*
* -Funzione per inserire un'immagine in un'altra immagine-
*  Il primo parametro indica l'immagine da inserire.
*  Il secondo parametro indica dove inserire l'immagine.
*  Il terzo e il quarto parametro indicano le coordinate dove posizionare l'immagine.
*/
void inserisci(IplImage*,IplImage*,int,int);
{% endhighlight %}

- C File
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
 
typedef struct
{
    int H,S,V;
}HSV;
 
typedef struct
{
    int xmin,xmax;
    int ymin,ymax;
}Rettangolo;
 
void leggiConfig(HSV *m,HSV *n,char *file)
{
    FILE *config;
    config = fopen(file,"r");
    if(config != NULL)
        fscanf(config,"%d,%d,%d,%d,%d,%d",&m->H,&m->S,&m->V,&n->H,&n->S,&n->V);
    else
    {
        m->H = m->S = m->V = 0;
        n->H = n->S = n->V = 255;
    }
 
    printf("--- VALORI PRESI ---\n## MIN ##\nH->%d S->%d V->%d\n## MAX ##\nH->%d S->%d V->%d\n",m->H,m->S,m->V,n->H,n->S,n->V);
    fclose(config);
}
 
void scriviConfig(HSV *m,HSV *n,char *file)
{
    FILE *config;
    config = fopen(file,"w");
    if(config != NULL)
    {
        fprintf(config,"%d,%d,%d,%d,%d,%d",m->H,m->S,m->V,n->H,n->S,n->V);
        puts("Valori scritti correttamente");
    }
    fclose(config);
}
 
IplImage* diminuisci(IplImage* file,int perc)
{
    IplImage *ris = cvCreateImage(cvSize((int)((file->width*perc)/100),(int)((file->height*perc)/100)),8,file->nChannels);
 
    cvResize(file,ris);
 
    return ris;
}
 
void riduciNoise(IplImage *src,IplImage *dst)
{
    IplImage *buff = cvCreateImage(cvGetSize(src),8,dst->nChannels);
    cvDilate(src,buff,NULL,1);
    cvErode(buff,buff,NULL,2);
    cvSmooth(buff,dst,CV_GAUSSIAN,5);
    cvReleaseImage(&buff);
}
 
void inserisci(IplImage *small,IplImage *big,int pos1,int pos2)
{
    CvRect dim = cvRect(pos1,pos2,(int)small->width,(int)small->height);
    cvSetImageROI(big,dim);
    cvCopy(small,big);
    cvResetImageROI(big);
}
{% endhighlight %}

Per avvicinarmi di più a quella che è la vera visione non potevo lasciare il tutto basato sull'RGB quindi l'ho abbandonato per usare l'**HSV**.   
Infatti dentro la libreria troviamo una struttura HSV che ho usato nei sorgenti per semplificarmi il lavoro durante l'utilizzo di questo modello additivo di composizione dei colori con 3 interi che rappresentano rispettivamente il valore della **tonalità (H = Hue)**, il valore della **saturazione (S = Saturation)** e il valore della **luminosità (V = Value)**.

Vediamo di parlare un po' di più del modello HSV.   
Il modello HSV è orientato per *emulare il più possibile la visione umana essendo basato sulla percezione che si ha di un colore basandosi in termini di tinta, sfumatura e tono*.   
![HSV](/images/examproject-parte-un-mezzo.png)   
Il sistema di coordinate è cilindrico ed è definito come un cono distorto.   
H = Angolo intorno all'asse verticale.   
S = La saturazione va da 0, sull'asse del cono, a uno sulla superficie del cono.   
V = La luminosità è l'altezza del cono.   
Soppianteremo quindi l'[RGB](http://it.wikipedia.org/wiki/RGB) per usare l'HSV così da aumentare la possibilità ed ottimizzare il tracking del colore che ci serve.

Dopo troviamo una semplice struttura *Rettangolo* che ci mette a disposizione degli interi per definire i vertici della figura geometrica.   
Questa struttura ci consentirà di definire il rettangolo che potremmo disegnare sull'immagine o anche solo rilevare se abbiamo o meno toccato un punto specifico da noi interessato.

Proseguendo troviamo la funzione *leggiConf*, a cui passati i puntatori alla struttura HSV  per i primi due parametri più il file di configurazione da cui trarremmo i dati ci permetterà di prelevare i diversi valori che ci consentiranno di trovare il nostro colore all'interno dell'immagine per poi stampare su console i valori presi.   
Questa funzione verrà usata all'interno di ogni programma che dovrà trackare un determinato colore scelto da noi.




Continuando troviamo la funzione scriviConf che accetta gli stessi parametri della funzione *leggiConf* con la differenza che questa funzione permette il salvataggio dei dati sul file di configurazione.   
Questa funzione ci permetterà di applicare a tutti i sorgenti che usano il file di configurazione di apportare le modifiche del colore una volta sola senza bisogno di editare o ricompilare i sorgenti tutte le volte che vorremmo riusare un programma.

La funzione *diminuisci* che troviamo dopo permette di ridurre la grandezza di un'immagine di una determinata percentuale scelta da noi.   
La funzione ritorna un puntatore alla struttura IplImage definita nelle openCV che ci consente la gestione delle immagini e accetta come parametri un puntatore a **IplImage** (quindi un'immagine) e un intero che c'indicherà la percentuale di riduzione.
All'interno della funzione creiamo un'immagine più piccola grazie alla funzione *cvCreateImage* indicando la grandezza dell'immagine da creare (in questo caso la larghezza moltiplicata la percentuale diviso 100), i il numero di bit per pixel (8) e il numero di canali dell'immagine (va da 1 a 4).   
Passiamo il tutto alla funzione *CvResize* che si occuperà di ridurre l'immagine iniziale ed adattarla all'immagine successivamente creata per poi ritornare il puntatore a quell'immagine.

Per abbreviare le cose mi sono creato una funzione riduciNoise che riceve come parametri un'immagine sorgente e un'immagine destinataria effettuando la cosiddetta "apertura" e "chiusura" dell'immagine **dilatando** l'immagine, **erodendola** per poi applicare lo **smooth**.   
La dilatazione genera un incremento delle dimensioni dell'immagine riempiendo i buchi e le aree mancanti unendole tra di loro.   
L'erosione genera un decremento delle dimensioni dell'immagine rimuovendo piccole anomalie.   
Lo smooth è un particolare **filtro** per pulire l'immagine da disturbi vari evidenziando i pattern significativi attenuando il rumore.   
Questa funzione verrà utilizzata per migliorare il controllo sull'immagine così da avere meno disturbi e quindi **meno falsi positivi**.

Dato che in giro si trova poco ho anche creato la funzione *inserisci* che prende come parametri un'immagine piccola, un'immagine grande e 2 interi.   
Tale funzione permette d'inserire un'immagine piccola in un'immagine più grande in delle coordinate definite dai 2 interi passatogli.   
Una volta dentro la funzione creiamo una variabile con le coordinate ricevute e la dimensione dell'immagine piccola.   
Dopo di che settiamo un **ROI** nell'immagine grande di dimensioni contenute in dim nella posizione contenuta sempre in dim, dopo copiamo l'immagine piccola nell'immagine grande per poi resettare il [ROI](http://en.wikipedia.org/wiki/Region_of_Interest).

## Vediamo cosa è il ROI.
Chiamato anche Region of interest, cioè un'area di un'immagine che ci interessa.   
![ROI](/images/examproject-parte-un-mezzo-1.gif)   
Una volta definito il ROI molte funzioni di OpenCV andranno a lavorare **SOLO** in quella locazione tralasciando tutto il resto. (come vediamo in *cvCopy* che copierà la piccola immagine SOLO nel ROI grande quanto se stessa lasciando stare tutto il resto)   
_Una volta resettato il ROI tutto torna normale_   
Il ROI è utilissimo soprattutto per concentrarci su porzioni dell'immagine senza processare su tutta l'immagine, il che non solo ci permetterà di alleggerire i calcoli ma anche d'incentrarci maggiormente su dettagli che altrimenti verrebbe scomodo analizzare.

**La libreria conclude qui.**   
Le funzioni potevano essere implementate ancor di più ma dato il semplice progetto da me creato non ne ho visto il bisogno. (anche per mancanza di tempo)   
Spero che la prima vera parte di questo progetto vi abbia incuriosito, presto metterò il resto sperando che possa essere utile a qualcuno.

Saluti, DLion

---

[English Version](https://domenicoluciani.com/2013/02/26/exam-project-prelude.html)
