# Task 3 - Cómo explica cada rol

Presentación completa (16 diapositivas): https://claude.ai/artifact/ABU4Z5atZXANhjqqo6vxw4
Diapositiva única para los 2 minutos oficiales: `task3/slide.pdf`. Guion y banco de preguntas: `task3/GUION_Y_DEFENSA.md`.

> **Ojo con el tiempo:** la exposición oficial dura **2 minutos** y la Task 3.1 pide **una sola diapositiva**. Para los 2 minutos usen `slide.pdf` con el guion de `GUION_Y_DEFENSA.md`. La presentación completa sirve para estudiar y como respaldo en las preguntas.

Cada rol tiene:
- **Qué decir:** unos 30 segundos, si hay tiempo o si le preguntan.
- **Cómo lo explico:** la lógica para entenderlo, no solo repetirlo.
- **Si me preguntan:** las respuestas cortas a lo más probable.

---

## Introducción (cualquiera) · diapositivas 1–3

**Qué decir:** «Somos el comité técnico de RemesaSegura. Cada día analizamos 6,000 remitentes y esperamos 12 casos de lavado (0.2%). Cada alerta la revisa una persona. Recomendamos **lanzar con condiciones**: el lunes arrancamos en U4 con los 8 analistas y bajamos a U3 y U2 a medida que crece el equipo.»

**Cómo lo explico:** todo el caso gira alrededor de una restricción, que cada alerta la revise una persona. Por eso el umbral no lo elige el mejor modelo, sino la capacidad del equipo.

---

## Rol 1 · José Ruíz · Modelado y entrenamiento · diapositiva 4

**Qué decir:** «El modelo que se evaluó está sobreajustado. La pérdida de validación tocó su mínimo de 0.21 en la época 6 y subió a 0.34 en la época 20, pero se evaluó la época 20. Proponemos dos cambios: parada temprana con regularización y una pérdida ponderada. Además, la precisión de 0.95 venía de una validación mitad positivos y mitad negativos; con la prevalencia real es **3.9%**.»

**Cómo lo explico:**
- **Curvas:** si la pérdida de entrenamiento baja y la de validación sube, el modelo memoriza en vez de generalizar. El mejor modelo es el de la época 6.
- **Cambio 1 (parada temprana + weight decay/dropout):** corrige el sobreajuste. Se guarda el checkpoint con menor pérdida de validación y se penalizan pesos grandes.
- **Cambio 2 (BCE ponderada o focal):** corrige el desbalance. Con 0.2% de positivos, decir «nunca» casi no cuesta pérdida. `pos_weight = 0.998 / 0.002 ≈ 499` hace que cada positivo pese como 499 negativos. La focal loss baja el peso de los ejemplos fáciles.
- **Precisión 3.9%:** TP = 0.80 × 12 = 9.6; FP = 0.04 × 5,988 = 239.5; precisión = 9.6 / 249.1 = 3.85%.

**Si me preguntan:**
- *¿Por qué cambia la precisión y no el recall?* El recall solo mira positivos y la FPR solo negativos, así que no dependen de la proporción. La precisión mezcla ambos: con 499 negativos por positivo, los falsos positivos dominan.
- *Mi corrección a la IA:* dio una pérdida de 0.014 para un predictor constante sin notar que contradice el 0.08 de entrenamiento. A mano: −(0.002·ln 0.002 + 0.998·ln 0.998) = 0.0144.

---

## Rol 2 · José Auyón · Transferencia y ajuste fino · diapositiva 5

**Qué decir:** «Recomendamos ajustar solo los dos últimos bloques del codificador del socio, con tasa de aprendizaje 10 veces menor. Es el mejor AUC de la tabla, **0.87**, con solo 18% de parámetros entrenables y 1.5 GPU-horas. El ajuste completo rinde menos porque sobreajusta con 1,200 positivos. LoRA es la alternativa barata. Y a gerencia se le reporta 0.81, el AUC temporal.»

**Cómo lo explico:**
- **Por qué dos bloques:** las capas tempranas aprendieron patrones generales de 3 millones de secuencias y conviene conservarlas. Las últimas se adaptan al corredor nuevo. La tasa 10× menor evita borrar lo aprendido.
- **Por qué el ajuste completo rinde menos:** tiene 100% de parámetros para solo 1,200 positivos, así que memoriza. Por eso llega a menor pérdida de entrenamiento pero peor AUC (0.84), y además olvida lo que traía (olvido catastrófico).
- **LoRA r=8:** AUC 0.86 con 2% de parámetros y 1 GPU-h. Si en validación temporal iguala 0.81, conviene por más barata.
- **Riesgos del socio:** otra escala de normalización de montos (hay que re-normalizar) y otra distribución de montos (deriva, que se mide con PSI o KS). También puede haber patrones de lavado distintos.

**Si me preguntan:**
- *¿Por qué no extracción de características, si es la más barata?* Con el codificador congelado no se adapta al corredor: AUC 0.82.
- *Mi corrección a la IA:* ordenó las estrategias con AUC de partición aleatoria. Solo la elegida tiene AUC temporal (0.81), así que el orden de las demás no está validado.

---

## Rol 3 · Gerardo Fernández · Validación temporal y métricas · diapositiva 6

**Qué decir:** «Las métricas de los roles 1 y 2 están infladas porque usaron partición aleatoria. Con partición temporal, entrenando con enero a marzo y validando con abril y mayo, el AUC baja de 0.87 a **0.81**: esa es la cifra para gerencia. Por los ciclos de 7, 15 y 30 días recomendamos una ventana de al menos 31 días. Y la exactitud no sirve: no alertar nunca da 99.8%.»

**Cómo lo explico:**
- **Fuga:** al repartir ventanas al azar, ventanas del mismo remitente, casi idénticas por autocorrelación, quedan en entrenamiento y en validación. El modelo reconoce al remitente en lugar de generalizar. Además entrena con el «futuro» para predecir el «pasado».
- **Picos de autocorrelación:** 7 días es el ciclo semanal, 15 el quincenal y 30 el mensual (pagos de salario). Un monto alto el día de pago es normal.
- **Ventana ≥ 31 días:** para ver el rezago de 30 hacen falta dos días separados por 30. Una ventana de 30 días solo cubre rezagos de 0 a 29.
- **Métricas:** recall (cuántos de los 12 detectamos), precisión a prevalencia real, PR-AUC y recall con 40 alertas/día, que es la capacidad real.

**Si me preguntan:**
- *¿Por qué ROC-AUC también puede engañar?* Con tantos negativos, una FPR pequeña ya son cientos de falsos positivos, y ROC no lo refleja. PR-AUC sí.
- *Mi corrección a la IA:* propuso una ventana de 30 días para captar el rezago de 30; se corrigió a ≥ 31.

---

## Rol 4 · David Juárez · Datos sintéticos y privacidad · diapositiva 7

**Qué decir:** «Recomendamos sintéticos de difusión, solo para entrenar. Cubren 6 de 7 tipologías y suben el AUC temporal de 0.81 a **0.83**. La GAN no ayuda y el VAE produce secuencias borrosas. La prueba se hace solo con datos reales de abril y mayo. Como el 3% de los sintéticos casi copia a clientes reales, primero filtramos y auditamos.»

**Cómo lo explico:**
- **GAN, 2 de 7 tipologías:** sufre colapso de modo. El generador solo necesita engañar al discriminador y, cuando encuentra un patrón que funciona (un fraccionamiento), lo repite. Nada lo obliga a cubrir toda la distribución.
- **VAE, borroso:** optimiza una reconstrucción tipo MSE más una KL hacia N(0, I). Ante la incertidumbre, la mejor reconstrucción es el promedio, así que suaviza montos y borra patrones exactos, como los montos justo bajo un umbral.
- **Difusión:** aprende a quitar ruido paso a paso y modela bien la distribución completa. Cuesta 3 días y puede memorizar, de ahí el 3% de casi copias.
- **Validación:** el generador se entrena solo con positivos de enero a marzo y la prueba usa solo reales de abril y mayo. Si se probara con sintéticos, la medición sería circular.
- **Privacidad:** filtrar por distancia al vecino real más cercano, auditar membership inference (o entrenar con DP-SGD) y no sacar los sintéticos de la empresa.

**Si me preguntan:**
- *¿+0.02 vale la pena?* Es modesto y se midió antes del filtro de privacidad: hay que repetirlo después. La tipología 7 sigue sin cubrirse.
- *Mi corrección a la IA:* propuso filtrar por distancia sin fijar un umbral; el criterio quedó en 0 casi duplicados tras la auditoría.

---

## Rol 5 · Juan Diego Solis · Narrativas con LLM · diapositiva 8

**Qué decir:** «El LLM solo redacta borradores. Con las transacciones de la alerta en contexto y temperatura 0.2, los hechos inventados bajan de 15% a **5%**. Pero 5% sigue siendo inaceptable, así que exigimos un validador automático de montos y la firma del oficial. Ahorra 4.5 de 6 minutos por narrativa.»

**Cómo lo explico:**
- **Tasas:** 6/40 = 15% y 2/40 = 5%, una reducción del 67%. Con n = 40 el intervalo de confianza del 5% llega hasta ~16.5%.
- **Por qué alucina:** el LLM genera el token más probable dado el contexto; no consulta una base de datos. Sin las transacciones, rellena con un monto verosímil: los USD 9,800, justo bajo el umbral de reporte de 10,000, un patrón «típico» que aprendió.
- **Por qué ayudan contexto y temperatura:** el contexto le da hechos para copiar en lugar de inventar. La temperatura baja concentra el muestreo en los tokens más probables. Ninguno de los dos elimina el problema.
- **Ahorro:** 6 − 1.5 = 4.5 min (75%), unos 180 min/día con 40 alertas.

**Si me preguntan:**
- *¿Qué condiciones?* Contexto con las transacciones, T ≤ 0.2, validador que bloquea montos, fechas o IDs inexistentes, trazabilidad (prompt, versión, editor), auditoría mensual y coherencia con SHAP.
- *Mi corrección a la IA:* afirmó que el ahorro no aumenta la capacidad, pero el caso no da el tiempo total de revisión. Quedó como limitación, no como conclusión.

---

## Rol 6 · Dillary Cruz · Explicabilidad · diapositiva 9

**Qué decir:** «SHAP es la explicación oficial: su error de eficiencia es **3×10⁻⁷**, así que las contribuciones suman exactamente el puntaje. LIME es inestable y la atención no explica. Cada audiencia recibe algo distinto. Al cliente nunca le decimos qué cambiar para evitar la alerta.»

**Cómo lo explico:**
- **SHAP:** la propiedad de eficiencia dice que la suma de contribuciones es igual a la predicción menos el valor base. Un error de 3×10⁻⁷ significa que se cumple. No prueba causalidad y depende del fondo de 100 secuencias.
- **LIME:** un Jaccard de 0.45 entre conjuntos de 5 variables equivale a unas 3 de 5 iguales entre corridas (J = k / (10 − k)). Con 5,000 muestras mejora a 0.78 (unas 4.4 de 5), con 10 veces más costo.
- **Atención:** su correlación con SHAP es 0.35. Dice dónde «mira» el modelo, no cuánto aporta cada entrada al puntaje.
- **Audiencias:**
  - Oficial: top-5 SHAP y las transacciones que lo originan, para investigar.
  - Regulador: metodología, AUC 0.81, pruebas de estabilidad y registro por alerta.
  - Cliente: aviso genérico, sin la lógica del modelo.

**Si me preguntan:**
- *¿Por qué la política prohíbe decirle al cliente qué cambiar?* Sería un contrafactual que enseña a evadir el sistema, por ejemplo fraccionando montos, y alertaría al sospechoso (*tipping-off*).
- *¿Qué validación extra antes de lanzar?* Fidelidad (quitar las top-5 variables SHAP debe bajar el puntaje más que quitar 5 al azar), estabilidad con distintos fondos y coherencia de las narrativas con SHAP.
- *Mi corrección a la IA:* presentó umbrales como Jaccard ≥ 0.90 sin fuente en los datos; se marcaron como propuesta.

---

## Rol 7 · Alex de la Cruz · Operaciones y cumplimiento · diapositiva 10

**Qué decir:** «8 analistas por 5 alertas son **40 revisiones al día**. El piloto en U2 generó 250: más de 6 veces la capacidad, 3,000 dólares diarios y 50 analistas para revisarlas todas. El piloto confirmó unos 10 casos diarios, el 83% de los 12 esperados. Por eso arrancamos en U4 y crecemos por etapas.»

**Cómo lo explico:**
- **Capacidad:** 8 × 5 = 40 alertas/día (USD 480). Piloto: 250 × 12 = USD 3,000/día, o sea 6.25× la capacidad, con 210 alertas sin revisar.
- **Detección:** 4% × 250 = 10 confirmados/día, que son 10/12 = 83%.
- **Decisión:** U4 da unas 36 alertas y cabe en la capacidad. Pasar de U4 a U2 cuesta (249.1 − 35.9) × 12 = USD 2,558/día en revisión y evita 3.6 × 50,000 = USD 180,000/día.

**Si me preguntan:**
- *¿Cómo se midió el 4% si solo podían revisar 40?* Buena pregunta: tuvo que ser una muestra o personal extra, y los 12 casos son una estimación. Por eso marcamos el piloto como cifra optimista.
- *Mi corrección a la IA:* calculó U4 → U2 como 210 × 12 = USD 2,520 mezclando bases; con la misma base es USD 2,558.

---

## Integración (cualquiera) · diapositivas 11–16

- **Conciliación (11):** recall y FPR no dependen de la prevalencia, la precisión sí (0.95 → 3.9%). La partición aleatoria infla el AUC (0.87 → 0.81). La tabla y el piloto coinciden en U2 (249 frente a ≈250 alertas).
- **Umbrales (12):** detectados = recall × 12; falsos positivos = FPR × 5,988; revisión = alertas × 12; daño = no detectados × 50,000. Solo U4 cabe en 40.
- **Plan (13):** U4 con 8 analistas, luego U3 con 14 y la meta de U2 con 50, más 5 condiciones medibles.
- **Riesgo (14):** en U4 quedan unos 6 de 12 casos/día sin detectar (≈USD 300,000/día). Decide la Gerencia General y el Oficial de Cumplimiento opera.
- **IA (15):** 5 errores reales, cada uno verificado a mano o contra el PDF.
- **Por qué no U1:** exige 122 analistas, tiene precisión de 1.8% y no está validado.
- **Por qué no «no lanzar»:** cada día sin sistema se pierden detecciones, y U4 ya detecta unos 6 casos diarios con alertas que caben en la capacidad.
