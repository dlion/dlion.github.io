---
title: Muovere il puntatore con il joystick
summary: Il joystick, il tuo secondo mouse
categories: Programming
layout: post
---
Cercando un po’ in rete ho trovato una libreria piuttosto cool che mi permetteva di interfacciare un qualsiasi joystick usb che avevo in casa con il pc ricevendo tutti i segnali inviati da esso; quindi se premevo un pulsante, se ruotavo l’analogico, se andavo su o giù e così via.   
Allora mi son chiesto se era possibile spostare il puntatore del mouse semplicemente muovendo l’analogico del joystick,  mi sono messo all’opera…

La libreria di cui parlo è la PLib, una volta installata sarà una passeggiata utilizzarla.   
Ovviamente ho fatto tutto su Linux, ecco il sorgente:

{% highlight c lineanchors %}
/*###############################################################################
*# @Author : Domenico Luciani aka DLion
*# @Description: Simple snippet for move the pointer on the screen using a joystick
*# @How compile: g++ <source> -o <binary> -lplibjs -lplibsl -lplibsm -lplibul -lm -lX11
*# @Copyright : 2012
*# @Site : http://www.about.me/DLion
*# @License : GNU AGPL v3 http://www.gnu.org/licenses/agpl.html
*###############################################################################*/
 
#include <X11/Xlib.h>
#include <X11/X.h>
#include <X11/Xutil.h>
#include <plib/js.h>
 
int main()
{
    Display *monitor;
    Window win;
    jsJoystick *js[1];
    float *ax[1];
    int x=0,y=0,b;
 
    //Init plib
    jsInit();
 
    //get first joystick
    js[0] = new jsJoystick();
    
    //Check if the joystick is present
    if(js[0]->notWorking())
        printf("Joystick not detected!\n");
    else
    {
        //Get axes
        ax[0] = (float*)malloc((js[0]->getNumAxes())*sizeof(float));
        //Functions for X
        monitor = XOpenDisplay(0);
        win = XRootWindow(monitor,0);
        XSelectInput(monitor,win,NoEventMask);
 
        while(1)
        {
            //Get Button pressed and Axes
            js[0]->read(&b,ax[0]);
            
            //Increment o decrement x and y axes
            x+=ax[0][0];
            y+=ax[0][1];
          
            //Move pointer on the screen
            XWarpPointer(monitor,None,win,0,0,0,0,x,y);
            XFlush(monitor);
            
            //Sleep 1 second
            usleep(1000);
        }
    }
 
    return 0;
}
{% endhighlight %}

Per potervi interfacciare con il joystick vi basterà includere la libreria *plib/js.h* e per muovere il puntatore vi basterà includere le altre tre librerie di X.   
Il sorgente è piuttosto semplice, inizializzo il tutto, rilevo il joystick, prendo le coordinate e le setto al puntatore mostrando il tutto a schermo.   
Per compilare il tutto vi basta questo comando:
{% highlight sh lineanchors %}
g++ joystick.cpp -o joystick -lplibjs -lplibsl -lplibsm -lplibul -lm -lX11
./joystick
{% endhighlight %}

Se avete domande in proposito basta chiedere, il tutto funziona su qualsiasi joystick venga rilevato dalla vostra distro Linux.   
Ovviamente questo è solo un esempio scemo, l’unico limite è la vostra fantasia.

Saluti, DLion

---

[English Version](https://domenicoluciani.com/2013/02/26/move-cursor-pointer-with-joystick.html)
