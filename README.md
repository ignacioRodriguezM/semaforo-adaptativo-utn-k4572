# Simulador en Tiempo Real: Semáforo Adaptativo de Lazo Cerrado (IoT / Control PD)

**Universidad Tecnológica Nacional — Facultad Regional Buenos Aires (UTN FRBA)**  
**Cátedra:** Teoría de Control — Curso K-4572  
**Profesor:** Mgtr. Ing. Omar Oscar Civale  
**Integrantes:**  
* Ezequiel Reichel (Legajo: 176.353-2)  
* Ignacio Rodríguez (Legajo: 214.074-3)  

---

## 1. Descripción del Proyecto

Este repositorio contiene el simulador interactivo en tiempo real desarrollado para el **Trabajo Práctico Integrador** de Teoría de Control. El sistema modela una intersección urbana semaforizada operada mediante un **Sistema de Control en Lazo Cerrado**, comparando su desempeño frente a un semáforo tradicional de tiempos fijos (**Lazo Abierto**).

La simulación integra la dinámica física del tránsito, la detección mediante magnetómetros embebidos en la calzada, un tablero de control convencional para parametrización en vivo, la inyección de perturbaciones externas/internas, el modo de resiliencia local ante cortes de Internet (arquitectura IoT / Edge Computing) y la graficación simultánea de las **6 señales fundamentales del lazo de control**.

---

## 2. Requisitos y Cómo Ejecutar la Simulación

El simulador está desarrollado en **HTML5, CSS3 y JavaScript (ES6+)** puro utilizando la API nativa `Canvas 2D`, sin dependencias ni librerías externas que requieran instalación.

### Opción A: Ejecución directa en navegador (Recomendada)
1. Descargar o clonar este repositorio:
   ```bash
   git clone [https://github.com/](https://github.com/)<tu-usuario>/semaforo-adaptativo-utn-k4572.git
