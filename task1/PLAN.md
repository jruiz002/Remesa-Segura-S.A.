# Plan - Task 1 (análisis por rol)

## Estrategia

- 7 subagentes en paralelo, uno por rol (Tasks 1.1 a 1.7). Cada rol es independiente en Task 1.
- Cada subagente escribe **solo** `task1/rol-N.md` con: respuestas a cada inciso, sección del memo (4–6 líneas, recomendación + cifra) y bloque "Cifras a compartir / dependencias" para la Task 2.
- Fuente única: `README.md` (reorganización del PDF del examen). No se inventan datos.
- Al terminar, el orquestador verifica cada cálculo contra las cifras de control de abajo y corrige discrepancias.

## Formato común de cada `rol-N.md`

1. Encabezado: rol, task, sección del memo.
2. Un apartado por inciso (1.N.a, 1.N.b, ...), con cálculos explícitos cuando aplique.
3. **Sección N del memo** (4–6 líneas, recomendación concreta + al menos una cifra).
4. **Cifras a compartir / dependencias** (qué números usan otros roles y qué supuestos tienen).

## Cifras de control (verificación del orquestador)

Base: 6,000 remitentes/día, prevalencia 0.2% → 12 positivos y 5,988 negativos por día.

| Umbral | TP = recall×12 | FP = FPR×5,988 | Alertas/día | Precisión real | No detectados | Costo revisión (USD 12) |
|---|---:|---:|---:|---:|---:|---:|
| U1 | 10.8 | 598.8 | ≈ 610 | ≈ 1.8% | 1.2 | ≈ 7,315 |
| U2 | 9.6 | 239.5 | ≈ 249 | ≈ 3.9% | 2.4 | ≈ 2,989 |
| U3 | 7.8 | 59.9 | ≈ 68 | ≈ 11.5% | 4.2 | ≈ 812 |
| U4 | 6.0 | 29.9 | ≈ 36 | ≈ 16.7% | 6.0 | ≈ 431 |

- Rol 1: precisión reportada (0.90–0.99) es de validación 50/50; la real es 1.8%–16.7%. Mejor época = 6 (val 0.21).
- Rol 2: ajuste fino de 2 bloques = AUC 0.87 (aleatoria) con 18% params y 1.5 GPU-h; LoRA 0.86 / 2% / 1 GPU-h.
- Rol 3: AUC temporal 0.81 (caída de 0.06); ventana ≥ 30 días; exactitud trivial 99.8%.
- Rol 4: difusión +0.02 AUC temporal (0.81 → 0.83); 3% casi duplicados.
- Rol 5: alucinación 6/40 = 15% vs 2/40 = 5%; ahorro 6 − 1.5 = 4.5 min (75%).
- Rol 6: SHAP eficiencia 3×10⁻⁷ (aditividad se cumple); LIME 0.45 → 0.78; atención vs SHAP ρ = 0.35.
- Rol 7: capacidad 8×5 = 40/día (USD 480); piloto 250×12 = USD 3,000/día; 250/40 ≈ 6.25×; confirmados 0.04×250 = 10/día = 83% de 12; analistas para 250 = 50.
