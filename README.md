Guía completa — Cuaderno de Python (ETL y EDA)
Caso: Nómina Inteligente | CIA6041 · Semana 7
Nombre del archivo:
Sigan los pasos en orden. Cada celda de código va por separado — no las peguen todas juntas.

Paso 0 — Preparar el entorno
1.	Vayan a  colab.research.g oog le.com → "Nuevo cuaderno"
2.	Renombren el cuaderno (clic en el nombre arriba a la izquierda):

3.	En el panel izquierdo, ícono de carpeta → botón de subir → suban
desde su computadora

Paso 1 — Abrir y cargar el archivo

Qué hace: las líneas import cargan 3 herramientas: pandas (para tablas de datos),
(cálculos numéricos) y matplotlib (gráﬁcos). pd.read_csv(...) lee el archivo y lo convierte
 
en una tabla que Python puede manipular, guardada en la variable
 
.
 
da el tamaño
 
(ﬁlas, columnas).
cargó bien.
 
muestra las primeras 5 ﬁlas, solo para conﬁrmar visualmente que
 
Resultado esperado: "Filas: 1470 | Columnas: 35" y una tabla con las primeras ﬁlas.

Paso 2 — Medir la calidad antes de tocar nada
Celda 2a:
 
 

 
Qué hace:
 
revisa celda por celda si está vacía (True/False), y
 
cuenta
 
cuántas hay por columna; el segundo	suma todas esas columnas para dar el total del
 
dataset completo.
las cuenta.
Resultado esperado: 0 y 0.
Celda 2b:







Qué hace:
 
marca qué ﬁlas son copias exactas de otra ﬁla anterior, y














cuenta cuántos valores distintos tiene una columna. La primera
 
línea recorre todas las columnas (	) y se queda solo con las que tienen
exactamente 1 valor distinto — es decir, columnas donde todos los empleados tienen el mismo
 
dato, y por eso no sirven para análisis. El modo de evidencia.
Resultado esperado:
 
de abajo solo imprime cuál es ese valor único, a
 

 
Paso 3 — Limpiar, reconstruyendo desde la fuente
Celda 3a:


 
Qué hace:
 
crea una copia nueva de la tabla (
 
) sin las
 
columnas que encontramos en el paso anterior. El en una variable con nombre distinto.
 
original no se toca — por eso se guarda
 
Resultado esperado: "Columnas antes: 35 | Columnas después: 32"

 
A partir de aquí, siempre se trabaja sobre intacto.
Celda 3b:
 
, no sobre
 
— así el original queda
 

 
Qué hace:	es una veriﬁcación automática — comprueba que la condición sea
verdadera, y si no lo es, detiene el programa con el mensaje de error que le pusimos. Aquí
 
conﬁrma que el número de ﬁlas de
no debería eliminar ningún empleado).
 
sigue siendo igual al de
 
(eliminar columnas
 
Resultado esperado: "1470 empleados intactos."

Paso 4 — Crear las columnas derivadas

 
Qué hace:	crea una función — una especie de mini-fórmula
reutilizable — que recibe un número y devuelve una categoría de texto según las reglas escritas con if/elif/else (las mismas 3 categorías y cortes que usaron en Power Query).
.apply(banda_salarial) aplica esa función a cada una de las 1,470 ﬁlas de la columna MonthlyIncome , generando una columna nueva. Se repite la misma lógica para la antigüedad. La última línea solo muestra 8 ﬁlas de ejemplo para conﬁrmar que las categorías quedaron bien asignadas.
Resultado esperado: tabla de 8 ﬁlas con las categorías asignadas correctamente.

Paso 5 — Agrupar: departamento y antigüedad
Celda 5a:

Qué hace: .groupby("Department") junta a todos los empleados del mismo departamento en un solo grupo. ["MonthlyIncome"].agg(["count", "mean"]) calcula, para cada grupo, cuántos
empleados tiene ( count ) y su salario promedio ( mean ), en un solo paso.	redondea
a 2 decimales. La siguiente línea solo les pone nombres más claros a las columnas resultantes.
Celda 5b:

Qué hace: mismo principio que la celda anterior, pero agrupando por	y
calculando solo el promedio. El	fuerza el orden lógico (Nuevo → Intermedio
→ Senior); sin él, pandas ordenaría las categorías alfabéticamente, sin ningún sentido real.

 
Paso 6 — Los cuatro indicadores

Qué hace: .sum() y .mean() sobre MonthlyIncome dan el total y el promedio de salario de los 1,470 empleados. (df_limpio["Attrition"] == "Yes") crea una lista de True/False (True donde el empleado renunció); .mean() sobre esa lista da directamente el porcentaje de "True" (la tasa de rotación), y .sum() cuenta cuántos "True" hay en total (el número de renuncias).
Las líneas print(f"...") solo dan formato de texto legible a esos 4 números.
Resultado esperado (debe coincidir con las tarjetas de Power BI):

Costo total de nómina:	$9, 559, 309. 00
Salario promedio:	$6, 502. 93
Tasa de rotación:	16. 12%
Empleados que renunciaron:	237

Paso 7 — Los tres gráﬁcos
Celda 7a — dispersión:
 
 

 
Qué hace:
 
crea el lienzo en blanco donde se dibuja el gráﬁco. El
 
recorre
 
cada departamento por separado y dibuja sus puntos ( ax.scatter ) con un color distinto automático, para poder compararlos. ax.set_xlabel/ylabel/title ponen los textos de los ejes y el título. ax.legend() agrega el recuadro que indica qué color es cada departamento.
plt.savefig(...) guarda el gráﬁco como imagen, y	lo muestra en pantalla.
Celda 7b — barras por antigüedad:

Qué hace:	convierte directamente la tabla por_antiguedad (calculada en el
Paso 5) en un gráﬁco de barras, una barra por categoría. La lista de color=[...] asigna un color especíﬁco a cada barra, en el mismo orden (Nuevo, Intermedio, Senior), usando los
 
colores de identidad del proyecto.
categorías horizontales, sin inclinar.
Celda 7c — barras por horas extra:
 
mantiene los nombres de las
 

tasa_por_overtime = df_limpio. groupby(" OverTime") [" Attrition"] . apply( lambda x: ( x == " Yes") . mean() )

fig, ax = plt. subplots( figsize=( 6, 5) )
tasa_por_overtime. plot( kind=" bar",  ax=ax,  color=["#1E2761",  "#C0392B"] )
ax. set_ylabel(" Tasa de rotación")
ax. set_title(" Las horas extra multiplican el riesgo de renuncia") ax. set_xlabel("¿Hace horas extra?")
plt. xticks( rotation=0)
plt. tight_layout()
plt. savefig(" grafico_overtime. png",  dpi=120)
plt. show()


multiplicador = tasa_por_overtime[" Yes"] / tasa_por_overtime[" No"]
print( f" Multiplicador de riesgo ( OverTime=Yes vs. No) : {multiplicador: . 2f}x")

Qué hace: agrupa a los empleados por si hacen horas extra o no, y calcula la tasa de rotación dentro de cada grupo (misma lógica de "True/False → mean" del Paso 6, aquí aplicada dentro de
un	). El resto del código dibuja el gráﬁco de barras igual que antes. La última línea
divide la tasa de quienes SÍ hacen horas extra entre la de quienes NO, dando el multiplicador de riesgo.
Resultado esperado:

Bonus — Candidatos a promoción
Celda extra 1:

Qué hace: crea una nueva columna que vale 1 si el empleado lleva 3 o más años sin promoción,
y 0 si no —	convierte el True/False en 1/0 para que sea más fácil sumarlo
después. Las siguientes líneas cuentan cuántos "1" hay y calculan el porcentaje sobre el total.
Celda extra 2:
 
 
Qué hace: mismo patrón de	que usaron en el Paso 5 — aquí, por
departamento, cuenta cuántos candidatos a promoción hay (	, ya que son 1s y 0s) y qué
proporción representan (	).
Celda extra 3 (gráﬁco):

Qué hace: mismo tipo de gráﬁco de barras que los anteriores, esta vez con la columna de la tabla recién calculada.
⚠ Importante: corran esta celda y miren qué departamento sale con la proporción más alta ANTES de dejar el título ﬁjo. Ajusten el nombre del departamento en
según lo que muestren sus propios datos — no copien "Ventas" sin veriﬁcarlo, cada quien debe conﬁrmar su propio resultado.

Paso 8 — Veriﬁcar y exportar
Celda 8a:
 
 
Qué hace: solo vuelve a imprimir los 4 indicadores del Paso 6, junto con el tamaño ﬁnal de la tabla, como resumen de cierre antes de exportar — sirve para comparar de un vistazo contra las tarjetas del dashboard de Power BI.
Celda 8b (exportar — vuelvan a correrla después de agregar el bonus):

Qué hace: .to_csv(...) guarda la tabla completa como archivo CSV, listo para descargar. index=False evita que pandas agregue una columna extra con el número de ﬁla. Las líneas de resumen = pd.DataFrame({...}) arman una tabla pequeña nueva, solo con los 4 indicadores, y la exportan aparte — útil para adjuntar en la documentación sin tener que abrir el CSV completo de 1,470 ﬁlas.
Para descargar: panel izquierdo (ícono de carpeta) → clic derecho sobre cada CSV → "Descargar".
Veriﬁcación ﬁnal:

 
Qué hace:	imprime la lista completa de nombres de columna, para
conﬁrmar visualmente que todo quedó incluido (las derivadas y el bonus).

 
Deberían ver 35 columnas, incluyendo
.
 
,	y
 

 
Errores comunes (por si se traban)
 	en el gráﬁco de promoción: revisen que estén usando el nombre de columna
correcto — si renombraron
, deben usar	.
 	El notebook no encuentra el CSV: conﬁrmen que subieron
en el Paso 0, y que el nombre está escrito exactamente igual (mayúsculas y guiones incluidos).
 	Los números del Paso 6 no coinciden con Power BI: revisen que estén usando
(no	) en todas las celdas desde el Paso 4 en adelante.

Nota sobre modelaje
Este notebook es solo ETL y EDA (limpieza + análisis exploratorio) — no entrena ningún modelo predictivo. El modelo con LazyPredict y K-Means es un notebook distinto, hecho en una etapa anterior del proyecto.

## Sobre este proyecto
Nómina Inteligente es un proyecto del curso CIA6041 (Inteligencia Artificial y Gestión de Datos Organizacionales, Broward International University) que busca predecir el riesgo de renuncia de empleados y segmentarlos según su perfil salarial y de antigüedad, usando el caso hipotético de una empresa de tecnología (Advanced Tech Group).
Dataset: IBM HR Analytics Employee Attrition & Performance — 1,470 empleados, dataset público y ficticio, licencia DbCL v1.0.
Herramientas: Herramientas: Python (pandas, matplotlib) en Google Colab para limpieza y análisis exploratorio; Power BI y Tableau para los dashboards. El proyecto contempla además segmentación (K-Means) y un modelo de clasificación predictivo, desarrollados en una etapa posterior del proyecto.
Este notebook cubre la etapa de ETL (extracción, transformación y carga) y EDA (análisis exploratorio de datos): limpieza del dataset original, creación de variables derivadas, cálculo de indicadores clave y visualizaciones iniciales.

