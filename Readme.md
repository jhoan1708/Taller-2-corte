# Taller 2


<p align="center">
  <img src="Foto/Captura de pantalla 2026-04-21 170238.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170322.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170400.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170426.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170505.png" width="90%">
</p> 



# Desarrollo

```bash
#include <xc.h>

// CONFIG
#pragma config FOSC = HS
#pragma config WDTE = OFF
#pragma config PWRTE = ON
#pragma config CP = OFF

#define _XTAL_FREQ 4000000

//================ DELAY =================
void delay_custom(unsigned int t) {
    for(unsigned int i = 0; i < t; i++) {
        for(unsigned int j = 0; j < 10; j++);
    }
}

//================ SONIDO =================
void sonido(unsigned int f) {
    RC2 = 1;
    delay_custom(f);
    RC2 = 0;
    delay_custom(f);
}

//================ NUMEROS =================
// a b c d e f g (ánodo común ? invertido)
unsigned char numeros[] = {
    ~0b0111111, // 0
    ~0b0000110, // 1
    ~0b1011011, // 2
    ~0b1001111, // 3
    ~0b1100110, // 4
    ~0b1101101, // 5
    ~0b1111101, // 6
    ~0b0000111  // 7
};

//================ MOSTRAR =================
void mostrar(unsigned char num) {
    RC3 = 0;              // activar display
    PORTD = ~numeros[num];
}

//================ MAIN =================
void main(void) {

    TRISB = 0xFF;
    TRISC = 0x00;
    TRISD = 0x00;

    PORTC = 0x00;
    PORTD = 0xFF;

    ADCON1 = 0x06;
    OPTION_REGbits.nRBPU = 0;

    while(1) {

        if (RB0 == 0) { // DO ? 1
            RC1 = 1;
            while(RB0 == 0) {
                sonido(191);
                mostrar(1);
            }
        }

        else if (RB1 == 0) { // RE ? 2
            RC1 = 1;
            while(RB1 == 0) {
                sonido(170);
                mostrar(2);
            }
        }

        else if (RB2 == 0) { // MI ? 3
            RC1 = 1;
            while(RB2 == 0) {
                sonido(151);
                mostrar(3);
            }
        }

        else if (RB3 == 0) { // FA ? 4
            RC1 = 1;
            while(RB3 == 0) {
                sonido(143);
                mostrar(4);
            }
        }

        else if (RB4 == 0) { // SOL ? 5
            RC1 = 1;
            while(RB4 == 0) {
                sonido(127);
                mostrar(5);
            }
        }

        else if (RB5 == 0) { // LA ? 6
            RC1 = 1;
            while(RB5 == 0) {
                sonido(113);
                mostrar(6);
            }
        }

        else if (RB6 == 0) { // SI ? 7
            RC1 = 1;
            while(RB6 == 0) {
                sonido(101);
                mostrar(7);
            }
        }

        else {
            RC2 = 0;
            RC1 = 0;
            PORTD = 0xFF;
        }
    }
}
```



* ## Simulacion de circuito en simullde

<p align="center">
<img src="Foto/Captura de pantalla 2026-04-21 182817.png" width="90%">
</p> 


https://github.com/user-attachments/assets/9e8b98cf-56ec-4555-a7a8-f1c1b37355e3




<p align="center">
  <img src="Foto/Captura de pantalla 2026-04-21 170520.png" width="90%">
</p> 

# Introduccion
En este laboratorio presentamos el desarrollo de un sistema de clasificación de objetos en tiempo real utilizando YOLO (You Only Look Once), uno de los modelos más conocidos y eficientes en el área de visión por computador. Nuestro propósito es mostrar, de manera práctica, cómo es posible aplicar técnicas de aprendizaje profundo para detectar y clasificar múltiples elementos dentro de una escena capturada por una cámara.
A lo largo de este trabajo implementamos un modelo capaz de identificar diferentes elementos de juguete, lo que nos permite simular un entorno controlado y enfocado en el aprendizaje. El sistema fue diseñado para realizar detecciones en tiempo real, mostrando en pantalla las cajas delimitadoras (bounding boxes) junto con el nivel de confianza asociado a cada predicción realizada por el modelo.
Los objetos que el sistema es capaz de clasificar son:

* Motocicletas de juguete
* Carros de juguete
* Ciclas de juguete
* Buses de juguete
* Peatones de juguete

Durante la ejecución, el modelo YOLO analiza cada fotograma de video y determina la presencia de uno o varios de estos elementos, lo que nos permite evidenciar el funcionamiento del proceso de detección y clasificación de forma visual e interactiva.
Finalmente, documentamos todo el proceso en un repositorio de GitHub, donde describimos la estructura del proyecto, el funcionamiento del modelo y la lógica empleada para llevar a cabo la clasificación en tiempo real. De esta manera, no solo presentamos los resultados obtenidos, sino también el proceso completo de desarrollo, facilitando la comprensión y replicabilidad del laboratorio.

## Tecnologías utilizadas
- YOLO
- Roboflow
- Python
- OpenCV
- GitHub

# Creación de la base de datos (Roboflow)
Para el desarrollo de este laboratorio, comenzamos con la creación de la base de datos necesaria para el entrenamiento del modelo YOLO. Este proceso se llevó a cabo utilizando la plataforma Roboflow, la cual facilita la gestión, anotación y preparación de conjuntos de datos orientados a proyectos de visión por computador.
Inicialmente, recopilamos un conjunto de imágenes que contienen los objetos definidos para el laboratorio, correspondientes a elementos de juguete. Estas imágenes fueron seleccionadas procurando incluir diferentes ángulos, tamaños y fondos, con el fin de mejorar la capacidad del modelo para generalizar durante el proceso de detección en tiempo real.

Una vez cargadas las imágenes en Roboflow, realizamos el proceso de etiquetado manual, asignando una clase a cada objeto presente en las imágenes.

<img width="1915" height="835" alt="image" src="https://github.com/user-attachments/assets/0bd3217d-d82a-4a9b-9f04-820687633069" />


Cada objeto fue delimitado mediante cajas delimitadoras (bounding boxes), asegurando que estas encerraran correctamente el elemento correspondiente para garantizar una correcta interpretación por parte del modelo durante el entrenamiento.

<img width="1772" height="813" alt="image" src="https://github.com/user-attachments/assets/c223ca69-2f5b-4822-bb45-ee91cd85d026" />


Posteriormente, utilizamos las herramientas de Roboflow para realizar la división del conjunto de datos en tres subconjuntos: entrenamiento, validación y prueba. Esta separación permite evaluar el rendimiento del modelo y verificar su capacidad de detección sobre imágenes no vistas durante el entrenamiento.

<img width="309" height="522" alt="image" src="https://github.com/user-attachments/assets/cc92f680-9387-4ea9-a504-f501983364e9" />

Finalmente, la base de datos fue exportada en el formato compatible con YOLO, lo que permitió su utilización directa en la etapa de entrenamiento del modelo. Todo este proceso queda documentado en este repositorio, junto con las evidencias visuales correspondientes, mostrando así cada una de las etapas realizadas en la creación del dataset.

# Entrenamiento del modelo y generación del archivo .pt
(Descripción del entrenamiento del modelo YOLO y de cómo se obtuvo el archivo de pesos)

# Implementación del código para la detección de objetos en tiempo real (YOLO)
(Explicación del código, cámara, inferencia y visualización de resultados)

# Resultados obtenidos
(Qué logra detectar, ejemplo de ejecución, confianza, etc.)




```
import cv2
from ultralytics import YOLO
 
# 1. CARGA DEL MODELO
model = YOLO(r"C:\Users\PIPE\Desktop\best.pt")
 
# 2. CONFIGURACIÓN DE LA CÁMARA
cap = cv2.VideoCapture(0)
 
# Forzamos una resolución estándar para que no haya "puntos ciegos"
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
 
if not cap.isOpened():
    print("Error: No se pudo abrir la cámara.")
    exit()
 
print("Detección TOTAL activa. Apunta a tus juguetes o a la pantalla de tu celular.")
 
while True:
    ret, frame = cap.read()
    if not ret:
        break
 
    # --- EL SECRETO PARA DETECTAR EN CUALQUIER POSICIÓN Y CELULARES ---
    # conf=0.20: Bajamos mucho la confianza. Las fotos en celulares se ven distintas a los
    # objetos reales, por eso necesitamos que la IA sea más "sensible".
    # iou=0.3: Esto permite que detecte muchos objetos aunque estén pegados o amontonados.
    # augment=True: Este comando hace que la IA analice la imagen varias veces (girada,
    # con más luz, etc.) para encontrar objetos en cualquier posición.
    
    results = model(frame, stream=True, conf=0.20, iou=0.3, augment=True, verbose=False)
 
    for r in results:
        # Dibujamos los cuadros
        annotated_frame = r.plot()
 
        # 4. MOSTRAR EN PANTALLA
        cv2.imshow('Detector Universal - Pipe', annotated_frame)
 
    # Presiona 'q' para cerrar
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
 
cap.release()
cv2.destroyAllWindows()
```



<p align="center">
  <img src="Foto/Captura de pantalla 2026-04-21 170531.png" width="90%">
</p> 


