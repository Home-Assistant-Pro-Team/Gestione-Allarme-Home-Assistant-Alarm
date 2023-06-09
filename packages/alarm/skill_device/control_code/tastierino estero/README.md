# Realizzazione di un pannello remoto da integrare in HA con ESPhome

Il progetto si sviluppa con un nuovo device in EspHOME che richiama due stack in C (indipendenti). 
I file dello stack vanno caricati nella cartella "esphome"

![3X4-3X3-Matrix-12-9-Key-Membrane-Switch-Keypad-3X4-3X3-Matrix-Keyboard](https://user-images.githubusercontent.com/48358142/227930435-ab25367f-1f9a-42cd-bc6d-0e4a4c4f3f35.jpg)

Vengono crearti in HA 2 sensori:
- sensor.tastierino_esterno
- sensor.tastierino_esterno_input

Il sensore tastierino_esterno_input mostra per un secondo il tasto premuto e poi ritorna in uno stato di "sconosciuto".

Lo stack tiene tiene conto di tutti i tasti premuti entro un certo lasso di tempo e li passa in HA in un unica stringa quando si preme il tasto * aggiornando il valore di sensor.tastierino_esterno. La stringa viene svuotata dopo qualche sencondo.
Il tasto # vuota tutto.

il file keypad_sensor.h crea il sensore tastierino_esterno_input
il file keypad_textsensor.h crea il sensore tastierino_esterno


## Lista della spesa
- 3x4 12 Key Matrix
- D1 Mini
