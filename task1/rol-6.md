# Rol 6 - Responsable de explicabilidad | Task 1.6 | Memo §6

## 1.6.a

- **SHAP (100 secuencias de fondo), error de eficiencia máx. 3 × 10⁻⁷ log-odds.** La propiedad de eficiencia (aditividad) se cumple: las contribuciones de cada alerta suman la diferencia entre la predicción y el valor base, con un error numérico despreciable. Esto confirma que la descomposición es exacta, pero **no** demuestra fidelidad causal ni estabilidad. Además, el valor base y las atribuciones dependen del fondo elegido (100 secuencias), así que conviene medir cuánto cambian con otros fondos.
- **LIME, 500 muestras → Jaccard 0.45 en el top-5 (20 corridas).** Para dos conjuntos de 5 variables, J = k/(10−k). J = 0.45 equivale a k ≈ 3.1, es decir, unas **3 de las 5** variables se repiten entre corridas (3/7 = 0.43). Es inestable: la misma alerta recibe explicaciones distintas en cada ejecución. **Con 5,000 muestras → 0.78** (k ≈ 4.4, entre 4 y 5 de 5). Mejora, pero aún varía y cuesta 10× más evaluaciones del modelo.
- **Atención vs SHAP, correlación de rangos 0.35.** La concordancia es débil. La atención muestra *dónde mira* el modelo, no *cuánto contribuye* cada entrada al puntaje. No es una explicación fiel.
- **Recomendación:** **SHAP** como explicación oficial. **LIME no** como explicación oficial; a lo sumo como apoyo exploratorio con ≥ 5,000 muestras. **La atención no** se usa como explicación.

## 1.6.b

| Audiencia | Qué recibe | Justificación |
|---|---|---|
| **Oficial de cumplimiento** | Top-5 variables SHAP de cada alerta, con su signo y magnitud, y las transacciones concretas que las originan (montos, fechas, contrapartes). | Necesita investigar y decidir si reporta. Las variables deben poder rastrearse hasta hechos verificables. |
| **Regulador** | Metodología (modelo de dos etapas, SHAP y su fondo), validación (AUC temporal 0.81, no 0.87 aleatoria; pruebas de estabilidad y fidelidad de 1.6.c), importancia global SHAP, registro auditable de cada alerta (puntaje, umbral, SHAP, versión del modelo, revisor y decisión) y limitaciones (dependencia del fondo, inestabilidad de LIME, atención descartada). | Obligación de explicar el motivo de cada alerta y demostrar que el sistema está controlado y es reproducible. |
| **Cliente señalado** | Comunicación genérica conforme a la ley (por ejemplo, "operación en revisión según la normativa vigente") y un canal de atención y reclamo. Nunca recibe variables, lógica del modelo ni contrafactuales. En AML normalmente ni se le informa que existe un reporte (prohibición de *tipping-off*). | Revelar la lógica le permitiría evadir el sistema y alertaría a un posible sospechoso. |

## 1.6.c

**Sentido de la política.** Decirle a un cliente "qué tendría que cambiar para evitar la alerta" es un **contrafactual**. En AML funciona como un manual de evasión: por ejemplo, fraccionar montos por debajo de los umbrales o espaciar los envíos. Además, equivale a avisar al sospechoso (*tipping-off*). La política protege la eficacia del sistema y cumple la ley.

**Validación adicional exigida antes del lanzamiento** (los umbrales son **propuesta de Rol 6**):

1. **Fidelidad de SHAP (borrado/perturbación).** Se reemplazan las top-5 variables SHAP por valores del fondo y se mide la caída del puntaje. *Criterio propuesto:* caída media ≥ 2× la que produce borrar 5 variables al azar, en ≥ 90% de las alertas de la validación temporal (abril–mayo).
2. **Estabilidad de SHAP.** Se recalcula con 3 fondos distintos de 100 secuencias y distintas semillas. *Criterio propuesto:* Jaccard top-5 ≥ 0.90 entre fondos, por encima del 0.78 de LIME, y error de eficiencia ≤ 10⁻⁶.
3. **Cero contrafactuales hacia el cliente.** Se revisan todas las plantillas y salidas dirigidas al cliente. *Criterio:* 0 salidas con variables, umbrales o sugerencias de cambio.
4. **Coherencia con las narrativas del LLM (Rol 5).** *Criterio propuesto:* 100% de las variables citadas en cada narrativa aparecen en el top-5 SHAP de su alerta y 0 transacciones inexistentes (como la de USD 9,800).

## Sección 6 del memo

Recomendamos SHAP como explicación oficial de cada alerta: su error de eficiencia es ≤ 3 × 10⁻⁷ log-odds, así que la descomposición es exacta. LIME es inestable (Jaccard top-5 de 0.45 con 500 muestras, ≈ 3 de 5 variables; 0.78 con 5,000, a 10× el costo) y la atención no explica el puntaje (ρ = 0.35 con SHAP). El oficial recibe el top-5 SHAP con sus transacciones; el regulador, la metodología, el AUC temporal 0.81 y un registro por alerta; el cliente, solo un aviso legal genérico, sin contrafactuales (*tipping-off*). Condición de lanzamiento: aprobar las pruebas de fidelidad y estabilidad de SHAP (Jaccard ≥ 0.90 entre fondos, propuesta) y que el 100% de las narrativas LLM coincidan con SHAP.

## Cifras a compartir / dependencias

- **Rol 5:** cada narrativa debe citar solo variables del top-5 SHAP y transacciones reales de la alerta (criterio de 100% de coincidencia). Las narrativas no salen hacia el cliente.
- **Rol 7:** el oficial recibe el top-5 SHAP y sus transacciones, lo que agiliza la revisión. LIME a 5,000 muestras cuesta 10× y no se usa en el flujo diario. El registro por alerta es un requisito operativo.
- **Rol 3 / grupo:** al regulador se le reporta el AUC temporal 0.81, no el 0.87 de la partición aleatoria.
- **Rol 4 / grupo:** el fondo SHAP debe salir de datos reales, no de sintéticos de difusión (3% casi duplicados de clientes reales).
- **Grupo (defensa regulatoria):** SHAP con eficiencia 3 × 10⁻⁷; LIME 0.45 → 0.78; atención ρ = 0.35 (descartada); prohibición de contrafactuales y *tipping-off*; pruebas de fidelidad y estabilidad como condición medible de lanzamiento.
