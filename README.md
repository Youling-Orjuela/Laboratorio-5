# Laboratorio 5: Variabilidad de la Frecuencia Cardiaca usando la Transformada Wavelet
## INVESTIGACION PREVIA
### 1. Actividad simpática y parasimpática del sistema nervioso autónomo:
El sistema nervioso autónomo controla la presión arterial, la frecuencia cardíaca, la temperatura corporal, el peso, la digestión, el metabolismo, el balance hidroelectrolítico, la sudoración, la micción, la defecación, la respuesta sexual y otros procesos. Muchos órganos están regulados especialmente por el sistema simpático o parasimpático, aunque pueden recibir aferencias de ambos; en ocasiones, las funciones son recíprocas (p. ej., la activación simpática acelera la frecuencia cardíaca; la parasimpática la disminuye).

El sistema nervioso parasimpático es catabólico; activa las respuestas de lucha o huida.

El sistema nervioso parasimpático es anabólico; conserva y restablece.


### 2. Efecto de la actividad simpática y parasimpática en la frecuencia cardiaca
1. **Inervación parasimpática:**

Las porciones parasimpáticas del plexo cardíaco solo reciben contribuciones del nervio vago. Las fibras preganglionares, que se ramifican desde el nervio vago derecho e izquierdo, llegan al corazón para luego entrar en el plexo cardíaco haciendo sinapsis con los ganglios de este plexo y las paredes de los atrios.

- La inervación parasimpática es responsable de, reducir la frecuencia cardíaca, reducir la fuerza de contracción del corazón, vasoconstricción (estrechamiento) de las arterias coronarias.

2. **Inervación simpática:**
 
La porción simpática del plexo cardíaco está compuesta por fibras del tronco simpático, las cuales tienen su origen a partir de los segmentos superiores de la médula espinal torácica. Las fibras del tronco simpático llegan al plexo cardíaco mediante los nervios cardíacos. Las fibras preganglionares se ramifican desde la médula espinal torácica superior y hacen sinapsis en los ganglios cervicales inferiores y torácicos superiores. Las fibras postganglionares se extienden desde los ganglios hasta el plexo cardíaco.

- Los nervios simpáticos son responsables de, aumentar la frecuencia cardíaca, aumentar la fuerza de contracción del miocardio, la respuesta de ‘lucha o huida’, que aumenta la frecuencia cardíaca.
  

### 3.	Variabilidad de la frecuencia cardiaca (HRV) medida como fluctuaciones en el intervalo R-R, y las frecuencias de interés en este análisis

El punto fiducial reconocido en el trazado del ECG que identifica un complejo QRS puede basarse en el máximo o baricentro del complejo, en la determinación del máximo de una curva de interpolación, o encontrarse mediante la comparación con una plantilla u otros marcadores de eventos. Para localizar el punto fiducial, las normas voluntarias para equipos de ECG de diagnóstico son satisfactorias en términos de relación señal-ruido, rechazo de modo común, ancho de banda, etc.  Un límite de frecuencia de corte de banda superior sustancialmente inferior al establecido para equipos de diagnóstico (≈200 Hz) puede generar fluctuación en el reconocimiento del punto fiducial del complejo QRS, lo que introduce un error en los intervalos RR medidos. De igual forma, una frecuencia de muestreo limitada induce un error en el espectro de la VFC que aumenta con la frecuencia, afectando así a los componentes de mayor frecuencia.  Una interpolación de la señal de ECG submuestreada puede reducir este error. Con una interpolación adecuada, incluso una frecuencia de muestreo de 100 Hz puede ser suficiente.


### 4.	Transformada Wavelet: definición, usos y tipos de wavelet utilizadas en señales biológicas.

La Transformada Wavelet es una herramienta matemática que permite descomponer una señal en componentes de diferente escala o resolución. A diferencia de la Transformada de Fourier, que analiza la señal en el dominio de la frecuencia de manera global, la Wavelet permite un análisis multiresolución: localiza tanto en el tiempo como en la frecuencia. Esto la hace especialmente útil para analizar señales no estacionarias, como las biológicas.

**Usos en señales biológicas:**

En el contexto del análisis de señales biológicas (como EEG, ECG, EMG, PPG, entre otras), la Transformada Wavelet se emplea para:

-	Eliminar ruido (filtrado de artefactos).
-	Detectar eventos transitorios (como picos o complejos QRS en ECG).
-	Extraer características relevantes para clasificación automática (por ejemplo, para diagnóstico).
-	Compresión de datos sin perder información significativa.
-	Análisis de variabilidad y dinámica de señales fisiológicas.
  
La Wavelet Morlet es una de las más utilizadas en neurociencia y análisis de señales cerebrales (EEG, MEG) debido a su capacidad de representar frecuencias específicas con gran resolución temporal y frecuencia. Esta es ideal para detectar oscilaciones neuronales en bandas específicas (alfa, beta, gamma, etc.), tiene buena resolución en frecuencia, lo cual permite analizar ritmos cerebrales con precisión y es adecuada para transformada wavelet continua

[![Inicio.png](https://i.postimg.cc/3JCXcN5d/Inicio.png)](https://postimg.cc/2VyLqz5D)

## **Captar la señal**
```matlab
clc;
clearvars;
close all;
clear all

%% Cerrar puertos seriales abiertos previamente
puertosAbiertos = serialportfind;
if ~isempty(puertosAbiertos)
    for idx = 1:length(puertosAbiertos)
        delete(puertosAbiertos(idx));
    end
end

%% Configuración del puerto serial
puerto   = "COM5" + ...
    "";   % Ajusta según corresponda
baudRate = 115200;
s = serialport(puerto, baudRate);
configureTerminator(s, "LF");

%% Parámetros de adquisición y visualización
voltaje_ref  = 5;        % Voltaje de referencia del ADC (V)
adc_max      = 4095;      % Resolución 8 bits
fs           = 1000;     % Frecuencia de muestreo estimada (Hz)
dt           = 1/fs;     % Intervalo de muestreo (s)
num_muestras = 50;     % Puntos en la ventana de pantalla
offset       = 0;      % Offset (V)

tiempoVentana = (0:num_muestras-1) * dt;
buffer        = zeros(1, num_muestras);

dataLog = [];
timeLog = [];

%% Crear interfaz con botón Stop
gcfHandle = figure('Name','EMG en Tiempo Real','NumberTitle','off');
set(gcfHandle, 'UserData', struct('stopFlag', false)); % Inicializar UserData
set(gcfHandle, 'CloseRequestFcn', @(src,~) figureClose(src));

uicontrol('Style','pushbutton','String','Stop','Position',[10 10 50 20],...
    'Callback',@(src,~) stopAcquisition(src));

hLine = plot(tiempoVentana, buffer, 'b', 'LineWidth', 1.5);
ylim([0, voltaje_ref + offset]);
xlim([tiempoVentana(1), tiempoVentana(end)]);
xlabel('Tiempo (s)');
ylabel('Voltaje (V)');
title('Señal EMG en Tiempo Real');
grid on;

disp('Iniciando adquisición. Pulse Stop para finalizar y guardar.');

%% Bucle principal de adquisición mientras no se presione Stop
userData = get(gcfHandle, 'UserData');
%I=0;
while ~userData.stopFlag && ishandle(gcfHandle)
    if s.NumBytesAvailable > 0
        nuevosBytes = read(s, s.NumBytesAvailable, 'uint8')
        nuevosVolt  = double(nuevosBytes)/adc_max * voltaje_ref + offset;
        %datos[i]=nuevosVolt

        for v = nuevosVolt(:).'
            buffer = [buffer(2:end), v];
            dataLog(end+1) = v;
            timeLog(end+1) = (length(dataLog)-1) * dt;
        end
        set(hLine, 'YData', buffer);
        drawnow limitrate;
    else
        pause(0.001);
    end
    userData = get(gcfHandle, 'UserData'); % Actualizar UserData
    %I=I+1;
end
save("misenal.m",'datos')
%% Guardar datos cuando se detiene
if ~isempty(dataLog)
    filename = fullfile(pwd, 'Laboratorio_Corazon2.csv');
    T = table(timeLog.', dataLog.', 'VariableNames', {'Tiempo_s','Voltaje_V'});
    writetable(T, filename);
    disp(['Datos guardados en: ', filename]);
end

%% Cerrar puerto serial y figura
delete(s);
if ishandle(gcfHandle)
    delete(gcfHandle);
end

%% Callbacks
function stopAcquisition(src)
    userData = get(src.Parent, 'UserData');
    userData.stopFlag = true;
    set(src.Parent, 'UserData', userData);
end

function figureClose(src)
    userData = get(src, 'UserData');
    userData.stopFlag = true;
    set(src, 'UserData', userData);
    delete(src);
end
```
### **Procesamiento de la señal**

**Librerias**
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy.signal import butter, lfilter, find_peaks
import pywt
from scipy.interpolate import interp1d
```
numpy: para cálculos numéricos con arreglos, como diferencias entre tiempos.
pandas: para leer archivos .csv y trabajar con columnas de datos.
matplotlib.pyplot: para graficar señales y resultados.
scipy.signal: herramientas para diseñar y aplicar filtros, y detectar picos.
pywt: librería de transformadas wavelet.
interp1d: para interpolar valores cuando necesitamos una señal continua a intervalos regulares.

**Carga de archivos CSV**
```python
archivo1 = "Laboratorio_Corazon1.csv"
archivo2 = "Laboratorio_Corazon2.csv"
datos1 = pd.read_csv(archivo1)
datos2 = pd.read_csv(archivo2)
```
Define los nombres de los archivos.
Lee cada archivo y guarda los datos en datos1 y datos2.

**Extracción de columnas de tiempo y voltaje**
```python
tiempo1 = datos1["Tiempo_s"]
voltaje1 = datos1["Voltaje_V"]
tiempo2 = datos2["Tiempo_s"]
voltaje2 = datos2["Voltaje_V"]
```
Extrae la columna de tiempo ("Tiempo_s") y voltaje ("Voltaje_V") de cada archivo.
Guarda los datos como vectores separados.

**Función para diseñar un filtro IIR Butterworth y función para aplicar el filtro**
```python
def disenar_filtro_iir(fs, f_low, f_high, orden=2):
    nyq = fs / 2
    low = f_low / nyq
    high = f_high / nyq
    b, a = butter(orden, [low, high], btype='band')
    return b, a
def aplicar_filtro_iir(senal, b, a):
    return lfilter(b, a, senal)
```
Calcula la frecuencia de Nyquist (la mitad de la frecuencia de muestreo).
Normaliza las frecuencias baja y alta con respecto a Nyquist.
Usa butter() para crear un filtro pasa banda de orden 2.
Devuelve los coeficientes del filtro (b, a).
Aplica el filtro IIR a una señal usando los coeficientes b y a.

**Filtro de las señales**
```python
fs = 250
f_low = 0.5
f_high = 40
b, a = disenar_filtro_iir(fs, f_low, f_high)
voltaje1_filtrado = aplicar_filtro_iir(voltaje1, b, a)
voltaje2_filtrado = aplicar_filtro_iir(voltaje2, b, a)
```
Define la frecuencia de muestreo y el rango del filtro.
Filtra ambas señales ECG para eliminar ruido de baja y alta frecuencia.

**Visualización de señal original vs filtrada**
```python
# ======= Graficar señal original y filtrada - ECG 1 =======
plt.figure(figsize=(10, 4))
plt.plot(tiempo1, voltaje1, label="Señal original ECG 1", color='purple')
plt.title("ECG 1 - Señal original")
plt.xlabel("Tiempo (s)")
plt.ylabel("Voltaje (V)")
plt.grid(True)
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 4))
plt.plot(tiempo1, voltaje1_filtrado, label="Señal filtrada ECG 1", color='lightblue')
plt.title("ECG 1 - Señal filtrada (IIR orden 2)")
plt.xlabel("Tiempo (s)")
plt.ylabel("Voltaje (V)")
plt.grid(True)
plt.tight_layout()
plt.show()

# ======= Graficar señal original y filtrada - ECG 2 =======
plt.figure(figsize=(10, 4))
plt.plot(tiempo2, voltaje2, label="Señal original ECG 2", color='lightgreen')
plt.title("ECG 2 - Señal original")
plt.xlabel("Tiempo (s)")
plt.ylabel("Voltaje (V)")
plt.grid(True)
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 4))
plt.plot(tiempo2, voltaje2_filtrado, label="Señal filtrada ECG 2", color='lightblue')
plt.title("ECG 2 - Señal filtrada (IIR orden 2)")
plt.xlabel("Tiempo (s)")
plt.ylabel("Voltaje (V)")
plt.grid(True)
plt.tight_layout()
plt.show()
```

Muestra gráficamente la señal original y la filtrada para comparar visualmente.
[![ECG1.jpg](https://i.postimg.cc/26KvnLZK/ECG1.jpg)](https://postimg.cc/ftYJNL5c)
[![IIR.jpg](https://i.postimg.cc/mgsdPKRF/IIR.jpg)](https://postimg.cc/KR07Vq1Z)
[![ECG2.jpg](https://i.postimg.cc/wjQwLFT4/ECG2.jpg)](https://postimg.cc/CRdCVG8b)
[![IIR2.jpg](https://i.postimg.cc/25j2wv1k/IIR2.jpg)](https://postimg.cc/NKV6GK8S)

**Segmento de 1 segundo**
```python
duracion = 1
segmento1 = tiempo1 <= duracion
segmento2 = tiempo2 <= duracion
voltaje1_vis = voltaje1_filtrado[segmento1]
voltaje2_vis = voltaje2_filtrado[segmento2]
tiempo1_vis = tiempo1[segmento1]
tiempo2_vis = tiempo2[segmento2]
```
Se selecciona solo el primer segundo de señal ECG.
Este fragmento corto permite visualizar mejor los picos R.

** Detección de picos R en segmento visible y detección de picos R en toda la señal**
```python
min_dist = int(0.3 * fs)
picos1_vis, _ = find_peaks(voltaje1_vis, height=0.5, distance=min_dist)
picos2_vis, _ = find_peaks(voltaje2_vis, height=0.5, distance=min_dist)
picos1, _ = find_peaks(voltaje1_filtrado, height=0.5, distance=min_dist)
picos2, _ = find_peaks(voltaje2_filtrado, height=0.5, distance=min_dist)
```
find_peaks localiza los máximos locales que representan los picos R.
Se filtran por altura mínima (0.5) y distancia mínima entre picos (0.3 segundos).

**Gráficas 1s de las señales filtradas ECG1 y ECG2**
```python
plt.figure(figsize=(10, 4))
plt.plot(tiempo1_vis, voltaje1_vis, label="ECG 1 filtrada (1s)", color='lightblue')
plt.plot(tiempo1.values[picos1_vis], voltaje1_filtrado[picos1_vis], 'ro', label='Picos R')
plt.title("ECG 1 - Primer segundo con picos R detectados")
plt.xlabel("Tiempo (s)")
plt.ylabel("Voltaje (V)")
plt.grid(True)
plt.legend()
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 4))
plt.plot(tiempo2_vis, voltaje2_vis, label="ECG 2 filtrada (1s)", color='lightblue')
plt.plot(tiempo2.values[picos2_vis], voltaje2_filtrado[picos2_vis], 'ro', label='Picos R')
plt.title("ECG 2 - Primer segundo con picos R detectados")
plt.xlabel("Tiempo (s)")
plt.ylabel("Voltaje (V)")
plt.grid(True)
plt.legend()
plt.tight_layout()
plt.show()
```
Se muestran las gráficas de el primer segundo de ambas señales para una mejoria visual de lo que se esta evaluando.
[![r.jpg](https://i.postimg.cc/SxnvFcyg/r.jpg)](https://postimg.cc/yDzP0g3R)
[![r2.jpg](https://i.postimg.cc/zBrtgMTQ/r2.jpg)](https://postimg.cc/KK0DX0q7)

**Cálculo de intervalos R-R**
```python
rr_intervals1 = np.diff(tiempo1.values[picos1])
rr_intervals2 = np.diff(tiempo2.values[picos2])
```
Se calcula la diferencia de tiempo entre cada par de picos R consecutivos.
Los valores obtenidos son los intervalos R-R, base para el análisis de HRV.

***Análisis de HRV en el dominio del tiempo***
```python
def analizar_hrv(rr_intervals):
    mean_rr = np.mean(rr_intervals)
    std_rr = np.std(rr_intervals)
    print(f"Media RR: {mean_rr:.4f} s")
    print(f"Desviación estándar RR: {std_rr:.4f} s")
    return mean_rr, std_rr

mean1, std1 = analizar_hrv(rr_intervals1)
mean2, std2 = analizar_hrv(rr_intervals2)
```
mean_rr: promedio de los intervalos R-R (refleja el ritmo cardíaco).
std_rr: desviación estándar, una medida simple de la variabilidad del ritmo cardíaco.

**Graficar intervalos R-R**
```python
# ======= Graficar intervalos R-R =======
plt.figure(figsize=(10, 4))
plt.plot(rr_intervals1, marker='o', linestyle='-', color='red')
plt.title("Intervalos R-R - ECG 1")
plt.xlabel("Número de intervalo")
plt.ylabel("Duración (s)")
plt.grid(True)
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 4))
plt.plot(rr_intervals2, marker='o', linestyle='-', color='darkorange')
plt.title("Intervalos R-R - ECG 2")
plt.xlabel("Número de intervalo")
plt.ylabel("Duración (s)")
plt.grid(True)
plt.tight_layout()
plt.show()
```
[![r3.jpg](https://i.postimg.cc/Jhq6jxK4/r3.jpg)](https://postimg.cc/Z9WxSr11)
[![r4.jpg](https://i.postimg.cc/Tw3k0bHj/r4.jpg)](https://postimg.cc/7CjMY54b)
***Análisis de wavelate**
***Definición de la función, interpolación de la señal RR, ransformada Wavelet Continua**
```python
def cwt_hrv(rr_intervals, tiempo_picos, fs_rr=4, wavelet='cmor1.5-1.0', comparar=False):
    tiempo_interp = np.linspace(tiempo_picos[0], tiempo_picos[-1], int((tiempo_picos[-1] - tiempo_picos[0]) * fs_rr))
    tiempo_rr = (tiempo_picos[1:] + tiempo_picos[:-1]) / 2
    interp_rr = interp1d(tiempo_rr, rr_intervals, kind='cubic', fill_value="extrapolate")(tiempo_interp)

    scales = np.arange(1, 256)
    coef, freqs = pywt.cwt(interp_rr, scales, wavelet, 1/fs_rr)
    power = np.abs(coef) ** 2
```
rr_intervals: array con los intervalos RR (diferencia de tiempo entre picos R sucesivos del ECG).
tiempo_picos: array con los tiempos en los que ocurrieron los picos R.
fs_rr: frecuencia de muestreo deseada para interpolar la serie RR.
wavelet: tipo de wavelet usada para el análisis.
comparar: si es True y se usa cmor1.5-1.0, la función también se ejecuta con la wavelet mexh para comparación.
tiempo_interp: crea una serie de tiempo uniforme desde el primer al último pico R con una resolución dada por fs_rr.
tiempo_rr: estima el tiempo medio entre cada par de picos, ubicando los RR entre los picos.
interp_rr: interpola la señal RR en tiempo_interp usando interpolación cúbica. Esto es necesario porque la CWT requiere una señal muestreada de forma uniforme.
scales: conjunto de escalas usadas en la CWT. Escalas más grandes corresponden a frecuencias más bajas.
coef: coeficientes complejos obtenidos al aplicar la CWT a la señal interpolada.
freqs: mapea cada escala a su frecuencia equivalente (según la wavelet y fs_rr).
power: calcula la potencia en cada punto tiempo-frecuencia como el cuadrado del módulo del coeficiente.

**Cálculo de Potencia LF, HF y Relación LF/HF**
```python
lf_band = (freqs >= 0.04) & (freqs <= 0.15)
hf_band = (freqs > 0.15) & (freqs <= 0.4)
lf_power = np.sum(power[lf_band, :])
hf_power = np.sum(power[hf_band, :])
ratio = lf_power / hf_power if hf_power != 0 else np.nan
```
Define las bandas de frecuencia estándar:
LF: 0.04–0.15 Hz → actividad simpática y parasimpática.
HF: 0.15–0.4 Hz → actividad parasimpática.
lf_power y hf_power: potencia total integrada en esas bandas.
ratio: relación LF/HF. Se interpreta como un índice del balance autonómico (más simpático si sube).

**Visualización con Espectrograma Wavelet e  impresión de resultados**
```python
plt.figure(figsize=(12, 6), dpi=150)
plt.imshow(power, extent=[tiempo_interp[0], tiempo_interp[-1], freqs[-1], freqs[0]],
           cmap='jet', aspect='auto', interpolation='bilinear')
plt.colorbar(label='Potencia')
plt.title(f"Espectrograma Wavelet de HRV ({wavelet})")
plt.xlabel("Tiempo (s)")
plt.ylabel("Frecuencia (Hz)")
plt.axhline(0.04, color='white', linestyle='--', label='LF límite inferior')
plt.axhline(0.15, color='white', linestyle='--', label='LF/HF límite')
plt.axhline(0.4, color='white', linestyle='--', label='HF límite superior')
plt.legend(loc='upper right')
plt.tight_layout()
plt.show()

print(f"Potencia LF ({wavelet}): {lf_power:.2f}")
print(f"Potencia HF ({wavelet}): {hf_power:.2f}")
print(f"Relación LF/HF ({wavelet}): {ratio:.2f}\n")
```
Se genera un espectrograma tiempo-frecuencia de la señal interpolada.
[![w1.jpg](https://i.postimg.cc/QC60vdvL/w1.jpg)](https://postimg.cc/4H969sJw)
[![w2.jpg](https://i.postimg.cc/wTvWLDs9/w2.jpg)](https://postimg.cc/vDJLdgzK)
[![w3.jpg](https://i.postimg.cc/7YWKVbg0/w3.jpg)](https://postimg.cc/6yCV6Wbp)
[![w4.jpg](https://i.postimg.cc/VNj4mR2W/w4.jpg)](https://postimg.cc/06y7Cpkz)
Ejes:
X: tiempo.
Y: frecuencia en Hz.
Colores: intensidad (potencia) de cada frecuencia en cada instante.
Se marcan con líneas blancas los límites de las bandas LF y HF para facilitar la interpretación visual.
Imprime en consola los valores numéricos clave del análisis: potencias en las bandas y su relación.

**Comparación con otra wavelet**
```python
if comparar and wavelet == 'cmor1.5-1.0':
    print(" Comparando con wavelet 'mexh'...\n")
    cwt_hrv(rr_intervals, tiempo_picos, fs_rr=fs_rr, wavelet='mexh', comparar=False)
```
Si la bandera comparar es True y se está usando la wavelet cmor1.5-1.0, se vuelve a llamar a la misma función con la wavelet mexh (Mexican Hat) para comparar el comportamiento.

**Llamadas a la función para dos señales y comparación tiempo vs tiempo-frecuencia**

```python
tiempo_picos1 = tiempo1.values[picos1]
tiempo_picos2 = tiempo2.values[picos2]

print(" Análisis señal ECG 1")
cwt_hrv(rr_intervals1, tiempo_picos1, comparar=True)

print(" Análisis señal ECG 2")
cwt_hrv(rr_intervals2, tiempo_picos2, comparar=True)

print("\n Comparación entre análisis en el dominio del tiempo y tiempo-frecuencia:")
print(" Una mayor desviación estándar de los intervalos RR puede reflejar mayor variabilidad del ritmo cardíaco.")
print(" En el espectrograma wavelet, esto puede traducirse en mayor potencia en la banda LF.")
print(" Por otro lado, una frecuencia cardíaca más estable se asocia con menor potencia en LF y mayor en HF.")
print(f" ECG 1: std RR = {std1:.4f} s")
print(f" ECG 2: std RR = {std2:.4f} s")
print("Interpreta estos valores junto con la relación LF/HF para evaluar el balance simpático/parasimático.\n")
```
Se extraen los tiempos correspondientes a los picos R detectados en dos señales ECG distintas.
Se llama a cwt_hrv para ambas señales y se permite la comparación de wavelets.
Compara los resultados del dominio del tiempo (desviación estándar de RR) con los del dominio tiempo-frecuencia (potencia LF/HF).
Relaciona mayor variabilidad (más dispersión de RR) con mayor potencia en LF y, por tanto, mayor actividad simpática.

**Resultados**
Media RR: 0.0426 s
Desviación estándar RR: 0.0035 s
Media RR: 0.0285 s
Desviación estándar RR: 0.0039 s
 Análisis señal ECG 1
Potencia LF (cmor1.5-1.0): 5.05
Potencia HF (cmor1.5-1.0): 0.11
Relación LF/HF (cmor1.5-1.0): 47.32

 Comparando con wavelet 'mexh'...

Potencia LF (mexh): 11.29
Potencia HF (mexh): 0.11
Relación LF/HF (mexh): 103.33

 Análisis señal ECG 2
Potencia LF (cmor1.5-1.0): 2.33
Potencia HF (cmor1.5-1.0): 0.04
Relación LF/HF (cmor1.5-1.0): 63.58

 Comparando con wavelet 'mexh'...

Potencia LF (mexh): 5.42
Potencia HF (mexh): 0.04
Relación LF/HF (mexh): 148.42

## **Análisis de Resultados**
El análisis de la variabilidad de la frecuencia cardíaca (HRV) se abordó desde dos enfoques complementarios: el dominio del tiempo y el dominio tiempo-frecuencia mediante transformada wavelet. En el análisis temporal, se calcularon los intervalos R-R y se extrajeron parámetros estadísticos como la media y la desviación estándar (SDRR). Para la señal ECG 1, se obtuvo una media RR de 0.0426 segundos y una SDRR de 0.0035 s, mientras que para ECG 2, la media fue de 0.0285 s y la SDRR de 0.0039 s. Esto indica que, aunque ECG 1 tiene una menor frecuencia cardíaca promedio, presenta una variabilidad ligeramente menor en comparación con ECG 2, cuyas fluctuaciones son un poco más marcadas.

Por otro lado, el análisis en el dominio tiempo-frecuencia utilizando transformada wavelet (CWT) permitió observar cómo se distribuye la potencia de la señal en distintas bandas de frecuencia a lo largo del tiempo. Se utilizaron las wavelets cmor1.5-1.0 y mexh, y se calcularon las potencias en las bandas de baja (LF) y alta frecuencia (HF), así como la relación LF/HF. En ambas señales, la potencia en LF fue significativamente mayor que en HF, lo cual indica un predominio de la actividad simpática. Específicamente, la señal ECG 1 mostró una relación LF/HF de 47.32 con cmor y 103.33 con mexh, mientras que ECG 2 alcanzó relaciones aún más elevadas: 63.58 y 148.42 respectivamente. Estos resultados revelan un mayor dominio simpático en ECG 2.

## **Conclusiones**

En conclusión, los resultados obtenidos cumplen con los objetivos propuestos en la guía, que buscaban relacionar la variabilidad en el dominio del tiempo con la distribución espectral obtenida mediante wavelets. Se logró observar cómo estas dos formas de análisis se complementan para ofrecer una visión más completa de la modulación cardíaca y su relación con el sistema nervioso autónomo.

## Bibliografía
[1] L. Veloza, C. Jiménez, D. Quiñones, F. Polanía, L. C. Pachón-Valero, and C. Y. Rodríguez-Triviño, “Variabilidad de la frecuencia cardiaca como factor predictor de las enfermedades cardiovasculares,” Revista Colombiana De Cardiología, vol. 26, no. 4, pp. 205–210, Jun. 2019, doi: 10.1016/j.rccar.2019.01.006.

[2] T. F. of the E. S. of C. the N. A. Electrophysiology, “Heart rate variability,” Circulation, vol. 93, no. 5, pp. 1043–1065, Mar. 1996, doi: 10.1161/01.cir.93.5.1043.

[3] https://biblus.us.es/bibing/proyectos/abreproy/11511/fichero/PFC+Silvia+Blasco+Vadillo%252FCap%C3%ADtulo+9+-+Anexo+2.pdf+#:~:text=La%20Transformada%20Wavelet%20es%20un,im%C3%A1genes%20m%C3%A9dicas%20y%20se%C3%B1ales%20biol%C3%B3gicas.


## Colaboradores
1. Youling Andrea Orjuela Bermúdez (5600815)
2. Jose Manuel Gomez Carrillo (5600793)
3. Juan Camilo Quintero Velandia (5600745)

