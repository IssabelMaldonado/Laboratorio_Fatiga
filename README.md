# Laboratorio 4 || Fatiga muscular

- Isabel Sofía Maldonado Roa
- Daniel Guillermo Espinosa Parra.

## INTRODUCCIÓN:
El análisis de señales EMG permite evaluar la fatiga muscular a través de cambios en la amplitud y frecuencia. En este laboratorio, se utilizará Python para procesar datos obtenidos con un DAQ (Data Acquisition System), un sistema de adquisición de datos que permite capturar y digitalizar señales del mundo real.   además de calcular métricas como la Mediana de Frecuencia (MF) y la Frecuencia Media (MNF). Estos indicadores ayudarán a identificar el agotamiento durante una contracción sostenida.

## REQUERIMIENTOS: 

- Interfaz de python (para este caso 3.12)
- numpy 
- matplotlib.pyplot
- scipy.signal
- scipy.fftpack
- nidaqmx
- time

## ADQUSICIÓN DE LA SEÑAL EMG:
Como primera medida para adquirir la señal EMG, se utilizaron 3 electrodos en posiciones especificas, las cuales se mostraran en la siguiente imagen:

![image](https://github.com/user-attachments/assets/248ef36c-4496-4c9b-af65-1869b8352d5a)

Dos electrodos (marcados en rojo) se colocaron sobre el músculo para captar su actividad eléctrica, mientras que otro (en verde) se ubicó en una zona más estable, funcionando como referencia o tierra para minimizar interferencias. Esta configuración es habitual en estudios de electromiografía de superficie. Durante el procedimiento, la persona realizó contracciones musculares durante aproximadamente 2 minutos para inducir la fatiga. Sin embargo, al no ser muy frecuentes, los resultados reflejan un nivel bajo de fatiga.

### TOMA DE LA SEÑAL MEDIANTE DAQ:
La DAQ (Data Acquisition) es un sistema que captura y digitaliza señales del entorno para su análisis. Para esta práctica la señal electromiográfica enviada por el sensor es recibida por el dispositivo mediante un ADC luego se envía mediante una conexión al computador.Emplando la documentación referenciada en la guía propuesta, se reailzó un programa en python para grabar los datos de la señal y almacenarlos en un archivo .npy.

El código implementado para captar la señal, se muestra a continuación:

```bash
import nidaqmx  ## Permite la comunicación con el hardware DAQ de National Instruments.
import numpy as np ## manejar los datos adquiridos como arreglos numéricos 
import time ## Se emplea para medir tiempos y mostrar progreso en la adquisición.
import matplotlib.pyplot as plt ## libreria para gráficar los datos adquiridos

# Configuración
DAQ = "Dev1"      # Nombre del dispositivo DAQ
CANAL = "ai1"          # Canal de entrada analógica
FM = 1000     # Frecuencia de muestreo en Hz
TIME = 120           # Duración de la adquisición en segundos (2 minutos)
ARCHIVO = "datos_señal.npy"  # Archivo para guardar en formato NumPy

def adquirir_datos():
    numMUE= FM * TIME # Número total de muestras a adquirir
    data = np.zeros(numMUE)  #  crea un arreglo de ceros con el tamaño total de muestras para almacenar los datos
    
    with nidaqmx.Task() as task: ## Se crea una tarea para la adquisición de datos.
        task.ai_channels.add_ai_voltage_chan(f"{DAQ}/{CANAL}") ## Se añade un canal analógico para medir voltaje.
        task.timing.cfg_samp_clk_timing(FM, samps_per_chan=num_samples) ## Se configura el reloj de muestreo con la frecuencia deseada.

        print(f"Adquiriendo datos durante {TIME// 60} minutos...") ## Muestra un mensaje indicando que la adquisición ha comenzado.
        start_time = time.time() ##  almacena el tiempo de inicio para calcular el tiempo transcurrido.

        # Adquirir datos en segmentos de 1 segundo
        for i in range(TIME):
            chunk = task.read(number_of_samples_per_channel=FM)  ##Se lee la señal en bloques de 1000 muestras por segundo.
            data[i * FM: (i + 1) * FM] = chunk  # Se almacenan los datos en la posición correspondiente del arreglo data.


            elapsed = time.time() - start_time
            print(f"Progreso: {i+1}/{TIME} segundos ({elapsed:.1f} s transcurridos)", end="\r") ## Se calcula y muestra el tiempo transcurrido para monitorear el progreso.


    print("\nAdquisición completada.")
    return data ## Al finalizar la adquisición, se imprime un mensaje y se devuelven los datos adquiridos.



def verificar_datos(data): ## Comprueba si todos los valores en data son ceros, lo que indicaría una falla en la adquisición.
    """ Verifica que los datos no sean solo ruido o ceros """
    if np.all(data == 0):
        print("No se detecta señal, verifica la conexión.")
        return False
    if np.std(data) < 0.001: ## detectar desviación estandar, Si la desviación estándar es muy baja, la señal es demasiado estable, lo que podría indicar una falla en la medición.
        print(" Se detectaron datos que pueden no ser correctos")
        return False ## si los datos no presentan anomalias
    print("Señal detectada correctamente.")
    return True

def guardar_datos_npy(data, filename): ## Guarda los datos en un archivo .npy, para su posterior manipulación 
    np.save(filename, data)
    print(f"Datos guardados en {filename}")

def graficar_datos(data):
    """ Grafica la señal adquirida con una selección representativa si es muy grande """
    tiempo = np.arange(0, len(data)) / FM  # Crear vector de tiempo, cada muestra representa un instante de tiempo

    plt.figure(figsize=(12, 5))
    
    if len(data) > 50000:  # Si hay más de 50,000 puntos, graficar una muestra representativa (reducida para una mejor visualización)
        step = len(data) // 50000
        plt.plot(tiempo[::step], data[::step], label="Señal adquirida (muestra reducida)", color='b', linewidth=0.5)
    else:
        plt.plot(tiempo, data, label="Señal adquirida", color='b', linewidth=0.5)

    plt.xlabel("Tiempo (s)")
    plt.ylabel("Voltaje (V)")
    plt.title("Señal Adquirida desde DAQ ( 2 minutos)")
    plt.legend()
    plt.grid(True)
    
    plt.show()

# Adquirir, verificar, guardar y graficar datos, se llama a loas funciones definidas para que se ejecuten
datos = adquirir_datos()
if verificar_datos(datos):
    guardar_datos_npy(datos, FILENAME_NPY)
    graficar_datos(datos)

```
Luego de ejecutar el código, se obtuvo la señal guardada con la extención deseada, Además los datos se gráficaron y el resultado se muestra a continuación:

![image](https://github.com/user-attachments/assets/a3a6c445-7e20-4194-8e5b-649227fa6aaa)







## Filtrado de la señal: 
En este caso para las especificaciones dadaas por el laboratorio se necesitó un filtro BUtterworth pasa banda de orden 4 para poder tener una mayor efeiciencia y a la vez una mayor atenuación del ruido ya que es de orden 4, en el código de python se implementó de la siguiente manera: 

```bash

def filtrar_senal(senal, fs, lowcut=20, highcut=450): # Se crea la funcuón para filtrar la señal poniendo la frecuencias de corte tanto bajas ( 20 Hz ) Y altas ( 450 Hz)  
    nyq = 0.5 * fs # Se calcula la frecuencia de Nyquist con la frecuencia de muestreo.
    b, a = signal.butter(4, [lowcut / nyq, highcut / nyq], btype='band') # Al usal signal.butter al filtro de le da la caracteristica de que es Butterworth 
    return signal.filtfilt(b, a, senal) # Se aplica signal.filtfilt() para filtrar la señal sin generar desfases


    
```

## Analísis de la señal a partir de la transformada de Fourier. 

En este caso para el código en python con respecto al analisis, se utilizó un metodo de ventanas par poder dividir o seccionar la señal para analizarla de manera más completa y facil, en python se realizó de la siguiente manera
  
```bash
def analizar_ventanas(senal, fs, ventana_size=1024, overlap=512): # Divide la señal en segmentos  con un intervalo llamado overlap  para hacer análisis espectral de la señal.
    step = ventana_size - overlap 
    num_ventanas = (len(senal) - overlap) // step 
    frecuencias = np.fft.fftfreq(ventana_size, d=1/fs) # Se calculan las transformadas de fourier correspondientes 
    espectros = []
    medianas = []


```
Luego se calculo la transdormada individualmente para apliclarla ventana por ventana. 

```bash
 for i in range(num_ventanas):
        start = i * step
        end = start + ventana_size
        if end > len(senal):
            break
        ventana = senal[start:end] * np.hamming(ventana_size) # Se aplica la ventana de hamming para evitar discontinuidades
        fft_senal = np.abs(fft(ventana))[:ventana_size//2] # Se calcula la transformada y se almacena solo la mitad del espectro
        espectros.append(fft_senal)
```

## Calculos de fercuencia: 

```bash
    potencias = fft_senal ** 2
        total_potencia = np.sum(potencias)
        suma_parcial = np.cumsum(potencias)
        mediana_freq = frecuencias[:ventana_size//2][np.where(suma_parcial >= total_potencia / 2)[0][0]]
        medianas.append(mediana_freq)
```
