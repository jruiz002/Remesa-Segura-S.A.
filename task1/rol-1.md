# Rol 1 - Líder de modelado | Task 1.1 | Memo §1

## 1.1.a Diagnóstico de las curvas de pérdida y dos cambios

**Diagnóstico: sobreajuste a partir de la época 6.** La pérdida de entrenamiento baja de forma continua (0.42 → 0.08), pero la de validación toca su mínimo de **0.21 en la época 6** y después sube hasta **0.34 en la época 20** (+0.13, un 62% más). A partir de la época 6 el modelo memoriza el conjunto de entrenamiento (donde hay muy pocos positivos) en lugar de generalizar. Además se evaluó y reportó el modelo de la **época 20**, con una pérdida de validación 0.13 mayor que la de la época 6: las métricas de la tabla corresponden a un modelo sobreajustado. Hay que agregar que el *nivel* absoluto de ambas pérdidas no se compara directamente, porque entrenamiento usa la prevalencia real y validación está remuestreada al 50/50. La señal de sobreajuste está en la *tendencia*: validación sube mientras entrenamiento sigue bajando.

| # | Cambio concreto | Qué problema corrige |
|---|---|---|
| 1 | **Parada temprana con restauración del mejor checkpoint**: monitorear la pérdida de validación, paciencia de unas 3 épocas, y quedarse con los pesos de la mejor época (aquí la 6, val = 0.21) en vez de los de la 20. | Que el modelo entregado siga entrenando después del punto óptimo. Elimina las 14 épocas de memorización y entrega el modelo con menor error de generalización observado. |
| 2 | **Regularización explícita**: *weight decay* (L2, p. ej. AdamW con 1e-4 a 1e-2) y *dropout* (0.2 a 0.3) en la cabeza clasificadora y en las capas densas. | La alta capacidad sin restricciones (hoy no hay ninguna regularización). Penaliza pesos grandes y la co-adaptación de neuronas, lo que reduce la brecha entre entrenamiento y validación y retrasa o suaviza el sobreajuste. |

Los dos cambios se complementan: la regularización reduce el sobreajuste y la parada temprana elige el mejor punto que queda. Para escoger la época y los hiperparámetros hay que usar una **validación temporal** (ver Rol 3), no una aleatoria.

## 1.1.b ¿Es apropiada BCE sin ponderar con 0.2% de positivos?

**No.** Con 0.2% de positivos, el 99.8% de los términos de la pérdida provienen de negativos, así que el gradiente lo dominan los negativos. El modelo puede bajar mucho la pérdida si predice casi siempre una probabilidad cercana a 0. Como referencia, un predictor constante p = 0.002 logra BCE ≈ 0.014 sin detectar ningún caso. Por eso un valor bajo de pérdida no garantiza que el modelo detecte lavado: los ~12 positivos diarios casi no influyen en el aprendizaje, y la frontera de decisión de los positivos queda mal estimada.

**Alternativa justificada:** BCE **ponderada** con `pos_weight = 0.998 / 0.002 = 499`, que hace que positivos y negativos aporten lo mismo en total a la pérdida. Si 499 vuelve inestable el entrenamiento (gradientes ruidosos con tan pocos positivos), conviene una versión moderada, p. ej. √499 ≈ 22, y fijar el valor por validación. Otra opción es la **focal loss** (γ ≈ 2, α según la clase), que baja el peso de los negativos fáciles y concentra el aprendizaje en los ejemplos difíciles. **Precauciones:**
- La ponderación o la focal loss **descalibran** las probabilidades (las inflan para la clase positiva). Hay que recalibrar después (Platt o isotónica) sobre datos con la prevalencia real.
- El **umbral** de alerta se fija al final, con la prevalencia real (0.2%) y la capacidad de revisión, no con 0.5 ni con la validación 50/50.

## Advertencia sobre la tabla de umbrales

Las métricas de la tabla se calcularon con **validación remuestreada 50/50**, **partición aleatoria** y el **modelo de la época 20**. Recall y FPR son tasas condicionadas a la clase y **no dependen de la prevalencia**. La **precisión sí depende**: con 50% de positivos se infla artificialmente. Con 6,000 remitentes/día y 0.2% hay **12 positivos y 5,988 negativos**: TP = recall × 12, FP = FPR × 5,988, precisión = TP / (TP + FP).

| Umbral | Recall | FPR | TP/día | FP/día | Alertas/día | Precisión reportada (50/50) | **Precisión real (0.2%)** |
|---|---:|---:|---:|---:|---:|---:|---:|
| U1 | 0.90 | 0.10 | 10.8 | 598.8 | ≈ 610 | 0.90 | **1.8%** |
| U2 | 0.80 | 0.04 | 9.6 | 239.5 | ≈ 249 | 0.95 | **3.9%** |
| U3 | 0.65 | 0.01 | 7.8 | 59.9 | ≈ 68 | 0.98 | **11.5%** |
| U4 | 0.50 | 0.005 | 6.0 | 29.9 | ≈ 36 | 0.99 | **16.7%** |

Comprobación: en U2 el cálculo da ≈ 249 alertas/día con 3.9% de precisión, lo que coincide con el piloto del Rol 7 (≈ 250 alertas/día, ~4% confirmadas). La precisión de 0.95 reportada no describe la operación real.

## Sección 1 del memo

**§1 Modelado.** El modelo evaluado (época 20) está sobreajustado: la pérdida de validación subió de 0.21 (época 6) a 0.34. Recomendamos reentrenar con parada temprana (restaurando el mejor checkpoint), *weight decay* y *dropout*, y BCE ponderada (`pos_weight` hasta 499) o focal loss, con recalibración posterior. La precisión reportada de 0.90–0.99 proviene de una validación 50/50. Con la prevalencia real de 0.2%, la precisión es de 1.8% (U1) a 16.7% (U4), y en U2 sería **3.9% con ≈ 249 alertas/día**, en línea con el piloto. Ningún umbral debe fijarse hasta recalcular la tabla con el modelo corregido sobre partición temporal y prevalencia real.

## Cifras a compartir / dependencias

- **Rol 3:** la tabla viene de partición **aleatoria** y validación **50/50**. Con partición temporal el AUC cae de 0.87 a 0.81, así que recall y FPR por umbral probablemente sean **optimistas**. Le pido la versión temporal para recalcular.
- **Rol 7:** alertas/día y precisión real por umbral (U2 ≈ 249 alertas y 3.9%, coherente con su piloto). Necesito su capacidad (8 × 5 = 40 alertas/día) y su costo (USD 12/alerta, USD 50,000 por caso no detectado) para elegir umbral.
- **Grupo (Task 2.2):** recall/FPR por umbral y la base de 12 positivos / 5,988 negativos. Supuestos que hay que declarar: (1) partición aleatoria, (2) validación 50/50 (solo la precisión se corrige con la prevalencia), (3) modelo de la **época 20**, que está sobreajustado. Las tres hacen que las cifras sean un **techo optimista**, no la expectativa de producción.
