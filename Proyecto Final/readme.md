# 🫀 Sistema uFISIO: Plataforma de Ensayos y Validación Metrológica (VOP / PWV)

![Estado del Proyecto](https://img.shields.io/badge/Estado-En_Desarrollo_%2F_Validaci%C3%B3n-blue)
![Hardware](https://img.shields.io/badge/MCU-STM32F103%2F104_(Blue_Pill)-green)
![Institución](https://img.shields.io/badge/UTN.BA-GIBIO_%2F_CIBIO-orange)

Este repositorio/proyecto integra el desarrollo de la **Plataforma Integral de Ensayos para el Sistema uFISIO** y el marco de **Validación Metrológica y Clínica** para la medición no invasiva de la Velocidad de la Onda del Pulso Arterial ($VOP$ o $PWV$, por sus siglas en inglés), desarrollado en el ámbito del Grupo/Centro de Investigación en Ingeniería Biomédica (**GIBIO / CIBIO**) y la cátedra de **Medidas Electrónicas I** de la **Universidad Tecnológica Nacional - Facultad Regional Buenos Aires (UTN.BA)**.

---

## 📋 Tabla de Contenidos
1. [Introducción y Marco Teórico](#-1-introducción-y-marco-teórico)
2. [Arquitectura de Hardware](#-2-arquitectura-de-hardware)
3. [Arquitectura de Firmware y RTOS](#-3-arquitectura-de-firmware-y-rtos)
4. [Protocolo de Comunicación](#-4-protocolo-de-comunicación)
5. [Configuración de la Placa (Jumpers y Pines)](#-5-configuración-de-la-placa-jumpers-y-pines)
6. [Plan de Validación Metrológica y Clínica (MEI)](#-6-plan-de-validación-metrológica-y-clínica-mei)
7. [Estructura del Proyecto](#-7-estructura-del-proyecto)
8. [Clientes de Prueba y Software](#-8-clientes-de-prueba-y-software)
9. [Referencias y Normativa](#-9-referencias-y-normativa)

---

## 🩺 1. Introducción y Marco Teórico

Las enfermedades cardiovasculares (ECV) constituyen la principal causa de muerte a nivel mundial. Un biomarcador no invasivo fundamental para evaluar el daño arterial temprano y el riesgo cardiovascular es la **rigidez arterial**, la cual se estima de forma indirecta a través de la **Velocidad de Onda de Pulso ($VOP$ / $PWV$)**.

### Principio Físico y Fisiológico
La relación entre la rigidez de la pared arterial y la velocidad de propagación de la onda de presión está gobernada por la ecuación de **Moens-Korteweg** y la ecuación de **Bramwell-Hill**:

$$PWV = \sqrt{\frac{E_{inc} \cdot h}{\rho \cdot d}} = \sqrt{\frac{\Delta P \cdot V}{\Delta V \cdot \rho}} = \sqrt{\frac{1}{\chi \cdot \rho}}$$

Donde:
* $E_{inc}$: Módulo elástico incremental de Young de la pared arterial.
* $h$: Espesor de la pared arterial.
* $d$: Diámetro del vaso.
* $\rho$: Densidad de la sangre.
* $\chi$: Distensibilidad volumétrica.

### Cálculo Clínico Estándar (Gold Standard)
El estándar de oro (*gold-standard*) aceptado por las guías internacionales (ESC/ESH) para la evaluación no invasiva es la **VOP carótido-femoral ($VOP_{CF}$ o $cfPWV$)**. Se determina registrando simultáneamente (o secuencialmente sincronizado con ECG) la onda de pulso en la arteria carótida (proximal) y en la arteria femoral (distal).

utilizando el método del "pie a pie" (*foot-to-foot*) y la corrección anatómica del $80\%$ sobre la distancia directa superficial:

$$VOP_{CF} = 0.8 \cdot \frac{D}{\Delta t} \quad [\text{m/s}]$$

* $D$: Distancia directa superficial medida entre el sitio carotídeo y femoral ($\text{m}$).
* $\Delta t$: Desfasaje temporal (*Transit Time*, $TT$) entre los pies de onda (ascenso sistólico) de ambas señales ($\text{s}$).

Además, la plataforma calcula el **Factor de Forma ($FF$)** arterial a partir de la presión arterial media ($MAP$), sistólica ($SBP$) y diastólica ($DBP$):

$$FF = \frac{MAP - DBP}{SBP - DBP}$$

---

## 🛠️ 2. Arquitectura de Hardware

La plataforma fue diseñada en **Altium Designer** para dotar al sistema uFISIO de versatilidad experimental, permitiendo adquirir señales mediante sensores tonométricos/piezoeléctricos o fotopletismográficos (PPG).

* **Microcontrolador:** Módulo *Blue Pill* (STM32F103C8T6 / STM32F104C8T6) corriendo a $72\text{ MHz}$.
* **Adquisición Analógica de Alta Precisión (SPI1):** Conversor ADC de 24 bits **ADS1192** (específico para biopotenciales/ECG y sensores tonométricos piezoeléctricos), comandado por interrupción externa (`DRDY`).
* **Fotopletismografía - PPG (I2C1):** Sensor óptico **MAX30102 / MAX30100** (reflexión de luz roja e infrarroja), comandado por interrupción externa.
* **Medición de Distancia (ADC1 - Canal 0):** Entrada analógica (`PA0`) asociada a un potenciómetro multivuelta (simulando un sistema de cordón retráctil para medir la distancia carótida-femoral entre $20\text{ cm}$ y $120\text{ cm}$).
* **Visualización Local (I2C1 con DMA):** Pantalla **OLED de 1.3" ($128 \times 64\text{ px}$)** que muestra en tiempo real la frecuencia cardíaca ($\text{lpm}$), $VOP$, $\Delta t$, factor de forma ($FF$) proximal/distal y las curvas capturadas.
* **Comunicaciones Inalámbricas y Seriales:**
  * **Bluetooth:** Módulo **HC-05** conectado a `UART2` ($115200\text{ bps}$) mediante matriz de jumpers (`CN1`) que permite aislarlo para conexión directa por USB-TTL o configuración AT.
  * **Wi-Fi:** Interfaz preparada para módulo **ESP-01 (ESP8266)** en `UART1` ($115200\text{ bps}$).
* **Alimentación:** Fuente lineal regulada de $3.3\text{ V}$ basada en el **AIC1117**.

---

## 💻 3. Arquitectura de Firmware y RTOS

El firmware cuenta con un **Sistema Operativo de Tiempo Real (RTOS) propio** inspirado en conceptos de FreeRTOS, optimizado para bajo consumo de recursos y determinismo en el muestreo.

* **System Tick:** Configurado a **$1\text{ ms}$ ($1\text{ kHz}$)** mediante `SysTick`, garantizando una base de tiempo estricta para la adquisición de muestras.
* **Cambio de Contexto:** Implementado en lenguaje ensamblador (`context_switching.s`) utilizando la interrupción `PendSV`.
* **Esquema de Tareas Híbrido:**
  1. **Tareas Apropiativas (Preemptive):** Se ejecutan obligatoriamente en cada ciclo de reloj y finalizan con un `return`. No admiten bloqueos ni demoras activas:
     * `Lectura_Presion()`: Adquisición desde ADS1192 o señal de prueba en memoria.
     * `Lectura_SpO2()`: Adquisición desde el sensor MAX3010x.
     * `Lectura_Distancia()`: Muestreo del ADC interno con filtro de media móvil (50 muestras).
  2. **Tareas No Apropiativas (Cooperativas / Background):** Se ejecutan en el tiempo excedente del CPU dentro de un bucle `while(1)` infinito, siendo desalojadas por el *scheduler* cuando ocurre un Tick:
     * `Tarea de Análisis de Datos`: Detecta el pie de onda (ascenso sistólico) y el pico sistólico en un buffer circular de memoria compartida (`SHARED_MEMORY_SIZE = 2048`), calculando ritmo cardíaco, factor de forma y el desfasaje temporal ($\Delta t$).
     * `Tarea de Visualización`: Renderiza el buffer de video y lo transfiere a la pantalla OLED vía **DMA** para no penalizar el uso de CPU.
* **Estadísticas de Uso de CPU:** El *scheduler* monitorea los tiempos de ejecución mejores, peores y acumulados de cada tarea.

### Uso de Memoria (STM32)
* **RAM:** $16.29\text{ KB} / 20\text{ KB}$ ($81.45\%$)
* **FLASH:** $62.21\text{ KB} / 64\text{ KB}$ ($97.21\%$)

---

## 📡 4. Protocolo de Comunicación

El sistema implementa soporte para el protocolo heredado (**v2**) y un nuevo **Protocolo Universal (v3)** con control de integridad y detección de pérdida de paquetes. Para optimizar el canal serie, las señales de presión y PPG se empaquetan en bloques de **32 muestras**.

### Estructura de la Trama (Protocolo v3)
| Campo | SYNC (4 bytes) | COMANDO | NONCE | CHECKSUM | PAYLOAD LENGTH | PAYLOAD |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Tipo / Valor** | `0x23 0x23 0x13 0x10` | `uint32_t` | `uint32_t` | `uint16_t` | `uint16_t` | *Custom Data* |

* **COMANDO (`CMD_TYPES`):**
  * `0`: `BAD_CMD` (Descartar)
  * `1`: `ACK`
  * `2`: `UFISIO_PRESSURE_DATA` (Bloque de 32 muestras carótida + femoral)
  * `3`: `UFISIO_DISTANCE` (Distancia medida por el ADC)
  * `4`: `UFISIO_SPO2_DATA` (Bloque de 32 muestras Rojo + IR)
  * `5`: `UFISIO_VOP_HBR` ($VOP$ calculada y período cardíaco en cada latido)
  * `6`: `UFISIO_CPU_REPORT` (Estadísticas de tiempos de ejecución del RTOS)
* **NONCE:** Contador incremental de paquetes (inicia en 1) para auditar paquetes perdidos.
* **CHECKSUM:** Suma byte a byte en `uint16_t` calculada exclusivamente sobre el *Payload*.

---

## ⚙️ 5. Configuración de la Placa (Jumpers y Pines)

La plataforma dispone de 4 entradas digitales (`PB12` a `PB15`) con resistencias de *pull-up* internas que permiten configurar el modo de operación en caliente colocando jumpers a `GND`:

| Pin | Nombre en Código | Sin Jumper (Alto) | Con Jumper a GND (Bajo) |
| :---: | :--- | :--- | :--- |
| **PB12** | `PROTOCOL_SELECTION_PIN` | Protocolo v2 (Solo presión) | **Protocolo v3 (Completo)** |
| **PB13** | `SIGNAL_SOURCE_PIN` | Señal de prueba en memoria Flash | **Datos reales del ADS1192 (SPI)** |
| **PB14** | `INTERFACE_SELECTION_PIN` | **Bluetooth / Serie (`UART2`)** | Wi-Fi ESP8266 (`UART1`) |
| **PB15** | `GEN_INPUT_4` | Modo sensor PPG (MAX3010x) | **Modo Display OLED activo** |

### Indicadores LED
* **LED Rojo (Blue Pill):** Presencia de alimentación.
* **LED Verde (`PC13`):** Destella en caso de error crítico del Sistema Operativo.
* **LED 1 (`PA12` - Rojo en PCB):** Destella indicando la detección de un latido cardíaco.
* **LED 2 (`PA11` - Rojo en PCB):** Disponible para propósito general.

---

## 🔬 6. Plan de Validación Metrológica y Clínica (MEI)

Para que el sistema uFISIO sea transferido a entornos clínicos (Unidad de Prevención Cardiovascular del **CIBIO**), debe superar el protocolo de validación metrológica establecido por la cátedra de **Medidas Electrónicas I**:

### Etapas de Validación Requeridas
1. **Validación Funcional:** Verificación de adquisición, sincronización inter-canal, detección de pie de onda, cálculo de distancia, tiempo de tránsito ($TT$) y $PWV$ ante distintas frecuencias cardíacas, amplitudes y relaciones señal/ruido ($SNR$).
2. **Validación Temporal:** Evaluación mediante generadores de funciones, simuladores fisiológicos o señales digitales desfasadas con precisión conocida para determinar error absoluto temporal, resolución ($1\text{ ms}$), linealidad y estabilidad.
3. **Validación de Distancia:** Evaluación de la incertidumbre del método de medición anatómica, repetibilidad inter-operador y validación de la corrección geométrica ($0.8 \times L_{CF}$).
4. **Validación del Algoritmo:** Prueba frente a bases de datos de referencia evaluando sensibilidad, especificidad, tasa de falsos positivos/negativos en detección del pie de onda y robustez ante artefactos de movimiento.
5. **Ensayos de Repetibilidad y Reproducibilidad:**
   * *Repetibilidad:* Mínimo 10 mediciones consecutivas por sujeto en intervalos $< 2\text{ min}$ (Cálculo de media, $SD$, $CV$ y error estándar).
   * *Reproducibilidad:* Evaluación frente a distintos operadores, días y reposicionamiento de transductores (Cálculo de $ICC$).
6. **Validación Clínica y Análisis Estadístico:** Comparación contra un equipo estándar de referencia (ej. Complior, SphygmoCor, PulsePen) aplicando regresión lineal ($r, R^2$), **Análisis de Bland-Altman** (sesgo/Bias y límites de concordancia al $95\%$), $MAE$, $RMSE$ y $MAPE$.
7. **Gestión de Riesgos (ISO 14971):** Mitigación y alarmas ante mala colocación de sensores, pérdida de sincronismo, arritmias o baja calidad de señal.

### Criterios Mínimos de Aceptación Metrológica
| Parámetro Evaluado | Criterio de Aceptación |
| :--- | :--- |
| **Repetibilidad** | Coeficiente de Variación ($CV$) $\le 10\%$ |
| **Reproducibilidad** | Coeficiente de Correlación Intraclase ($ICC$) $\ge 0.90$ |
| **Correlación con Referencia** | Coeficiente de correlación ($r$) $\ge 0.90$ |
| **Sesgo (Bias)** | Cercano a $0\text{ m/s}$ (En pruebas preliminares: $6.57\text{ m/s}$ vs $6.48\text{ m/s}$ de ref.) |
| **Disponibilidad de Mediciones** | $> 95\%$ de intentos válidos |

---

## 📂 7. Estructura del Proyecto

```text
├── PCB Plataforma de Pruebas/       # Archivos de diseño esquemático y PCB (Altium Designer v22.11)
├── UOS/                             # Código fuente del Firmware y Sistema Operativo en Tiempo Real
│   ├── Drivers/                     # Drivers de bajo nivel (ADS1192, MAX30100, MAX30102)
│   ├── HAL_Layer/                   # Capa de abstracción de hardware (HAL_Layer.c/.h)
│   ├── Hardware_Functions/          # Funciones específicas del MCU (stm32_framework_functions.c/.h)
│   ├── Inc/                         # Cabeceras de configuración (config.h, pin_asigment.h, sample_wave.h)
│   ├── OS/                          # Núcleo del sistema operativo y memoria compartida (os.c/.h)
│   ├── Process/                     # Implementación de tareas apropiativas y no apropiativas (procesos.c/.h)
│   ├── Protocol/                    # Empaquetado, serialización y checksum v2/v3 (protocol.c/.h)
│   └── Scheduler/                   # Planificador de tareas y cambio de contexto ASM (scheduler.c, context_switching.s)
└── Clientes/                        # Herramientas de recepción y visualización en PC
    ├── Python/                      # Cliente de pruebas en tiempo real (test.py)
    └── C++/                         # Clases base de deserialización para futura integración en Qt5
```

---

## 🖥️ 8. Clientes de Prueba y Software

El sistema fue ensayado satisfactoriamente con dos entornos de software:
1. **Software Oficial GIBIO uFISIO:** Ejecutado bajo entorno **Anaconda** (utilizando Protocolo v2). En ensayos con señales almacenadas para una distancia simulada de $60\text{ cm}$, el software arrojó una $VOP_{CF}$ estimada de **$6.48 \pm 5.86\%\text{ m/s}$**.
2. **Cliente de Pruebas en Python (`Clientes/Python/test.py`):** Desarrollado para validar el **Protocolo v3**. Permite graficar simultáneamente las curvas de presión carótida/femoral, el ritmo cardíaco, el `VOP Delay` ($\Delta t$), las señales PPG (Roja e IR) y auditar por consola el porcentaje de uso de CPU de cada tarea del RTOS. Con los datos procesados localmente por el microcontrolador ($\Delta t \approx 73\text{ ms}$), se obtuvo una $VOP_{CF}$ de **$6.57\text{ m/s}$**, demostrando alta concordancia con el software de referencia.

---

## 📚 9. Referencias y Normativa

### Normativa Aplicable
* **ISO 13485:** Sistemas de gestión de la calidad para dispositivos médicos.
* **ISO 14971:** Gestión de riesgos para dispositivos médicos.
* **IEC 60601-1 / IEC 60601-1-2:** Seguridad eléctrica, desempeño esencial y compatibilidad electromagnética.
* **IEC 62304:** Ciclo de vida del software para dispositivos médicos.
* **Guías ESC/ESH:** *Guidelines for the Management of Arterial Hypertension* (Reconocimiento de $cfPWV$ como estándar de referencia).

### Bibliografía Principal
1. **G. Fiori et al.**, *"Non-Invasive Methods for PWV Measurement in Blood Vessel Stiffness Assessment"*, IEEE Reviews in Biomedical Engineering, Vol. 15, pp. 169-183, 2022.
2. **M. De Luca, L. J. Cymberknop et al.**, *"uFISIO-VOP: Implementación de un Sistema Modular para la Evaluación de la Rigidez Arterial en Entornos Clínicos"*, XXII Congreso Argentino de Bioingeniería (SABI), 2020.
3. **L. M. Van Bortel et al.**, *"Expert consensus document on the measurement of aortic stiffness in daily practice using carotid-femoral pulse wave velocity"*, Journal of Hypertension, 2012.
4. **J. L. Izzo et al.**, *"Impact of arterial location, pressure wave indicators, and measurement devices on arterial form factor and mean and central arterial pressure"*, 2022.

---
**Autor del Desarrollo de Plataforma (PPS):** Hernán Matías Travado  
**Supervisor de Campo (GIBIO / CIBIO):** Dr. Ing. Leandro J. Cymberknop  
**Tutor Universitario:** Ing. Sergio Díaz  
**Institución:** Universidad Tecnológica Nacional - Facultad Regional Buenos Aires (UTN.BA)