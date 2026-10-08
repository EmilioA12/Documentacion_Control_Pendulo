---
layout: default
title: Modelado del sistema
nav_order: 2
---

# Modelado del sistema

## 1. Plataforma y objetivo

El Quanser Qube-Servo 3, en configuración de péndulo invertido,
está formado por un brazo rotatorio accionado por un motor y un
péndulo que gira libremente alrededor de su articulación.

El motor actúa sobre el brazo. El movimiento del brazo permite
influir en el péndulo mediante el acoplamiento mecánico.

El objetivo es mantener el péndulo cerca de la vertical superior
y regular la posición angular del brazo.

## 2. Variables y estados

Se utiliza el siguiente vector de estados:

$$
x=
\begin{bmatrix}
\theta & \alpha & \dot{\theta} & \dot{\alpha}
\end{bmatrix}^{T}
$$

| Variable | Significado | Unidad |
|---|---|---|
| θ | Posición angular del brazo respecto a su referencia | rad |
| α | Ángulo del péndulo respecto a la vertical superior | rad |
| θ̇ | Velocidad angular del brazo | rad/s |
| α̇ | Velocidad angular del péndulo | rad/s |
| u | Voltaje de entrada del modelo del motor | V |

La posición invertida corresponde a α = 0.

Los encoders proporcionan las posiciones angulares del brazo
y del péndulo. En la implementación final, cada posición medida
alimenta un observador que estima su velocidad angular.

Es importante distinguir el vector de estados físicos del
modelo del vector utilizado en la realimentación.

El modelo matemático describe los estados:

$$
x=
\begin{bmatrix}
\theta & \alpha & \dot{\theta} & \dot{\alpha}
\end{bmatrix}^{T}.
$$

En Simulink, el controlador recibe:

$$
x_c=
\begin{bmatrix}
\theta & \alpha & \widehat{\dot{\theta}} &
\widehat{\dot{\alpha}}
\end{bmatrix}^{T}.
$$

Los dos primeros elementos son las posiciones medidas por
los encoders. Los dos últimos son las velocidades estimadas
por los subsistemas `Observador Theta` y `Observador Alpha`.

Las posiciones estimadas por los observadores se comparan
con las mediciones en los Scopes, pero no sustituyen las
posiciones medidas dentro del vector de realimentación.

El modelo base de Quanser obtiene las velocidades mediante
derivación filtrada. En nuestra implementación final, las
velocidades utilizadas por el LQR proceden de los observadores.

El diseño de sus ganancias y su implementación fueron
realizados por Emilio Acuña siguiendo el esquema de la
figura 3 del documento del curso. Su desarrollo se presenta
en la sección dedicada al observador.

Las conversiones de cuentas a radianes y los signos utilizados
en la interfaz se documentan en la sección de implementación.

## 3. Parámetros utilizados

Los siguientes valores aparecen en el código MATLAB y coinciden
con los parámetros del PDF del proyecto y del archivo oficial
`qube3_rotpen_param.m` del laboratorio LQR de Quanser.

| Parámetro | Símbolo | Valor | Unidad |
|---|---|---|---|
| Resistencia del motor | Rm | 7.5 | Ω |
| Constante de torque | kt | 0.0422 | N·m/A |
| Constante contraelectromotriz | km | 0.0422 | V·s/rad |
| Masa del brazo | mr | 0.095 | kg |
| Longitud del brazo | r | 0.085 | m |
| Amortiguamiento del brazo | br | 0.001 | N·m·s/rad |
| Masa del péndulo | mp | 0.024 | kg |
| Longitud del péndulo | Lp | 0.129 | m |
| Distancia al centro de masa | l | 0.0645 | m |
| Amortiguamiento del péndulo | bp | 0.00005 | N·m·s/rad |
| Aceleración de la gravedad | g | 9.81 | m/s² |

Las inercias se calculan aproximando los elementos como barras
uniformes y usando sus ejes de giro:

$$
J_r=\frac{m_r r^2}{3},
\qquad
J_p=\frac{m_p L_p^2}{3},
\qquad
l=\frac{L_p}{2}.
$$

**Alcance de los parámetros:** los archivos entregados no
incluyen una identificación experimental propia. En el recurso
oficial, los amortiguamientos incluyen una nota de ajuste
heurístico para aproximar la respuesta del QUBE-Servo 2.
Se emplean aquí como parámetros del modelo proporcionado,
sin afirmar que hayan sido medidos en nuestro equipo.

## 4. Equilibrio y linealización

El modelo se utiliza alrededor del equilibrio invertido:

$$
\alpha=0,\qquad
\dot{\theta}=0,\qquad
\dot{\alpha}=0,\qquad
u=0.
$$

La posición del brazo se expresa respecto a una referencia
constante; para el equilibrio de origen se toma θ = 0.

Alrededor de la vertical superior se aplican las aproximaciones:

$$
\sin(\alpha)\approx\alpha,
\qquad
\cos(\alpha)\approx1.
$$

En la aproximación de primer orden se descartan productos de
pequeñas desviaciones y términos de orden superior.

Esto permite obtener un modelo lineal local. No significa que
el péndulo se comporte linealmente en todo su recorrido.

La zona de activación de ±10° solicitada en el proyecto es una
condición de operación del controlador; no constituye por sí
sola una demostración de estabilidad para todos los estados
dentro de ese intervalo.

## 5. Reconstrucción de las ecuaciones linealizadas

El script MATLAB implementa directamente los coeficientes de
A y B. A continuación se desarrolla una reconstrucción algebraica
compatible con esas expresiones y su convención de signos.

Para simplificar la escritura, se definen:

$$
M=J_r+m_p r^2,
\qquad
h=m_p l r,
\qquad
G=m_p g l.
$$

Las ecuaciones mecánicas linealizadas son:

$$
M\ddot{\theta}-h\ddot{\alpha}
+b_r\dot{\theta}=\tau
$$

$$
-h\ddot{\theta}+J_p\ddot{\alpha}
+b_p\dot{\alpha}-G\alpha=0.
$$

El término gravitacional corresponde al equilibrio superior,
que es inestable sin control.

El parámetro h representa el acoplamiento: acelerar el brazo
influye en la aceleración del péndulo.

## 6. Conversión de voltaje a torque

Despreciando la dinámica de la inductancia del motor:

$$
i=\frac{u-k_m\dot{\theta}}{R_m},
\qquad
\tau=k_t i.
$$

Por tanto:

$$
\tau=
\frac{k_t}{R_m}u
-\frac{k_t k_m}{R_m}\dot{\theta}.
$$

El primer término representa la acción del voltaje; el segundo
proviene de la fuerza contraelectromotriz.

En los parámetros empleados se cumple numéricamente kt = km.
Por ello, el script utiliza km en los términos de entrada
y km² en los términos de amortiguamiento eléctrico.

Se define:

$$
b_e=b_r+\frac{k_t k_m}{R_m}.
$$

Al sustituir el torque en las ecuaciones mecánicas:

$$
\begin{bmatrix}
M & -h\\
-h & J_p
\end{bmatrix}
\begin{bmatrix}
\ddot{\theta}\\
\ddot{\alpha}
\end{bmatrix}
=
\begin{bmatrix}
\frac{k_t}{R_m}u-b_e\dot{\theta}\\
G\alpha-b_p\dot{\alpha}
\end{bmatrix}.
$$

## 7. Despeje de las aceleraciones

El determinante de la matriz de inercia es:

$$
J_t=MJ_p-h^2
=(J_r+m_p r^2)J_p-m_p^2l^2r^2.
$$

Aunque se denomina Jt en el código, esta expresión es el
determinante de la matriz de inercia y tiene unidades de
kg²·m⁴.

Al invertir esa matriz:

$$
\ddot{\theta}=
\frac{hG}{J_t}\alpha
-\frac{J_p b_e}{J_t}\dot{\theta}
-\frac{h b_p}{J_t}\dot{\alpha}
+\frac{J_p k_t}{R_mJ_t}u
$$

$$
\ddot{\alpha}=
\frac{MG}{J_t}\alpha
-\frac{h b_e}{J_t}\dot{\theta}
-\frac{M b_p}{J_t}\dot{\alpha}
+\frac{h k_t}{R_mJ_t}u.
$$

Estas expresiones permiten identificar directamente las
filas tercera y cuarta de A y B.

## 8. Representación en espacio de estados

El sistema tiene la forma:

$$
\dot{x}=Ax+Bu.
$$

Con las definiciones anteriores:

$$
A=
\begin{bmatrix}
0&0&1&0\\
0&0&0&1\\
0&\frac{hG}{J_t}&-\frac{J_p b_e}{J_t}&-\frac{h b_p}{J_t}\\
0&\frac{MG}{J_t}&-\frac{h b_e}{J_t}&-\frac{M b_p}{J_t}
\end{bmatrix}
$$

$$
B=
\begin{bmatrix}
0\\
0\\
\frac{J_p k_t}{R_mJ_t}\\
\frac{h k_t}{R_mJ_t}
\end{bmatrix}.
$$

Sustituyendo los parámetros:

$$
A\approx
\begin{bmatrix}
0&0&1&0\\
0&0&0&1\\
0&55.15252&-4.54706&-0.18159\\
0&168.58098&-4.49419&-0.55506
\end{bmatrix}
$$

$$
B\approx
\begin{bmatrix}
0\\
0\\
20.67551\\
20.43509
\end{bmatrix}.
$$

Las primeras dos filas indican que la derivada de cada posición
es su velocidad. Las últimas dos describen cómo la gravedad,
el amortiguamiento y el voltaje producen aceleraciones.

Para representar como salidas los dos ángulos medidos:

$$
y=Cx+Du,
\qquad
C=
\begin{bmatrix}
1&0&0&0\\
0&1&0&0
\end{bmatrix},
\qquad
D=
\begin{bmatrix}
0\\
0
\end{bmatrix}.
$$

C y D se incorporan aquí para explicar las mediciones.
El script entregado calcula A y B, pero no define C y D.

## 9. Alcance y limitaciones

- El modelo describe el comportamiento cerca de la vertical superior.
- Se utilizan inercias aproximadas y amortiguamiento viscoso.
- No se representa explícitamente la inductancia del motor.
- No se incluyen todos los efectos de fricción, cableado,
  ruido, cuantización y límites de voltaje.
- La coincidencia con las matrices de referencia verifica
  la formulación, pero no sustituye una validación experimental.

## 10. Fuentes y trazabilidad

### Material del curso

El documento del proyecto de Control Avanzado y Robótica,
en la sección del Equipo B y el Cuadro 2, proporciona los
parámetros y las expresiones de las matrices utilizadas
para el péndulo invertido.

### Recursos de Quanser

- [Parámetros del péndulo: qube3_rotpen_param.m](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control/hardware/matlab/qube3_rotpen_param.m).
- [Procedimiento del laboratorio LQR](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control/hardware/matlab/lab_procedure_lqr_control.pdf).
- [Guía teórica del control LQR](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control/application_guide_lqr_control.pdf).
- [Modelo base de Simulink: qs3_lqr_ctrl.slx](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control/hardware/matlab/qs3_lqr_ctrl.slx).

Los parámetros del script del proyecto coinciden con
`qube3_rotpen_param.m`. Las matrices numéricas A y B
coinciden, al redondeo mostrado, con las publicadas en
el procedimiento del laboratorio LQR de Quanser.

El archivo `rotpen_ABCD_eqns_ip.m` incluido en esa carpeta
del laboratorio es una plantilla que debe completarse.
La comprobación de las matrices se realiza contra los
resultados publicados en el procedimiento y las
expresiones del documento del curso.

### Archivos finales del proyecto

- Código MATLAB: `ProyectoControlAvanzadoFinalPenduloCode.m`.
- Modelo Simulink: `qs3_lqr_ctrl_simulinkFInal.slx`.

El script implementa los parámetros y las matrices A y B,
calcula la ganancia LQR y obtiene las ganancias del observador.

El modelo Simulink integra las mediciones del equipo,
los observadores y la ley de control.

### Alcance del desarrollo presentado

Las ecuaciones intermedias de esta página explican cómo
se obtienen las matrices implementadas en el código,
conservando su convención de signos.

La coincidencia con las referencias verifica la formulación
del modelo nominal. La validación experimental requiere
comparar sus resultados con las mediciones del equipo,
lo cual se aborda en la sección de resultados.