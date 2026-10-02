---
layout: default
title: Inicio
nav_order: 1
---

# Control de un péndulo invertido
{: .fs-9 }

Quanser Qube-Servo 3 · Control Avanzado y Robótica
{: .fs-6 .fw-300 }

Este portafolio documenta el modelado del péndulo invertido
rotatorio, el diseño de un controlador LQR y el desarrollo
del observador solicitado en el proyecto.

## Objetivo

Mantener el péndulo cerca de su posición vertical superior
mediante realimentación de estados y documentar el diseño,
la implementación y la validación del sistema.

El péndulo se levanta manualmente hasta la zona de balance.
El proyecto no utiliza un controlador de swing-up.

## Plataforma de trabajo

| Elemento | Descripción |
|---|---|
| Equipo | Quanser Qube-Servo 3 |
| Configuración | Brazo rotatorio con péndulo invertido |
| Actuador | Motor de corriente directa |
| Mediciones | Posiciones angulares del brazo y del péndulo mediante encoders |
| Software | MATLAB, Simulink y QUARC |
| Control | Regulador cuadrático lineal, LQR |
| Estimación | Observador basado en la figura 3 de las instrucciones del proyecto |

## Recorrido del proyecto

1. Parámetros físicos, modelado y linealización.
2. Diseño LQR y análisis de estabilidad.
3. Implementación en Simulink y comunicación con el equipo.
4. Diseño e integración del observador.
5. Resultados experimentales y análisis de errores.
6. Discusión, conclusiones y evidencias.
7. Programas descargables y referencias.

## Estado de la documentación

El diseño matemático del LQR y las ganancias del observador
se encuentran calculados.

En el modelo inicialmente revisado, el balance se activa
dentro de ±15°, mientras que las instrucciones solicitan ±10°.
El observador aparece sin conexión a la medición ni al LQR.

Las correcciones y las pruebas se documentarán conforme se
realicen. Los resultados experimentales se respaldarán con
registros, capturas y videos del laboratorio.

## Equipo de trabajo

Integrantes: Emilio Acuña, Angel, Joel.

## Fuentes y créditos

- Plataforma y recursos técnicos:
  [Quanser](https://www.quanser.com/products/qube-servo-3/).
- Estructura de documentación:
  [guía de Huber Giron](https://hubergiron.github.io/portafolio-just-the-docs/).
- Requisitos y observador: instrucciones del proyecto de
  Control Avanzado y Robótica.