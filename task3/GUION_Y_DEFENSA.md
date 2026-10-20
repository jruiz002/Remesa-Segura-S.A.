# Task 3 - Presentación y defensa

Diapositiva: `task3/slide.pdf` (fuente: `slide.html`). Memo: `task2/memo.pdf`.

## 3.1 Guion de 2 minutos (≈280 palabras)

Lo puede decir una persona o repartirse por bloques. **Ensayar una vez con cronómetro** y anotar el tiempo: ______.

**[0:00–0:20] Decisión.**
"Recomendamos **lanzar con condiciones**. La regla que manda es que cada alerta debe revisarla una persona de cumplimiento, y hoy el equipo puede revisar 40 alertas al día."

**[0:20–0:50] La cifra que más pesó.**
"En el piloto con U2 el sistema generó unas **250 alertas diarias para 40 revisiones posibles**. Además, la precisión de 0.95 que reportó modelado venía de una validación mitad positivos y mitad negativos. Con la prevalencia real de 0.2%, la precisión es **3.9%**: unas 25 de cada 26 alertas son falsas. El AUC también estaba inflado: con partición temporal baja de 0.87 a **0.81**."

**[0:50–1:20] Umbral y capacidad.**
"Por eso el lunes arrancamos en **U4 con los 8 analistas**: unas 36 alertas diarias, que caben en la capacidad, por USD 431 al día. Luego bajamos a U3 con 14 analistas y llegamos a **U2 con 50**, el único punto que el piloto corrobora, con unos 10 de 12 casos diarios detectados. Cada paso se justifica: pasar de U4 a U2 cuesta USD 2,558 diarios en revisión y evita unos USD 180,000 de daño esperado."

**[1:20–1:45] Condiciones.**
"Las condiciones son medibles:
- cero alertas sin revisar;
- reentrenar con parada temprana y pérdida ponderada, y revalidar con partición temporal antes de bajar de U4;
- narrativas del LLM con las transacciones en contexto, validador automático y firma del oficial;
- SHAP como explicación, sin contrafactuales al cliente;
- sintéticos de difusión solo después del filtro de privacidad."

**[1:45–2:00] Riesgo y responsable.**
"El principal riesgo: en U4 dejamos de detectar unos **6 de 12 casos diarios** hasta ampliar el equipo, y lo reconocemos ante el regulador. La decisión la asume la Gerencia General a propuesta de este comité; el Oficial de Cumplimiento responde por la operación."

## 3.1.b Las siete secciones en una línea (todos deben saberlas)

| § | Idea | Cifra |
|---|---|---|
| 1 Modelado | Sobreajuste: parar en la época 6, regularizar, usar BCE ponderada o focal | Validación 0.21 → 0.34; precisión real U2 3.9% |
| 2 Transferencia | Ajuste fino de 2 bloques con LR 10× menor; el ajuste completo sobreajusta con 1,200 positivos | AUC 0.87 / 18% parámetros / 1.5 GPU-h; temporal 0.81 |
| 3 Series de tiempo | La partición aleatoria filtra información entre ventanas del mismo remitente; ventana ≥ 31 días por los ciclos 7/15/30 | AUC 0.87 → 0.81; exactitud trivial 99.8% |
| 4 Generativos | Difusión solo para entrenar y prueba solo con datos reales; la GAN colapsa a un modo y el VAE promedia | 6 de 7 tipologías; AUC 0.81 → 0.83; 3% casi copias |
| 5 LLM | Borrador con contexto y T = 0.2, validador automático y firma humana | Alucinación 15% → 5%; ahorro 4.5 min (75%) |
| 6 Explicabilidad | SHAP sí; LIME inestable; atención no; sin contrafactuales al cliente | SHAP 3×10⁻⁷; LIME 0.45 / 0.78; ρ = 0.35 |
| 7 Operaciones | Capacidad 40 vs 250 → U4 al inicio y ampliar a U3 y U2 | 8 × 5 = 40; 250 × 12 = USD 3,000; 10 de 12 = 83% |

## 3.2 Preguntas probables y respuestas cortas

### Decisión y números
- **¿Por qué no lanzar en U2 directamente?** Con 8 analistas solo se revisan 40 de 250 alertas; 210 diarias quedarían sin revisión humana, y el caso exige revisar cada una.
- **¿Por qué no "no lanzar"?** Cada día sin sistema se pierde detección. Incluso U4 detecta unos 6 casos diarios con alertas que caben en la capacidad. Bajar el umbral evita más daño del que cuesta (U4→U3: USD 381 frente a USD 90,000).
- **¿Por qué no U1, si minimiza el costo total?** Exige 122 analistas, tiene precisión de 1.8% (fatiga de revisión) y no está validado; el piloto solo corrobora U2.
- **¿Cómo calcularon las alertas?** TP = recall × 12; FP = FPR × 5,988; alertas = TP + FP. U2: 9.6 + 239.5 = 249.1. Precisión 9.6 / 249.1 = 3.85%.
- **¿Por qué 0.95 y 3.9% son ambas ciertas?** Recall y FPR no dependen de la prevalencia; la precisión sí. Con 50/50 hay tantos positivos como negativos; con 0.2% los negativos son 499 veces más.
- **¿Qué cifra es optimista?** El AUC 0.87 (partición aleatoria), la precisión de 0.90 a 0.99 (50/50), el AUC 0.83 (antes del filtro de privacidad), el 5% de alucinación (n = 40) y el propio piloto: con 40 revisiones diarias no se revisaron las 250 alertas, así que el 4% viene de una muestra.

### Por sección
- **(1) ¿Qué muestran las curvas?** El entrenamiento baja de 0.42 a 0.08 mientras validación sube de 0.21 (época 6) a 0.34: sobreajuste. Se evaluó el peor checkpoint (época 20).
- **(1) ¿Por qué no BCE sin ponderar?** Con 0.2% de positivos los negativos dominan la pérdida, y predecir "nunca" ya da pérdida baja. Con pos_weight ≈ 499 (0.998 / 0.002) o focal loss se fuerza a aprender los positivos.
- **(2) ¿Por qué el ajuste completo rinde menos con menor pérdida de entrenamiento?** Sobreajusta: tiene 100% de parámetros para 1,200 positivos y olvida lo aprendido con 3 millones de secuencias (olvido catastrófico).
- **(2) ¿Riesgo de usar el modelo del socio?** Otra escala de normalización de montos y otra distribución de montos. Se verifica re-normalizando y midiendo la deriva (PSI o KS) con validación temporal propia.
- **(3) ¿Por qué ≥ 31 días y no 30?** Para observar el rezago de 30 hacen falta dos días separados por 30, y una ventana de 30 días solo cubre rezagos de 0 a 29.
- **(3) ¿Qué métrica en vez de exactitud?** Recall, precisión a prevalencia real, PR-AUC y recall con 40 alertas diarias. Un sistema que nunca alerta logra 99.8% de exactitud y detecta 0 casos.
- **(4) ¿Por qué la GAN cubre pocas tipologías?** Por colapso de modo: el generador solo necesita engañar al discriminador y se queda en un patrón que funciona.
- **(4) ¿Por qué el VAE es borroso?** La reconstrucción tipo MSE más la KL empuja hacia la media condicional, que promedia montos y borra patrones exactos.
- **(4) ¿Cómo se valida con sintéticos?** Se entrena con reales más sintéticos de enero a marzo y se prueba solo con reales de abril y mayo. Nunca se prueba con sintéticos.
- **(5) ¿Por qué alucina el LLM?** Genera el token más probable, no consulta hechos. Sin las transacciones inventa montos verosímiles, como USD 9,800, justo bajo el umbral de USD 10,000. El contexto le da hechos que copiar y la temperatura baja reduce el muestreo de tokens improbables.
- **(6) ¿Por qué SHAP y no atención?** El error de eficiencia de SHAP es 3×10⁻⁷, así que las contribuciones suman exactamente el puntaje. La atención solo correlaciona 0.35 con SHAP: indica dónde mira el modelo, no cuánto aporta cada entrada.
- **(6) ¿Por qué no decirle al cliente qué cambiar?** Es un contrafactual que enseña a evadir el sistema, por ejemplo fraccionando montos, y alerta al sospechoso (*tipping-off*).
- **(7) ¿Cuántos casos detectó el piloto?** 4% × 250 = 10 por día, el 83% de 12. Ojo: 12 es una estimación y el 4% salió de una muestra.

### Uso de la IA (pregunta segura)
- **¿Cómo usaron la IA?** Un agente por rol para la Task 1, un segundo agente verificador independiente y la integración con IA. Cada cifra se recalculó por separado.
- **¿Dónde falló?**
  - Mezcló bases en U4→U2 (USD 2,520 frente a 2,558 correctos).
  - Propuso una ventana de 30 días que no capta el rezago de 30.
  - Tomó el 4% del piloto sin preguntar cómo se revisaron 250 alertas con capacidad para 40.
  - Dio una pérdida de referencia (0.0144) que contradice el dato del caso.
  - Presentó umbrales propios, como Jaccard ≥ 0.90, sin fuente.
- **¿Cómo verificaron?** Recalculando a mano y con un script independiente, y comparando cada cifra con el PDF del examen. Cada integrante debe poder rehacer su corrección a mano (ver contribuciones en el memo).
- **¿Quién responde por la decisión?** La Gerencia General, a propuesta del comité técnico. El Oficial de Cumplimiento responde por la operación y el reporte. La IA no responde: es una herramienta.

## Checklist antes de la defensa
- [ ] Ensayo cronometrado de 2:00 hecho (tiempo: ____)
- [ ] Cada integrante explicó en voz alta las 7 líneas de la tabla
- [ ] Cada integrante puede rehacer a mano su corrección a la IA
- [ ] Alguien recalculó U2 a mano (Task 2.3.b)
