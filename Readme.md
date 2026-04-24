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



``
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
´´



<p align="center">
  <img src="Foto/Captura de pantalla 2026-04-21 170531.png" width="90%">
</p> 


