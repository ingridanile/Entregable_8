# M8 - Modelo de datos y medidas DAX (RetailPro)

Archivo: Perez_Ingrid_Checkpoint2.pbix
Parte del archivo Pipeline_ETL_Perez_Ingrid.pbix del M6.

## Relaciones (todas 1:N, dirección única, activas)

- Dim_Clientes[id_cliente] -> Fact_Ventas[id_cliente]
- Dim_Productos[id_producto] -> Fact_Ventas[id_producto]
- Dim_Categorias[id_categoria] -> Dim_Productos[id_categoria]
- Dim_Fechas[Date] -> Fact_Ventas[fecha_venta]

Dim_Productos no tenía id_categoria (solo el nombre de la categoría).
Lo agregué en Power Query combinando con Dim_Categorias por nombre_categoria.

Imagen del modelo: modelo.png

## Tabla calendario (marcada como tabla de fechas)

Dim_Fechas = ADDCOLUMNS(
    CALENDAR(MIN(Fact_Ventas[fecha_venta]), MAX(Fact_Ventas[fecha_venta])),
    "Año", YEAR([Date]),
    "Mes Número", MONTH([Date]),
    "Mes Nombre", FORMAT([Date], "MMMM"),
    "Trimestre", "T" & QUARTER([Date]),
    "Semana", WEEKNUM([Date])
)

Mes Nombre está ordenada por Mes Número.

## Tabla _Medidas (5 medidas)

Total Ventas = SUM(Fact_Ventas[total_venta])

Ventas Online = CALCULATE([Total Ventas], Fact_Ventas[canal] = "Online")

Ventas YTD = TOTALYTD([Total Ventas], Dim_Fechas[Date])

Ventas LY = CALCULATE([Total Ventas], SAMEPERIODLASTYEAR(Dim_Fechas[Date]))

% Crecimiento Anual =
VAR VentasActual = [Total Ventas]
VAR VentasAnterior = [Ventas LY]
RETURN
    DIVIDE(VentasActual - VentasAnterior, VentasAnterior)

## Validación (página "Validación")

- Total Ventas 2023: 28.764 / 2024: 18.814 (datos hasta julio 2024)
- Ventas YTD acumula mes a mes y vuelve a empezar en enero 2024
- Ventas LY en 2023 queda vacía (no hay año anterior)
- Ventas LY en 2024 muestra los valores de 2023
- % Crecimiento Anual 2024: 44,43 %

Imagen de la matriz: validacion.png
