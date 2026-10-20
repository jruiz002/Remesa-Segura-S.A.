# Rol 4 - Especialista en modelos generativos | Task 1.4 | Memo §4

## 1.4.a

- **GAN (2 de 7 tipologías; repite una sola forma de fraccionamiento) → colapso de modo.** El objetivo es adversarial: el generador solo necesita engañar al discriminador, no cubrir toda la distribución de positivos. No hay término de verosimilitud ni de cobertura que lo penalice por ignorar tipologías. Si encuentra un modo que el discriminador acepta (un patrón de fraccionamiento), le conviene quedarse ahí. Por eso sus sintéticos no aportan información nueva: **el AUC no cambia**.
- **VAE (5 de 7 tipologías; secuencias borrosas) → promedio bajo incertidumbre.** Maximiza el ELBO: reconstrucción tipo gaussiana/MSE más un término KL que empuja el latente hacia N(0, I). La KL obliga a que el latente sea suave y solapado, y con pérdida MSE la mejor reconstrucción ante incertidumbre es la **media condicional**. El resultado cubre más modos que la GAN, pero promedia montos y borra los patrones exactos que definen una tipología (p. ej., montos justo bajo un umbral de reporte).
- **Difusión (6 de 7 tipologías; mayor diversidad y nitidez).** Aprende a revertir el ruido paso a paso con un objetivo de eliminación de ruido que equivale a modelar la distribución completa. No colapsa como la GAN ni promedia como el VAE. El costo: **3 días de entrenamiento** (frente a 4 h de la GAN y 2 h del VAE), y su fidelidad trae riesgo de **memorizar** muestras reales (ver 1.4.c).

## 1.4.b

- **Recomendación:** usar sintéticos **solo de difusión** y **solo para aumentar el entrenamiento**. Con ellos el AUC temporal pasa de **0.81 a 0.83 (+0.02)**; con la GAN no cambia, así que se descarta. El VAE no se recomienda porque pierde los patrones exactos.
- **Datos de entrenamiento:** positivos y negativos reales de **enero a marzo**, más sintéticos de difusión. El generador se entrena **solo con positivos del periodo de entrenamiento** para que no se filtre información del periodo de prueba.
- **Datos de prueba:** **solo datos reales** con partición temporal (validación **abril y mayo**, posterior al entrenamiento). **Nunca sintéticos**: medirían qué tan bien el modelo reconoce al generador, no qué tan bien detecta lavado real.
- **Advertencias:** +0.02 es una mejora modesta, y la tipología que ningún generador cubre (1 de 7) sigue sin representación. Hay que reportar el **recall por tipología** en la validación temporal, y no solo el AUC agregado, para confirmar que la mejora no se concentra en las tipologías que ya se detectaban.

## 1.4.c

El **3% de las secuencias sintéticas de difusión** son casi idénticas a secuencias reales de clientes. Eso es memorización: compartirlas o usarlas expone datos personales y financieros. Mitigación propuesta:

1. **Filtrar:** descartar cada sintético cuya distancia al vecino real más cercano quede bajo un umbral, que se fija con la auditoría de privacidad.
2. **Atacar la causa:** reentrenar con **privacidad diferencial (DP-SGD)** o, como mínimo, deduplicar el set de entrenamiento del generador para reducir la memorización.
3. **Auditar antes de usar o compartir:** prueba de **membership inference** y repetición de la auditoría de casi duplicados, con resultado documentado.
4. **Contener:** los sintéticos no salen del entorno controlado de RemesaSegura; no se comparten con terceros (p. ej., el socio del codificador).

Después del filtrado hay que **volver a medir el AUC temporal**: parte del +0.02 podría venir de las secuencias memorizadas.

## Sección 4 del memo

Recomendamos usar datos sintéticos **solo de difusión** y **solo en entrenamiento**: cubre 6 de 7 tipologías y sube el AUC temporal de **0.81 a 0.83**, mientras que la GAN (2 de 7) no mejora el AUC. La prueba se hace exclusivamente con datos reales de abril y mayo. Como el **3% de los sintéticos son casi copias de clientes reales**, se exige antes del uso: filtrado por distancia al vecino real más cercano, auditoría de membership inference y prohibición de compartirlos fuera de la empresa. El +0.02 es modesto y debe confirmarse tras el filtrado con recall por tipología; la tipología restante (1 de 7) sigue sin cobertura.

## Cifras a compartir / dependencias

- **Rol 3:** el AUC temporal base es **0.81** (dato del Rol 3) y con difusión sube a **0.83** (+0.02). Con GAN no cambia. La prueba es solo con datos reales de abril y mayo; hay que conciliarlo en Task 2.1 (cruce 3–4).
- **Rol 6:** el **3%** de los sintéticos de difusión son casi idénticos a clientes reales. Es un riesgo de privacidad y regulatorio que afecta la defensa ante el regulador.
- **Grupo:** el +0.02 es optimista hasta que se vuelva a medir tras el filtrado; la cobertura es de 6 de 7 tipologías; la difusión cuesta **3 días** de entrenamiento, lo que importa para el plazo del lunes. Posible condición de lanzamiento: sintéticos solo después del filtrado y la auditoría de privacidad.
