---
layout: default
title: Resultados y discusión
nav_order: 6
---

# Resultados experimentales y discusión

## 1. Descripción del ensayo

Se realizaron pruebas con el péndulo invertido del
Quanser Qube-Servo 3 utilizando el controlador LQR
y los observadores implementados en Simulink.

El procedimiento experimental consistió en:

1. Iniciar con el péndulo en la posición inferior.
2. Elevarlo manualmente hasta la zona de balance.
3. Permitir que el controlador mantuviera el péndulo
   cerca de la vertical superior.
4. Aplicar perturbaciones manuales mediante pequeños
   empujones con el dedo.
5. Observar la respuesta del brazo, del péndulo
   y de los observadores.

Las evidencias disponibles son cuatro capturas:
Base (deg), Pendulum (deg), Scope Theta y Scope Alpha.

Las figuras corresponden al equipo físico.
Los valores mencionados son lecturas visuales aproximadas,
no métricas calculadas a partir de registros numéricos.

## 2. Lectura de las capturas

La posición invertida del péndulo corresponde a α = 0.
La posición inferior se representa cerca de ±180°
o, equivalentemente, ±π rad.

Las capturas Base y Pendulum muestran registros de
aproximadamente 18.9 s. Las de los observadores indican
un tiempo final cercano a 15.6 s.

Scope Theta presenta además un desplazamiento temporal
indicado como `Offset = 10`: sus etiquetas horizontales
deben interpretarse considerando ese desplazamiento.

Debido a estas diferencias, las capturas se analizan
por separado. No se establece una correspondencia
punto a punto entre sus eventos.

Los ángulos de Base y Pendulum están en grados.
Según las conexiones del modelo, las comparaciones
de Scope Theta y Scope Alpha están en radianes.

## 3. Respuesta del péndulo

![Posición angular del péndulo en grados]({{ '/assets/images/resultados/pendulo_grados.png' | relative_url }})

[Abrir gráfica a tamaño completo]({{ '/assets/images/resultados/pendulo_grados.png' | relative_url }})

### Posición inicial

Durante los primeros segundos, el ángulo aparece
alternando entre valores cercanos a −180° y +180°.

Ambos valores corresponden a orientaciones próximas
a la posición inferior. Los saltos son compatibles
con el cambio de representación producido por la
operación módulo utilizada en la conversión angular.

Por tanto, las líneas verticales no deben interpretarse
automáticamente como giros físicos completos del péndulo.

### Elevación manual y entrada al balance

Aproximadamente entre 3.6 y 6 s, el ángulo cambia
desde la posición inferior hasta las cercanías de cero.

Este tramo corresponde a la elevación manual descrita
en el ensayo. Al llegar a la zona superior, el controlador
toma el balance y la señal permanece cerca de α = 0.

La captura permite identificar la llegada a la vertical,
pero no determina el instante exacto de habilitación,
porque no incluye la señal lógica del selector.

### Respuesta a las perturbaciones

En la parte posterior del registro se observan
desviaciones breves alrededor de cero, especialmente
a partir de aproximadamente 12 s.

De acuerdo con el procedimiento del ensayo,
estas variaciones se relacionan con los empujones
aplicados manualmente.

Después de varias de estas desviaciones, el ángulo
regresa hacia las cercanías de cero. Esto aporta
evidencia de recuperación del balance frente a las
perturbaciones aplicadas durante este registro.

La escala vertical incluye todo el recorrido del péndulo,
por lo que las pequeñas oscilaciones del balance aparecen
comprimidas. No se asigna una amplitud exacta ni un tiempo
de establecimiento a partir de esta imagen.

## 4. Respuesta del brazo rotatorio

![Posición angular del brazo y referencia cero]({{ '/assets/images/resultados/base_grados.png' | relative_url }})

[Abrir gráfica a tamaño completo]({{ '/assets/images/resultados/base_grados.png' | relative_url }})

La línea horizontal amarilla representa la referencia
cero. La curva azul muestra la posición angular del brazo.

En la primera parte del registro aparecen desplazamientos
relativamente pequeños. Posteriormente, las excursiones
del brazo aumentan de forma apreciable.

Por lectura visual, se observan extremos cercanos a
+28° y −22° en el conjunto del registro.

El movimiento del brazo permite actuar sobre el péndulo
mediante el acoplamiento mecánico. Por ello, durante
la recuperación del balance pueden aparecer movimientos
importantes de la base aunque la desviación del péndulo
respecto a la vertical sea pequeña.

Sin embargo, la gráfica también muestra que la posición
del brazo no permanece exactamente en su referencia.
Se observan desviaciones sostenidas y movimientos
transitorios considerables.

El final de la captura todavía presenta una desviación
del brazo. Por tanto, esta evidencia no permite afirmar
que haya regresado completamente a cero al terminar
el registro.

## 5. Observador de la posición del brazo

![Comparación de la posición del brazo y su estimación]({{ '/assets/images/resultados/observador_theta.png' | relative_url }})

[Abrir gráfica a tamaño completo]({{ '/assets/images/resultados/observador_theta.png' | relative_url }})

Scope Theta compara la posición medida del brazo
con la posición estimada por su observador.

Las dos curvas permanecen próximas durante gran parte
del tramo visible, incluso cuando la posición presenta
variaciones amplias.

Las diferencias más visibles aparecen durante los cambios
rápidos y cerca de los extremos de la trayectoria.
Este comportamiento es compatible con la dinámica
transitoria del estimador.

Hacia el final del tramo mostrado, ambas curvas se
aproximan a un valor casi constante de alrededor
de 0.2 rad.

La cercanía entre las curvas indica seguimiento de la
medición en ese intervalo. No significa que el brazo
se encuentre exactamente en su referencia cero.

Tampoco implica que el error de estimación sea nulo:
para cuantificarlo sería necesario disponer de las
series temporales y calcular la diferencia entre señales.

El rango angular de esta captura es distinto del mostrado
en Base (deg). Por ello, ambas imágenes no se utilizan
como registros directamente equivalentes mediante
una simple conversión de unidades.

## 6. Observador de la posición del péndulo

![Comparación de la posición del péndulo y su estimación]({{ '/assets/images/resultados/observador_alpha.png' | relative_url }})

[Abrir gráfica a tamaño completo]({{ '/assets/images/resultados/observador_alpha.png' | relative_url }})

Scope Alpha compara la posición medida del péndulo
con su estimación.

### Posición inferior y elevación

Al inicio, la señal medida se encuentra cerca de
−π rad, correspondiente a la posición inferior.

La estimación presenta un transitorio inicial y
posteriormente se aproxima a la medición.

Durante la elevación, ambas curvas avanzan hacia cero
y permanecen próximas durante buena parte de la maniobra.

### Balance y perturbaciones

Aproximadamente entre 2.7 y 11 s de esta captura,
las señales se mantienen cerca de cero.

Se observan variaciones transitorias durante ese intervalo.
Las dos curvas siguen una trayectoria similar, lo que
aporta evidencia cualitativa del seguimiento de la
posición medida por parte del observador durante el balance.

### Salida de la zona de balance

A partir de aproximadamente 11.1 s, la señal medida
se aleja de cero y posteriormente aparecen cambios
entre valores próximos a −π y +π rad.

Este tramo indica que el péndulo ya no permanece
cerca de la vertical superior.

La captura no permite determinar si esta salida
se debió a una perturbación, a una intervención manual
o a la finalización del ensayo.

Los saltos entre −π y +π son compatibles con el
cambio de representación angular alrededor de la
posición inferior.

Frente a estas discontinuidades, la curva estimada
presenta picos que alcanzan visualmente valores
cercanos a ±5 rad.

Estos picos corresponden a la respuesta del estimador.
No deben interpretarse como una medición directa
de giros adicionales del péndulo.

La figura muestra que el seguimiento durante el balance
es diferente de la respuesta del observador frente
a cambios abruptos de la señal angular.

## 7. Discusión de los resultados

### Balance del péndulo

La captura Pendulum muestra que, después de la elevación
manual, el péndulo permanece cerca de la vertical superior
y se recupera de varias perturbaciones del ensayo.

Este resultado respalda el funcionamiento del control
de balance en las condiciones observadas.

No demuestra estabilidad para cualquier perturbación
ni para todo el recorrido angular.

### Regulación del brazo

La base presenta movimientos apreciables durante
las correcciones y no permanece exactamente en cero.

Esto evidencia que mantener la vertical del péndulo
y regular la posición del brazo son objetivos relacionados,
pero no equivalentes.

En el diseño documentado, el error angular del péndulo
tiene mayor ponderación que el del brazo. Esta elección
es coherente con la prioridad del balance, aunque las
capturas por sí solas no permiten atribuir toda la respuesta
a esa ponderación.

También influyen las perturbaciones aplicadas, las
condiciones iniciales y las diferencias entre el modelo
nominal y el equipo físico.

### Desempeño de los observadores

Las comparaciones de posiciones muestran seguimiento
cercano en varios intervalos, con diferencias durante
transitorios rápidos.

En particular, Scope Alpha evidencia una respuesta
pronunciada del estimador ante los saltos de la
representación angular.

Esto permite identificar una limitación práctica:
la estabilidad de la dinámica interna del observador
no garantiza estimación exacta ante señales discontinuas.

Los Scopes presentados comparan posiciones.
Aunque el controlador utiliza las velocidades estimadas,
estas cuatro figuras no constituyen una validación
directa del error de estimación de velocidad.

## 8. Alcance de la evidencia

| Aspecto | Evidencia disponible |
|---|---|
| Elevación manual hacia la vertical | Visible en las capturas del péndulo |
| Permanencia cerca de la vertical | Visible durante intervalos de balance |
| Respuesta ante empujones | Desviaciones y recuperaciones en los registros descritos |
| Movimiento correctivo del brazo | Visible en Base (deg) |
| Seguimiento de posiciones por los observadores | Comparación de dos curvas en Theta y Alpha |
| Respuesta ante discontinuidades angulares | Picos del estimador visibles en Scope Alpha |
| Voltaje aplicado y saturación | No evaluables con estas cuatro capturas |
| Error de velocidad estimada | No evaluable directamente con estas figuras |
| Error RMS y tiempos precisos | Requieren los registros numéricos |
| Instante exacto de activación | Requiere la señal de habilitación |

Las capturas no muestran las ganancias ni el umbral
configurado durante cada adquisición. Los valores del
diseño se documentan en las secciones de LQR,
observador e implementación.

## 9. Conclusiones

Las pruebas aportan evidencia del balance del péndulo
tras su elevación manual y de recuperación frente
a varias perturbaciones aplicadas durante el ensayo.

El brazo realiza movimientos correctivos importantes,
por lo que no se considera demostrada una regulación
perfecta de su posición alrededor de cero.

Los observadores siguen de cerca las posiciones medidas
en varios intervalos. Las diferencias transitorias y
los picos ante discontinuidades muestran los límites
de ese seguimiento.

La salida de la zona de balance visible en Scope Alpha
se conserva como parte de la evidencia experimental.
No se afirma que todos los registros mantengan el
péndulo invertido hasta su finalización.

En conjunto, las gráficas permiten relacionar el diseño
matemático con el comportamiento físico observado,
incluyendo tanto los resultados favorables como
las limitaciones de la implementación.