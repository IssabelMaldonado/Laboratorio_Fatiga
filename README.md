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

## ADQUSICIÓN DE LA SEÑAL EMG:
Como primera medida para adquirir la señal EMG, se utilizaron 3 electrodos en posiciones especificas, las cuales se mostraran en la siguiente imagen:

![image](https://github.com/user-attachments/assets/248ef36c-4496-4c9b-af65-1869b8352d5a)

Dos electrodos (marcados en rojo) se colocaron sobre el músculo para captar su actividad eléctrica, mientras que otro (en verde) se ubicó en una zona más estable, funcionando como referencia o tierra para minimizar interferencias. Esta configuración es habitual en estudios de electromiografía de superficie. Durante el procedimiento, la persona realizó contracciones musculares durante aproximadamente 2 minutos para inducir la fatiga. Sin embargo, al no ser muy frecuentes, los resultados reflejan un nivel bajo de fatiga.

### TOMA DE LA SEÑAL MEDIANTE LA DAQ:


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
