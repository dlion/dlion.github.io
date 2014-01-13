---
layout: post
title: Edge Detector
---
Parliamo ancora delle OpenCV, uno strumento utile ed immenso che ci permette di fare cose impressionanti.   
Oggi vedremo come **rilevare i contorni** degli oggetti all'interno di una determinata immagine.   
Per fare ciò usaremo [Canny](http://en.wikipedia.org/wiki/Canny_edge_detector), un particolare filtro che ci permette di mettere in risalto i contorni di un qualcosa all’interno di una determinata immagine.

Prima di tutto vediamo un piccolo sorgente che ci consente di fare ciò:
{% highlight c linenos %}
/*
* Rilevatore di bordi con canny
* Snippet by Domenico Luciani aka DLion
*/
 
#include <highgui.h>
#include <cv.h>
 
int main()
{
    IplImage *frame,*out;
    CvCapture *immagine;
    
    //Catturo dalla cam
    immagine = cvCaptureFromCAM(0);
    //Catturo il primo frame
    frame= cvQueryFrame(immagine);
    //Creo out ompatibile con frame, ad un solo canale.
     out = cvCreateImage(cvGetSize(frame),IPL_DEPTH_8U,1);
 
    //Creo una finestra chiamata "Bordi"
    cvNamedWindow("Bordi",CV_WINDOW_AUTOSIZE);
 
    //Ciclo finchè frame è disponibile
    while(frame)
    {
        //Converto l'immagine nella scala di grigi
        cvCvtColor(frame, out, CV_BGR2GRAY);
        //Giro l'immagine
        cvFlip(out,out,1);
        //Applico il filtro di canny
        cvCanny(out,out,40,50,3);
        //Dilato l'immagine per risaltare i contorni trovati
        cvDilate(out,out,NULL,1);
        //Mostro l'immagine
        cvShowImage("Bordi",out);
        //Prendo il frame successivo
        frame = cvQueryFrame(immagine);
        //Aspetto 10 millisecondi per vedere viene premuto q (tasto di uscita)
        if(cvWaitKey(10) == 'q') break;
    }
    //Rimuovo tutto
    cvReleaseCapture(&immagine);
    cvReleaseImage(&frame);
    cvReleaseImage(&out);
    cvDestroyWindow("Bordi");
    return 0;
}
 
//If you want to see the example result you can see here: http://i.imgur.com/Peo11.jpg
{% endhighlight %}

Come potete ben vedere il sorgente è pieno di commenti.   
Vediamo il procedimento esatto:

1. Dichiaro le variabili necessarie.
2. Setto la webcam
3. Prendo un primo frame
4. Creo una immagine con le stesse dimensioni del primo frame **ma ad un canale solo**. ([immagine binaria](http://it.wikipedia.org/wiki/Immagine_binaria))
5. Creo una finestra dove inserire i frame. (visualizzazione)
6. Entro in un ciclo che si ripeterà finché frame non darà un valore false
7. Dentro di esso converto l'immagine originale (il frame preso dalla webcam) in una immagine in bianco e nero tramite la funzione *cvCvtColor*, mettendola nell'immagine preparata precedentemente.
8. Inverto l'immagine con la funzione *cvFlip*.
9. Applico *Canny* tramite la funzione [cvCanny](http://opencv.willowgarage.com/documentation/c/imgproc_feature_detection.html) impostando dei valori di [threshold](http://en.wikipedia.org/wiki/Thresholding_%28image_processing%29) ottimali per rendere i bordi visibili il più possibile.
10. Dilato l'immagine con la funzione *cvDilate* così da rendere più visibili i bordi
11. Mostro l’immagine dentro la finestra creata al punto 5.
12. Prendo un nuovo frame e sovrascrivo quello vecchio. (punto 3)
13. Controllo se premo il tasto ‘q’ tramite la funzione cvWaitKey (aspettando 10ms), se così è esco dal ciclo
14. Il ciclo viene ripetuto.
15. Dopo di ciò libero tutte le risorse allocate precedentemente terminando il programma.

Questo è il risultato:
![Canny]({{site.image_url}}/edge-detector.jpg)

Cambiando i valori di threshold potrete decidere voi se far vedere più contorni o farne vedere meno, io ho scelto di filtrare quasi tutto lasciando solo quello che mi interessava, provateci!

Per compilare il sorgente vi basta aver installato le OpenCV e digitare sulla vostra shell:
{% highlight sh linenos %}
g++ `pkg-config --cflags --libs opencv` source.c -o source
{% endhighlight %}

Le funzioni messe a disposizione dalle OpenCV ci permettono di fare il tutto con poche righe di codice, come avete appena potuto notare.   
Questo è solo un esempio scemo, presto vedremo altri usi di queste fantastiche librerie messe a disposizione dall'Intel.

Saluti, DLion.
