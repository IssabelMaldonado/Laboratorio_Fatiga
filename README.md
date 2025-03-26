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

  


