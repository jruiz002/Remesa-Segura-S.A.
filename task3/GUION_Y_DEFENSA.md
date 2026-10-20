# Task 3 - Guion y defensa

- Presentación completa (16 diapositivas): https://claude.ai/artifact/ABU4Z5atZXANhjqqo6vxw4
- Diapositiva única: `task3/slide.pdf`
- Explicación a fondo por rol: `task3/README.md`

Hay dos versiones del guion:

| Versión | Cuándo usarla | Duración | Material |
|---|---|---|---|
| **A. Oficial** | Exposición calificada (Task 3.1) | **2:00** | `slide.pdf` (una sola diapositiva) |
| **B. Completa** | Ensayo, estudio, respaldo durante las preguntas o si el docente da más tiempo | ≈ 7:30 | Presentación de 16 diapositivas |

---

## A. Guion oficial de 2 minutos (una diapositiva)

Lo puede decir una persona o repartirse en bloques. **Ensayar una vez con cronómetro** y anotar el tiempo: ______.

| Tiempo | Habla | Texto |
|---|---|---|
| 0:00–0:20 | José Auyón | «Recomendamos **lanzar con condiciones**. La regla que manda es que cada alerta la revisa una persona de cumplimiento, y hoy el equipo puede revisar 40 alertas al día.» |
| 0:20–0:50 | José Ruíz | «En el piloto con U2 el sistema generó unas **250 alertas diarias para 40 revisiones posibles**. La precisión de 0.95 venía de una validación 50/50; con la prevalencia real de 0.2% es **3.9%**: unas 25 de cada 26 alertas son falsas. Y con partición temporal el AUC baja de 0.87 a **0.81**.» |
| 0:50–1:20 | Alex de la Cruz | «Por eso el lunes arrancamos en **U4 con los 8 analistas**: unas 36 alertas diarias, que caben, por USD 431 al día. Luego U3 con 14 analistas y la meta de **U2 con 50**, el único punto que el piloto corrobora, con unos 10 de 12 casos detectados. Pasar de U4 a U2 cuesta USD 2,558 diarios y evita unos USD 180,000 de daño esperado.» |
| 1:20–1:45 | Dillary Cruz | «Las condiciones son medibles: cero alertas sin revisar; modelo reentrenado y revalidado con partición temporal antes de bajar de U4; narrativas del LLM con validador automático y firma del oficial; SHAP como explicación, sin contrafactuales al cliente; y sintéticos solo después del filtro de privacidad.» |
| 1:45–2:00 | Gerardo Fernández | «El principal riesgo: en U4 dejamos de detectar unos **6 de 12 casos diarios** hasta ampliar el equipo, y lo reconocemos ante el regulador. La decisión la asume la Gerencia General a propuesta de este comité; el Oficial de Cumplimiento responde por la operación.» |

---

## B. Guion completo por diapositiva

**Reparto:** cada integrante presenta la diapositiva de su rol (4–10) y una más de introducción o de integración. Todos hablan unas dos veces, así que todos practican.

| Diap. | Tema | Habla | Tiempo |
|---:|---|---|---:|
| 1–3 | Portada, caso y decisión | José Auyón | 0:50 |
| 4 | §1 Modelado | José Ruíz | 0:30 |
| 5 | §2 Transferencia | José Auyón | 0:30 |
| 6 | §3 Validación temporal | Gerardo Fernández | 0:30 |
| 7 | §4 Sintéticos | David Juárez | 0:30 |
| 8 | §5 LLM | Juan Diego Solis | 0:30 |
| 9 | §6 Explicabilidad | Dillary Cruz | 0:30 |
| 10 | §7 Operaciones | Alex de la Cruz | 0:30 |
| 11 | Conciliación de cifras | Gerardo Fernández | 0:30 |
| 12 | Tabla de umbrales | José Ruíz | 0:35 |
| 13 | Plan y condiciones | Alex de la Cruz | 0:30 |
| 14 | Riesgo y regulador | Dillary Cruz | 0:30 |
| 15 | Crítica de la IA | Juan Diego Solis | 0:30 |
| 16 | Equipo y cierre | David Juárez | 0:15 |
| | **Total** | | **≈ 7:30** |

Cada diapositiva trae:
- **Decir:** el texto para leer en voz alta.
- **Señalar:** el dato de la pantalla al que se apunta.
- **Pasa a:** la frase que entrega la palabra al siguiente.
- **Si preguntan:** la pregunta más probable sobre esa diapositiva y su respuesta.

---

### Diapositiva 1 · Portada — José Auyón (0:10)

**Decir:** «Buenas. Somos el comité técnico de RemesaSegura. La gerencia nos pidió decidir si el sistema de detección de lavado sale el próximo lunes, y venimos a darles nuestra recomendación.»

**Señalar:** la pregunta del título.

### Diapositiva 2 · El caso — José Auyón (0:20)

**Decir:** «Cada día analizamos unos **6,000 remitentes**. El 0.2% lava dinero: unos **12 casos por día**. El sistema tiene dos etapas: primero aprende cómo se ve un remitente normal y mide cuánto se aparta; luego un clasificador asigna un puntaje de riesgo. Si el puntaje supera un umbral, se genera una alerta. Y la regla que manda todo: **el 100% de las alertas debe revisarlas una persona**.»

**Señalar:** la tarjeta oscura del 100%.

### Diapositiva 3 · Decisión — José Auyón (0:20)

**Decir:** «Nuestra decisión es **lanzar con condiciones**. El lunes arrancamos en U4, el umbral más exigente, porque es el único cuyas alertas caben en lo que el equipo puede revisar. Después bajamos el umbral por etapas a medida que crece el equipo. Cada rol les va a mostrar de dónde sale esta decisión.»

**Pasa a:** «Empieza José con el modelo.»

**Si preguntan:**
- *¿Qué es U4?* El más alto de los cuatro umbrales: genera unas 36 alertas diarias y detecta 6 de 12 casos.
- *¿Por qué no "no lanzar"?* Sin sistema se pierden detecciones todos los días; U4 ya detecta unos 6 casos diarios con alertas que el equipo sí puede revisar.

---

### Diapositiva 4 · §1 Modelado — José Ruíz (0:30)

**Decir:** «El modelo que se evaluó está sobreajustado. La pérdida de entrenamiento bajó de 0.42 a 0.08, pero la de validación tocó su mínimo de 0.21 en la época 6 y subió a 0.34 en la época 20. Y se evaluó la época 20, el peor punto. Proponemos dos cambios:
- **parada temprana** con weight decay y dropout, contra el sobreajuste;
- **pérdida ponderada** o focal, contra el desbalance, porque con 0.2% de positivos la entropía cruzada simple se minimiza diciendo casi siempre que no.

Y un dato clave: la precisión de 0.95 se midió con mitad positivos y mitad negativos. Con la prevalencia real es **3.9%**.»

**Señalar:** el 3.9% de la tarjeta oscura.

**Pasa a:** «Ese modelo se construye sobre un codificador de un socio. José explica cómo adaptarlo.»

**Si preguntan:**
- *¿Por qué baja tanto la precisión?* El recall y la FPR no dependen de la proporción de positivos, pero la precisión sí. Con 499 negativos por positivo: 9.6 TP frente a 239.5 FP, o sea 3.9%.
- *¿Qué es pos_weight 499?* 0.998 / 0.002: cada positivo pesa como 499 negativos en la pérdida.

### Diapositiva 5 · §2 Transferencia — José Auyón (0:30)

**Decir:** «El socio nos da un codificador entrenado con 3 millones de secuencias, y nosotros solo tenemos 1,200 positivos. De las cinco estrategias elegimos **ajustar los dos últimos bloques con tasa de aprendizaje 10 veces menor**: mejor AUC de la tabla, **0.87**, con solo 18% de parámetros y 1.5 GPU-horas. El ajuste completo logra menor pérdida de entrenamiento pero peor AUC, 0.84: con 100% de parámetros y tan pocos positivos sobreajusta y olvida lo que traía. Antes de usarlo hay que alinear la escala de montos del socio y medir la deriva. Ojo: 0.87 es con partición aleatoria.»

**Señalar:** el 0.87 y la nota «con partición temporal: 0.81».

**Pasa a:** «Gerardo explica por qué ese 0.87 no es la cifra real.»

**Si preguntan:**
- *¿Por qué no LoRA?* Queda a 0.01 de AUC con 2% de parámetros y es la alternativa. Si con partición temporal iguala 0.81, la preferimos por barata.

### Diapositiva 6 · §3 Validación temporal — Gerardo Fernández (0:30)

**Decir:** «Las cifras anteriores usaron partición aleatoria: ventanas del mismo remitente quedaron en entrenamiento y en validación, así que el modelo reconocía al remitente. Con **partición temporal**, entrenando con enero a marzo y validando con abril y mayo, el AUC baja de 0.87 a **0.81**. Esa es la cifra para gerencia. La autocorrelación muestra picos a 7, 15 y 30 días, los ciclos de pago, así que la ventana debe tener **al menos 31 días**. Y la exactitud no sirve: un sistema que nunca alerta logra 99.8% y detecta cero casos.»

**Señalar:** el 0.81.

**Pasa a:** «Con tan pocos positivos, David evaluó si conviene generar datos sintéticos.»

**Si preguntan:**
- *¿Por qué 31 y no 30?* Para ver el rezago de 30 hacen falta dos días separados por 30; una ventana de 30 días solo cubre rezagos de 0 a 29.
- *¿Qué métricas usan?* Recall, precisión a prevalencia real, PR-AUC y recall con 40 alertas diarias.

### Diapositiva 7 · §4 Sintéticos — David Juárez (0:30)

**Decir:** «Probamos tres generadores para crear casos sintéticos de lavado:
- **GAN:** cubre solo 2 de 7 tipologías por colapso de modo; repite el patrón que engaña al discriminador.
- **VAE:** cubre 5 pero promedia los montos y pierde los patrones exactos.
- **Difusión:** cubre 6 de 7 y sube el AUC temporal de 0.81 a 0.83, **+0.02**.

La usamos solo para entrenar y probamos solo con datos reales de abril y mayo. Pero el 3% de lo generado casi copia a clientes reales, así que primero filtramos y auditamos privacidad.»

**Señalar:** el +0.02.

**Pasa a:** «Cada alerta necesita una narrativa escrita. Juan Diego explica el uso del LLM.»

**Si preguntan:**
- *¿Por qué no probar con sintéticos?* Sería circular: el modelo se evaluaría con datos de la misma fuente que lo entrenó.

### Diapositiva 8 · §5 LLM — Juan Diego Solis (0:30)

**Decir:** «Un LLM redacta el borrador de la narrativa de cada alerta. Sin las transacciones en contexto y con temperatura 0.7, el 15% de las narrativas inventaba hechos, como una transacción de 9,800 dólares que no existía. Con las transacciones en contexto y temperatura 0.2 baja a **5%**. Pero 5% sigue siendo inaceptable en un proceso regulado. Por eso exigimos:
- que sea solo un borrador;
- un validador automático de montos, fechas e IDs;
- la firma del oficial.

Ahorra 4.5 de 6 minutos por narrativa.»

**Señalar:** el 5%.

**Pasa a:** «Además el regulador exige explicar cada alerta. Dillary.»

**Si preguntan:**
- *¿Por qué alucina?* Genera el token más probable, no consulta hechos. El contexto le da datos para copiar y la temperatura baja reduce las opciones improbables.

### Diapositiva 9 · §6 Explicabilidad — Dillary Cruz (0:30)

**Decir:** «Usamos **SHAP** como explicación oficial. Su error de eficiencia es de 3 por 10 a la menos 7, así que las contribuciones suman exactamente el puntaje. LIME es inestable: entre corridas solo coinciden unas 3 de 5 variables. La atención casi no coincide con SHAP, 0.35. Cada audiencia recibe algo distinto:
- **el oficial:** las variables y transacciones;
- **el regulador:** la metodología y el registro;
- **el cliente:** solo un aviso genérico.

Nunca le decimos qué cambiar para evitar la alerta, porque le enseñaría a evadir el sistema.»

**Señalar:** el 3×10⁻⁷.

**Pasa a:** «Todo esto tiene que caber en lo que el equipo puede revisar. Alex.»

**Si preguntan:**
- *¿Qué validación extra exigen?* Fidelidad (quitar las 5 variables principales debe bajar el puntaje más que quitar 5 al azar) y estabilidad con distintos fondos.

### Diapositiva 10 · §7 Operaciones — Alex de la Cruz (0:30)

**Decir:** «Tenemos 8 analistas que revisan 5 alertas cada uno: **40 al día**. El piloto en U2 generó 250, más de **6 veces** la capacidad. Revisarlas todas costaría 3,000 dólares diarios y requeriría 50 analistas. El piloto confirmó unos 10 casos diarios, el 83% de los 12 esperados. Así que no podemos lanzar en U2 sin dejar 210 alertas diarias sin revisar. Empezamos en U4 y crecemos por etapas.»

**Señalar:** el 6.25×.

**Pasa a:** «Al juntar las siete secciones, varias cifras no cuadraban. Gerardo.»

**Si preguntan:**
- *¿Cómo midieron el 4% si solo podían revisar 40?* Tuvo que ser una muestra o personal extra, y los 12 casos son una estimación. Por eso lo marcamos como cifra optimista.

---

### Diapositiva 11 · Conciliación — Gerardo Fernández (0:30)

**Decir:** «Cinco cifras chocaban:
- **Precisión 0.95 frente a 4%:** ambas son ciertas. Una es de validación 50/50 y la otra de la prevalencia real; usamos 3.9%.
- **AUC 0.87 frente a 0.81:** aleatoria frente a temporal; usamos 0.81.
- **AUC 0.83:** se midió antes del filtro de privacidad, así que aún no cuenta.
- **Tabla frente a piloto en U2:** coinciden, unas 250 alertas. Eso nos da confianza en U2, pero U3 y U4 no están validados.

Las cifras optimistas son la precisión reportada, el AUC aleatorio y el 4% del piloto, que salió de una muestra.»

**Señalar:** la columna «Usamos».

**Pasa a:** «Con las cifras reconciliadas, José calculó cada umbral.»

### Diapositiva 12 · Umbrales — José Ruíz (0:35)

**Decir:** «Con la tabla de umbrales, 6,000 remitentes y 0.2% calculamos cada umbral:
- detectados = recall por 12;
- falsas alarmas = FPR por 5,988;
- costo = alertas por 12 dólares;
- daño = casos no detectados por 50,000.

**Solo U4 cabe en 40 revisiones**: unas 36 alertas. Pero bajar el umbral siempre conviene: pasar de U4 a U3 cuesta 381 dólares diarios y evita 90,000. U1 tendría el menor costo total, pero exige 122 analistas y nadie lo ha validado.»

**Señalar:** la fila resaltada «U4 · lunes» y luego «U2 · meta».

**Pasa a:** «Alex presenta el plan por etapas.»

**Si preguntan:**
- *Calcule U2 a mano:* 0.80 × 12 = 9.6; 0.04 × 5,988 = 239.5; alertas 249.1; precisión 3.85%; costo USD 2,989; daño 2.4 × 50,000 = USD 120,000.

### Diapositiva 13 · Plan y condiciones — Alex de la Cruz (0:30)

**Decir:** «Tres etapas:
- **el lunes, U4 con 8 analistas:** unas 36 alertas y 6 de 12 casos;
- **luego, U3 con 14 analistas:** unos 7.8 casos;
- **la meta, U2 con 50:** unos 10 de 12, el punto que el piloto corrobora.

Cada etapa exige condiciones medibles: cero alertas sin revisar; modelo reentrenado con AUC temporal de al menos 0.81 y recall de 0.80; narrativas con cero hechos inventados; SHAP validado; y sintéticos solo filtrados.»

**Señalar:** las tres tarjetas y luego la lista numerada.

**Pasa a:** «Ninguna decisión está libre de riesgo. Dillary.»

### Diapositiva 14 · Riesgo y regulador — Dillary Cruz (0:30)

**Decir:** «El principal riesgo: mientras operemos en U4 dejamos de detectar unos **6 de 12 casos diarios**, unos 300,000 dólares de daño esperado al día. Ante el regulador lo reconocemos y presentamos la evidencia: el AUC temporal de 0.81, el piloto, la explicación SHAP de cada alerta y el registro de cada revisión humana. Nos comprometemos a reportar cada mes recall, precisión, deriva y la cola de alertas sin revisar en cero. La decisión la asume la Gerencia General; el Oficial de Cumplimiento responde por la operación.»

**Señalar:** el «≈6 de 12».

**Pasa a:** «Usamos IA en todo el proceso, y también la auditamos. Juan Diego.»

### Diapositiva 15 · Crítica de la IA — Juan Diego Solis (0:30)

**Decir:** «Usamos un agente de IA por rol y un segundo agente que verificaba al primero, y recalculamos todo a mano. Encontramos errores reales:
- mezcló bases en un costo: 2,520 en vez de 2,558;
- propuso una ventana de 30 días que no capta el rezago de 30;
- aceptó el 4% del piloto sin preguntar cómo se revisaron 250 alertas con capacidad para 40;
- dio una pérdida de referencia que contradecía el caso;
- puso criterios sin fuente.

Cada uno se verificó a mano o contra el PDF.»

**Señalar:** la columna «Verificación».

**Pasa a:** «David cierra.»

### Diapositiva 16 · Equipo — David Juárez (0:15)

**Decir:** «Somos siete roles con una sola decisión: lanzar con condiciones, porque es lo que el equipo puede sostener hoy sin dejar alertas sin revisar, y porque nos da un camino medible para detectar más. Cualquiera de nosotros puede explicar las siete secciones. Gracias, quedamos atentos a sus preguntas.»

---

## Las siete secciones en una línea (todos deben saberlas)

| § | Idea | Cifra |
|---|---|---|
| 1 Modelado | Sobreajuste: parar en la época 6, regularizar, usar BCE ponderada o focal | Validación 0.21 → 0.34; precisión real U2 3.9% |
| 2 Transferencia | Ajuste fino de 2 bloques con LR 10× menor; el completo sobreajusta con 1,200 positivos | AUC 0.87 / 18% / 1.5 GPU-h; temporal 0.81 |
| 3 Series de tiempo | La partición aleatoria filtra información entre ventanas; ventana ≥ 31 días por los ciclos 7/15/30 | AUC 0.87 → 0.81; exactitud trivial 99.8% |
| 4 Generativos | Difusión solo para entrenar y prueba solo con reales; la GAN colapsa a un modo y el VAE promedia | 6 de 7 tipologías; AUC 0.81 → 0.83; 3% casi copias |
| 5 LLM | Borrador con contexto y T = 0.2, validador y firma humana | 15% → 5%; ahorro 4.5 min (75%) |
| 6 Explicabilidad | SHAP sí; LIME inestable; atención no; sin contrafactuales al cliente | SHAP 3×10⁻⁷; LIME 0.45 / 0.78; ρ = 0.35 |
| 7 Operaciones | 40 de capacidad frente a 250 → U4 al inicio, luego U3 y U2 | 8 × 5 = 40; 250 × 12 = USD 3,000; 10 de 12 = 83% |

## Banco de preguntas para la persona sorteada

### Decisión y números
- **¿Qué es U4?** El más alto de los cuatro umbrales de puntaje. Genera unas 36 alertas diarias (caben en 40) y detecta 6 de 12 casos.
- **¿Por qué no lanzar en U2 directamente?** Con 8 analistas solo se revisan 40 de 250 alertas; 210 diarias quedarían sin revisión humana, y el caso exige revisar cada una.
- **¿Por qué no "no lanzar"?** Cada día sin sistema se pierden detecciones. Bajar el umbral evita más daño del que cuesta (U4→U3: USD 381 frente a USD 90,000).
- **¿Por qué no U1, si minimiza el costo total?** Exige 122 analistas, tiene precisión de 1.8% (fatiga de revisión) y no está validado; el piloto solo corrobora U2.
- **¿Por qué 0.95 y 3.9% son ambas ciertas?** Recall y FPR no dependen de la prevalencia; la precisión sí. Con 50/50 hay tantos positivos como negativos; con 0.2% los negativos son 499 veces más.
- **¿Qué cifras son optimistas?** El AUC 0.87, la precisión de 0.90 a 0.99, el AUC 0.83 sin filtrar, el 5% con n = 40 y el 4% del piloto (salió de una muestra).

### Uso de la IA
- **¿Cómo la usaron?** Un agente por rol para la Task 1, un segundo agente verificador independiente y la integración con IA. Cada cifra se recalculó por separado y contra el PDF.
- **¿Dónde falló?** En los 5 casos de la diapositiva 15; cada integrante tiene el suyo en la tabla de contribuciones del memo.
- **¿Quién responde por la decisión?** La Gerencia General, a propuesta del comité. La IA es una herramienta y no responde.

Las preguntas por sección y sus respuestas están en `task3/README.md`.

## Checklist antes de la defensa
- [ ] Ensayo de la versión A cronometrado (tiempo: ____)
- [ ] Ensayo de la versión B completo, con las entregas de palabra (tiempo: ____)
- [ ] Cada integrante explicó en voz alta las 7 líneas de la tabla
- [ ] Cada integrante puede rehacer a mano su corrección a la IA
- [ ] Alguien recalculó U2 a mano (Task 2.3.b)
- [ ] La presentación está compartida con el grupo (menú Share)
