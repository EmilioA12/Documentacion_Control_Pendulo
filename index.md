---
layout: default
title: Inicio
nav_order: 1
---

# Control de un péndulo invertido
{: .fs-9 }

Quanser Qube-Servo 3 · Control Avanzado y Robótica
{: .fs-6 .fw-300 }

Este portafolio presenta el modelado del péndulo invertido
rotatorio, el diseño del controlador LQR y la implementación
de observadores para estimar las velocidades angulares
utilizadas en la realimentación.

El desarrollo utiliza como base los materiales del laboratorio
de control LQR de Quanser y los requisitos del proyecto
de Control Avanzado y Robótica.

## Objetivo

Mantener el péndulo cerca de su posición vertical superior
y regular la posición angular del brazo alrededor de cero.

El péndulo se eleva manualmente hasta la zona de balance.
El sistema no utiliza un controlador de swing-up.

## Plataforma de trabajo

| Elemento | Descripción |
|---|---|
| Equipo | Quanser Qube-Servo 3 |
| Configuración | Brazo rotatorio con péndulo invertido |
| Actuador | Motor de corriente directa |
| Mediciones | Ángulos del brazo y del péndulo mediante encoders |
| Software | MATLAB, Simulink y QUARC |
| Controlador | Regulador cuadrático lineal, LQR |
| Estimación | Dos observadores: uno para el brazo y otro para el péndulo |
| Referencia | Posición cero del brazo y vertical superior del péndulo |

## Base del desarrollo y aportaciones

Los materiales de Quanser proporcionaron la base para los
parámetros físicos, el procedimiento de control LQR y la
estructura de comunicación con el equipo.

Para este proyecto se implementaron las matrices del modelo,
se establecieron las ponderaciones del LQR y se adaptó
el diagrama de Simulink a la regulación alrededor del origen.

El diseño de ganancias y la implementación del observador
fueron realizados por Emilio Acuña a partir del esquema
solicitado en la figura 3 del documento del curso.

Se implementaron dos copias del observador para estimar
el movimiento del brazo y del péndulo.

## Funcionamiento de la versión final

1. Los encoders proporcionan las posiciones angulares.
2. Las cuentas se convierten a radianes.
3. Cada posición medida alimenta a su observador.
4. Las velocidades estimadas se integran al vector de estados.
5. El LQR calcula el voltaje de control.
6. La lógica de habilitación permite aplicar el control
   dentro de la zona de balance.

El controlador utiliza las posiciones medidas y las
velocidades estimadas. Las posiciones estimadas también
se muestran en Scopes para compararlas con las mediciones.

## Organización del portafolio

1. Modelado y representación en espacio de estados.
2. Diseño LQR y análisis de estabilidad.
3. Diseño y cálculo de ganancias de los observadores.
4. Implementación en Simulink y comunicación mediante QUARC.
5. Resultados experimentales y discusión.
6. Programas, evidencias y referencias.

## Configuración y documentación

La versión final revisada incluye ambos observadores
conectados a las mediciones y al controlador LQR.

El archivo conserva una ventana de activación de ±15°,
mientras que la consigna y el procedimiento de Quanser
indican ±10°. Esta diferencia se documentará en la sección
de implementación junto con su resolución.

Las gráficas y los videos del laboratorio se incorporarán
a la sección de resultados para respaldar el análisis
del funcionamiento físico.

## Equipo de trabajo

Emilio Acuña, Angel y Joel.

## Fuentes y créditos

- [Quanser: recursos del laboratorio LQR](https://github.com/quanser/Quanser_Academic_Resources/tree/dev-windows/6_teaching/1_Controls/Qube_Servo_3/sp6_pendulum_control/1b_lqr_control).
- Documento del curso: Proyecto Práctico de Control Avanzado
  y Robótica, sección del Qube-Servo 3 y figura 3.
- Diseño de ganancias e implementación del observador:
  Emilio Acuña, siguiendo el esquema del curso.
- [Guía de portafolio de Huber Giron](https://hubergiron.github.io/portafolio-just-the-docs/).