---
layout: default
title: Observador de estados
nav_order: 4
---

# Diseño e implementación del observador

## 1. Objetivo y aportación

El controlador LQR utiliza las posiciones y velocidades
angulares del brazo y del péndulo.

Las posiciones se obtienen mediante los encoders.
Para estimar las velocidades se implementaron dos
observadores: uno para el brazo y otro para el péndulo.

El diseño de ganancias y la implementación en Simulink
fueron realizados por Emilio Acuña a partir del esquema
solicitado en la figura 3 del documento del curso.

La base de control LQR y comunicación con el equipo
procede del material de Quanser. La incorporación de estos
observadores corresponde al desarrollo del proyecto.

## 2. Señales de entrada y salida

Cada observador recibe únicamente el ángulo medido.
No recibe el voltaje aplicado al motor.

| Observador | Entrada | Salidas |
|---|---|---|
| Observador Theta | Posición medida del brazo, θ | Posición y velocidad estimadas del brazo |
| Observador Alpha | Posición medida del péndulo, α | Posición y velocidad estimadas del péndulo |

Las dos copias utilizan la misma estructura y las mismas
ganancias, pero mantienen estados internos independientes.

Para desarrollar las ecuaciones se utiliza una notación
general:

| Símbolo | Significado |
|---|---|
| y | Ángulo medido: θ o α |
| p̂ | Posición angular estimada |
| v̂ | Velocidad angular estimada |
| q | Estado del integrador de la rama auxiliar |
| L, m, β | Ganancias del observador |

La ganancia L se escribe con mayúscula para distinguirla
de la distancia física l = Lp/2 del modelo del péndulo.
En la figura del profesor aparece como l minúscula.

## 3. Obtención de las ecuaciones

El primer sumador compara la posición medida con la estimada:

$$
\varepsilon=y-\hat p.
$$

Esta diferencia alimenta dos ramas.

### Rama de estimación de velocidad y posición

La señal de entrada al integrador de velocidad es
la suma de Lε y mq:

$$
\dot{\hat v}=L\varepsilon+mq.
$$

La salida de ese integrador es la velocidad estimada.
Un segundo integrador proporciona la posición estimada:

$$
\dot{\hat p}=\hat v.
$$

### Rama auxiliar

En esta rama se resta βq al error de posición:

$$
\dot q=\varepsilon-\beta q.
$$

El resultado se integra para obtener q, que se realimenta
a través de las ganancias β y m.

### Ecuaciones completas

Sustituyendo ε = y − p̂:

$$
\boxed{
\begin{aligned}
\dot{\hat p}&=\hat v,\\
\dot{\hat v}&=L(y-\hat p)+mq,\\
\dot q&=y-\hat p-\beta q.
\end{aligned}
}
$$

Estas ecuaciones corresponden a los sumadores,
integradores y ganancias implementados en Simulink.

## 4. Representación en espacio de estados

Se define el vector interno:

$$
z=
\begin{bmatrix}
\hat p\\
\hat v\\
q
\end{bmatrix}.
$$

Las ecuaciones anteriores pueden escribirse como:

$$
\dot z=A_o z+B_o y,
$$

donde:

$$
A_o=
\begin{bmatrix}
0&1&0\\
-L&0&m\\
-1&0&-\beta
\end{bmatrix},
\qquad
B_o=
\begin{bmatrix}
0\\
L\\
1
\end{bmatrix}.
$$

Ao describe la dinámica interna del observador.
Bo representa cómo entra la posición medida.

Estas matrices corresponden al observador y son distintas
de las matrices A y B de la planta.

## 5. Polinomio característico

Para seleccionar las ganancias se calcula:

$$
P_o(s)=\det(sI-A_o).
$$

La matriz del determinante es:

$$
sI-A_o=
\begin{bmatrix}
s&-1&0\\
L&s&-m\\
1&0&s+\beta
\end{bmatrix}.
$$

Al desarrollar por la primera fila:

$$
P_o(s)=s^2(s+\beta)+L(s+\beta)+m.
$$

Por tanto:

$$
\boxed{
P_o(s)=s^3+\beta s^2+Ls+(L\beta+m).
}
$$

Esta expresión permite relacionar las ganancias con
la ubicación deseada de los polos.

## 6. Selección de polos y cálculo de ganancias

Se seleccionan tres polos nominales iguales en −a:

$$
P_d(s)=(s+a)^3.
$$

Al expandir:

$$
P_d(s)=s^3+3as^2+3a^2s+a^3.
$$

Igualando coeficientes con el polinomio del observador:

| Coeficiente | Igualdad |
|---|---|
| s² | β = 3a |
| s | L = 3a² |
| Término independiente | Lβ + m = a³ |

Se obtiene:

$$
\beta=3a,
\qquad
L=3a^2,
\qquad
m=a^3-L\beta.
$$

En el código del proyecto se utiliza:

$$
a=100\ \mathrm{s}^{-1}.
$$

Por tanto:

$$
\boxed{\beta=300}
$$

$$
\boxed{L=30\,000}
$$

$$
\boxed{
m=100^3-(30\,000)(300)=-8\,000\,000.
}
$$

El signo negativo de m es resultado de la igualación
de coeficientes. No representa por sí mismo una
inestabilidad del observador.

Las unidades consistentes son β en s⁻¹, L en s⁻²
y m en s⁻³.

## 7. Verificación de la dinámica interna

Sustituyendo las ganancias:

$$
P_o(s)=s^3+300s^2+30\,000s+1\,000\,000.
$$

Este polinomio cumple:

$$
P_o(s)=(s+100)^3.
$$

Así, los polos nominales son:

$$
\boxed{s_1=s_2=s_3=-100.}
$$

Todos tienen parte real negativa, por lo que la dinámica
interna homogénea es asintóticamente estable.

El siguiente bloque reproduce numéricamente el cálculo:

```matlab
a = 100;

beta = 3*a;
L = 3*a^2;
m = a^3 - L*beta;

Aobs = [0 1 0;
       -L 0 m;
       -1 0 -beta];

Bobs = [0; L; 1];

disp('Ganancias [L beta m]:');
disp([L beta m]);

disp('Polos del observador:');
disp(eig(Aobs));
```

Este bloque es una presentación numérica equivalente del
diseño. Define Bobs con la ganancia L ya calculada.
En Simulink las ecuaciones se implementan mediante bloques,
sin utilizar directamente la variable Bobs.

Al calcular los autovalores pueden aparecer pequeñas
diferencias numéricas alrededor de −100 debido a que
se trata de una raíz triple.

El parámetro a determina la ubicación de los polos.
Su elección debe evaluarse considerando rapidez de
estimación, ruido de medición y paso de ejecución.

## 8. Ecuaciones de cada observador

### Observador del brazo

La entrada es la posición medida θ:

$$
\begin{aligned}
\dot{\hat\theta}&=\hat v_\theta,\\
\dot{\hat v}_\theta
&=L(\theta-\hat\theta)+mq_\theta,\\
\dot q_\theta
&=\theta-\hat\theta-\beta q_\theta.
\end{aligned}
$$

La señal de velocidad utilizada por el controlador es:

$$
\widehat{\dot\theta}=\hat v_\theta.
$$

### Observador del péndulo

La entrada es la posición medida α:

$$
\begin{aligned}
\dot{\hat\alpha}&=\hat v_\alpha,\\
\dot{\hat v}_\alpha
&=L(\alpha-\hat\alpha)+mq_\alpha,\\
\dot q_\alpha
&=\alpha-\hat\alpha-\beta q_\alpha.
\end{aligned}
$$

La señal de velocidad utilizada por el controlador es:

$$
\widehat{\dot\alpha}=\hat v_\alpha.
$$

## 9. Integración con el controlador LQR

Los observadores se encuentran dentro de:

`Qube With Pendulum → State X`

Las conexiones finales son:

| Señal | Destino |
|---|---|
| θ medida | Entrada 1 del Mux de estados |
| α medida | Entrada 2 del Mux de estados |
| Salida `dot` de Observador Theta | Entrada 3 del Mux de estados |
| Salida `dot` de Observador Alpha | Entrada 4 del Mux de estados |
| θ estimada | Scope Theta |
| α estimada | Scope Alpha |

El vector recibido por el controlador es:

$$
x_c=
\begin{bmatrix}
\theta\\
\alpha\\
\widehat{\dot\theta}\\
\widehat{\dot\alpha}
\end{bmatrix}.
$$

Con referencia cero, durante el balance se aplica:

$$
u=-Kx_c.
$$

Las posiciones estimadas se comparan con las mediciones
en los Scopes. El controlador conserva las posiciones
medidas y utiliza las velocidades estimadas.

Los filtros de derivación de la versión inicial permanecen
en el archivo, pero sus salidas no alimentan al LQR final.

## 10. Error de estimación y alcance de la estabilidad

Los polos negativos de Ao demuestran estabilidad de
la dinámica interna. Para estudiar el error respecto
a la señal medida se definen:

$$
e_p=\hat p-y,
\qquad
e_v=\hat v-\dot y.
$$

A partir de las ecuaciones del observador:

$$
\dot e_p=e_v,
$$

$$
\dot e_v=-Le_p+mq-\ddot y,
$$

$$
\dot q=-e_p-\beta q.
$$

En forma matricial:

$$
\frac{d}{dt}
\begin{bmatrix}
e_p\\
e_v\\
q
\end{bmatrix}
=
A_o
\begin{bmatrix}
e_p\\
e_v\\
q
\end{bmatrix}
+
\begin{bmatrix}
0\\
-\ddot y\\
0
\end{bmatrix}.
$$

Esta expresión muestra dos aspectos:

- La parte homogénea del error es estable.
- La aceleración de la señal medida excita la dinámica
  del error durante el movimiento.

Para una señal constante o una rampa ideal, la aceleración
es cero y los errores tienden a cero bajo las hipótesis
del modelo continuo.

Para movimientos generales, ruido o cambios rápidos,
puede existir error dinámico. Por ello, los polos negativos
no garantizan estimación exacta para cualquier trayectoria.

El estado auxiliar q participa en la corrección del
observador. En esta implementación no se define una
salida calibrada de torque perturbador; no debe
interpretarse directamente como una medición de
perturbación física.

## 11. Validación experimental

La comparación de posiciones se realiza mediante:

$$
e_\theta=\hat\theta-\theta,
\qquad
e_\alpha=\hat\alpha-\alpha.
$$

Para documentar el desempeño se incorporarán:

- Gráfica de θ medida y θ estimada.
- Gráfica de α medida y α estimada.
- Gráficas de los errores de posición.
- Gráficas de las velocidades estimadas.
- Condiciones y duración de las pruebas.
- Relación entre la estimación y el comportamiento
  del péndulo durante el balance.

Las posiciones y sus estimaciones deben compararse
con las mismas unidades y el mismo eje de tiempo.

Comparar la posición estimada con el encoder evalúa
el seguimiento de la medición. No proporciona por sí
solo una validación independiente de la velocidad.

Las métricas experimentales se calcularán sobre los
registros del laboratorio y se presentarán en la
sección de resultados.

## 12. Fuentes y autoría

- Esquema solicitado: figura 3 del documento del curso,
  Proyecto Práctico de Control Avanzado y Robótica.
- Diseño de ganancias e implementación del observador:
  Emilio Acuña.
- Código final: `ProyectoControlAvanzadoFinalPenduloCode.m`.
- Modelo final: `qs3_lqr_ctrl_simulinkFInal.slx`.
- Base de control e interfaz:
  [laboratorio LQR de Quanser](https://github.com/quanser/Quanser_Academic_Resources/tree/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control).

El desarrollo presentado explica las ecuaciones y las
ganancias utilizadas en los observadores del modelo final.