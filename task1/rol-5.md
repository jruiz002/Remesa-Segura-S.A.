# Rol 5 - Líder de lenguaje natural | Task 1.5 | Memo §5

## 1.5.a Tasas de alucinación y su causa

| Configuración | Hechos inventados | Tasa | IC 95% (Wilson, n = 40) |
|---|---:|---:|---:|
| Sin transacciones en contexto, T = 0.7 | 6/40 | **15%** | 7.1% – 29.1% |
| Con transacciones en contexto, T = 0.2 | 2/40 | **5%** | 1.4% – 16.5% |

- Reducción relativa: (15 − 5) / 15 = **67%**.
- **Advertencia muestral:** con 40 narrativas por configuración la estimación es muy incierta. El 5% podría ser en realidad hasta ~16.5%, y los dos intervalos se traslapan, así que la mejora es plausible pero no está demostrada estadísticamente. Hace falta una auditoría más grande antes de fijar una cifra definitiva.
- **Por qué ocurre:** un LLM genera el token más plausible condicionado al contexto; no consulta una base de hechos. Si no tiene las transacciones de la alerta, completa la narrativa con montos *verosímiles*. El USD 9,800 inventado está justo por debajo del umbral de reporte de USD 10,000 en EE. UU., un patrón "típico" de fraccionamiento que el modelo aprendió de textos de lavado. Es estadísticamente plausible y fácticamente falso.
- **Por qué ayudan contexto y temperatura:** el *grounding* (transacciones en el prompt) le da al modelo los hechos que debe copiar en lugar de inventar. Una temperatura baja (0.2) concentra el muestreo en los tokens de mayor probabilidad y reduce los desvíos creativos. **Ninguna de las dos medidas elimina el problema:** un 5% de narrativas con hechos falsos sigue siendo inaceptable en un reporte regulatorio si no hay revisión.

## 1.5.b Ahorro de tiempo y condiciones de uso regulado

- Ahorro por narrativa: 6 − 1.5 = **4.5 min (75%)**. Los 1.5 min ya incluyen la revisión humana.
- **Escala realista (capacidad del equipo):** 8 analistas × 5 alertas = 40 alertas/día → 40 × 4.5 = **180 min/día (3 h)** ahorrados.
- Solo como referencia, con las ~250 alertas/día del piloto el ahorro sería 1,125 min ≈ 18.75 h/día. **No es alcanzable:** el cuello de botella es la capacidad de revisión (40/día), no la redacción. El LLM no resuelve el exceso de alertas.
- **Condiciones para usarlo en un proceso regulado:**
  1. Usar solo la configuración con transacciones en contexto y **T ≤ 0.2**. La configuración sin contexto queda prohibida.
  2. El LLM produce un **borrador**: el oficial de cumplimiento lo edita, lo firma y responde por él.
  3. **Validación automática:** cada monto, fecha e ID citado debe existir en la secuencia de la alerta; si alguno no existe, la narrativa se bloquea.
  4. **Trazabilidad:** registrar el prompt, la versión del modelo, los datos usados y el editor humano.
  5. **Auditoría muestral periódica** con meta de **0 hechos inventados en narrativas enviadas** al regulador.
  6. La narrativa debe ser **consistente con la explicación SHAP** del Rol 6 (mismas variables y transacciones como motivo de la alerta).
  7. No enviar datos de clientes a servicios externos sin control (contrato, anonimización o despliegue interno).

## Sección 5 del memo

Recomendamos usar el LLM solo como redactor de borradores, con las transacciones de la alerta en contexto y temperatura 0.2. Esta configuración baja las narrativas con hechos inventados de 15% a 5% (6/40 → 2/40) y ahorra 4.5 de 6 min por narrativa (75%), unos 180 min/día con las 40 alertas que el equipo puede revisar. Como 5% sigue siendo inaceptable y n = 40 deja un IC 95% de hasta ~16.5%, ninguna narrativa se envía sin que el oficial la firme y sin que un validador automático compruebe que cada monto, fecha e ID existe en la alerta. Además exigimos trazabilidad completa, auditoría muestral con meta de 0 hechos inventados y coherencia con la explicación SHAP.

## Cifras a compartir / dependencias

- **→ Rol 7:** ahorro de 4.5 min/narrativa (75%); 180 min/día con 40 alertas/día (8 × 5). Las 18.75 h/día con 250 alertas no son alcanzables por capacidad. Tasas de alucinación: 15% vs 5% (IC 95% del 5%: 1.4%–16.5%). La revisión humana sigue siendo obligatoria y el LLM no amplía la capacidad de revisión.
- **→ Rol 6:** la narrativa debe citar las mismas transacciones y variables que la explicación SHAP. Ninguna narrativa dirigida al cliente debe revelar qué cambiar para evitar la alerta (política de cumplimiento). El validador de hechos complementa la explicabilidad ante el regulador.
- **← Rol 7:** capacidad de 40 alertas/día (dato usado para escalar el ahorro).
