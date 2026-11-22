# Sistema de Control de Lazo Cerrado para Semáforo Adaptativo (IoT / Control PD)

**Universidad Tecnológica Nacional — Facultad Regional Buenos Aires (UTN FRBA)**  
**Departamento de Ingeniería en Sistemas de Información (DISI)**  
**Cátedra:** Teoría de Control / Tecnologías para la Automatización — **Curso K-4572**  
**Docente:** Drando. Omar Oscar Civale  

**Integrantes del Equipo:**
* **Ezequiel Reichel** — Legajo: `176.353-2`
* **Ignacio Rodríguez** — Legajo: `214.074-3`

---

## Acceso Directo al Simulador Web (GitHub Pages)

Para facilitar la evaluación sin necesidad de instalar entornos, compiladores ni dependencias externas, el simulador se encuentra desplegado y ejecutable directamente desde el navegador a través de **GitHub Pages**:

> **Enlace de ejecución en vivo:** `https://<tu-usuario>.github.io/<nombre-del-repositorio>/`  
> *(Nota: Reemplazar con la URL generada en Settings $\rightarrow$ Pages de este repositorio).*

El sistema corre íntegramente del lado del cliente en tiempo real a $60\text{ FPS}$, integrando la animación física de la intersección vial, un tablero de control convencional, el diagrama de bloques con telemetría instantánea y un osciloscopio de **6 señales sincronizadas**.

---

## 1. Fundamentación desde la Teoría de Control

El proyecto modela una intersección urbana semaforizada inspirada en los sistemas adaptativos reales implementados en la Ciudad Autónoma de Buenos Aires (como **SCATS / SCOOT** en Av. del Libertador), contrastando el paradigma tradicional de **Lazo Abierto** (tiempos fijos) frente a un **Sistema de Control en Lazo Cerrado**.

### Identificación de la Estructura del Sistema
* **Entrada de Referencia $R(t)$:** Función escalón constante (*Setpoint*) que establece la longitud máxima admisible de vehículos esperando en cola (valor nominal por defecto: $R(t) = 15\text{ autos}$).
* **Elemento de Comparación (Punto Suma):** Nodo lógico en el microcontrolador que calcula la desviación instantánea:
  $$E(t) = C(t) - R(t)$$
  Incorpora una **zona de normalidad / banda de histéresis** de $\pm 5\text{ autos}$ para evitar conmutaciones erráticas ante fluctuaciones mínimas del tránsito.
* **Controlador $G_c(s)$:** Algoritmo **Proporcional-Derivativo (PD)** operando sobre el microcontrolador local:
  $$U(t) = U_{\text{base}} + K_p \cdot E(t) + K_d \cdot \frac{dE(t)}{dt}$$
  El término proporcional ($K_p$) ajusta la duración del verde en relación directa con el exceso de vehículos, mientras que el término derivativo dinámico ($K_d$) aporta acción anticipatoria frente a variaciones abruptas en la tasa de llegada de vehículos.
* **Actuador $G_a(s)$:** Relés de estado sólido que conmutan las luminarias LED del semáforo, ejecutando la variable manipulada $U(t)$ (duración de luz verde acotada por seguridad vial entre $15\text{ s}$ y $60\text{ s}$).
* **Planta / Proceso $G_p(s)$:** La intersección vehicular y su dinámica física de acumulación y evacuación de vehículos durante cada ciclo semafórico.
* **Elemento de Medición / Transductor $H(s)$:** Magnetómetros inductivos embebidos bajo la calzada. Aprovechando la **relación de solidaridad electromagnética**, la masa metálica de cada vehículo altera la inductancia de la espira, generando pulsos digitales discretos que realimentan la longitud real de la cola $C(t)$.
* **Perturbaciones $D(t)$:**
  * *Externas:* Oleadas repentinas de caudal vehicular (*Hora Pico*) y degradación del coeficiente de evacuación por condiciones climáticas adversas o incidentes (*Lluvia / Accidente*).
  * *Internas:* Deriva térmica en la sensibilidad de los sensores de calzada y pérdida de visibilidad por suciedad en las ópticas LED.

---

## 2. Vinculación con IoT, Red Local y Problemática de Conectividad

En una arquitectura de **Movilidad Urbana Inteligente (Smart Cities)**, cada gabinete semafórico de vereda constituye un nodo **IoT** conectado mediante fibra óptica o enlaces celulares 4G/5G con el Centro de Control de Tránsito de la Ciudad para monitoreo remoto y sincronización de onda verde.

### Problemática de la Conexión a Internet vs. Control en Red Local (Shell del Microcontrolador)
1. **Riesgo de dependencia de la nube (WAN):** Ejecutar el lazo de control cerrado en un servidor remoto a través de Internet introduce latencia no determinística, pérdida de paquetes y vulnerabilidad ante cortes de conectividad. Si se interrumpe Internet, el semáforo perdería capacidad de respuesta o comprometería los tiempos de seguridad vial.
2. **Solución de Arquitectura Edge / Red Local:** El lazo crítico en tiempo real se ejecuta **localmente dentro de la shell / firmware del microcontrolador (Arduino / PLC industrial)** alojado en el gabinete IP65 de la esquina, comunicado por bus cableado directo con los magnetómetros y actuadores de la red local. La conexión a Internet se reserva únicamente para telemetría supervisoria y ajuste de consignas globales.
3. **Verificación en el Simulador:** El tablero incluye el interruptor **"Simular Corte de Internet (Modo Edge Local)"**. Al accionarlo, se interrumpe el enlace con la nube (`WAN OFFLINE`), evidenciando que el controlador local continúa regulando la intersección de forma autónoma y estable mientras almacena las tramas en un buffer local hasta el restablecimiento del enlace.

---

## 3. Representación de las 6 Señales en Tiempo Real

El osciloscopio multicanal del simulador grafica simultáneamente la evolución temporal (ciclo a ciclo) de las 6 variables fundamentales del sistema:

| # | Señal | Símbolo | Unidad | Descripción Funcional en el Lazo |
| :-: | :--- | :---: | :---: | :--- |
| **1** | **Referencia (Setpoint)** | $R(t)$ | $\text{autos}$ | Consigna escalón que fija el umbral máximo deseable de vehículos en cola. |
| **2** | **Variable Controlada (Medida)** | $C(t)$ | $\text{autos}$ | Salida real de la planta medida por los magnetómetros. Identifica visualmente mediante franjas de color el **Estado Estable**, el **Estado Transitorio** y la **Saturación**. |
| **3** | **Perturbación Externa** | $D(t)$ | $\text{autos/ciclo}$ | Registra el instante y la magnitud de las alteraciones exógenas inyectadas (*Hora Pico* o *Lluvia/Accidente*). |
| **4** | **Señal de Error** | $E(t)$ | $\text{autos}$ | Diferencia $E(t) = C(t) - R(t)$ a la salida del comparador, destacando la banda de normalidad ($\pm 5\text{ autos}$). |
| **5** | **Derivativo Dinámico** | $U_d(t)$ | $\text{segundos}$ | Aporte anticipatorio $K_d \cdot \frac{dE(t)}{dt}$ que reacciona ante la velocidad de crecimiento de la fila. |
| **6** | **Señal de Control (Actuación)** | $U(t)$ | $\text{segundos}$ | Orden de mando enviada al actuador (duración efectiva de luz verde entre $15\text{ s}$ y $60\text{ s}$). |

---

## 4. Tablero de Control Convencional y Funcionalidades

El panel lateral izquierdo replica una consola de operación industrial permitiendo intervenir el sistema en tiempo real:

* **Selector de Ley de Control ($G_c$) con Habilitación Dinámica de Ganancias:** Permite conmutar entre **Lazo Abierto**, **Control P**, **Control PD** (configuración principal del TFI), **Control PI** y **Control PID**. La interfaz habilita o bloquea automáticamente los controles de $K_p$, $K_d$ y $K_i$ según la arquitectura elegida.
* **Ajuste Paramétrico en Vivo:** Deslizadores y campos numéricos sincronizados para modificar el *Setpoint* $R(t)$, el *Tiempo Base de Luz Verde*, las constantes $K_p$, $K_d$, $K_i$ y el *Flujo Vehicular Base*.
* **Generador de Perturbaciones:** Botones dedicados para inyectar una oleada de **Hora Pico ($+25\text{ autos/ciclo}$)** o condiciones de **Lluvia / Accidente** (reducción de la capacidad de evacuación de la calzada).
* **Reinicio Completo del Sistema (`Reiniciar Sistema`):** Restablece el tiempo de simulación a $t = 0$, limpia los historiales de las 6 gráficas, vacía las colas excedentes y pone a cero los acumuladores derivativos e integrales para iniciar ensayos desde condiciones iniciales limpias.
* **Exportación de Telemetría (`Descargar Datos CSV`):** Genera un archivo `.csv` con el registro histórico completo de las 6 señales, el régimen de respuesta y el estado de red IoT para su posterior análisis cuantitativo.

---

## 5. Protocolo Sugerido de Ensayo (Guía de Demostración)

Para verificar todas las capacidades del modelo en menos de un minuto desde el enlace de **GitHub Pages**:

1. **Verificación de Estado Estable:** Con la configuración por defecto (**Controlador PD**, $R(t) = 15\text{ autos}$, Tiempo Base $= 30\text{ s}$), observar que el indicador superior permanece en **ESTADO ESTABLE** y el error $E(t)$ se mantiene acotado dentro de la banda de normalidad.
2. **Respuesta Transitoria y Acción Derivativa:** Presionar **"Perturbación: Hora Pico (+25 autos)"**. Observar cómo el sistema pasa a **ESTADO TRANSITORIO**, la señal **Derivativo Dinámico $U_d(t)$** genera un impulso preventivo inmediato y la actuación **$U(t)$** extiende la luz verde hasta $50\text{ s}\text{--}60\text{ s}$ hasta absorber la congestión y retornar al estado estable.
3. **Resiliencia IoT ante Corte de Internet:** Activar **"Simular Corte de Internet (Modo Edge Local)"** y volver a inyectar una perturbación. Verificar que, pese a la desconexión con la nube (`WAN OFFLINE`), el control local PD sigue estabilizando la intersección sin degradación.
4. **Comparativa contra Lazo Abierto (Inestabilidad):** Presionar **"Reiniciar Sistema"**, seleccionar **Lazo Abierto (Semáforo Fijo)** y aplicar **Hora Pico**. Se comprobará que, al no existir realimentación, $U(t)$ permanece clavado en $30\text{ s}$ y la cola crece sin límite llevando el sistema a **SATURACIÓN / INESTABILIDAD**.

---

## 6. Estructura del Repositorio

```text
├── index.html   # Aplicación web autocontenida (Interfaz HMI, motor físico-matemático y osciloscopios Canvas 2D)
└── README.md    # Documentación técnica y guía de evaluación para la cátedra
```