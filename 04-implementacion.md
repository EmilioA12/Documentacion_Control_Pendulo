---
layout: default
title: Implementación en Simulink
nav_order: 5
---

# Implementación en Simulink y QUARC

## 1. Organización del modelo

La implementación final se encuentra en:

`qs3_lqr_ctrl_simulinkFInal.slx`

El modelo integra la adquisición de posiciones angulares,
los observadores, el controlador LQR y la escritura del
comando al motor del Qube-Servo 3.

El recorrido principal de las señales es:

1. Lectura de los encoders.
2. Conversión de cuentas a posiciones angulares.
3. Estimación de velocidades mediante los observadores.
4. Construcción del vector de realimentación.
5. Cálculo del comando LQR.
6. Selección entre control de balance y 0 V.
7. Envío del comando al equipo.

Los Scopes permiten visualizar las posiciones,
las estimaciones, el comando y la habilitación del balance.

## 2. Preparación de las variables en MATLAB

Antes de ejecutar el modelo se utiliza el script:

`ProyectoControlAvanzadoFinalPenduloCode.m`

Este archivo define los parámetros físicos, construye las
matrices A y B, calcula la ganancia K del LQR y obtiene
las ganancias de los observadores.

Las principales variables utilizadas por los bloques son:

| Variable | Uso en Simulink |
|---|---|
| K | Ganancia de realimentación LQR |
| L | Ganancia aplicada al error de posición del observador |
| beta | Ganancia de realimentación de la rama auxiliar |
| m | Ganancia de la rama auxiliar hacia la estimación de velocidad |

El script debe ejecutarse antes de actualizar o compilar
el diagrama para que estas variables estén disponibles.

Las matrices A y B se utilizan para diseñar K en MATLAB.
En la ejecución con hardware, la planta física es el Qube:
no se sustituye por un bloque de simulación de esas matrices.

## 3. Configuración guardada

| Parámetro | Valor |
|---|---|
| Plataforma | Qube-Servo 3 |
| Modo | Hardware |
| Configuración mecánica | Pendulum |
| Tipo de tarjeta | `qube_servo3_usb` |
| Número de tarjeta | 0 |
| Objetivo de compilación | `quarc_win64.tlc` |
| Solver | `ode1` |
| Paso fijo | 0.002 s |
| Frecuencia correspondiente | 500 Hz |
| Tiempo final | `inf` |

El paso de 0.002 s corresponde a:

$$
f_s=\frac{1}{T_s}
=\frac{1}{0.002}
=500\ \mathrm{Hz}.
$$

Estos valores describen la configuración guardada.
El cumplimiento del tiempo de ejecución en el equipo
debe comprobarse durante las pruebas con QUARC.

## 4. Comunicación con el Qube-Servo 3

Dentro de `Qube With Pendulum` se encuentra el subsistema:

`Qube-Servo 3 - IO (QAL)`

Este contiene los bloques de comunicación de QUARC.

| Bloque | Función |
|---|---|
| HIL Initialize | Inicializa y configura la tarjeta de adquisición |
| HIL Read Timebase | Lee las señales utilizando la base de tiempo del equipo |
| HIL Write | Escribe las señales de salida, incluido el comando al motor |

La interfaz proporciona cuentas de encoder, información
de velocidad, corriente del motor y detección de bloqueo.

En esta implementación se utilizan los dos encoders para
obtener las posiciones. Las salidas de velocidad del hardware,
la corriente y la señal de detección de bloqueo terminan
en bloques Terminator y no alimentan al LQR.

Las velocidades utilizadas por el controlador proceden
de los observadores implementados en `State X`.

## 5. Conversión de cuentas a ángulos

El subsistema `Counts to Angles` transforma las cuentas
de los encoders en posiciones angulares.

El factor empleado es:

$$
c=\frac{2\pi}{512\cdot4}
=\frac{2\pi}{2048}
\quad \mathrm{rad/cuenta}.
$$

### Posición angular del brazo

La conversión incluye una inversión de signo:

$$
\theta=-c\,n_\theta,
$$

donde nθ es la lectura del encoder de la base.

La ganancia −1 forma parte de la convención de orientación
utilizada por el modelo.

### Posición angular del péndulo

La lectura del péndulo se convierte a radianes,
se aplica una operación módulo y se resta π:

$$
\alpha=
\operatorname{mod}(c\,n_\alpha,2\pi)-\pi.
$$

Con la inicialización prevista de los encoders,
esta transformación expresa el ángulo respecto
a la vertical superior:

$$
\alpha=0
\quad\text{en la posición invertida}.
$$

La operación módulo mantiene el ángulo en el intervalo
[−π, π). Al cruzar sus límites puede aparecer un salto
en la representación angular.

La referencia inicial de los encoders debe comprobarse
al preparar el montaje, porque una inicialización incorrecta
modifica la interpretación de las posiciones.

## 6. Observadores y vector de realimentación

Las posiciones θ y α entran al subsistema `State X`.

Cada posición sigue tres rutas:

- Alimenta directamente al Mux de realimentación.
- Entra a su observador.
- Se envía al Scope que la compara con su estimación.

Los observadores entregan una velocidad estimada
y una posición estimada.

| Entrada del Mux | Señal |
|---|---|
| 1 | θ medida |
| 2 | α medida |
| 3 | Velocidad estimada por Observador Theta |
| 4 | Velocidad estimada por Observador Alpha |

Por tanto:

$$
x_c=
\begin{bmatrix}
\theta\\
\alpha\\
\widehat{\dot\theta}\\
\widehat{\dot\alpha}
\end{bmatrix}.
$$

Los ángulos están en radianes y las velocidades
estimadas en radianes por segundo.

Las posiciones estimadas se envían a los Scopes
para compararlas con las mediciones. No sustituyen
a los ángulos medidos dentro del vector de realimentación.

### Bloques conservados de la versión anterior

Los subsistemas `Filtro grado1 Theta` y
`Filtro grado1 Alpha` permanecen conectados a las
posiciones, pero sus salidas no se utilizan en el LQR.

También permanece el bloque `alpha_dot`,
con función de transferencia:

$$
G(s)=\frac{50s}{s+50}.
$$

Este bloque tampoco alimenta la realimentación final.
La fuente de velocidades del controlador son
las salidas `dot` de los observadores.

## 7. Implementación de la ley LQR

La referencia del brazo es cero y la posición deseada
del péndulo es la vertical superior.

El bloque `Ground` proporciona la referencia cero
a la entrada positiva del sumador. El vector de estados
entra por la entrada negativa.

Así, el sumador entrega:

$$
0-x_c=-x_c.
$$

El bloque denominado `u = -K*x` tiene como parámetro
de ganancia la variable K y realiza una multiplicación
matricial.

Su salida es:

$$
u_{\mathrm{LQR}}=K(-x_c)=-Kx_c.
$$

El signo negativo se obtiene en el sumador.
No se aplica una segunda negación dentro de la ganancia K.

El orden de las señales en el Mux coincide con el orden
de los estados utilizado al calcular K en MATLAB.

## 8. Habilitación del control de balance

El LQR está diseñado alrededor de la vertical superior.
Por ello, se utiliza una lógica que permite aplicar
el control solamente dentro de una ventana angular.

La posición medida del péndulo se convierte a grados
antes de entrar al bloque MATLAB Function.

La función guardada implementa:

```matlab
function y = fcn(alpha)

if alpha <= 15 && alpha >= -15
    y = 1;
else
    y = 0;
end

end
```

En esta función, alpha está expresada en grados.

La salida controla el selector
`Enable Balance Control Switch`:

| Habilitación | Señal seleccionada |
|---|---|
| 0 | 0 V |
| 1 | Comando calculado por el LQR |

La señal seleccionada puede expresarse como:

$$
u_{\mathrm{sel}}=
\begin{cases}
-Kx_c,
& |\alpha_{\mathrm{deg}}|\leq15^\circ,\\
0,
& |\alpha_{\mathrm{deg}}|>15^\circ.
\end{cases}
$$

El péndulo se eleva manualmente hasta entrar en la
ventana de balance. No se implementa swing-up.

La habilitación se evalúa continuamente: si el péndulo
sale de la ventana, se selecciona nuevamente 0 V.

### Correspondencia con la consigna

El documento del curso y el procedimiento de Quanser
indican una ventana de ±10°.

El archivo final revisado conserva ±15°. Esta página
registra el valor realmente programado. La diferencia
debe resolverse mediante su ajuste y validación,
o documentando la justificación aprobada para utilizarlo.

## 9. Comando enviado a la interfaz

La señal seleccionada entra al subsistema
`Qube With Pendulum`.

Antes de llegar a la interfaz de hardware, atraviesa
el bloque `For +ve CCW`, cuya ganancia es −1:

$$
u_{\mathrm{HIL}}=-u_{\mathrm{sel}}.
$$

Esta inversión forma parte de la convención de signos
de la interfaz. Debe interpretarse junto con los signos
de los encoders y del modelo utilizado para diseñar K.

El Scope `Vm (V)` registra el comando seleccionado
antes de esta inversión. Por tanto, representa
u_sel y no una medición independiente del voltaje
real en los terminales del motor.

No hay un bloque explícito `Saturation` en la trayectoria
principal de control del modelo revisado.

La ponderación R = 10 del LQR no representa un límite
de voltaje de 10 V. Los límites del equipo y la
configuración de la interfaz se analizan por separado.

## 10. Visualización de señales

| Scope | Señales mostradas | Unidad |
|---|---|---|
| Base (deg) | Referencia cero y posición medida del brazo | Grados |
| Pendulum (deg) | Posición medida del péndulo | Grados |
| Vm (V) | Comando seleccionado antes de invertir su signo | Voltios |
| Scope del nivel principal | Habilitación del balance | 0 o 1 |
| Scope Theta | Posición medida y estimada del brazo | Radianes |
| Scope Alpha | Posición medida y estimada del péndulo | Radianes |

Los Scopes de los observadores comparan posiciones.
Para documentar las velocidades estimadas deben
registrarse también las salidas `dot`.

Las conversiones a grados se utilizan para visualización
y para la lógica de habilitación. La realimentación LQR
utiliza radianes y radianes por segundo.

## 11. Secuencia de preparación y ejecución

La secuencia de trabajo del modelo es:

1. Preparar el equipo y comprobar las conexiones del
   péndulo y de sus encoders.
2. Colocar el montaje en la posición inicial prevista
   por el procedimiento del laboratorio.
3. Ejecutar el script MATLAB para cargar K, L, beta y m.
4. Abrir el modelo y actualizar el diagrama.
5. Verificar la configuración de hardware, el paso de
   ejecución y la ventana de activación.
6. Compilar e iniciar la ejecución mediante QUARC.
7. Elevar manualmente el péndulo hasta la zona de balance.
8. Observar y registrar las señales de la prueba.
9. Detener la ejecución al terminar y conservar
   los registros utilizados para el análisis.

La descripción del diagrama y sus parámetros no
sustituye las evidencias de ejecución en el equipo.

## 12. Capturas y evidencias de implementación

Para acompañar esta explicación se incorporarán:

- Captura del nivel principal del modelo.
- Captura del subsistema `Qube With Pendulum`.
- Captura de `Counts to Angles`.
- Captura de `State X` mostrando ambos observadores
  y las conexiones al Mux.
- Captura de la estructura interna de los observadores.
- Captura de la configuración de ejecución.
- Gráficas y video de las pruebas en el equipo físico.

Las capturas deben corresponder a la misma versión
de los programas que se entrega y documenta.

## 13. Archivos y referencias

- Código MATLAB: `ProyectoControlAvanzadoFinalPenduloCode.m`.
- Modelo final: `qs3_lqr_ctrl_simulinkFInal.slx`.
- Documento del curso: requisitos del Qube-Servo 3
  y esquema del observador.
- [Quanser: modelo base de Simulink](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control/hardware/matlab/qs3_lqr_ctrl.slx).
- [Quanser: procedimiento del laboratorio LQR](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control/hardware/matlab/lab_procedure_lqr_control.pdf).

