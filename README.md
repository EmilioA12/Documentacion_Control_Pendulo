# Péndulo invertido — Quanser Qube-Servo 3

Proyecto de Control Avanzado y Robótica.

## Objetivo

Documentar el modelado y la linealización del péndulo invertido
rotatorio, el diseño de un controlador LQR y el desarrollo del
observador solicitado en el proyecto.

El control de balance se activa después de levantar manualmente
el péndulo cerca de la posición vertical superior. No se utiliza
control de swing-up.

## Alcance de la documentación

- Parámetros físicos y modelo en espacio de estados.
- Diseño LQR y análisis de estabilidad.
- Implementación en MATLAB, Simulink y QUARC.
- Diseño e integración del observador.
- Resultados, discusión y evidencias experimentales.
- Programas descargables y referencias.

## Estado de revisión

El diseño LQR y las ganancias del observador están calculados.

En la versión inicial revisada, el umbral de balance es ±15°,
frente a ±10° solicitado. El observador tiene pendiente su
conexión a la medición y a la realimentación del controlador.

La validación experimental se documentará con los registros
y videos disponibles del laboratorio.

## Equipo

Pendiente de completar.

## Créditos

Plataforma y recursos de referencia: Quanser.

Estructura del portafolio basada en la guía de Huber Giron:
https://hubergiron.github.io/portafolio-just-the-docs/