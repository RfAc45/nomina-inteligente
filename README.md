# Guía completa — Cuaderno de Python (ETL y EDA)

### Caso: Nómina Inteligente 

### Nombre del archivo: `CIA6041_NominaInteligente_ETL_EDAb.ipynb`

\---

## Paso 0 — Preparar el entorno

1. Vayan a [colab.research.google.com](https://colab.research.google.com) → "Nuevo cuaderno"
2. Renombren el cuaderno (clic en el nombre arriba a la izquierda): `NominaInteligente_ETL_EDA`
3. En el panel izquierdo, ícono de carpeta → botón de subir → subir `WA_Fn-UseC_-HR-Employee-Attrition.csv` desde su computadora

\---

## Paso 1 — Abrir y cargar el archivo

```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read\_csv("WA_Fn-UseC_-HR-Employee-Attrition.csv")

print("Filas:", df.shape\[0], "| Columnas:", df.shape\[1])
df.head()
```

**Qué hace:** las líneas `import` cargan 3 herramientas: `pandas` (para tablas de datos), `numpy` (cálculos numéricos) y `matplotlib` (gráficos). `pd.read\_csv(...)` lee el archivo y lo convierte en una tabla que Python puede manipular, guardada en la variable `df`. `df.shape` da el tamaño (filas, columnas). `df.head()` muestra las primeras 5 filas, solo para confirmar visualmente que cargó bien.

**Resultado esperado:** "Filas: 1470 | Columnas: 35" y una tabla con las primeras filas.

\---

## Paso 2 — Medir la calidad antes de tocar nada

**Celda 2a:**

```
vacios = df.isnull().sum()
print("Total de valores vacíos en todo el dataset:", vacios.sum())

duplicados = df.duplicated().sum()
print("Filas duplicadas:", duplicados)
```

**Qué hace:** `df.isnull()` revisa celda por celda si está vacía (True/False), y `.sum()` cuenta cuántas hay por columna; el segundo `.sum()` suma todas esas columnas para dar el total del dataset completo. `df.duplicated()` marca qué filas son copias exactas de otra fila anterior, y `.sum()` las cuenta.

**Resultado esperado:** 0 y 0.

**Celda 2b:**

```
constantes = \[col for col in df.columns if df\[col].nunique() == 1]
print("Columnas constantes encontradas:", constantes)

for col in constantes:
    print(f"  {col}: valor único = {df\[col].unique()\[0]}")
```

**Qué hace:** `df\[col].nunique()` cuenta cuántos valores *distintos* tiene una columna. La primera línea recorre todas las columnas (`for col in df.columns`) y se queda solo con las que tienen exactamente 1 valor distinto — es decir, columnas donde todos los empleados tienen el mismo dato, y por eso no sirven para análisis. El `for` de abajo solo imprime cuál es ese valor único, a modo de evidencia.

**Resultado esperado:** `\['EmployeeCount', 'Over18', 'StandardHours']`

\---

## Paso 3 — Limpiar, reconstruyendo desde la fuente

**Celda 3a:**

```
df\_limpio = df.drop(columns=constantes)

print("Columnas antes:", df.shape\[1], "| Columnas después:", df\_limpio.shape\[1])
print("Se eliminaron:", constantes)
```

**Qué hace:** `.drop(columns=constantes)` crea una copia nueva de la tabla (`df\_limpio`) sin las columnas que encontramos en el paso anterior. El `df` original no se toca — por eso se guarda en una variable con nombre distinto.

**Resultado esperado:** "Columnas antes: 35 | Columnas después: 32"

> A partir de aquí, siempre se trabaja sobre `df\_limpio`, no sobre `df` — así el original queda intacto.

**Celda 3b:**

```
assert df\_limpio.shape\[0] == df.shape\[0], "¡Se perdieron filas al limpiar!"
print("Filas verificadas: ninguna se perdió.", df\_limpio.shape\[0], "empleados intactos.")
```

**Qué hace:** `assert` es una verificación automática — comprueba que la condición sea verdadera, y si no lo es, detiene el programa con el mensaje de error que le pusimos. Aquí confirma que el número de filas de `df\_limpio` sigue siendo igual al de `df` (eliminar columnas no debería eliminar ningún empleado).

**Resultado esperado:** "1470 empleados intactos."

\---

## Paso 4 — Crear las columnas derivadas

```
def banda\_salarial(ingreso):
    if ingreso < 3000:
        return "Bajo"
    elif ingreso < 8000:
        return "Medio"
    else:
        return "Alto"

def grupo\_antiguedad(anios):
    if anios < 3:
        return "Nuevo"
    elif anios < 10:
        return "Intermedio"
    else:
        return "Senior"

df\_limpio\["Banda salarial"] = df\_limpio\["MonthlyIncome"].apply(banda\_salarial)
df\_limpio\["Grupo de antigüedad"] = df\_limpio\["YearsAtCompany"].apply(grupo\_antiguedad)

df\_limpio\[\["MonthlyIncome", "Banda salarial", "YearsAtCompany", "Grupo de antigüedad"]].head(8)
```

**Qué hace:** `def banda\_salarial(ingreso):` crea una función — una especie de mini-fórmula reutilizable — que recibe un número y devuelve una categoría de texto según las reglas escritas con `if/elif/else`. `.apply(banda\_salarial)` aplica esa función a cada una de las 1,470 filas de la columna `MonthlyIncome`, generando una columna nueva. Se repite la misma lógica para la antigüedad. La última línea solo muestra 8 filas de ejemplo para confirmar que las categorías quedaron bien asignadas.

**Resultado esperado:** tabla de 8 filas con las categorías asignadas correctamente.

\---

## Paso 5 — Agrupar: departamento y antigüedad

**Celda 5a:**

```
por\_departamento = df\_limpio.groupby("Department")\["MonthlyIncome"].agg(\["count", "mean"]).round(2)
por\_departamento.columns = \["Empleados", "Salario promedio"]
por\_departamento
```

**Qué hace:** `.groupby("Department")` junta a todos los empleados del mismo departamento en un solo grupo. `\["MonthlyIncome"].agg(\["count", "mean"])` calcula, para cada grupo, cuántos empleados tiene (`count`) y su salario promedio (`mean`), en un solo paso. `.round(2)` redondea a 2 decimales. La siguiente línea solo les pone nombres más claros a las columnas resultantes.

**Celda 5b:**

```
por\_antiguedad = df\_limpio.groupby("Grupo de antigüedad")\["MonthlyIncome"].mean().round(2)
por\_antiguedad = por\_antiguedad.reindex(\["Nuevo", "Intermedio", "Senior"])
por\_antiguedad
```

**Qué hace:** mismo principio que la celda anterior, pero agrupando por `Grupo de antigüedad` y calculando solo el promedio. El `.reindex(\[...])` fuerza el orden lógico (Nuevo → Intermedio → Senior); sin él, pandas ordenaría las categorías alfabéticamente, sin ningún sentido real.

\---

## Paso 6 — Los cuatro indicadores

```
costo\_total\_nomina = df\_limpio\["MonthlyIncome"].sum()
salario\_promedio = df\_limpio\["MonthlyIncome"].mean()
tasa\_rotacion = (df\_limpio\["Attrition"] == "Yes").mean()
empleados\_que\_renunciaron = (df\_limpio\["Attrition"] == "Yes").sum()

print(f"Costo total de nómina:      ${costo\_total\_nomina:,.2f}")
print(f"Salario promedio:           ${salario\_promedio:,.2f}")
print(f"Tasa de rotación:           {tasa\_rotacion:.2%}")
print(f"Empleados que renunciaron:  {empleados\_que\_renunciaron}")
```

**Qué hace:** `.sum()` y `.mean()` sobre `MonthlyIncome` dan el total y el promedio de salario de los 1,470 empleados. `(df\_limpio\["Attrition"] == "Yes")` crea una lista de True/False (True donde el empleado renunció); `.mean()` sobre esa lista da directamente el porcentaje de "True" (la tasa de rotación), y `.sum()` cuenta cuántos "True" hay en total (el número de renuncias). Las líneas `print(f"...")` solo dan formato de texto legible a esos 4 números.

**Resultado esperado:**

```
Costo total de nómina:      $9,559,309.00
Salario promedio:           $6,502.93
Tasa de rotación:           16.12%
Empleados que renunciaron:  237
```

\---

## Paso 7 — Los tres gráficos

**Celda 7a — dispersión:**

```
fig, ax = plt.subplots(figsize=(8, 5))
for depto, grupo in df\_limpio.groupby("Department"):
    ax.scatter(grupo\["YearsAtCompany"], grupo\["MonthlyIncome"], label=depto, alpha=0.5, s=15)
ax.set\_xlabel("Años en la empresa")
ax.set\_ylabel("Ingreso mensual (USD)")
ax.set\_title("Los salarios más altos se concentran en empleados con más antigüedad")
ax.legend()
plt.tight\_layout()
plt.savefig("grafico\_dispersion.png", dpi=120)
plt.show()
```

**Qué hace:** `plt.subplots()` crea el lienzo en blanco donde se dibuja el gráfico. El `for` recorre cada departamento por separado y dibuja sus puntos (`ax.scatter`) con un color distinto automático, para poder compararlos. `ax.set\_xlabel/ylabel/title` ponen los textos de los ejes y el título. `ax.legend()` agrega el recuadro que indica qué color es cada departamento. `plt.savefig(...)` guarda el gráfico como imagen, y `plt.show()` lo muestra en pantalla.

**Celda 7b — barras por antigüedad:**

```
fig, ax = plt.subplots(figsize=(7, 5))
por\_antiguedad.plot(kind="bar", ax=ax, color=\["#CADCFC", "#02C39A", "#1E2761"])
ax.set\_ylabel("Salario promedio (USD)")
ax.set\_title("El salario promedio aumenta con la antigüedad del empleado")
ax.set\_xlabel("")
plt.xticks(rotation=0)
plt.tight\_layout()
plt.savefig("grafico\_barras.png", dpi=120)
plt.show()
```

**Qué hace:** `.plot(kind="bar")` convierte directamente la tabla `por\_antiguedad` (calculada en el Paso 5) en un gráfico de barras, una barra por categoría. La lista de `color=\[...]` asigna un color específico a cada barra, en el mismo orden (Nuevo, Intermedio, Senior), usando los colores de identidad del proyecto. `plt.xticks(rotation=0)` mantiene los nombres de las categorías horizontales, sin inclinar.

**Celda 7c — barras por horas extra:**

```
tasa\_por\_overtime = df\_limpio.groupby("OverTime")\["Attrition"].apply(lambda x: (x == "Yes").mean())

fig, ax = plt.subplots(figsize=(6, 5))
tasa\_por\_overtime.plot(kind="bar", ax=ax, color=\["#1E2761", "#C0392B"])
ax.set\_ylabel("Tasa de rotación")
ax.set\_title("Las horas extra multiplican el riesgo de renuncia")
ax.set\_xlabel("¿Hace horas extra?")
plt.xticks(rotation=0)
plt.tight\_layout()
plt.savefig("grafico\_overtime.png", dpi=120)
plt.show()

multiplicador = tasa\_por\_overtime\["Yes"] / tasa\_por\_overtime\["No"]
print(f"Multiplicador de riesgo (OverTime=Yes vs. No): {multiplicador:.2f}x")
```

**Qué hace:** agrupa a los empleados por si hacen horas extra o no, y calcula la tasa de rotación dentro de cada grupo (misma lógica de "True/False → mean" del Paso 6, aquí aplicada dentro de un `.groupby`). El resto del código dibuja el gráfico de barras igual que antes. La última línea divide la tasa de quienes SÍ hacen horas extra entre la de quienes NO, dando el multiplicador de riesgo.

**Resultado esperado:** `2.93x`

\---

## Bonus — Candidatos a promoción

**Celda extra 1:**

```
df\_limpio\["CandidatoPromocion"] = (df\_limpio\["YearsSinceLastPromotion"] >= 3).astype(int)

candidatos = df\_limpio\["CandidatoPromocion"].sum()
total = df\_limpio.shape\[0]
print(f"Empleados candidatos a promoción (3+ años sin ascenso): {candidatos} de {total} ({candidatos/total:.1%})")
```

**Qué hace:** crea una nueva columna que vale 1 si el empleado lleva 3 o más años sin promoción, y 0 si no — `.astype(int)` convierte el True/False en 1/0 para que sea más fácil sumarlo después. Las siguientes líneas cuentan cuántos "1" hay y calculan el porcentaje sobre el total.

**Celda extra 2:**

```
candidatos\_por\_depto = df\_limpio.groupby("Department")\["CandidatoPromocion"].agg(\["sum", "mean"]).round(2)
candidatos\_por\_depto.columns = \["Cantidad", "Proporción"]
candidatos\_por\_depto
```

**Qué hace:** mismo patrón de `.groupby(...).agg(\[...])` que usaron en el Paso 5 — aquí, por departamento, cuenta cuántos candidatos a promoción hay (`sum`, ya que son 1s y 0s) y qué proporción representan (`mean`).

**Celda extra 3 (gráfico):**

```
fig, ax = plt.subplots(figsize=(7, 5))
candidatos\_por\_depto\["Proporción"].plot(kind="bar", ax=ax, color=\["#1E2761", "#02C39A", "#CADCFC"])
ax.set\_ylabel("Proporción de candidatos a promoción")
ax.set\_title("Ventas tiene la mayor proporción de empleados sin ascenso reciente")
ax.set\_xlabel("")
plt.xticks(rotation=0)
plt.tight\_layout()
plt.savefig("grafico\_promocion.png", dpi=120)
plt.show()

print(candidatos\_por\_depto)
```

**Qué hace:** mismo tipo de gráfico de barras que los anteriores, esta vez con la columna `Proporción` de la tabla recién calculada.

## Paso 8 — Verificar y exportar

**Celda 8a:**

```
print("=== Verificación contra el dashboard ===")
print(f"CostoTotalNomina:        ${costo\_total\_nomina:,.2f}")
print(f"SalarioPromedio:         ${salario\_promedio:,.2f}")
print(f"TasaRotacion:            {tasa\_rotacion:.2%}")
print(f"EmpleadosQueRenunciaron: {empleados\_que\_renunciaron}")
print()
print("Filas finales:", df\_limpio.shape\[0], "| Columnas finales:", df\_limpio.shape\[1])
```

**Qué hace:** solo vuelve a imprimir los 4 indicadores del Paso 6, junto con el tamaño final de la tabla, como resumen de cierre antes de exportar — sirve para comparar de un vistazo contra las tarjetas del dashboard de Power BI.

**Celda 8b (exportar — vuelvan a correrla después de agregar el bonus):**

```
df\_limpio.to\_csv("dataset\_nomina\_limpio.csv", index=False)
print("Exportado: dataset\_nomina\_limpio.csv")

resumen = pd.DataFrame({
    "Indicador": \["CostoTotalNomina", "SalarioPromedio", "TasaRotacion", "EmpleadosQueRenunciaron"],
    "Valor": \[costo\_total\_nomina, salario\_promedio, tasa\_rotacion, empleados\_que\_renunciaron],
})
resumen.to\_csv("resumen\_indicadores.csv", index=False)
print("Exportado: resumen\_indicadores.csv")
```

**Qué hace:** `.to\_csv(...)` guarda la tabla completa como archivo CSV, listo para descargar. `index=False` evita que pandas agregue una columna extra con el número de fila. Las líneas de `resumen = pd.DataFrame({...})` arman una tabla pequeña nueva, solo con los 4 indicadores, y la exportan aparte — útil para adjuntar en la documentación sin tener que abrir el CSV completo de 1,470 filas.

**Para descargar:** panel izquierdo (ícono de carpeta) → clic derecho sobre cada CSV → "Descargar".

**Verificación final:**

```
print(df\_limpio.columns.tolist())
print("Total de columnas:", df\_limpio.shape\[1])
```

**Qué hace:** `.columns.tolist()` imprime la lista completa de nombres de columna, para confirmar visualmente que todo quedó incluido (las derivadas y el bonus).

Deberían ver 35 columnas, incluyendo `Banda salarial`, `Grupo de antigüedad` y `CandidatoPromocion`.

\---

## Errores comunes 

* **`KeyError` en el gráfico de promoción:** revisen que estén usando el nombre de columna correcto — si renombraron `candidatos\_por\_depto.columns = \["Cantidad", "Proporción"]`, deben usar `candidatos\_por\_depto\["Proporción"]`, no `\["mean"]`.
* **El notebook no encuentra el CSV:** confirmen que subieron `WA\_Fn-UseC\_-HR-Employee-Attrition.csv` en el Paso 0, y que el nombre está escrito exactamente igual (mayúsculas y guiones incluidos).
\---

## Nota sobre modelaje

Este notebook es solo **ETL y EDA** (limpieza + análisis exploratorio) — no entrena ningún modelo predictivo. 

