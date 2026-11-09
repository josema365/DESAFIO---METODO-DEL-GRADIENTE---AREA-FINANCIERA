# GCP en finanzas: simulador de asignación presupuestaria

Desafío académico de **Métodos Numéricos**: aplicación del **Método de Gradientes Conjugados Precondicionado (GCP)** a un problema financiero.

- **Estudiante:** Jose Manuel Huanca Tito
- **Mención:** Desarrollo de Sistemas
- **Modalidad:** individual
- **Página web (pública):** https://josema365.github.io/DESAFIO---METODO-DEL-GRADIENTE---AREA-FINANCIERA/
- **Repositorio:** https://github.com/josema365/DESAFIO---METODO-DEL-GRADIENTE---AREA-FINANCIERA
                      
## 1. Descripción

La página resuelve un sistema lineal `Ax = b` con **n = 5** incógnitas, donde `A` es simétrica y definida positiva (SPD), en tres escenarios:

| Escenario | Descripción |
|---|---|
| Ideal | Situación estable: `A` tridiagonal con diagonal 4 y acoplamiento 1; κ(A) bajo. |
| Bajo estrés | Inflación del 15 % en los costos unitarios (diagonal ×1,15) y cambios de demanda en `b`. |
| Mal condicionado | `A = D·C·D` con `D = diag(1, 4, 16, 64, 256)` (mal escalamiento) y `C` con correlación 0,8^\|i−j\| (fuerte acoplamiento). |

### Modelo financiero

Se reparte un fondo (miles de Bs) entre cinco áreas: **x₁ Personal, x₂ Operaciones, x₃ Tecnología, x₄ Marketing, x₅ Reserva**. El costo de desviarse del plan es `f(x) = ½ xᵀAx − bᵀx`. Su mínimo cumple `∇f = Ax − b = 0`, es decir `Ax = b`.

- `aᵢᵢ`: costo marginal de ajustar el área i.
- `aᵢⱼ = aⱼᵢ`: interdependencia entre las áreas i y j.
- `bᵢ`: necesidad de financiamiento del área i.

Los datos son ilustrativos y están definidos por el autor para este ejercicio.

## 2. Qué calcula

Para cada escenario:

1. **Validación** de `A`: simetría, definida positiva (factorización de Cholesky, criterio de Sylvester con los menores principales y autovalores).
2. **Solución numérica de referencia** con el método directo (Cholesky `A = LLᵀ`).
3. **GCP** con precondicionador seleccionable: Jacobi (por defecto), Cholesky incompleto IC(0) o ninguno.
4. **GC simple** (sin precondicionar), solo como contraste.
5. **Errores** por variable (absoluto, relativo y porcentual) y residuo relativo global `‖b − Ax‖₂ / ‖b‖₂`.
6. **Números de condición** `κ(A)` y `κ(M⁻¹A)`, calculados con autovalores (método de Jacobi cíclico).
7. **Tabla comparativa** de los tres escenarios.

Criterio de parada: `‖rₖ‖₂ / ‖b‖₂ < tolerancia` (por defecto 10⁻⁶) o número máximo de iteraciones.

## 3. Algoritmo GCP (Suñagua, 2020)

```
x₀ dado;  r₀ = b − A·x₀
Resolver M·z₀ = r₀;  p₀ = z₀
Para k = 0, 1, 2, ...
    αₖ   = (rₖᵀ zₖ) / (pₖᵀ A pₖ)
    xₖ₊₁ = xₖ + αₖ pₖ
    rₖ₊₁ = rₖ − αₖ A pₖ
    Si ‖rₖ₊₁‖₂ / ‖b‖₂ < tol: parar
    Resolver M·zₖ₊₁ = rₖ₊₁
    βₖ   = (rₖ₊₁ᵀ zₖ₊₁) / (rₖᵀ zₖ)
    pₖ₊₁ = zₖ₊₁ + βₖ pₖ
```

Con `M = I` se obtiene el GC simple. Con `M = diag(A)` se obtiene Jacobi. Con `M = L·Lᵀ` se obtiene IC(0), donde `L` conserva el patrón de ceros de `A`.

## 4. Estructura del repositorio

```
.
├── index.html   # aplicación completa 
└── README.md    
```

## 5. Arquitectura

La aplicación es una página estática de un solo archivo, sin dependencias externas ni backend. El código JavaScript se organiza en cuatro capas:

| Capa | Contenido | Funciones principales |
|---|---|---|
| Datos | Definición de los escenarios y sus descripciones | `SC`, `DESC` |
| Núcleo numérico | Álgebra lineal, validación y algoritmos | `fac` (Cholesky y IC(0)), `fs`/`bs` (sustituciones), `eig` (autovalores), `val` (validación SPD), `cg` (GC/GCP), `run` (ejecuta un caso completo) |
| Interfaz | Entrada de datos, tablas y gráfico | `grid`, `readAB`, `opts`, `load`, `upd`, `cmp`, `chart` |
| Pruebas (QA) | Casos de prueba ejecutables | `qa` |

**Flujo:** cualquier cambio en la interfaz (evento `input`) llama a `upd()`, que lee `A` y `b`, ejecuta `run()` y vuelve a pintar validación, solución de referencia, GCP vs GC, errores, tabla comparativa y conclusiones. Los errores de validación se capturan y se muestran como aviso, sin detener la página.

## 6. Instalación y ejecución local

No requiere instalación ni dependencias.

```bash
git clone https://github.com/josema365/DESAFIO---METODO-DEL-GRADIENTE---AREA-FINANCIERA.git
cd DESAFIO---METODO-DEL-GRADIENTE---AREA-FINANCIERA
```

Abre `index.html` con doble clic en cualquier navegador moderno. Opcionalmente, puedes usar un servidor local:

```bash
python -m http.server 8000
# abrir http://localhost:8000
```

## 7. Módulo de pruebas (QA)

La sección 9 de la página ejecuta y documenta cuatro pruebas con el formato Entrada, Resultado esperado, Resultado obtenido y Conclusión:

1. **Carga de escenarios:** los tres escenarios convergen con GCP.
2. **Matriz no válida:** una matriz no simétrica y otra no definida positiva se rechazan con un mensaje de error, sin colapsar.
3. **Edición dinámica:** al modificar coeficientes se recalculan la solución de referencia, el GCP y los errores.
4. **Precondicionamiento:** en el escenario mal condicionado, GCP necesita menos o las mismas iteraciones que GC.
