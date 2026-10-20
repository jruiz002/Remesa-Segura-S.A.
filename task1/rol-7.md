# Rol 7 - Responsable de operaciones y cumplimiento | Task 1.7 | Memo §7

Datos: 8 analistas × 5 alertas/día; USD 12 por alerta; piloto de 2 semanas con U2: ≈250 alertas/día, ≈4% confirmadas; 12 casos/día esperados; USD 50,000 por caso no detectado. **Cada alerta debe ser revisada por una persona de cumplimiento.**

## 1.7.a

- **Capacidad diaria:** 8 × 5 = **40 alertas/día**. Revisarlas cuesta 40 × 12 = **USD 480/día**.
- **Costo de revisar todo el piloto:** 250 × 12 = **USD 3,000/día**.
- **Comparación:** las alertas superan la capacidad **6.25×** (250 / 40). Quedan **210 alertas/día sin revisar (84%)**, y eso no se puede aceptar porque toda alerta requiere revisión humana. Para revisar 250 se necesitan 250 / 5 = **50 analistas (42 más)**.

## 1.7.b

- **Confirmados detectados en el piloto:** 4% × 250 = **10 casos/día**, es decir **10 / 12 = 83%** de los casos esperados (recall ≈ 0.83). Es consistente con el recall 0.80 de U2 del Rol 1: 0.80 × 12 = 9.6/día.
- **Cifra a conciliar:** la precisión real del piloto es **≈4%**, y el Rol 1 reportó **0.95** en validación remuestreada 50/50. Con la prevalencia real, U2 da 9.6 VP + 0.04 × 5,988 = 239.5 FP ≈ 249 alertas, con una precisión de ≈3.9%. Eso reproduce el piloto, así que el 0.95 solo vale para una muestra balanceada.
- **No detectados:** 12 − 10 ≈ **2 casos/día**, que dan 2 × 50,000 = **USD 100,000/día** de daño esperado.

## 1.7.c

Base: 6,000 remitentes/día con 0.2% de prevalencia (12 positivos y 5,988 negativos), tabla del Rol 1. Aquí solo calculo lo necesario para la recomendación. La tabla completa de umbrales es grupal (Task 2.2).

| Opción | Alertas/día | Detectados | No detectados → daño/día | Revisión/día |
|---|---:|---:|---:|---:|
| (i) U2, revisar solo las 40 de mayor puntaje | 40 de ≈250 | ≥1.6 (sin dato del piloto) | ≥10.4 → ≤USD 520,000 | USD 480 |
| (ii) Subir a U4 | 6 + 29.9 ≈ 36 | 6 | 6 → USD 300,000 | ≈USD 431 |
| (ii) Subir a U3 | 7.8 + 59.9 ≈ 68 | 7.8 | 4.2 → USD 210,000 | ≈USD 812 (14 analistas) |
| (iii) Ampliar capacidad para U2 | ≈250 | 9.6 (piloto: 10) | 2.4 → USD 120,000 | ≈USD 3,000 (50 analistas) |

- **(i) Priorizar por puntaje:** si las 40 alertas revisadas tuvieran la precisión del piloto (~4%), saldrían 40 × 0.04 = **1.6 casos/día**. Ordenar por puntaje debería dar más, pero el piloto no lo mide. Si el puntaje ordena igual que la tabla del Rol 1, las 40 de mayor puntaje se parecen mucho a U4, que tiene 36 alertas. Además, las otras 210 alertas que genera U2 quedarían sin revisar, lo que choca con la regla de revisión humana.
- **(ii) Subir el umbral:** U4 es la única opción que cabe hoy en 40, pero detecta solo **6 de 12**. Eso deja 6 × 50,000 = **USD 300,000/día** de daño esperado. U3 detecta 7.8 con ≈68 alertas, lo que requiere ampliar el equipo a 14 analistas (6 más).
- **(iii) Ampliar capacidad** (contratación, personal temporal u outsourcing supervisado): llegar a U2 cuesta en revisión ≈210 × 12 = **USD 2,520/día** adicionales. A cambio detecta ≈3.6 casos más que U4 (9.6 − 6), es decir 3.6 × 50,000 = **USD 180,000/día** de daño evitado. El beneficio supera al costo unas 70 veces, aunque el costo de contratar no viene en los datos.
- **LLM con transacciones en contexto (Rol 5):** la narrativa baja de 6 a 1.5 min, un ahorro de 4.5 min por alerta (≈18.75 h/día con 250 alertas). Tiene 2/40 = 5% de hechos inventados, así que siempre requiere revisión humana. No conozco el tiempo total de revisión por alerta, por eso no puedo convertir ese ahorro en alertas/analista adicionales.
- **Recomendación escalonada (lanzar con condiciones):**
  1. **Inmediato:** operar en **U4** (≈36 ≤ 40 alertas/día), con el 100% de las alertas revisadas por una persona.
  2. **Corto plazo:** ampliar a 14 analistas y bajar a **U3**.
  3. **Meta:** llegar a ≈50 analistas o capacidad equivalente y operar en **U2**, solo cuando la cola sin revisar sea 0 y el recall del piloto sea ≥0.80.
  4. **Narrativas:** usar el LLM con contexto, con revisión obligatoria.
  5. **Seguimiento:** medir semanalmente las alertas/día, las confirmaciones y los casos perdidos.

## Sección 7 del memo

El equipo revisa 40 alertas/día (8 × 5). U2 genera ≈250 en el piloto (USD 3,000/día), así que 210 alertas (84%) quedarían sin revisión humana. El piloto confirma 10/día (83% de 12) con una precisión real ≈4%, frente al 0.95 reportado en validación 50/50. Recomendamos **lanzar con condiciones**: empezar en U4 (≈36 alertas, cabe en la capacidad) sin dejar alertas sin revisar, y ampliar capacidad hacia U3 (14 analistas) y luego U2 (50). Cada caso adicional detectado evita USD 50,000, y pasar de U4 a U2 evita ≈USD 180,000/día a un costo de revisión de ≈USD 2,520/día.

## Cifras a compartir / dependencias

- **Capacidad:** 40 alertas/día (8 analistas × 5). Se necesitan 5 analistas por cada 25 alertas.
- **Costo:** USD 12 por alerta. USD 480/día con la capacidad actual y USD 3,000/día para revisar todo el piloto con U2.
- **Piloto U2:** ≈250 alertas/día, 4% confirmadas, 10 detectados/día (83%) y ≈2 no detectados/día.
- **Daño:** USD 50,000 por caso no detectado (en el piloto: ≈USD 100,000/día).
- **Para Rol 1 / Task 2.2:** la precisión real (≈4%) contradice el 0.95 de la validación 50/50. Todo umbral que pase de 40 alertas/día exige ampliar capacidad antes de operar.
- **Depende de Rol 5:** tasa de alucinación (5% con contexto) y ahorro de 4.5 min por narrativa.
