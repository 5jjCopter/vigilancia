Pues construiremos una alarma con raspberry pi para que cuando detecte un movimiento  realice lo siguiente.

Haga una foto y la guarde en una pendrive.
Envié un correo con aviso de alarma y foto adjunta.
Tenemos que tener raspberry con sistema operativo.

Entraremos en raspberry pi por ssh con putty.

Instalaremos Ssmtp escribiendo lo siguiente en consola.

sudo apt-get install ssmtp

sudo apt-get install mailutils

Ahora edite el archivo de configuración SSMTP con.

sudo nano /etc/ssmtp/ssmtp.conf

Tienes que incluir el siguiente texto.

Reemplaza correo_gmail@gmail.com con tu correo Gmail
root=xxxxxxx@gmail.com
mailhub=smtp.gmail.com:587
rewriteDomain=gmail.com
UseTLS=Yes
UseSTARTTLS=Yes
#Reemplaza correo_gmail@gmail.com con tu correo Gmail
AuthUser=xxxxxxx@gmail.com
#Reemplaza clave_correo_gmail con la clave de tu correo Gmail
AuthPass=xxxxxxxxxxxxx
FromLineOverride=YES

Guardar y Salir

Instalar mpack para enviar archivos adjuntos.

sudo apt-get install mpack

Ahora crearemos un scripts para que realice el trabajo de enviar el email.

sudo nano enviar-email.sh

y ponemos dentro.

!/bin/bash
{
echo To: correo-que-envia@gmail.com
echo From: correo-que-recive@gmail.com
echo Subject: PELIGRO ALARMA
echo adjunto foto

} | ssmtp nombre-correo-envia

Guardar y salir

ahora el scripts que realizara la foto.

sudo nano foto.sh

Y pondremos dentro



#!/bin/bash
DATE=$(date +"%Y-%m-%d_%H%M%S")
raspistill -vf -hf -o /media/usb/fotos/1.jpg
cd /home/pi
raspistill -vf -hf -o /media/usb/fotos/$DATE.jpg

Realiza primero una foto y la guarda como 1.jpg y después realiza otra y la guarda por fecha y hora.

Esto es para que 1.jpg siempre sea sobre escrita por la siguiente y pueda ser mandada.

Ahora crearemos el scrips que enviara la foto.

sudo nano enviar-foto.sh

Escribimos dentro lo suguiente

#!/bin/bash

sudo mpack -s "Test" /media/usb/fotos/1.jpg xxxxxxx@gmail.com

Ahora crearemos el scripts que hara que de ejecute todo.

sudo nano disparo.py

Y escrivimos lo siguiente.

#!/usr/bin/env python
# -*- coding: utf-8 -*-
import RPi.GPIO as GPIO    #Importamos la libreria GPIO
import time                #Importamos time
import subprocess
from time import gmtime, strftime  #importamos gmtime y strftime
GPIO.setmode(GPIO.BCM)             #Configuramos los pines GPIO como BCM
PIR_PIN = 7                        #Usaremos el pin GPIO nº7
GPIO.setup(PIR_PIN, GPIO.IN)       #Lo configuramos como entrada

#GPIO.setup(17, GPIO.OUT)          #Configuramos el pin 17 como salida (para un led)

try:
    while True:  #Iniciamos un bucle infinito
        if GPIO.input(PIR_PIN):  #Si hay señal en el pin GPIO nº7
#           GPIO.output(17,True) #Encendemos el led
            time.sleep(1)        #Pausa de 1 segundo
            timex = strftime("%d-%m-%Y %H:%M:%S", gmtime()) #Creamos una cadena de texto con la hora
            print timex + " MOVIMIENTO DETECTADO"  #La sacamos por pantalla
            subprocess.call(['/home/pi/foto.sh'])
            time.sleep(5)
            print timex + " ENVIANDO EMAIL"  #La sacamos por pantalla
            subprocess.call(['/home/pi/enviar-mail.sh'])
            print timex + " CORREO ENVIADO"  #La sacamos por pantalla
            time.sleep(1)  #Pausa de 1 segundo
            print timex + " ENVIANDO FOTO"  #La sacamos por pantalla
            subprocess.call(['/home/pi/enviar-foto.sh'])
            print timex + " FOTO ENVIADA"  #La sacamos por pantalla
#           GPIO.output(17,False)  #Apagamos el led
        time.sleep(1)              #Pausa de 1 segundo y vuelta a empezar
except KeyboardInterrupt:   #Si el usuario pulsa CONTROL + C...
    print "quit"            #Anunciamos que finalizamos el script
    GPIO.cleanup()          #Limpiamos los pines GPIO y salimos
	
	...
..













