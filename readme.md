
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

´´´
#Ejecutable en power shel
PS C:\Users\MONTAÑEZ> python "C:\Users\MONTAÑEZ\MPLABXProjects\Piano.X\dist\default\production\programar.py"
´´´
