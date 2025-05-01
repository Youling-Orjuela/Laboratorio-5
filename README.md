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

### **Captar la señal**
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
