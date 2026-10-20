# Validación de un Equipo de Medición de la Velocidad de la Onda del Pulso Arterial (VOP / PWV)

**Asignatura:** Medidas Electrónicas I  
**Especialidad:** Ingeniería Electrónica  
**Institución:** Centro de Investigación en Ingeniería Biomédica (CIBIO) - UTN.BA  
**A cargo de:** Cymberknop, Leandro Javier
**Destinatarios:** Unidad de Prevención Cardiovascular del CIBIO 
**Alumnos:** Lapilli Cañedo, Mariano Adrián

             Robledo, Leandro Nahuel 
             
             Oléa Gasalla, Ignacio Ezequiel
             
             Trombetta Nicolás
**Docentes:** De Césare, Pablo Nicolás

              Rodriguez Colmeiro, Ramiro
              
              Zaccra, Franco
---

## 📋 Resumen del Proyecto

La velocidad de la onda del pulso arterial (**VOP** o **PWV**, por sus siglas en inglés *Pulse Wave Velocity*) es un biomarcador no invasivo ampliamente utilizado para evaluar la rigidez arterial y estimar el riesgo cardiovascular. Su determinación se basa en el cálculo de la velocidad de propagación de la onda de presión generada por la eyección ventricular entre dos sitios arteriales, a partir de la distancia recorrida y el tiempo de tránsito de la onda.

Debido a su relevancia clínica, la obtención de mediciones precisas y reproducibles resulta esencial para garantizar una adecuada interpretación diagnóstica y el seguimiento de los pacientes. Este proyecto contempla la evaluación del desempeño metrológico, la verificación de los algoritmos de procesamiento de señales y la comparación con métodos de referencia, en concordancia con las normas y recomendaciones internacionales aplicables a dispositivos médicos.

---

## ⚙️ Principio de Funcionamiento

Los equipos de medición de PWV estiman este parámetro determinando:
1. El **tiempo de tránsito** (*Transit Time*, $TT$) entre dos sitios arteriales.
2. La **distancia recorrida** ($D$) por la onda entre dichos puntos anatómicos.

La velocidad se calcula mediante la siguiente expresión:

$$PWV = \frac{D}{TT}$$

Donde:
* **$D$**: Distancia efectiva recorrida por la onda arterial ($\text{m}$).
* **$TT$**: Tiempo de tránsito entre ambos sitios ($\text{s}$).

### Tecnologías de Medición
La estimación del tiempo de tránsito puede realizarse mediante diversas tecnologías:
* Tonometría por aplanamiento.
* Fotopletismografía (PPG).
* Ultrasonido Doppler.
* Sensores piezoeléctricos.
* ECG sincronizado con sensores periféricos.

> **Nota crítica:** En todos los casos, la exactitud del resultado depende tanto de la precisión temporal del sistema como de la correcta identificación del pie de la onda de pulso.

---

## 🔬 Requerimientos de Validación Metrológica y Clínica

El fabricante deberá demostrar que el equipo proporciona mediciones repetibles, reproducibles y clínicamente válidas conforme a las recomendaciones internacionales vigentes. La validación contempla **10 etapas**:

### 1. Validación Funcional
Verificar el correcto funcionamiento de cada subsistema:
* Adquisición de señales y sincronización entre canales.
* Detección automática del pie de onda.
* Cálculo del tiempo de tránsito ($TT$), distancia ($D$) y cálculo final de $PWV$.
* **Pruebas de estrés frente a variaciones de:** amplitudes, relaciones señal/ruido (SNR), frecuencias cardíacas y morfologías de pulso.

### 2. Validación Temporal
El sistema deberá garantizar la exactitud en la medición del tiempo empleando generadores, simuladores y señales de referencia. El error máximo deberá encontrarse dentro de los límites establecidos por las especificaciones del equipo.

### 3. Validación de Distancia
Dado que la medición entre sitios arteriales es una fuente importante de incertidumbre, se deberá verificar:
* Procedimiento de medición anatómica y repetibilidad entre operadores.
* Influencia de la antropometría.
* Corrección geométrica utilizada por el algoritmo (documentar y validar algoritmos automáticos si los hubiera).

### 4. Validación del Algoritmo
Evaluación mediante bases de datos con valores de referencia conocidos:
* **Verificación de:** correcta detección del pie de onda, robustez frente al ruido, comportamiento ante artefactos de movimiento y estabilidad frente a variaciones fisiológicas.
* **Métricas a evaluar:** sensibilidad, especificidad, tasa de detecciones erróneas y porcentaje de señales descartadas.

### 5. Repetibilidad
Múltiples mediciones consecutivas sobre un mismo sujeto bajo condiciones constantes:
* **Protocolo recomendado:** al menos $10\text{ mediciones}$ por sujeto con un intervalo inferior a $2\text{ minutos}$.
* **Cálculos requeridos:** media, desviación estándar, coeficiente de variación ($CV$) y error estándar.

### 6. Reproducibilidad
Evaluación de la influencia de distintos operadores, distintos días, reposicionamiento de sensores y diferentes condiciones ambientales.
* **Cálculos requeridos:** Coeficiente de Correlación Intraclase ($ICC$), Coeficiente de Variación ($CV$) y error de reproducibilidad.

### 7. Validación Clínica
Comparación con un método de referencia clínicamente aceptado (tonometría o tecnologías equivalentes validadas internacionalmente).
* **Población de estudio representativa de diferentes:** edades, sexos, índices de masa corporal (IMC), niveles de presión arterial y grados de rigidez arterial.

### 8. Análisis Estadístico
La comparación con el método de referencia deberá incluir:
* Regresión lineal, coeficiente de correlación ($r$) y coeficiente de determinación ($R^2$).
* Análisis de Bland-Altman, error medio (*Bias*) y límites de concordancia al $95\%$.
* Error Absoluto Medio ($MAE$), Error Cuadrático Medio ($RMSE$) y Error Relativo Medio ($MAPE$).
* *Opcionales según corresponda:* Concordance Correlation Coefficient ($CCC$), regresión de Passing-Bablok y regresión de Deming.

### 9. Robustez
Verificación del desempeño frente a condiciones de uso reales:
* Movimiento moderado y variaciones respiratorias.
* Diferentes temperaturas ambientales.
* Diferentes tonos de piel (especialmente crítico para sistemas PPG).
* Distintas posiciones corporales y frecuencias cardíacas.

### 10. Gestión del Riesgo
Demostrar que los errores del sistema no comprometen la interpretación clínica (conforme a **ISO 14971**). Cada riesgo debe contar con mecanismos de detección, mitigación y notificación al usuario:
* Mala colocación de sensores y pérdida de sincronización.
* Señales de baja calidad, arritmias y artefactos de movimiento.
* Errores en la introducción o estimación de distancia.

---

## ✅ Criterios Mínimos de Aceptación

Como criterios generales, el equipo deberá demostrar el cumplimiento de los siguientes parámetros:

| Parámetro | Criterio Recomendado |
| :--- | :--- |
| **Repetibilidad** | $CV \le 10\%$ |
| **Reproducibilidad** | $ICC \ge 0.90$ |
| **Correlación con referencia** | $r \ge 0.90$ |
| **Sesgo (Bias)** | Cercano a $0\text{ m/s}$ |
| **Límites de concordancia** | Clínicamente aceptables |
| **Error absoluto medio ($MAE$)** | Dentro de la incertidumbre especificada por el fabricante |
| **Disponibilidad de mediciones válidas** | $> 95\%$ de los intentos |

---

## 📜 Normativa y Documentos de Referencia

El diseño de la validación debe ser consistente con las principales normas y guías aplicables a dispositivos médicos:

* **ISO 13485:** Sistemas de gestión de la calidad para dispositivos médicos.
* **ISO 14971:** Gestión de riesgos para dispositivos médicos.
* **IEC 60601-1:** Requisitos generales de seguridad eléctrica y desempeño esencial.
* **IEC 60601-1-2:** Compatibilidad electromagnética.
* **IEC 62304:** Ciclo de vida del software para dispositivos médicos (cuando aplique).
* **IEC 62366-1:** Ingeniería de la usabilidad en dispositivos médicos.
* **ISO 14155:** Buenas prácticas clínicas para investigaciones clínicas con dispositivos médicos.
* **Guías Clínicas (ESH / ESC):** Guías de la *European Society of Hypertension* y de la *European Society of Cardiology* para la evaluación de la rigidez arterial (reconocen la PWV carotídeo-femoral como el estándar de referencia para la evaluación no invasiva de la rigidez aórtica).
