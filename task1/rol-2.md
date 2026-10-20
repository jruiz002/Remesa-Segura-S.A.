# Rol 2 - Ingeniero de transferencia | Task 1.2 | Memo §2

Datos: codificador del socio preentrenado con 3 millones de secuencias de otro corredor; RemesaSegura tiene 1,200 positivos etiquetados. Las cinco estrategias se evaluaron con **partición aleatoria**.

## 1.2.a

**Estrategia elegida: ajuste fino de los dos últimos bloques con tasa de aprendizaje 10× menor.**

- **AUC:** 0.87, la mejor de la tabla. Supera por 0.09 a entrenar desde cero (0.78), por 0.05 a la extracción de características (0.82) y por 0.03 al ajuste fino completo (0.84).
- **Parámetros entrenables:** 18%, contra el 100% del ajuste completo. Hay menos capacidad para memorizar los 1,200 positivos.
- **GPU-horas:** 1.5, contra 6 del ajuste completo o de entrenar desde cero (4× menos cómputo).
- **Comparación con LoRA r=8 (0.86 | 2% | 1 GPU-h):** LoRA queda a solo 0.01 de AUC y usa 9× menos parámetros (2% frente a 18%) y 0.5 GPU-h menos. Es una **alternativa válida con menor riesgo de sobreajuste**. Una diferencia de 0.01 bajo partición aleatoria puede no ser significativa. Además, solo el ajuste de dos bloques tiene AUC temporal medido (0.81, Rol 3). Se elige esa estrategia como candidata y LoRA queda como respaldo: si con partición temporal LoRA iguala o supera a 0.81, se prefiere LoRA por ser más barata y más robusta.

**¿Por qué el ajuste fino completo rinde menos (0.84) aunque logra una pérdida de entrenamiento mucho menor?**
1. **Sobreajuste.** Con 100% de parámetros entrenables y solo 1,200 positivos, el modelo memoriza el conjunto de entrenamiento. La pérdida de entrenamiento baja, pero el modelo no generaliza a validación.
2. **Olvido catastrófico.** Actualizar todo el codificador con el LR completo borra las representaciones generales aprendidas con 3 millones de secuencias, y esos pocos datos propios no bastan para reconstruirlas.
3. **Por qué el ajuste parcial evita ambos problemas.** Congelar los bloques tempranos conserva los rasgos generales (patrones de secuencia, periodicidad, montos), y el LR 10× menor limita cuánto se alejan los bloques superiores de los pesos preentrenados. Ambos actúan como regularización.

## 1.2.b

| Riesgo | Por qué importa | Cómo verificarlo / mitigarlo |
|---|---|---|
| **Escala de normalización de montos distinta** | El codificador recibe montos en otra escala, así que sus activaciones y desviaciones quedan desplazadas sin que nada lo indique. | Re-normalizar nuestros montos con la escala del socio, o reajustar la capa de entrada. Verificar que media, desviación y percentiles de la entrada normalizada coincidan con los del preentrenamiento. |
| **Cambio de distribución (dominio)** | Las distribuciones de montos de los dos corredores son diferentes, así que lo que el modelo considera "normal" no es el comportamiento normal de Guatemala–EE. UU. | Comparar las distribuciones de montos con pruebas KS o PSI. Revisar la distribución de la puntuación de desviación de la etapa 1 en datos propios. Validar con partición temporal en datos de RemesaSegura. |
| **Patrones de lavado distintos entre corredores** | Las tipologías del otro corredor pueden no coincidir con las 7 conocidas aquí, y el codificador puede no captar algunas. | Medir el recall por tipología en datos propios y revisar los falsos negativos con cumplimiento. |
| **AUC inflado por partición aleatoria** | Ventanas del mismo remitente quedan en entrenamiento y en validación (fuga de información). | Reportar solo el AUC temporal: **0.87 → 0.81** (enero–marzo / abril–mayo, Rol 3). Re-evaluar LoRA con la misma partición. |
| **Gobernanza y procedencia del modelo del socio** | Ante el regulador hay que justificar con qué datos se entrenó el modelo, si había consentimiento para usarlos y quién responde por él. | Obtener documentación de los datos, el preprocesamiento y la licencia. Versionar los pesos recibidos. |

## Sección 2 del memo

Recomendamos adaptar el codificador del socio con **ajuste fino de los dos últimos bloques y LR 10× menor**: es la mejor estrategia evaluada (AUC 0.87) con 18% de parámetros entrenables y 1.5 GPU-h, frente a 0.84 del ajuste completo, que sobreajusta con solo 1,200 positivos. LoRA r=8 (0.86, 2% de parámetros, 1 GPU-h) queda como alternativa de menor riesgo. **El AUC que debe reportarse a gerencia es el temporal, 0.81, no 0.87**, porque esta última cifra se midió con partición aleatoria. Condición previa al lanzamiento: re-normalizar los montos a la escala del socio y verificar la deriva entre corredores con PSI o KS.

## Cifras a compartir / dependencias

- **Para Rol 3:** las 5 estrategias se midieron con partición aleatoria. Solo la elegida tiene AUC temporal (0.87 → 0.81, una caída de 0.06). LoRA (0.86) y las demás estrategias aún no tienen AUC temporal.
- **Para Rol 4:** el AUC base para medir datos sintéticos es el temporal 0.81 de esta estrategia.
- **Para el grupo (Task 2.1):** la cifra optimista es 0.87 y la que se concilia es 0.81. El costo de entrenamiento es bajo (1.5 GPU-h). Riesgos abiertos: escala de normalización, deriva de dominio y procedencia del modelo del socio.
- **Dependencia:** la elección final entre el ajuste de dos bloques y LoRA depende de la validación temporal del Rol 3.
