---
title: Ostega, steganografia con le OpenCV
description: Nascondere del testo all'interno di una immagine con le opencv
category: Programming
layout: post
---
Dopo il lavoro avevo 10 minuti liberi e volero riprendere a far roba idiota con il C e le OpenCV.   
Anni fa lessi alcuni articoli ed "affrontai" in molti *Hacker Game* la tecnica della [Steganografia](http://it.wikipedia.org/wiki/Steganografia)…

Tale tecnica permette di "inserire" all’interno di una immagine un qualsiasi testo così da "nasconderlo" alla vista di chiunque.   
Cioè alla vista di tutti, quella che io passavo era una banalissima immagine quando invece dentro di essa si nascondeva un messaggio criptato leggibile solo da chi aveva il decripter.   
Quindi armato di buona volontà mi misi a leggere alcuni articoli su tale tecnica e su come attuarla.

Le OpenCV permettono l'**interazione diretta con i pixel di una immagine** quindi il risultato non fu poi così complesso così nacque **OStega**, un piccolo e banale progettino realizzato con le OpenCV e il C che mi permetteva di usare una qualsiasi immagine bitmap come "copertura" per nascondere una determinata parola che fornivo in input per poi ritrovarla dando in pasto al mio decripter la stessa immagine, vediamolo in dettaglio il progetto.

Il sorgente della libreria *OStega.h* contenente i prototipi delle 2 funzioni (cripta/decripta) è questo:
{% highlight c linenos %}
/*
A simple libraries to use steganography with OpenCV
Thinking and created by Domenico Luciani aka DLion
*/

#include "OStega.c"


/* Function to insert and crypt a message into an bmp image.
 * first parameter is an image, second parameter is a message.
 * It returns -1 to error or 0 to complete successfull.
 */
int imgStega(IplImage*, char*);

/* Function to get a steganographed message into an bmp image
 * first parameter is an image
 * It returns the decrypted message
 */
char *imgDestega(IplImage*);
{% endhighlight %}

Come potete vedere le funzioni sono 2:

* la prima serve per inserire la parola all'interno di una immagine fornendo come primo parametro una immagine e come secondo parametro la parola da inserire. Ritornerà un semplice intero per avvisarvi se tutto è andato per il verso giusto.
* La seconda funzione accetta come parametro l'immagine contenete la parola criptografata e ritornerà lo stesso decriptato.

Ora vediamo il sorgente delle 2 funzioni in questione:
{% highlight c linenos %}
/* Functions to OStega project.
* thinking and created by Domenico Luciani aka DLion
*/
  
int imgStega(IplImage *img, char *msg)
{
    int wid = img->width;
    uchar *data = (uchar*)img->imageData;
    
    int j,k=0;
    int len = strlen(msg);
    char *new_str = (char*)malloc((len+2)*sizeof(char));
 
    new_str[0] = '$';
 
    for(j=1; j <= len; j++)
        new_str[j] = msg[j-1];
 
    new_str[len+1] = '$';
 
    if(img->nChannels != 3)
        return -1;
 
    for(j=0,k=0; j < wid || k < (len+2); j+=3,k++)
        data[j*3] = new_str[k];
 
    return 0;
}
    
char *imgDestega(IplImage *img)
{
    int wid = img->width;
    uchar *data = (uchar*)img->imageData;
    
    int j,k=0;
    char find;
    char *buffer = NULL;
 
    for(j=0; j < wid; j+=3)
    {
        find = data[j*3];
 
        if(j == 0)
        {
            if(find != '$')
                exit(EXIT_FAILURE);
            else
                continue;
        }
        else
        {
            if(find == '$')
                break;
            else
            {
                buffer=(char*)realloc(buffer,(k+1)*sizeof(char));
                buffer[k] = find;
                k++;
            }
        }
    }
    
    buffer = (char*)realloc(buffer,(k+1)*sizeof(char));
    buffer[k] = '\0';
 
    return buffer;
}
{% endhighlight %}

Vediamo in dettaglio la funzione *imgStega*.

Come detto prima abbiamo l'immagine *img* e la parola *msg*   
Prendiamo la larghezza dell'immagine inserendola in *wid* e la lunghezza della parola.

###**Attenzione, l’immagine deve essere NECESSARIAMENTE in formato bmp perché tali formati non vengono compressi né pacchettizzati e questo semplice sorgente lavora pixel per pixel senza decomprimere o fare altre operazioni più complesse.**

Se il canale dell'immagine è diversa da 3 (quindi una immagine normale a colori deve essere) ritorna -1 uscendo.   
Vediamo l’algoritmo che si cela dietro il processo steganografico:

* Creiamo una stringa lunga la parola + 2 e mettiamo dentro la prima posizione e dentro l'ultima il carattere '$' poi dentro queste 2 posizioni inseriamo tutti i caratteri della nostra parola. In pratica se inseriamo 'cane' avremo '$cane$'.
* Dopo di ciò entriamo in un ciclo che viene eseguito finché non arriviamo alla fine dell'immagine o la variabile k supera la lunghezza della nuova stringa, la variabile j che sarà la nostra posizione all'interno dell'immagine verrà incrementata di 3 posizioni ogni volta. In poche parole ogni 3 pixel metti un carattere e se tutto andrà bene ritornerà 0.

Ora non ci resta che vedere l’altra funzione.   
Come abbiamo detto basta fornire l'immagine con la parola criptografata e lei dovrebbe trovare la parola in questione.

* Prendiamo la larghezza dell'immagine e facciamo un ciclo fin quando non raggiungiamo la fine dell'immagine.
* Se nella prima posizione troviamo un '$' significa che la nostra parola è presente all'interno dell'immagine così continuiamo, altrimenti usciamo.
* Se nelle iterazioni successive non troviamo il carattere '$'  mettiamo tutto in un buffer che incrementeremo di volta in volta e alla fine una volta trovare il dollaro ritorniamo la parola trovata.

Semplice, no ?   
Ora vediamo come usare il tutto:
{% highlight c linenos %}
#include <highgui.h>
#include <cv.h>
#include <string.h>
#include "OStega.h"
#include <stdlib.h>
 
int main(int argc, char **argv)
{
    IplImage *in;
    char *img,*message;
    int mode,result;
    
    if(argc <= 2)
    {
        printf("Usage: %s <mode> <image_in> <<message>>\n",argv[0]);
        return -1;
    }
 
    mode = atoi(argv[1]);
 
    if(mode != 1 && mode != 2 )
    {
        printf("You can specify what do you do.\n",argv[0]);
        return -1;
    }
 
    in = cvLoadImage(argv[2],CV_LOAD_IMAGE_COLOR);
    
    if(mode == 1 && argc == 4)
    {
        message = argv[3];
 
        if(strlen(message) >= (in->width*in->height))
        {
            puts("Message too long\n");
            return -1;
        }
        else
        {
            result = imgStega(in,message);
            if(result == 0)
            {
                puts("Message hidden");
                cvSaveImage("ImageHidden.bmp",in,0);
            }
            else
            {
                printf("Error: %d\n",result);
                return -1;
            }
        }
    }
    else if(mode == 2 && argc == 3)
            printf("Il messaggio trovato e': %s\n",imgDestega(in));
    
    cvReleaseImage(&in);
    return 0;
}
{% endhighlight %}

Includiamo le librerie necessarie, prendiamo i parametri da linea di comando in questo modo: `./stega 1 image.bmp cane` per criptare la parola "cane" dentro l’immagine image   
oppure `./stega 2 image.bmp` per ricavare la parola decriptata.

* Nel primo caso carichiamo l'immagine e controlliamo che la parola non sia troppo grande per l'immagine fornita (altrimenti ritorno -1).   
Uso la funzione *imgStega* vista in precedenza e salvo nella stessa directory una immagine chiamata *"ImageHidden.bmp"* che conterrà la parola criptografata.
* Nel secondo caso richiama la funzione *imgDestega* e stampa la parola per poi deallocare il tutto.

### **Sì lo so, il codice fa schifo, poteva essere implementato meglio, orribile, non è commentato, è una gran cazzata, ti odio, etc.**   
Ma sinceramente era solo una prova, quindi prendetela come base, io ho fatto un semplice algoritmo idiota che incapsula la parola fra i dollari e posiziona i caratteri ogni 3 pixel.   
Il tutto lo potete trovare sul mio GitHub: [https://github.com/dlion/OStega](https://github.com/dlion/OStega)   
**Ovviamente** per qualsiasi cosa, io sono qui.

Saluti, DLion.

---

[English Version](https://domenicoluciani.com/2013/03/15/ostega-opencv-steganography.html)
