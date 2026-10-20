# CC3045 - Deep Learning | Examen parcial

> **Documento de trabajo basado en `S16 - Examen Practico.pdf`.** Este README organiza el caso, los datos y las tareas del examen para que cada integrante o agente identifique su responsabilidad. No contiene soluciones. Los datos y requisitos provienen del PDF; las tablas de dependencias, las líneas «Información que debe compartir» y el checklist de cierre son apoyos de organización del grupo, no texto del PDF.

## 1. Objetivo del examen

Actuar como el **comité técnico de RemesaSegura S.A.** y recomendar a la gerencia una de estas tres decisiones para el próximo lunes:

1. **Lanzar** el sistema.
2. **Lanzar con condiciones**.
3. **No lanzar** el sistema.

La decisión debe sustentarse en análisis técnico, reconciliación de cifras, evaluación operativa, explicabilidad y riesgos regulatorios. El producto principal es un **memo de máximo dos páginas en PDF**, acompañado por una **presentación de dos minutos** y una **defensa oral**.

### Contexto del caso

- Empresa: **RemesaSegura S.A.**, con operaciones de remesas entre Guatemala y Estados Unidos.
- Remitentes activos analizados: aproximadamente **6,000 por día**.
- Prevalencia de lavado de dinero confirmado: **0.2%**, aproximadamente **12 casos por día**.
- Obligación: detectar los casos, reportarlos y explicar ante el regulador el motivo de cada alerta.
- El sistema propuesto tiene dos etapas:
  1. Una primera etapa aprende el comportamiento normal de las remesas y mide desviaciones.
  2. Una segunda etapa es un clasificador supervisado que reutiliza lo aprendido y asigna un puntaje de riesgo por remitente.
- Se genera una alerta si el puntaje excede un umbral. **Cada alerta debe ser revisada por una persona de cumplimiento**.

### Tiempos indicados en el documento

| Actividad | Tiempo indicado |
|---|---:|
| Preguntas individuales en papel, sin IA | 5 minutos |
| Caso grupal: resolución, memo y preparación de presentación | 50 minutos |
| Defensa de cada grupo | 5 minutos |

**Nota de fidelidad:** el PDF también afirma que el examen dura 90 minutos en total, aunque los tiempos de la tabla suman 60. Se preservan ambos datos sin suponer cómo se distribuyen los 30 minutos restantes. La parte individual ya terminó al abrir el documento.

## 2. Asignación de roles

### Regla oficial

1. Ordenar a los integrantes por **mes y día de cumpleaños**, de enero a diciembre.
2. En caso de empate, ordenar por **apellido**.
3. Asignar los roles siguiendo ese orden: primera persona = Rol 1, segunda = Rol 2, etcétera.
4. Si hay menos de siete integrantes, **la última persona también asume los roles sobrantes, comenzando por el Rol 7**.
5. Cada integrante responde por su sección, pero **debe poder explicar las siete secciones**.
6. Se permite y espera usar IA. Los datos entre roles pueden no coincidir; identificarlos y reconciliarlos es parte del examen.

### Matriz de responsables

| Rol | Especialidad | Task individual | Sección del memo | Responsable de |
|---|---|---|---|---|
| **1** | Líder de modelado | **1.1** | **1** | Diagnosticar entrenamiento, pérdidas, desbalance y recomendar correcciones. |
| **2** | Ingeniero de transferencia | **1.2** | **2** | Elegir y justificar la estrategia de transferencia/fine-tuning. |
| **3** | Analista de series de tiempo | **1.3** | **3** | Evaluar validez de métricas, particiones temporales y autocorrelación. |
| **4** | Especialista en modelos generativos | **1.4** | **4** | Evaluar GAN/VAE/difusión, datos sintéticos y privacidad. |
| **5** | Líder de lenguaje natural | **1.5** | **5** | Evaluar narrativas con LLM, alucinaciones, tiempos y controles. |
| **6** | Responsable de explicabilidad | **1.6** | **6** | Evaluar SHAP/LIME/atención y explicaciones según audiencia. |
| **7** | Responsable de operaciones y cumplimiento | **1.7** | **7** | Evaluar capacidad de revisión, costos, piloto y decisión operativa. |

**Propiedad de tareas:** cada rol es dueño de su análisis de **Task 1** y de su sección numerada del memo. Las **Tasks 2 y 3 son grupales**: no se asignan exclusivamente a un rol en el PDF.

## 3. Task 1 - Análisis por rol

Todos los roles deben completar los incisos de su task y redactar **su sección del memo en 4 a 6 líneas**, incluyendo una recomendación y al menos una cifra que la respalde.

### Rol 1 - Líder de modelado | Task 1.1 | Memo §1

**Responsabilidad:** diagnosticar el entrenamiento del modelo candidato y recomendar cómo entrenarlo.

**Datos asignados:**

- Todas las métricas internas se calcularon en validación **remuestreada con 50% de positivos y 50% de negativos**.
- Las particiones utilizadas fueron **aleatorias**.

| Umbral | Recall | Tasa de falsos positivos (FPR) | Precisión reportada |
|---|---:|---:|---:|
| U1 | 0.90 | 0.10 | 0.90 |
| U2 | 0.80 | 0.04 | 0.95 |
| U3 | 0.65 | 0.01 | 0.98 |
| U4 | 0.50 | 0.005 | 0.99 |

**Entrenamiento actual:**

- 20 épocas.
- Entropía cruzada binaria **sin ponderación**.
- Datos de entrenamiento en su proporción real de positivos.
- Sin regularización explícita.
- Pérdida de entrenamiento: **0.42 → 0.08**.
- Pérdida de validación: mínimo **0.21 en época 6**, luego **0.34 en época 20**.
- El modelo evaluado corresponde a la **época 20**.

**Entregables exclusivos del Rol 1:**

- [ ] **1.1.a** Diagnosticar las curvas de pérdida y proponer **dos cambios concretos** de entrenamiento. Explicar qué problema corrige cada cambio.
- [ ] **1.1.b** Explicar si la función de pérdida utilizada es apropiada con **0.2% de clase positiva** y justificar una alternativa.
- [ ] **1.1.c** Redactar **sección 1 del memo (4–6 líneas)** con recomendación y una cifra de respaldo.

**Información que debe compartir:** tabla de umbrales y supuestos de validación con Rol 3, Rol 7 y el grupo para Task 2.2.

### Rol 2 - Ingeniero de transferencia | Task 1.2 | Memo §2

**Responsabilidad:** seleccionar cómo adaptar un codificador preentrenado al nuevo corredor.

**Datos asignados:**

- Un socio comparte un codificador preentrenado con **3 millones de secuencias de transacciones** de otro corredor.
- RemesaSegura tiene **1,200 positivos etiquetados**.
- Las cinco estrategias se evaluaron usando **partición aleatoria**.

| Estrategia | AUC | Parámetros entrenables | GPU-horas |
|---|---:|---:|---:|
| Entrenar desde cero | 0.78 | 100% | 6 |
| Extracción de características (nueva cabeza, codificador congelado) | 0.82 | 0.5% | 0.5 |
| Ajuste fino de los dos últimos bloques con tasa de aprendizaje 10 veces menor | 0.87 | 18% | 1.5 |
| Ajuste fino completo | 0.84 | 100% | 6 |
| LoRA de rango 8 | 0.86 | 2% | 1 |

**Observaciones adicionales:**

- El ajuste fino completo alcanza pérdida de entrenamiento mucho menor que el parcial.
- El socio utilizó **otra escala de normalización de montos**.
- Las distribuciones de montos de ambos corredores son diferentes.

**Entregables exclusivos del Rol 2:**

- [ ] **1.2.a** Elegir una estrategia, justificarla con **al menos dos columnas** de la tabla y explicar por qué el ajuste fino completo rinde menos.
- [ ] **1.2.b** Explicar riesgos al reutilizar un modelo de otro corredor y cómo verificarlos.
- [ ] **1.2.c** Redactar **sección 2 del memo (4–6 líneas)** con recomendación y cifra de respaldo.

**Información que debe compartir:** resultados de AUC y metodología de validación con Rol 3; consideraciones de transferencia con el grupo.

### Rol 3 - Analista de series de tiempo | Task 1.3 | Memo §3

**Responsabilidad:** determinar cuánto se puede confiar en las métricas del candidato y recomendar validación temporal.

**Datos asignados:**

- Secuencias por remitente divididas en ventanas de **10 transacciones**.
- Roles 1 y 2 calcularon sus métricas con **partición aleatoria de ventanas** entre entrenamiento y validación.
- Con partición temporal del candidato (ajuste fino de dos bloques):
  - Entrenamiento: **enero a marzo**.
  - Validación: **abril y mayo**.
  - AUC: **0.87 → 0.81**.
- La autocorrelación del monto diario presenta picos a **7, 15 y 30 días**.
- Un sistema que **nunca emite alertas** alcanza **99.8% de exactitud**.

**Entregables exclusivos del Rol 3:**

- [ ] **1.3.a** Explicar por qué la partición aleatoria infla el AUC y qué cifra debe usar la gerencia.
- [ ] **1.3.b** Interpretar picos de autocorrelación y recomendar **longitud de ventana en días** con justificación.
- [ ] **1.3.c** Explicar por qué la exactitud no resulta útil y definir métricas más apropiadas.
- [ ] **1.3.d** Redactar **sección 3 del memo (4–6 líneas)** con recomendación y cifra de respaldo.

**Información que debe compartir:** AUC temporal y advertencias de validación con Roles 1, 2 y 4 y con el grupo para conciliación.

### Rol 4 - Especialista en modelos generativos | Task 1.4 | Memo §4

**Responsabilidad:** decidir si los datos sintéticos convienen y evaluar cobertura y privacidad.

**Datos asignados:**

- Se conocen **7 tipologías de lavado** en el corredor.
- Se propone generar secuencias sospechosas sintéticas para compensar los pocos positivos.

| Generador | Tipologías cubiertas | Observación | Costo de entrenamiento |
|---|---:|---|---|
| GAN | 2 de 7 | Repite una sola forma de fraccionamiento de montos | 4 horas |
| Autoencoder variacional (VAE) | 5 de 7 | Secuencias borrosas: promedia montos y pierde patrones exactos | 2 horas |
| Difusión | 6 de 7 | Mayor diversidad y nitidez | 3 días |

**Resultados adicionales:**

- Clasificador entrenado con datos reales y sintéticos de difusión: **AUC temporal 0.81 → 0.83**.
- Con sintéticos generados por GAN: **sin cambio de AUC**.
- Auditoría de privacidad: **3% de las secuencias sintéticas de difusión** son casi idénticas a secuencias reales de clientes.

**Entregables exclusivos del Rol 4:**

- [ ] **1.4.a** Explicar, según los mecanismos generativos vistos en clase, la baja cobertura de GAN y la borrosidad de VAE.
- [ ] **1.4.b** Recomendar si usar sintéticos y con qué generador; definir **datos de entrenamiento y de prueba**.
- [ ] **1.4.c** Proponer una mitigación del riesgo de privacidad.
- [ ] **1.4.d** Redactar **sección 4 del memo (4–6 líneas)** con recomendación y cifra de respaldo.

**Información que debe compartir:** resultados de AUC temporal con Rol 3 y riesgos de privacidad con Rol 6 y el grupo.

### Rol 5 - Líder de lenguaje natural | Task 1.5 | Memo §5

**Responsabilidad:** decidir cómo usar un LLM para redactar narrativas de alertas en un proceso regulado.

**Datos asignados:**

- El LLM redacta **borradores** de narrativas que un oficial de cumplimiento adjunta a las alertas.
- Se auditaron **40 narrativas por configuración**.

| Configuración | Narrativas con hechos inventados | Minutos por narrativa, con revisión |
|---|---:|---:|
| Sin transacciones de la alerta en contexto; temperatura 0.7 | 6 de 40 | 1.5 |
| Con transacciones de la alerta en contexto; temperatura 0.2 | 2 de 40 | 1.5 |
| Redacción manual | No aplica | 6 |

**Ejemplo de alucinación:** se cita una transacción de **USD 9,800** que no existe en la secuencia del remitente.

**Entregables exclusivos del Rol 5:**

- [ ] **1.5.a** Calcular las tasas de alucinación; explicar por qué ocurre la generación de hechos inventados y por qué contexto y temperatura pueden reducirlos.
- [ ] **1.5.b** Calcular ahorro de tiempo por narrativa y definir condiciones para usar LLM en un proceso regulado.
- [ ] **1.5.c** Redactar **sección 5 del memo (4–6 líneas)** con recomendación y cifra de respaldo.

**Información que debe compartir:** tasas de alucinación y ahorro de revisión con Rol 7; consideraciones sobre narrativas regulatorias con Rol 6.

### Rol 6 - Responsable de explicabilidad | Task 1.6 | Memo §6

**Responsabilidad:** identificar explicaciones confiables y adaptar su comunicación a cada audiencia.

**Contexto:** el regulador exige que cada alerta pueda explicarse, y el equipo de cumplimiento exige que la explicación sea confiable.

**Datos asignados:**

| Prueba | Resultado |
|---|---:|
| SHAP con 100 secuencias de fondo: error de eficiencia máximo, en log-odds | 3 × 10⁻⁷ |
| LIME con 500 muestras: Jaccard promedio en 20 corridas, usando las 5 variables principales | 0.45 |
| LIME con 5,000 muestras: Jaccard promedio en 20 corridas | 0.78 |
| Correlación de rangos entre pesos de atención y valores SHAP | 0.35 |

**Política de cumplimiento:** está prohibido mostrar a un cliente señalado **qué tendría que cambiar para evitar que se genere la alerta**.

**Entregables exclusivos del Rol 6:**

- [ ] **1.6.a** Interpretar todos los resultados y determinar qué explicación recomienda y cuál no.
- [ ] **1.6.b** Definir explicaciones diferenciadas para **oficial de cumplimiento, regulador y cliente señalado**, justificándolas.
- [ ] **1.6.c** Explicar el sentido de la política de cumplimiento y exigir una **validación adicional antes del lanzamiento**.
- [ ] **1.6.d** Redactar **sección 6 del memo (4–6 líneas)** con recomendación y cifra de respaldo.

**Información que debe compartir:** requisitos y limitaciones de explicabilidad con Roles 5 y 7 y con el grupo para la defensa regulatoria.

### Rol 7 - Responsable de operaciones y cumplimiento | Task 1.7 | Memo §7

**Responsabilidad:** evaluar si el equipo puede procesar las alertas y recomendar una acción operativa.

**Datos asignados:**

- Equipo de cumplimiento: **8 analistas**.
- Cada analista revisa **5 alertas por día**.
- Costo de revisión: **USD 12 por alerta**.
- Piloto de dos semanas con **umbral U2**.
- Alertas del piloto: **aproximadamente 250 por día**.
- Confirmaciones del piloto: **cerca del 4% de las alertas**.
- Casos confirmados esperados en la operación: **12 por día**.
- Daño por no detectar un caso confirmado: **USD 50,000** estimados entre sanción y reputación.
- Gerencia exige decidir el próximo lunes.

**Entregables exclusivos del Rol 7:**

- [ ] **1.7.a** Calcular **capacidad diaria** del equipo y **costo diario** de revisar todas las alertas del piloto; compararlos.
- [ ] **1.7.b** Calcular casos confirmados diarios detectados en el piloto y su fracción de los 12 casos esperados.
- [ ] **1.7.c** Proponer una decisión operativa ante la diferencia entre alertas y capacidad.
- [ ] **1.7.d** Redactar **sección 7 del memo (4–6 líneas)** con recomendación y cifra de respaldo.

**Información que debe compartir:** datos de capacidad, costos y piloto con Rol 1 y el grupo para elegir umbral y condiciones en Task 2.2.

## 4. Task 2 - Integración y decisión (todos los roles)

**Los siete integrantes son responsables de completar y validar esta task conjuntamente.** Los insumos se producen en Task 1, pero ninguna sección se da por concluida hasta reconciliar los datos.

### Task 2.1 - Conciliar las siete secciones

- [ ] **2.1.a** Cada integrante expone **en un minuto** la cifra clave de su rol y su recomendación. Registrar qué cifras dependen de otras secciones.
- [ ] **2.1.b** Identificar cifras que no coinciden entre secciones. Explicar **cuál cifra es correcta en su contexto, por qué ambas pueden ser ciertas y cómo afectan la decisión**.
- [ ] **2.1.c** Señalar cuáles cifras son **optimistas** y por qué.

**Dependencias para revisar en grupo:**

| Cruce | Roles involucrados | Datos que deben conciliarse |
|---|---|---|
| Métricas en validación vs operación real | 1, 3, 7 | Validación balanceada/aleatoria, prevalencia real y piloto. |
| Estrategia de transferencia vs validación temporal | 2, 3 | AUC de partición aleatoria frente a AUC de partición temporal. |
| Mejoras con datos sintéticos | 3, 4 | AUC temporal base y con difusión; riesgo de privacidad. |
| Narrativas y operación regulada | 5, 6, 7 | Revisión humana, alucinaciones, explicabilidad y volumen de alertas. |
| Umbral y capacidad operativa | 1, 7 | Recall, FPR, alertas, costos y capacidad de revisión. |

### Task 2.2 - Elegir umbral, capacidad y decisión

Usar explícitamente:

- Tabla de umbrales del **Rol 1**.
- **6,000 remitentes por día**.
- **0.2% de positivos**.

Para justificar la elección en 2.2.b se usan además la capacidad del equipo, el costo de revisión y el costo de no detectar (datos del **Rol 7**).

- [ ] **2.2.a** Calcular y completar para **cada umbral**: alertas diarias esperadas, casos detectados, casos no detectados, precisión esperada y costo diario de revisión.
- [ ] **2.2.b** Elegir **umbral y capacidad operativa**; justificar con capacidad del equipo, costo de revisión y costo de no detectar casos.
- [ ] **2.2.c** Decidir **lanzar / lanzar con condiciones / no lanzar** e indicar **al menos tres condiciones medibles** para el lanzamiento.

**Tabla para completar (no resuelta):**

| Umbral | Alertas diarias esperadas | Casos detectados | Casos no detectados | Precisión esperada | Costo diario de revisión |
|---|---:|---:|---:|---:|---:|
| U1 | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |
| U2 | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |
| U3 | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |
| U4 | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |

### Task 2.3 - Documentar los errores o limitaciones de la IA

- [ ] **2.3.a** Registrar al menos **dos casos** en que la IA produjo un resultado incorrecto o incompleto. Para cada caso documentar: **qué dijo, qué supuesto violó y cómo se verificó**.
- [ ] **2.3.b** Si no se encuentra ningún error, pedir a la IA repetir los cálculos de **Task 2.2** y **verificar manualmente al menos un resultado**.

**Formato de registro:**

| Caso | Respuesta de la IA | Supuesto incorrecto / información omitida | Verificación realizada |
|---|---|---|---|
| 1 | Pendiente | Pendiente | Pendiente |
| 2 | Pendiente | Pendiente | Pendiente |

No inventar errores: el registro debe reflejar interacciones reales y verificaciones efectuadas por el grupo.

### Task 2.4 - Memo final y entrega

- [ ] **2.4.a** Redactar una **defensa ante el regulador en máximo seis líneas** que incluya:
  - Evidencia que presentarían.
  - Riesgo residual reconocido.
  - Compromisos de monitoreo.
  - Quién responde por la decisión dentro de la empresa.
- [ ] **2.4.b** Preparar el **memo de máximo dos páginas** con la estructura indicada en la sección siguiente.
- [ ] **2.4.c** Añadir por integrante **tres líneas**: su aporte, una corrección a la IA y cómo la verificó, y un dato que necesitó de otro rol.
- [ ] **2.4.d** Exportar el memo a **PDF** y subirlo a **Canvas cuando el docente avise que han concluido los 50 minutos del caso**. Después de ese momento **no se edita**.

## 5. Estructura obligatoria del memo final

El documento completo no puede superar **dos páginas**. Integrar en este orden:

1. **Decisión general en tres líneas:** lanzar, lanzar con condiciones o no lanzar.
2. **Sección 1 - Rol 1:** modelado y entrenamiento (4–6 líneas).
3. **Sección 2 - Rol 2:** transferencia y ajuste fino (4–6 líneas).
4. **Sección 3 - Rol 3:** validación temporal y métricas (4–6 líneas).
5. **Sección 4 - Rol 4:** datos sintéticos y privacidad (4–6 líneas).
6. **Sección 5 - Rol 5:** narrativas mediante LLM (4–6 líneas).
7. **Sección 6 - Rol 6:** explicabilidad (4–6 líneas).
8. **Sección 7 - Rol 7:** operaciones y cumplimiento (4–6 líneas).
9. **Cifras inconsistentes:** cuáles no cuadraban, por qué y cómo se conciliaron.
10. **Crítica de la IA:** errores/incompletitudes y verificación.
11. **Defensa ante el regulador:** máximo seis líneas.
12. **Contribuciones individuales:** tres líneas por integrante con aporte, corrección verificada a la IA y dependencia de otro rol.

**Requisito por sección:** recomendación concreta y **al menos una cifra que la sostenga**. La decisión final debe estar alineada con las cifras reconciliadas, la capacidad de revisión y **al menos tres condiciones medibles** si se condiciona el lanzamiento.

## 6. Task 3 - Presentación y defensa

### Task 3.1 - Presentación grupal

- [ ] Preparar **con IA** **una sola diapositiva** o **un esquema de una página**.
- [ ] Incluir **decisión**, **umbral elegido**, **capacidad elegida**, **cifra más determinante** y **principal riesgo**.
- [ ] Ensayar **una vez**, con cronómetro, la exposición de **dos minutos**.
- [ ] Comprobar que **todos** pueden explicar las siete secciones.

### Task 3.2 - Defensa oral

- Duración total por grupo: **5 minutos**.
  - **2 minutos** de exposición.
  - **3 minutos** de preguntas.
- El docente sortea el orden de los grupos.
- Después de la exposición, **se sortea un integrante** que responderá **solo** las preguntas. Nadie sabe de antemano quién será.
- Las preguntas pueden tratar su propia sección, cualquier otra sección y **el uso de IA**.

## 7. Rúbrica de evaluación

**Total: 100 puntos**. Parte individual en papel: **20 puntos**. Parte grupal: **80 puntos**.

| Criterio grupal | Puntos | Qué se evalúa |
|---|---:|---|
| Razonamiento técnico | 25 | Uso correcto de datos y conceptos del curso con justificación en cada sección. |
| Integración y verificación crítica | 25 | Conciliación de cifras incompatibles y documentación de errores de IA. |
| Defensa oral | 15 | Exposición clara y respuestas precisas y honestas de la persona sorteada. |
| Responsabilidad | 15 | Decisión asumida y defensa fundamentada ante el regulador. |
| **Total grupal** | **80** | |

## 8. Checklist de cierre por responsable

| Responsable | Debe confirmar antes de cerrar |
|---|---|
| **Rol 1** | Task 1.1 a–c y sección 1 lista; tabla de umbrales comunicada. |
| **Rol 2** | Task 1.2 a–c y sección 2 lista; resultados y riesgos de transferencia comunicados. |
| **Rol 3** | Task 1.3 a–d y sección 3 lista; cifras de validación temporal compartidas. |
| **Rol 4** | Task 1.4 a–d y sección 4 lista; mejoras y privacidad consideradas. |
| **Rol 5** | Task 1.5 a–c y sección 5 lista; alucinaciones y ahorro calculados. |
| **Rol 6** | Task 1.6 a–d y sección 6 lista; explicaciones por audiencia especificadas. |
| **Rol 7** | Task 1.7 a–d y sección 7 lista; costos/capacidad del piloto calculados. |
| **Todo el grupo** | Tasks 2.1–2.4 resueltas, cifras conciliadas, umbral/capacidad elegidos, al menos tres condiciones medibles definidas y crítica de IA documentada. |
| **Todo el grupo** | Memo de máximo dos páginas en PDF; una diapositiva o esquema de una página; presentación ensayada y defensa preparada. |

---

**Fuente única:** `S16 - Examen Practico.pdf`, páginas 1 a 6. Este README es una reorganización de sus datos, responsabilidades e instrucciones; los campos «Pendiente» se dejan intencionalmente sin resolver.
