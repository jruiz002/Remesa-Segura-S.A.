# Rol 3 - Analista de series de tiempo | Task 1.3 | Memo §3

**Datos de partida (README):** ventanas de 10 transacciones por remitente; Roles 1 y 2 usaron partición aleatoria de ventanas; partición temporal del candidato (ajuste fino de dos bloques: train ene–mar, val abr–may) da AUC 0.87 → 0.81; autocorrelación del monto diario con picos a 7, 15 y 30 días; un sistema que nunca alerta logra 99.8% de exactitud; 6,000 remitentes/día, prevalencia 0.2% (≈12 casos/día).

## 1.3.a

- **Fuga entre particiones:** al repartir ventanas al azar, ventanas del **mismo remitente** quedan en entrenamiento y en validación. Las ventanas contiguas o solapadas de una secuencia están autocorrelacionadas (picos a 7, 15 y 30 días), así que la validación contiene casi copias de lo que el modelo ya vio. El modelo **reconoce al remitente o al periodo** en vez de generalizar a remitentes y meses nuevos.
- **Futuro para predecir pasado:** la partición aleatoria mete en el entrenamiento ventanas posteriores a las de validación. En producción el modelo solo tendrá el pasado. Además, no mide la deriva entre meses.
- **Efecto medido:** con partición temporal el AUC del candidato baja de **0.87 a 0.81 (−0.06)**. **La gerencia debe usar 0.81**. El 0.87 es optimista porque mide memoria del remitente, no detección.
- **Recomendación de protocolo:** partición temporal (ene–mar → abr–may) **y además agrupada por remitente** (ningún remitente en ambos lados). Agregar un **hueco o embargo de ≥ 30 días**, el mayor rezago con autocorrelación, o al menos descartar las ventanas que cruzan la frontera. Repetir el esquema con origen móvil para ver la estabilidad mes a mes.

## 1.3.b

- **Interpretación:** los picos a **7, 15 y 30 días** muestran ciclos **semanal, quincenal y mensual**. Esto coincide con el envío de remesas atado al pago de salario (semana, quincena, fin de mes). Un monto alto el día de pago es normal, no una anomalía.
- **Problema de la ventana actual:** 10 transacciones no corresponden a un horizonte fijo de días. Para un remitente frecuente cubren una o dos semanas (no llegan al ciclo mensual). Para uno esporádico cubren meses (mezclan varios ciclos). La primera etapa aprende "normalidad" con escalas de tiempo distintas por remitente.
- **Recomendación: ventana de al menos 31 días calendario** (un ciclo mensual completo: una ventana de exactamente 30 días solo contiene rezagos de 0 a 29 y no vería el de 30), con agregación diaria e indicadores de día de semana, quincena y fin de mes.
  - Es la ventana mínima que contiene los tres ciclos completos (≈4 semanas, 2 quincenas, 1 mes).
  - Una de 60 días cubriría dos ciclos mensuales, pero no cabe con el calendario disponible. Train ene–mar (≈90 días) daría apenas 1–2 ventanas independientes por remitente. Val abr–may (≈61 días) no admitiría una sola ventana de 60 días más el embargo.
- **Desviaciones:** medirlas contra el **patrón estacional propio del remitente** (su perfil por día del ciclo, estimado con meses anteriores), no contra un promedio global. Así el ciclo de salario no se confunde con riesgo.

## 1.3.c

- **Por qué la exactitud no sirve:** un sistema que **nunca alerta** logra 99.8% (= 1 − 0.002; 5,988 de 6,000 aciertos) y detecta **0 de 12 casos/día**. Con prevalencia de 0.2%, la exactitud la dominan los negativos y no mide el objetivo regulatorio.
- **Métricas apropiadas:**
  - **Recall (sensibilidad):** fracción de los ≈12 casos/día detectados.
  - **Precisión a la prevalencia real (0.2%)**, no a la de una validación balanceada 50/50. Ejemplo con U2 del Rol 1: 0.80 × 12 = 9.6 verdaderos positivos y 0.04 × 5,988 ≈ 240 falsos, es decir ≈ 250 alertas/día y una precisión ≈ 4% (frente a 0.95 reportado). Coincide con el piloto del Rol 7.
  - **PR-AUC / average precision**, cuyo nivel de referencia es 0.002.
  - **Recall con carga fija de alertas:** **recall@40 alertas/día**, la capacidad del equipo de cumplimiento (Rol 7: 8 analistas × 5).
  - **FPR y alertas diarias absolutas**, que son el costo operativo.
- **Por qué ROC-AUC también es optimista:** el FPR se divide entre ≈5,988 negativos, así que un FPR "bajo" de 0.04 son ≈240 falsas alertas/día. ROC no depende de la prevalencia y no muestra el colapso de la precisión. PR-AUC y recall@capacidad sí lo muestran.

## Sección 3 del memo

Las métricas de los Roles 1 y 2 se calcularon con partición aleatoria de ventanas y están infladas por fuga de información entre ventanas del mismo remitente. Con partición temporal (ene–mar → abr–may) el AUC baja de 0.87 a 0.81. **Usar 0.81 como cifra base** y exigir validación temporal agrupada por remitente, con embargo ≥ 30 días. Por los ciclos de 7, 15 y 30 días, redefinir la ventana como al menos 31 días calendario y medir desviaciones contra el patrón propio del remitente. La exactitud (99.8% sin alertar nada) se descarta: reportar recall, precisión a 0.2% de prevalencia, PR-AUC y recall@40 alertas/día.

## Cifras a compartir / dependencias

- **Grupo (Task 2.1/2.2):** AUC temporal **0.81** es la base. Toda cifra de partición aleatoria es optimista.
- **Rol 1:** la tabla de umbrales U1–U4 viene de partición aleatoria y de validación balanceada 50/50. Recall y FPR reales serán peores, y la precisión debe recalcularse a 0.2% (p. ej. U2 ≈ 4%, no 0.95). Su punto de parada temprana (época 6) también debe elegirse con validación temporal.
- **Rol 2:** el 0.87 del ajuste fino de dos bloques es de partición aleatoria. La comparación entre estrategias debe repetirse con partición temporal antes de darla por válida.
- **Rol 4:** su 0.81 → 0.83 sí es temporal y comparable con mi base. Condición: la prueba debe ser solo con datos reales de abr–may, sin sintéticos derivados de remitentes de validación (el 3% de casi copias podría filtrar información).
- **Rol 7:** recall@40 alertas/día es la métrica que conecta modelo y capacidad. El piloto con U2 (≈250 alertas, ≈4% confirmadas) confirma que la precisión a prevalencia real es baja.
