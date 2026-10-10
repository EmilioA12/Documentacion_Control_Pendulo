---
layout: default
title: Control LQR
nav_order: 3
---

# Diseño del controlador LQR

## 1. Objetivo del controlador

El controlador busca mantener el péndulo cerca de la vertical
superior y regular la posición angular del brazo.

Se utiliza un regulador cuadrático lineal, LQR, para calcular
la ganancia de realimentación a partir del modelo en espacio
de estados y de una función de costo.

El diseño corresponde al modelo linealizado alrededor del
equilibrio invertido. Su aplicación física requiere levantar
manualmente el péndulo hasta la zona de balance.

## 2. Estados y ley de control

El orden de los estados es:

$$
x=
\begin{bmatrix}
\theta & \alpha & \dot{\theta} & \dot{\alpha}
\end{bmatrix}^{T}.
$$

Para estabilizar el origen se utiliza:

$$
u=-Kx.
$$

La ganancia contiene cuatro componentes:

$$
K=
\begin{bmatrix}
k_\theta & k_\alpha & k_{\dot{\theta}} & k_{\dot{\alpha}}
\end{bmatrix}.
$$

Cada componente multiplica el estado correspondiente. Es
necesario conservar el mismo orden de estados en MATLAB
y en el vector construido en Simulink.

En la implementación final, la referencia del brazo es cero.
La posición deseada del péndulo corresponde a la vertical
superior y las velocidades deseadas también son cero:

$$
x_{\mathrm{ref}}=
\begin{bmatrix}
0 & 0 & 0 & 0
\end{bmatrix}^{T}.
$$

El controlador recibe las posiciones medidas y las
velocidades estimadas por los observadores:

$$
x_c=
\begin{bmatrix}
\theta & \alpha & \widehat{\dot{\theta}} &
\widehat{\dot{\alpha}}
\end{bmatrix}^{T}.
$$

Por tanto, la ley implementada durante el balance es:

$$
u=K(x_{\mathrm{ref}}-x_c)=-Kx_c.
$$

Desarrollando sus componentes:

$$
u=
-k_{\theta}\theta
-k_{\alpha}\alpha
-k_{\dot{\theta}}\widehat{\dot{\theta}}
-k_{\dot{\alpha}}\widehat{\dot{\alpha}}.
$$

La ganancia K se diseña utilizando el modelo físico A, B.
Los observadores proporcionan las velocidades necesarias
para aplicar esa realimentación en el equipo.

La referencia cero se implementa mediante un bloque
`Ground` conectado a la entrada positiva del sumador.
El vector de realimentación entra por la entrada negativa.

## 3. Función de costo

Para la regulación al origen, el LQR minimiza:

$$
J=
\int_0^\infty
\left(x^TQx+u^TRu\right)\,dt.
$$

Las matrices cumplen funciones diferentes:

- Q pondera las desviaciones de los estados.
- R pondera la magnitud del esfuerzo de control.

El controlador es óptimo respecto a este costo y al modelo
utilizado. Esto no significa que sea óptimo frente a cualquier
criterio experimental ni que incorpore automáticamente
restricciones de voltaje.

## 4. Ponderaciones utilizadas

El script entregado contiene:

```matlab
Q = diag([1 20 0.1 0.1]);
R = 10;
```

Por tanto:

$$
Q=
\begin{bmatrix}
1&0&0&0\\
0&20&0&0\\
0&0&0.1&0\\
0&0&0&0.1
\end{bmatrix},
\qquad
R=10.
$$

El integrando de la función de costo queda:

$$
x^TQx+u^TRu=
\theta^2+20\alpha^2
+0.1\dot{\theta}^{\,2}
+0.1\dot{\alpha}^{\,2}
+10u^2.
$$

### Interpretación de los valores

| Ponderación | Interpretación |
|---|---|
| Q11 = 1 | Penaliza la desviación de la posición del brazo. |
| Q22 = 20 | Da mayor peso al error angular del péndulo, coherente con el objetivo de mantenerlo vertical. |
| Q33 = 0.1 | Incluye una penalización sobre la velocidad del brazo. |
| Q44 = 0.1 | Incluye una penalización sobre la velocidad del péndulo. |
| R = 10 | Penaliza el esfuerzo de control mediante el cuadrado del voltaje. |

Como ambos ángulos se expresan en radianes, desviaciones
angulares iguales del péndulo y del brazo aportan al costo
en una proporción de 20 a 1.

Las velocidades tienen unidades diferentes. Por ello, comparar
sus pesos directamente con los de posición no basta para
determinar su importancia durante una trayectoria real.

R = 10 no significa que el voltaje esté limitado a 10 V.
Es un peso dentro de la función de costo. Los límites físicos
y cualquier saturación deben analizarse por separado.

### Alcance de la justificación

Estas ponderaciones son las presentes en el archivo entregado.

Su estructura permite justificar una prioridad sobre el balance
del péndulo y una penalización del esfuerzo de control.
Sin embargo, los archivos disponibles no contienen una bitácora
de pruebas de Q y R.

Por tanto, no se afirma que estos valores hayan sido obtenidos
mediante una búsqueda exhaustiva, una secuencia específica de
ajustes o una comparación experimental documentada.

## 5. Controlabilidad del modelo

Antes de diseñar la realimentación se puede comprobar si
la entrada permite influir en todos los estados del modelo.

La matriz de controlabilidad es:

$$
\mathcal{C}=
\begin{bmatrix}
B & AB & A^2B & A^3B
\end{bmatrix}.
$$

La comprobación numérica con los parámetros del proyecto da:

$$
\operatorname{rango}(\mathcal{C})=4.
$$

Como el sistema tiene cuatro estados, el modelo es controlable.
Esta comprobación complementa la revisión del diseño; no
aparece explícitamente en el script original.

## 6. Cálculo de la ganancia

La función `lqr` resuelve el problema de regulación óptima.
Para el caso continuo sin término cruzado, la matriz P satisface:

$$
A^TP+PA-PBR^{-1}B^TP+Q=0.
$$

A partir de ella se obtiene:

$$
K=R^{-1}B^TP.
$$

En el script del proyecto, el cálculo se realiza mediante:

```matlab
K = lqr(A, B, Q, R);
```

La reproducción numérica del cálculo, usando los parámetros
del archivo MATLAB, produce:

$$
K\approx
\begin{bmatrix}
-0.31623 & 23.24519 & -0.59482 & 1.87009
\end{bmatrix}.
$$

Los valores se muestran redondeados para documentarlos.
En Simulink se debe utilizar la variable K calculada con
precisión completa.

Los signos de sus componentes dependen de las convenciones
del modelo. No deben cambiarse de forma aislada: deben ser
coherentes con los ángulos, las conversiones de encoder
y el signo aplicado al comando del motor.

## 7. Estabilidad en lazo cerrado

Al sustituir la ley de control en el modelo:

$$
\dot{x}=Ax+B(-Kx),
$$

se obtiene:

$$
\dot{x}=(A-BK)x.
$$

Por tanto, la matriz del sistema en lazo cerrado es:

$$
A_{\mathrm{cl}}=A-BK.
$$

El script evalúa sus polos mediante:

```matlab
eig(A - B*K)
```

La reproducción numérica da los siguientes resultados:

| Polo | Valor aproximado |
|---|---:|
| λ1 | −14.37682 |
| λ2 | −12.28931 |
| λ3 | −2.89535 |
| λ4 | −1.45793 |

Todos tienen parte real negativa. Por ello, el modelo lineal
continuo con realimentación ideal de estados es asintóticamente
estable.

Esto significa que, sin entradas externas y bajo las hipótesis
del modelo, las desviaciones respecto al equilibrio tienden
a desaparecer.

### Alcance de esta verificación

El análisis de A−BK no incluye explícitamente:

- La dinámica de los dos observadores que proporcionan
  las velocidades estimadas al controlador.
- Los límites de voltaje del equipo.
- El muestreo y los retardos de ejecución.
- La lógica que activa y desactiva el balance.
- Las diferencias entre el modelo y la planta física.

Por tanto, estos polos verifican el diseño ideal del LQR.
La estabilidad y el desempeño de la implementación completa
requieren comprobaciones adicionales y evidencia experimental.

## 8. Correspondencia con el Simulink final

### Formación del vector de realimentación

El subsistema `State X` construye el vector que recibe
el controlador mediante un bloque Mux.

| Entrada del Mux | Señal utilizada | Procedencia |
|---|---|---|
| 1 | Posición angular del brazo, θ | Encoder y conversión a radianes |
| 2 | Posición angular del péndulo, α | Encoder y conversión a radianes |
| 3 | Velocidad estimada del brazo | Salida `dot` de `Observador Theta` |
| 4 | Velocidad estimada del péndulo | Salida `dot` de `Observador Alpha` |

Las posiciones estimadas por los observadores se envían
a `Scope Theta` y `Scope Alpha` para compararlas con las
mediciones. El controlador utiliza las posiciones medidas.

Los filtros de derivación de la versión anterior permanecen
en el archivo, pero sus salidas no alimentan el vector
de realimentación del controlador final.

### Cálculo del comando

La entrada positiva del sumador está conectada a `Ground`.
La entrada negativa recibe el vector de realimentación.
Por tanto, el sumador entrega:

$$
0-x_c=-x_c.
$$

Aunque el bloque de ganancia se llama `u = -K*x`,
su parámetro es `K`. El signo negativo ya se obtiene
en el sumador:

$$
u=K(-x_c)=-Kx_c.
$$

Los ángulos utilizados para este cálculo están en radianes
y las velocidades estimadas en radianes por segundo.

### Habilitación del balance

El ángulo medido del péndulo se convierte a grados
antes de entrar al bloque MATLAB Function que determina
si el controlador debe activarse.

En el archivo final revisado, la condición es:

$$
-15^\circ \leq \alpha_{\mathrm{deg}} \leq 15^\circ.
$$

Cuando se cumple, el selector `Enable Balance Control Switch`
envía el comando calculado por el LQR. Fuera de esa ventana,
envía 0 V.

El péndulo se eleva manualmente hasta la zona de balance;
no se implementa una maniobra automática de swing-up.

El documento del curso y el procedimiento de Quanser
indican una ventana de ±10°. El archivo revisado conserva
±15°; esta diferencia debe resolverse y documentarse
antes de declarar cumplimiento completo de ese requisito.

### Adaptación respecto al laboratorio de Quanser

La estructura de realimentación y la interfaz con el equipo
se basan en el laboratorio LQR de Quanser.

En este proyecto se utiliza una referencia cero del brazo,
las ponderaciones Q y R del script final y las velocidades
estimadas mediante los observadores desarrollados
por Emilio Acuña a partir del esquema del curso.

El archivo final no utiliza una referencia variable del brazo
ni una ganancia de 15 para escalarla.

## 9. Comprobación reproducible en MATLAB

Después de ejecutar el script original para disponer de
A y B, este bloque permite repetir y ampliar las verificaciones:

```matlab
% Ponderaciones presentes en el proyecto
Q = diag([1 20 0.1 0.1]);
R = 10;

% Comprobación adicional de controlabilidad
rango_controlabilidad = rank(ctrb(A, B));

% Diseño LQR continuo
% P_riccati: solución de Riccati
% polos_lqr: polos del lazo cerrado
[K, P_riccati, polos_lqr] = lqr(A, B, Q, R);

% Verificación directa de estabilidad
A_cl = A - B*K;
polos_verificados = eig(A_cl);
estable_modelo_lineal = all(real(polos_verificados) < 0);

disp('Rango de controlabilidad:');
disp(rango_controlabilidad);

disp('Ganancia K:');
disp(K);

disp('Polos de A-BK:');
disp(polos_verificados);

disp('Modelo lineal ideal estable:');
disp(estable_modelo_lineal);
```

Este bloque se propone para reproducir la revisión.
No se presenta como una prueba experimental ya realizada.


## 10. Referencias

- Código del equipo: `ProyectoControlAvanzadoFinalPenduloCode.m`.
- Modelo del equipo: `qs3_lqr_ctrl_simulinkFInal.slx`.
- Instrucciones del proyecto: primera parte de la evaluación.
- [Quanser: guía de control LQR](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control/application_guide_lqr_control.pdf).
- [MathWorks: documentación de lqr](https://www.mathworks.com/help/control/ref/lti.lqr.html).