# Taller 2


<p align="center">
  <img src="Foto/Captura de pantalla 2026-04-21 170238.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170322.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170400.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170426.png" width="90%">
  <img src="Foto/Captura de pantalla 2026-04-21 170505.png" width="90%">
</p> 

# 1. Instrumento musical electrónico

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

# 2. Clasificación de Objetos en Tiempo Real usando YOLO

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
Para el entrenamiento del modelo de detección de objetos utilizamos Google Colab, ya que esta plataforma nos permite acceder a recursos de hardware acelerado (GPU) sin necesidad de una configuración local avanzada. Antes de ejecutar el código, fue necesario cambiar el tipo de entorno de ejecución a GPU, lo cual optimiza significativamente el tiempo de entrenamiento del modelo YOLO.
A continuación, se describe el script utilizado, organizado por segmentos, explicando la función de cada uno dentro del proceso de entrenamiento y generación del archivo final .pt.

<img width="1890" height="820" alt="image" src="https://github.com/user-attachments/assets/8787e03a-fa5f-42c8-b3bd-379e63c94ff6" />


```
# ==========================================================
# SCRIPT COMPLETO PARA DETECTOR DE JUGUETES (GOOGLE COLAB)
# ==========================================================
# Este script tiene como objetivo entrenar un modelo YOLO
# para la detección de objetos de juguete y generar el
# archivo final de pesos (.pt) que será usado posteriormente
# para la detección en tiempo real.
# ==========================================================


# ==========================================================
# 1. INSTALACIÓN E IMPORTACIÓN DE LIBRERÍAS
# ==========================================================
# En este primer paso instalamos las librerías necesarias:
# - ultralytics: contiene la implementación del modelo YOLO
# - roboflow: nos permite descargar automáticamente el dataset
# desde la plataforma Roboflow

!pip install ultralytics roboflow

# Importamos librerías necesarias para ejecutar el entrenamiento
import os                          # Manejo de rutas y archivos del sistema
from roboflow import Roboflow      # Conexión y descarga de datasets desde Roboflow
from ultralytics import YOLO       # Clase principal para usar modelos YOLO


# ==========================================================
# 2. DESCARGA DEL DATASET DESDE ROBOFLOW
# ==========================================================
# En este segmento nos conectamos a Roboflow utilizando
# una API Key, accedemos al workspace y descargamos la
# versión del dataset previamente creado y etiquetado.

try:
    # Inicializamos la conexión con Roboflow usando la API Key
    rf = Roboflow(api_key="YTZ1hI9Jzu23WFr9DZkO")

    # Accedemos al workspace donde está almacenado el proyecto
    project = rf.workspace("andress-workspace-s8nj4").project("juguetes_detect")

    # Seleccionamos la versión del dataset
    version = project.version(1)

    # Descargamos el dataset en formato compatible con YOLO
    dataset = version.download("yolov11")

    # Mensaje de confirmación si la descarga fue exitosa
    print("✅ Dataset descargado con éxito.")

except Exception as e:
    # En caso de error, se muestra el mensaje correspondiente
    print(f"❌ Error al descargar de Roboflow: {e}")


# ==========================================================
# 3. CONFIGURACIÓN DEL MODELO YOLO Y ENTRENAMIENTO
# ==========================================================
# En este paso cargamos un modelo base de YOLO
# Utilizamos la versión YOLO Nano (yolo11n.pt),
# ya que es ligera y adecuada para entrenamientos académicos
# y pruebas en tiempo real.

model = YOLO('yolo11n.pt')


# Antes de iniciar el entrenamiento, mostramos un mensaje informativo
print("🚀 Iniciando entrenamiento... esto puede tardar unos minutos.")


# Iniciamos el entrenamiento del modelo
# Parámetros utilizados:
# - data: ruta al archivo data.yaml del dataset descargado
# - epochs: número de veces que el modelo analizará el dataset completo
# - imgsz: tamaño de las imágenes durante el entrenamiento
# - plots: genera gráficas automáticas del proceso de entrenamiento

results = model.train(
    data=os.path.join(dataset.location, "data.yaml"),
    epochs=50,
    imgsz=640,
    plots=True
)


# ==========================================================
# 4. GENERACIÓN Y UBICACIÓN DEL ARCHIVO FINAL .PT
# ==========================================================
# Al finalizar el entrenamiento, YOLO guarda automáticamente
# los pesos del mejor modelo entrenado en un archivo llamado:
# best.pt
# Este archivo contiene el "aprendizaje" del modelo y será
# utilizado posteriormente para la detección de objetos.

print("\n" + "="*50)
print("¡ENTRENAMIENTO FINALIZADO!")
print("El archivo del modelo entrenado se encuentra en:")
print("/content/runs/detect/train/weights/best.pt")
print("="*50)


# ==========================================================
# 5. DESCARGA AUTOMÁTICA DEL ARCHIVO .PT (OPCIONAL)
# ==========================================================
# Este segmento permite descargar automáticamente el archivo
# best.pt desde Google Colab al computador local del usuario,
# facilitando su uso en otros entornos o proyectos.

from google.colab import files

try:
    # Descarga del archivo de pesos entrenado
    files.download('/content/runs/detect/train/weights/best.pt')
    print("📥 Descargando el archivo 'best.pt' a tu computadora...")
except:
    # Mensaje en caso de que la descarga automática falle
    print("No se pudo iniciar la descarga automática, revisa manualmente la carpeta 'runs'.")
```

# Implementación del código para la detección de objetos en tiempo real (YOLO)
En esta etapa del laboratorio implementamos el código necesario para realizar la detección y clasificación de objetos en tiempo real utilizando el modelo YOLO previamente entrenado. Para ello, cargamos el archivo de pesos .pt generado en Google Colab y lo integramos con una cámara web mediante la librería OpenCV.
El objetivo principal de esta implementación es capturar video en tiempo real, analizar cada fotograma con el modelo YOLO y mostrar en pantalla los objetos detectados mediante cajas delimitadoras, junto con el nombre de la clase y el nivel de confianza. De esta manera, se valida el funcionamiento práctico del modelo entrenado y su capacidad para reconocer los diferentes elementos definidos en el laboratorio, incluso cuando estos se muestran desde la pantalla de un celular o en distintas posiciones.

```
# ==========================================================
# IMPLEMENTACIÓN DEL CÓDIGO PARA LA DETECCIÓN DE OBJETOS
# EN TIEMPO REAL UTILIZANDO YOLO
# ==========================================================
# Este script permite utilizar el modelo previamente entrenado
# (archivo .pt) para detectar y clasificar objetos en tiempo real
# usando la cámara del computador.
# ==========================================================


# ==========================================================
# 1. IMPORTACIÓN DE LIBRERÍAS
# ==========================================================
# Importamos OpenCV para el manejo de la cámara y visualización
# Importamos YOLO desde Ultralytics para cargar el modelo entrenado

import cv2
from ultralytics import YOLO


# ==========================================================
# 2. CARGA DEL MODELO ENTRENADO (.PT)
# ==========================================================
# En este paso cargamos el archivo best.pt generado durante
# la etapa de entrenamiento en Google Colab.
# Este archivo contiene los pesos del modelo entrenado para
# reconocer los juguetes definidos en el dataset.

model = YOLO(r"C:\Users\PIPE\Desktop\best.pt")


# ==========================================================
# 3. CONFIGURACIÓN DE LA CÁMARA
# ==========================================================
# Inicializamos la cámara web del computador.
# El valor 0 corresponde a la cámara principal del sistema.

cap = cv2.VideoCapture(0)

# Se fuerza una resolución estándar para mejorar la detección
# y evitar zonas con mala interpretación de la imagen.

cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

# Verificamos que la cámara se haya abierto correctamente
if not cap.isOpened():
    print("Error: No se pudo abrir la cámara.")
    exit()

print("Detección activa. Apunta a tus juguetes o a la pantalla de tu celular.")


# ==========================================================
# 4. BUCLE PRINCIPAL DE DETECCIÓN EN TIEMPO REAL
# ==========================================================
# Este ciclo captura continuamente los fotogramas de la cámara
# y los envía al modelo YOLO para su análisis.

while True:
    ret, frame = cap.read()

    # Si no se logra capturar el fotograma, se finaliza el programa
    if not ret:
        break


    # ======================================================
    # CONFIGURACIÓN DE PARÁMETROS DE DETECCIÓN
    # ======================================================
    # conf = 0.20:
    #   Reduce el umbral de confianza para permitir detecciones
    #   más sensibles, especialmente cuando los objetos se
    #   muestran desde la pantalla de un celular.
    #
    # iou = 0.3:
    #   Permite detectar varios objetos aunque estén cercanos
    #   o ligeramente superpuestos.
    #
    # augment = True:
    #   Aplica aumentos durante la inferencia, analizando la
    #   imagen en diferentes variaciones para mejorar resultados.
    #
    # stream = True:
    #   Permite procesar los fotogramas de forma continua.
    #
    # verbose = False:
    #   Evita mostrar mensajes innecesarios en consola.

    results = model(
        frame,
        stream=True,
        conf=0.20,
        iou=0.3,
        augment=True,
        verbose=False
    )


    # ======================================================
    # 5. VISUALIZACIÓN DE RESULTADOS
    # ======================================================
    # Recorremos los resultados obtenidos y dibujamos las
    # cajas delimitadoras con su respectiva etiqueta y confianza.

    for r in results:
        annotated_frame = r.plot()
        cv2.imshow('Detector Universal - Pipe', annotated_frame)


    # ======================================================
    # 6. CONDICIÓN DE SALIDA
    # ======================================================
    # Al presionar la tecla 'q', se cierra el programa.

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break


# ==========================================================
# 7. LIBERACIÓN DE RECURSOS
# ==========================================================
# Se liberan la cámara y las ventanas para cerrar correctamente
# la aplicación y evitar bloqueos del sistema.

cap.release()
cv2.destroyAllWindows()

```


# Resultados obtenidos

<img width="1864" height="1019" alt="image" src="https://github.com/user-attachments/assets/64ddbd2b-ab8b-4c91-bccb-a5ae89a1672e" />

<img width="1864" height="1019" alt="image" src="https://github.com/user-attachments/assets/197a77d0-f646-4931-b2af-c38c527efe3d" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/76cbca39-fa52-4fbe-891f-6e1fd4ab6cf2" />






<p align="center">
  <img src="Foto/Captura de pantalla 2026-04-21 170531.png" width="90%">
</p> 

# 3. Paisaje por medio de figuras de lissajaus
