# filosofos
En una mesa hay procesos que simulan el comportamiento de unos filósofos que intentan comer de un plato. Cada filósofo tiene un cubierto a su izquierda y uno a su derecha y para poder comer tiene que conseguir los dos. Si lo consigue, mostrará un mensaje en pantalla que indique «Filosofo 2 comiendo».

Despues de comer, soltará los cubiertos y esperará al azar un tiempo entre 1000 y 5000 milisegundos, indicando por pantalla «El filósofo 2 está pensando».

En general todos los objetos de la clase Filósofo está en un bucle infinito dedicándose a comer y a pensar.

Simular este problema en un programa Java que muestre el progreso de todos sin caer en problemas de sincronización ni de inanición.

../_images/Filosofos.png
Esquema de los filósofos

Boceto de solución
import java.util.Random;


public class Filosofo  implements Runnable{
        public void run(){
                String miNombre=Thread.currentThread().getName();
                Random generador=new Random();
                while (true){
                        /* Comer*/
                        /* Intentar coger palillos*/
                        /* Si los coge:*/
                        System.out.println(miNombre+" comiendo...");
                        int milisegs=(1+generador.nextInt(5))*1000;
                        esperarTiempoAzar(miNombre, milisegs);
                        /* Pensando...*/
                        //Recordemos soltar los palillos
                        System.out.println(miNombre+"  pensando...");                           milisegs=(1+generador.nextInt(5))*1000;
                        esperarTiempoAzar(miNombre, milisegs);
                }
        }

private void esperarTiempoAzar(String miNombre, int milisegs) {
                try {
                        Thread.sleep(milisegs);
                } catch (InterruptedException e) {
                        System.out.println(
                                miNombre+" interrumpido!!. Saliendo...");
                        return ;
                }
        }
}
Solución completa al problema de los filósofos
Gestor de recursos compartidos (palillos)
package com.iesmaestre.filosofos;

public class GestorPalillos {
        boolean palilloLibre[];
        public GestorPalillos(int numPalillos){
                palilloLibre=new  boolean[numPalillos];
                for (int i=0; i<numPalillos; i++){
                        palilloLibre[i]=true;
                } //Fin del for
        } //Fin del constructor
        public synchronized boolean
                intentarCogerPalillos(int pos1, int pos2)
        {
                boolean seConsigue=false;
                if (
                                (palilloLibre[pos1])
                                &&
                                (palilloLibre[pos2]) )
                {
                        palilloLibre[pos1]=false;
                        palilloLibre[pos2]=false;
                        seConsigue=true;
                } //Fin del if
                return seConsigue;
        }

        public void liberarPalillos(int pos1, int pos2){
                palilloLibre[pos1]=true;
                palilloLibre[pos2]=true;
        }
}
Simulación de un filósofo
package com.iesmaestre.filosofos;

import java.util.Random;

public class Filosofo implements Runnable{
        GestorPalillos gestorPalillos;
        int posPalilloIzq,posPalilloDer;
        public Filosofo(GestorPalillos g, int pIzq, int pDer){
                this.gestorPalillos=g;
                this.posPalilloDer=pDer;
                this.posPalilloIzq=pIzq;
        }
        public void run() {
                while (true){
                        boolean palillosCogidos;
                        palillosCogidos=
                          this.gestorPalillos.intentarCogerPalillos(
                                          posPalilloIzq, posPalilloDer);
                        if (palillosCogidos){
                                comer();
                                this.gestorPalillos.liberarPalillos(
                                                posPalilloIzq,
                                                posPalilloDer);
                                dormir();
                        } //Fin del if
                } //Fin del while true
        } //Fin del run()

        private void comer() {
                System.out.println("Filosofo "+
                                Thread.currentThread().getName()+
                                " comiendo");
                esperarTiempoAzar();
        }
        private void esperarTiempoAzar() {
                Random generador=new Random();
                int msAzar=generador.nextInt(3);
                try {
                        Thread.sleep(msAzar);
                } catch (InterruptedException ex) {
                        System.out.println("Fallo la espera");
                }
        }
        private void dormir(){
                System.out.println("Filosofo "+
                                Thread.currentThread().getName()+
                                " durmiendo (zzzzzz)");
                esperarTiempoAzar();
        }
}
Lanzador de hilos
package com.iesmaestre.filosofos;
public class Lanzador {
        public static void main(String[] args) {
                int NUM_PROCESOS=5;
                Filosofo filosofos[]=new Filosofo[NUM_PROCESOS];
                GestorPalillos gestorPalillos;
                gestorPalillos=new GestorPalillos(NUM_PROCESOS);
                Thread hilos[]=new Thread[NUM_PROCESOS];
                for (int i=1; i<NUM_PROCESOS; i++){
                        filosofos[i]=new Filosofo(
                                gestorPalillos, i, i-1);
                        hilos[i]=new Thread(filosofos[i]);
                        hilos[i].start();
                }
                filosofos[0]=new Filosofo(
                                gestorPalillos, 0, 4);
                hilos[0]=new Thread(filosofos[0]);
                hilos[0].start();
        } //Fin del main
} //Fin de la clase
