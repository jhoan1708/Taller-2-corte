### Procedimiento para configurar un pic16f887 sobre un arduino 

Creamos un circuito entre el pic y el arduino, se requiere los siguientes materiales y herramientas:

* Arduino uno
* Cable de serial arduino
* jumper machos a macho
* jumper macho a hembra
* protoboard
* relay de arduino
* resistencia de 10 k
* pic16f887
* python
* arduino.ide
* powershell


creamos el siguiente diseño:

* Los vdd del pic a 5 v der arduino
* los vccdel pic a gnd de arduino
* relay in a d10 para controlar
* relay gnd a gnd de arduino
* relay vcc a mclr de pic
* relay coom a 12v positivos de fuente
* gnd o negativo de fuente a gnd de arduino
* pines 39 y 40 de pic a pines d12 y d13 arduino para datos
* Mclr pic con resistencia a 5 v
  
´´´
Nota: todos los positivos voltaje deben de estar interconectados entre si y los mismo los gnd o negativos y mclr debe de ir con la resistencia de 10 k o cercana 
´´´


Una vez tenemos este circuito diseñado lo armamos en fisico conectamos el serial al pc validamos el puerto e iniciaños en arduino.ide y subimos el siguiente firware:


´´´
#define PGC 13
#define PGD 12
#define MCLR 10

String input = "";

void pulseClock() {
  digitalWrite(PGC, HIGH);
  delayMicroseconds(2);
  digitalWrite(PGC, LOW);
  delayMicroseconds(2);
}

void sendBit(bool b) {
  digitalWrite(PGD, b);
  pulseClock();
}

void sendCommand(unsigned int cmd) {
  for (int i = 0; i < 6; i++) {
    sendBit(cmd & 1);
    cmd >>= 1;
  }
}

void sendData(unsigned int data) {
  sendBit(0);
  for (int i = 0; i < 14; i++) {
    sendBit(data & 1);
    data >>= 1;
  }
  sendBit(0);
}

void enterProgramming() {
  digitalWrite(MCLR, HIGH); // activa relé (12V)
  delay(50);
}

void exitProgramming() {
  digitalWrite(MCLR, LOW);
}

void setup() {
  pinMode(PGC, OUTPUT);
  pinMode(PGD, OUTPUT);
  pinMode(MCLR, OUTPUT);

  digitalWrite(PGC, LOW);
  digitalWrite(PGD, LOW);
  digitalWrite(MCLR, LOW);

  Serial.begin(9600);
}

void loop() {
  while (Serial.available()) {
    char c = Serial.read();

    if (c == '\n') {
      procesarLinea(input);
      input = "";
    } else {
      input += c;
    }
  }
}

void procesarLinea(String linea) {

  if (linea == "E") {
    enterProgramming();

    // reset dirección
    for (int i = 0; i < 10; i++) {
      sendCommand(0b000110);
    }

    Serial.println("OK");
    return;
  }

  if (linea == "X") {
    exitProgramming();
    Serial.println("OK");
    return;
  }

  int sep = linea.indexOf(':');

  if (sep > 0) {
    int val = linea.substring(sep + 1).toInt();

    sendCommand(0b000000); // LOAD
    sendData(val);

    sendCommand(0b000010); // PROGRAM
    delay(10);

    sendCommand(0b000110); // NEXT

    Serial.println("OK");
  }
}
´´´


Una vez subimos copilamos el firware al arduino ejecutamos un codigo de prueba para comprobar que el relay funcione una vez hecho esto subimos los siguiente codigos en python y los ejecutamos.
  
Bash´´´
#Codigo ejecutablle en python leer 
def leer_hex(ruta):
    datos = []

    with open(ruta, 'r') as f:
        for linea in f:
            if linea.startswith(':'):
                longitud = int(linea[1:3], 16)
                direccion = int(linea[3:7], 16)
                tipo = int(linea[7:9], 16)

                if tipo == 0:
                    for i in range(longitud):
                        byte = int(linea[9+i*2:11+i*2], 16)
                        datos.append((direccion+i, byte))

    return datos


data = leer_hex("C:/Users/MONTAÑEZ/MPLABXProjects/Piano.X/dist/default/production/Piano.X.production.hex")

print("Bytes leídos:", len(data))
print(data[:10])
´´´

´´´
#Codigo 2 ejecutable python leer
def leer_hex(ruta):
    datos = []

    with open(ruta, 'r') as f:
        for linea in f:
            if linea.startswith(':'):
                longitud = int(linea[1:3], 16)
                direccion = int(linea[3:7], 16)
                tipo = int(linea[7:9], 16)

                if tipo == 0:
                    for i in range(longitud):
                        byte = int(linea[9+i*2:11+i*2], 16)
                        datos.append((direccion+i, byte))

    return datos
´´´


´´´
#Ejecutable python
import serial
import time

def leer_hex(ruta):
    datos = []
    with open(ruta, 'r') as f:
        for linea in f:
            if linea.startswith(':'):
                longitud = int(linea[1:3], 16)
                direccion = int(linea[3:7], 16)
                tipo = int(linea[7:9], 16)

                if tipo == 0:
                    for i in range(longitud):
                        byte = int(linea[9+i*2:11+i*2], 16)
                        datos.append((direccion+i, byte))
    return datos


PUERTO = 'COM8'
HEX = "C:/Users/MONTAÑEZ/MPLABXProjects/Piano.X/dist/default/production/Piano.X.production.hex"

ser = serial.Serial(PUERTO, 9600)
time.sleep(2)

datos = leer_hex(HEX)

palabras = []
for i in range(0, len(datos), 2):
    if i+1 < len(datos):
        low = datos[i][1]
        high = datos[i+1][1]
        palabra = (high << 8) | low
        palabras.append(palabra & 0x3FFF)  # 14 bits

print("Palabras:", len(palabras))

ser.write(b'E\n')
time.sleep(0.02)

print("Programando...")

for palabra in palabras:
    ser.write(f"0:{palabra}\n".encode())
    time.sleep(0.02)

    if ser.in_waiting:
        respuesta = ser.readline().decode().strip()
        print("Arduino:", respuesta)

ser.write(b'X\n')

print("FIN")
´´´


Una vez ejecutados en la powershell de nuestro equipo ejecutamos la siguiente linea 


´´´
#Ejecutable en power shel
PS C:\Users\MONTAÑEZ> python "C:\Users\MONTAÑEZ\MPLABXProjects\Piano.X\dist\default\production\programar.py"
´´´

una vez recibamos la siguiente respuesta ya nuestro pic quedo programado pero se debe de tener en cuenta que puede fallar ya que es casero y experimental.

´´´
PS C:\Users\MONTAÑEZ> python "C:\Users\MONTAÑEZ\MPLABXProjects\Piano.X\dist\default\production\programar.py"
Palabras: 343
Programando...
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
Arduino: OK
FIN
PS C:\Users\MONTAÑEZ>
´´´
