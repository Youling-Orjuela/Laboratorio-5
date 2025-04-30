# Laboratorio 5: Variabilidad de la Frecuencia Cardiaca usando la Transformada Wavelet
## INVESTIGACION PREVIA
### 1. Actividad simpática y parasimpática del sistema nervioso autónomo:
El sistema nervioso autónomo controla la presión arterial, la frecuencia cardíaca, la temperatura corporal, el peso, la digestión, el metabolismo, el balance hidroelectrolítico, la sudoración, la micción, la defecación, la respuesta sexual y otros procesos. Muchos órganos están regulados especialmente por el sistema simpático o parasimpático, aunque pueden recibir aferencias de ambos; en ocasiones, las funciones son recíprocas (p. ej., la activación simpática acelera la frecuencia cardíaca; la parasimpática la disminuye).

El sistema nervioso parasimpático es catabólico; activa las respuestas de lucha o huida.

El sistema nervioso parasimpático es anabólico; conserva y restablece.

### 2. Efecto de la actividad simpática y parasimpática en la frecuencia cardiaca
1. Inervación parasimpática: 
Las porciones parasimpáticas del plexo cardíaco solo reciben contribuciones del nervio vago. Las fibras preganglionares, que se ramifican desde el nervio vago derecho e izquierdo, llegan al corazón para luego entrar en el plexo cardíaco haciendo sinapsis con los ganglios de este plexo y las paredes de los atrios.

- La inervación parasimpática es responsable de, reducir la frecuencia cardíaca, reducir la fuerza de contracción del corazón, vasoconstricción (estrechamiento) de las arterias coronarias.

2. Inervación simpática: 
La porción simpática del plexo cardíaco está compuesta por fibras del tronco simpático, las cuales tienen su origen a partir de los segmentos superiores de la médula espinal torácica. Las fibras del tronco simpático llegan al plexo cardíaco mediante los nervios cardíacos. Las fibras preganglionares se ramifican desde la médula espinal torácica superior y hacen sinapsis en los ganglios cervicales inferiores y torácicos superiores. Las fibras postganglionares se extienden desde los ganglios hasta el plexo cardíaco.

- Los nervios simpáticos son responsables de, aumentar la frecuencia cardíaca, aumentar la fuerza de contracción del miocardio, la respuesta de ‘lucha o huida’, que aumenta la frecuencia cardíaca.

### 3.	Variabilidad de la frecuencia cardiaca (HRV) medida como fluctuaciones en el intervalo R-R, y las frecuencias de interés en este análisis

El punto fiducial reconocido en el trazado del ECG que identifica un complejo QRS puede basarse en el máximo o baricentro del complejo, en la determinación del máximo de una curva de interpolación, o encontrarse mediante la comparación con una plantilla u otros marcadores de eventos. Para localizar el punto fiducial, las normas voluntarias para equipos de ECG de diagnóstico son satisfactorias en términos de relación señal-ruido, rechazo de modo común, ancho de banda, etc.  Un límite de frecuencia de corte de banda superior sustancialmente inferior al establecido para equipos de diagnóstico (≈200 Hz) puede generar fluctuación en el reconocimiento del punto fiducial del complejo QRS, lo que introduce un error en los intervalos RR medidos. De igual forma, una frecuencia de muestreo limitada induce un error en el espectro de la VFC que aumenta con la frecuencia, afectando así a los componentes de mayor frecuencia.  Una interpolación de la señal de ECG submuestreada puede reducir este error. Con una interpolación adecuada, incluso una frecuencia de muestreo de 100 Hz puede ser suficiente.

### 4.	Transformada Wavelet: definición, usos y tipos de wavelet utilizadas en señales biológicas.

La Transformada Wavelet es una herramienta matemática que permite descomponer una señal en componentes de diferente escala o resolución. A diferencia de la Transformada de Fourier, que analiza la señal en el dominio de la frecuencia de manera global, la Wavelet permite un análisis multiresolución: localiza tanto en el tiempo como en la frecuencia. Esto la hace especialmente útil para analizar señales no estacionarias, como las biológicas.

Usos en señales biológicas:
En el contexto del análisis de señales biológicas (como EEG, ECG, EMG, PPG, entre otras), la Transformada Wavelet se emplea para:

-	Eliminar ruido (filtrado de artefactos).
-	Detectar eventos transitorios (como picos o complejos QRS en ECG).
-	Extraer características relevantes para clasificación automática (por ejemplo, para diagnóstico).
-	Compresión de datos sin perder información significativa.
-	Análisis de variabilidad y dinámica de señales fisiológicas.
  
La wavelet Morlet es una de las más utilizadas en neurociencia y análisis de señales cerebrales (EEG, MEG) debido a su capacidad de representar frecuencias específicas con gran resolución temporal y frecuencia. Esta es ideal para detectar oscilaciones neuronales en bandas específicas (alfa, beta, gamma, etc.), tiene buena resolución en frecuencia, lo cual permite analizar ritmos cerebrales con precisión y es adecuada para transformada wavelet continua

