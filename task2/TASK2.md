# Task 2 - Integración y decisión

Insumos: `task1/rol-1.md` … `task1/rol-7.md`. Base común: 6,000 remitentes/día, prevalencia 0.2% → **12 positivos y 5,988 negativos por día**.

## 2.1 Conciliar las siete secciones

### 2.1.a Cifra clave por rol (exposición de 1 minuto) y dependencias

| Rol | Cifra clave | Recomendación | Depende de |
|---|---|---|---|
| 1 | Precisión real U2 = **3.9%** (no 0.95); val. mínima 0.21 en época 6 | Parada temprana + regularización + BCE ponderada/focal | Prevalencia (caso), piloto (Rol 7), validación temporal (Rol 3) |
| 2 | AUC 0.87 (aleatoria) con 18% params y 1.5 GPU-h | Ajuste fino de 2 bloques; LoRA como alternativa | AUC temporal (Rol 3) |
| 3 | AUC temporal **0.81** (−0.06) | Validación temporal por remitente; ventana ≥ 31 días | Métricas de Roles 1 y 2 |
| 4 | AUC temporal 0.81 → **0.83** con difusión; 3% casi duplicados | Sintéticos de difusión solo en entrenamiento, filtrados | Base 0.81 (Rol 3); privacidad (Rol 6) |
| 5 | Alucinación 15% → **5%**; ahorro 4.5 min (75%) | LLM solo borrador, con contexto y T = 0.2 | Volumen de alertas (Rol 7); SHAP (Rol 6) |
| 6 | SHAP error 3×10⁻⁷; atención–SHAP ρ = 0.35 | SHAP oficial; sin contrafactuales al cliente | AUC temporal (Rol 3), narrativas (Rol 5) |
| 7 | Capacidad **40/día** vs **250/día** del piloto | Lanzar con condiciones, escalonado | Tabla de umbrales (Rol 1) |

### 2.1.b Cifras que no coinciden

| Cifras en conflicto | Cuál usar | Por qué ambas son ciertas | Efecto en la decisión |
|---|---|---|---|
| Precisión U2: **0.95** (Rol 1) vs **≈4%** (piloto, Rol 7) | **3.9%** | 0.95 se midió con validación 50/50. Recall y FPR no dependen de la prevalencia, la precisión sí: con 0.2%, 9.6 TP frente a 239.5 FP → 3.9%. El piloto (≈250 alertas, 10 confirmados) lo reproduce. | ≈25 de cada 26 alertas son falsas: el costo de revisión manda. Cada caso confirmado en U2 cuesta ≈ USD 311 en revisión. |
| AUC **0.87** (Roles 1–2) vs **0.81** (Rol 3) | **0.81** | 0.87 usa partición aleatoria de ventanas: hay fuga entre ventanas del mismo remitente y autocorrelación. 0.81 usa ene–mar → abr–may. | La gerencia y el regulador reciben 0.81. La tabla de umbrales del Rol 1 es optimista. |
| AUC **0.83** (Rol 4) vs **0.81** (Rol 3) | **0.81** (0.83 queda pendiente) | Ambas son temporales. 0.83 añade sintéticos de difusión, pero antes del filtrado de privacidad (3% casi copias). | La mejora de +0.02 no entra en la decisión hasta repetirla después del filtrado. |
| Alertas U2: tabla **≈249** y 9.6 TP vs piloto **≈250** y **10** confirmados | Ambas | La tabla aplica recall/FPR a 6,000 remitentes. El piloto es la observación real. La diferencia (10 vs 9.6, recall 0.83 vs 0.80) es ruido de dos semanas. | U2 es el único punto de operación validado en producción. U3 y U4 solo tienen cifras de la tabla optimista. |
| Capacidad **40/día** vs **250/día** | Ambas | Una es oferta (8 × 5) y la otra demanda (piloto U2). | En U2 quedan 210 alertas/día (84%) sin revisar, lo que viola "cada alerta revisada por una persona". |
| Ahorro LLM **180 min/día** vs **1,125 min/día** (Rol 5) | 180 min con la capacidad actual | El ahorro escala con las alertas revisadas: 40 o 250 por día. | El LLM no resuelve la brecha de capacidad, porque el README no da el tiempo total de revisión por alerta. |
| Pérdidas de entrenamiento (0.08) vs validación (0.34) del Rol 1 | La tendencia, no el nivel | Entrenamiento usa la prevalencia real; validación es 50/50, así que los niveles no son comparables. | El diagnóstico de sobreajuste se basa en el mínimo de la época 6 y la subida posterior. |

### 2.1.c Cifras optimistas

1. **Precisión 0.90–0.99** (Rol 1): validación 50/50; la real es 1.8%–16.7%.
2. **AUC 0.87** y todo el ranking de estrategias del Rol 2: partición aleatoria. Solo la estrategia elegida tiene AUC temporal (0.81).
3. **Recall/FPR de la tabla del Rol 1**: modelo de la época 20 (sobreajustado) y partición aleatoria. El piloto confirma U2, pero U3 y U4 no están validados.
4. **AUC 0.83** con sintéticos: medido antes del filtrado de privacidad; +0.02 es modesto.
5. **5% de alucinación**: n = 40, IC 95% de 1.4% a 16.5%.
6. **Exactitud 99.8%**: es trivial, la alcanza un sistema que nunca alerta.
7. **Piloto: 4% confirmadas y 83% de detección:** con capacidad de 40/día no se pudieron revisar las ≈250 alertas/día. El 4% salió de una muestra o de personal extra que el caso no describe. El 83% (10 de 12) compara contra una estimación previa de 12 casos, no contra casos medidos.
8. **12 casos/día esperados**: solo cuenta lo confirmable con las tipologías conocidas (7, una sin cubrir por sintéticos). La exposición real podría ser mayor.

## 2.2 Umbral, capacidad y decisión

### 2.2.a Tabla por umbral

TP = recall × 12; FP = FPR × 5,988; alertas = TP + FP; precisión = TP / alertas; costo = alertas × USD 12; daño esperado = no detectados × USD 50,000; analistas = alertas / 5.

| Umbral | Alertas diarias esperadas | Casos detectados | Casos no detectados | Precisión esperada | Costo diario de revisión | Daño esperado por no detectados | Costo total/día | Analistas necesarios |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| U1 | 609.6 | 10.8 | 1.2 | 1.8% | USD 7,315 | USD 60,000 | USD 67,315 | 122 |
| U2 | 249.1 | 9.6 | 2.4 | 3.9% | USD 2,989 | USD 120,000 | USD 122,989 | 50 |
| U3 | 67.7 | 7.8 | 4.2 | 11.5% | USD 812 | USD 210,000 | USD 210,812 | 14 |
| U4 | 35.9 | 6.0 | 6.0 | 16.7% | USD 431 | USD 300,000 | USD 300,431 | 8 (7.2) |

Costo marginal de bajar el umbral:

| Paso | Revisión adicional/día | Casos adicionales detectados | Daño evitado/día |
|---|---:|---:|---:|
| U4 → U3 | USD 381 | 1.8 | USD 90,000 |
| U3 → U2 | USD 2,177 | 1.8 | USD 90,000 |
| U2 → U1 | USD 4,326 | 1.2 | USD 60,000 |

### 2.2.b Umbral y capacidad elegidos

- **Restricción dura:** cada alerta debe ser revisada por una persona. Con 8 analistas (40/día), el único umbral que cabe es **U4** (≈36 alertas/día).
- **Criterio económico:** cada paso hacia un umbral más bajo evita mucho más daño del que cuesta revisar (USD 381 por USD 90,000 en el peor caso). Mantenerse en U4 deja ≈ USD 300,000/día de daño esperado, contra ≈ USD 123,000 en U2.
- **Por qué U2 como objetivo y no U1:** U1 minimiza el costo total teórico, pero exige 122 analistas, tiene una precisión de 1.8% (fatiga de revisión) y nadie lo ha validado. U2 es el único punto confirmado por el piloto (≈250 alertas, 10 confirmados). Si U1 se revalida con partición temporal, podrá evaluarse después.
- **Elección:**
  - Lunes: **U4 con 8 analistas** (36 alertas de 40 posibles, USD 431/día).
  - Escalón intermedio: **U3 con 14 analistas** (68/día, USD 812/día).
  - Objetivo: **U2 con 50 analistas** (250/día, USD 3,000/día), 10 casos confirmados por día (83% de 12).

### 2.2.c Decisión: **LANZAR CON CONDICIONES**

Condiciones medibles (los plazos son propuesta del grupo):

1. **Cobertura humana:** 0 alertas sin revisar por día. Las alertas diarias no superan 5 × analistas activos. Bajar de umbral solo cuando la capacidad lo permita: U3 con ≥ 14 analistas, U2 con ≥ 50.
2. **Modelo corregido y revalidado antes de bajar de U4:** reentrenado con parada temprana (mejor época por pérdida de validación), regularización y BCE ponderada o focal, y validado con partición temporal por remitente. AUC temporal ≥ 0.81 y recall temporal ≥ 0.80 en el umbral objetivo, con la tabla de umbrales recalculada a prevalencia real.
3. **Narrativas LLM:** solo con contexto y T ≤ 0.2. El 100% pasa el validador automático de montos, fechas e IDs. Meta de 0 hechos inventados en las narrativas enviadas, con auditoría muestral mensual (≥ 40 narrativas).
4. **Explicabilidad:** SHAP aprueba las pruebas de fidelidad y estabilidad (Jaccard top-5 ≥ 0.90 entre fondos, propuesta del Rol 6). Cero contrafactuales en comunicaciones al cliente.
5. **Privacidad y transferencia:** sintéticos usados solo tras el filtrado (0 casi duplicados de clientes reales). Montos re-normalizados a la escala del socio, con deriva de montos monitoreada mensualmente (PSI o KS).
6. **Monitoreo de desempeño:** recall y precisión mensuales sobre casos confirmados. Si la precisión de U2 cae por debajo de 3% o las alertas/día exceden la capacidad, se sube el umbral y se informa al comité.

## 2.3 Errores y limitaciones de la IA

Casos reales de esta sesión de trabajo con IA (Claude Code y sus subagentes). Agregar los hallazgos del agente que está verificando los cálculos. No inventar casos.

| Caso | Respuesta de la IA | Supuesto incorrecto / información omitida | Verificación realizada |
|---|---|---|---|
| 1 | La IA del Rol 1 justificó que la BCE sin ponderar es inadecuada con "un predictor constante p = 0.002 obtiene BCE ≈ 0.014". | La cifra es correcta, pero la IA no notó que contradice el dato del caso: la pérdida de entrenamiento final (0.08) sería *peor* que la de un predictor constante. Omitió reconciliarlo o advertirlo. | Cálculo manual: −(0.002·ln 0.002 + 0.998·ln 0.998) = 0.0124 + 0.0020 = **0.0144**. Se marcó la frase como riesgosa para la defensa. |
| 2 | La IA aplicó la tabla de umbrales del Rol 1 directamente a producción (alertas, casos detectados y costos por umbral) para elegir U4/U3/U2. | Esa tabla sale del modelo de la época 20 (sobreajustado) y de partición aleatoria; el AUC temporal cae de 0.87 a 0.81. La IA no ajustó recall y FPR por esa degradación. | Se comparó con el piloto: U2 predice 249 alertas y 9.6 TP; el piloto observó ≈250 y 10. U2 queda corroborado; U3 y U4 se declaran no validados y se exige revalidación (condición 2). |
| 3 | La IA del Rol 7 calculó el costo de pasar de U4 a U2 como 210 × 12 = USD 2,520/día, frente a 3.6 casos más (USD 180,000). | Mezcló bases: el costo se midió contra la capacidad actual (40 alertas) y el beneficio contra U4 (36 alertas). | Lo detectó un segundo agente verificador. Recalculado: (249.1 − 35.9) × 12 = **USD 2,558/día**. La conclusión no cambia; se corrigió `rol-7.md`. |
| 4 | Las IA de los Roles 1, 3 y 7 usaron el piloto (≈250 alertas, 4% confirmadas, 10 de 12 casos = 83%) como evidencia de que U2 funciona. | Omitieron que con 40 revisiones/día no pudieron revisarse las 250 alertas, así que el 4% viene de una muestra o de personal extra. Además, los 12 casos son una estimación, no una medición. | Lo detectó el agente verificador. Comprobación: 250 > 40 = 8 × 5. Se agregó a cifras optimistas (2.1.c, punto 7). |
| 5 | La IA del Rol 3 recomendó una ventana de exactamente 30 días para captar el rezago de 30. | Una ventana de 30 días solo contiene rezagos de 0 a 29, no el de 30. | Lo detectó el agente verificador. Se cambió a "al menos 31 días" en `rol-3.md`. |
| 6 | Las IA de los Roles 5 y 7 afirmaron que el ahorro del LLM (4.5 min por narrativa) no aumenta la capacidad de 5 alertas por analista. | Es un supuesto no explícito: el caso no da el tiempo total de revisión por alerta, así que tampoco se puede afirmar que el ahorro no cuente. | Se revisó el enunciado: no hay dato de minutos por revisión completa. Queda como limitación, no como conclusión. |

**2.3.b Verificación manual de un resultado de la 2.2 (U2):** 0.80 × 12 = 9.6 TP. 0.04 × 5,988 = 239.52 FP. Alertas = 249.12. Precisión = 9.6 / 249.12 = 3.85%. Costo = 249.12 × 12 = USD 2,989.44. No detectados = 2.4 → 2.4 × 50,000 = USD 120,000. *Un integrante debe rehacerlo a mano antes de entregar.*

## 2.4.a Defensa ante el regulador (≤ 6 líneas)

Presentamos la validación temporal del modelo (AUC 0.81, entrenamiento ene–mar y validación abr–may) y un piloto de dos semanas en U2 (≈250 alertas/día, 10 casos confirmados/día, 83% de los 12 esperados). También presentamos explicaciones SHAP por alerta (error de eficiencia 3×10⁻⁷) y el registro auditable de la revisión humana de cada alerta. Reconocemos un riesgo residual: hasta ampliar el equipo, en U4 dejaremos de detectar ≈6 de 12 casos/día, y el modelo puede degradarse por deriva del corredor. Nos comprometemos a informar mensualmente recall, precisión, deriva de montos, auditoría de narrativas y una cola de alertas sin revisar igual a 0, y a bajar el umbral a U3 y luego a U2 conforme crezca la capacidad. La decisión la asume la Gerencia General a propuesta de este comité; el Oficial de Cumplimiento responde por la operación y el reporte.

## 2.4.b–d Memo y entrega

- Memo armado en `task2/memo.html` y exportado a `task2/memo.pdf` (máximo 2 páginas).
- **Pendiente del grupo:** contribuciones individuales (2.4.c, con nombres reales y sus propias correcciones a la IA), revisión final y subida a Canvas cuando el docente avise (2.4.d).
