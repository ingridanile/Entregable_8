# M8 - Modelo de datos y medidas DAX

En este entregable seguí trabajando sobre el archivo del M6 (Pipeline_ETL_Perez_Ingrid.pbix) y lo guardé como Perez_Ingrid_Checkpoint2.pbix.

## Modelo

Armé un esquema en estrella con Fact_Ventas en el centro. Relacioné las tablas así:

- Dim_Clientes con Fact_Ventas por id_cliente
- Dim_Productos con Fact_Ventas por id_producto
- Dim_Categorias con Dim_Productos por id_categoria
- Dim_Fechas con Fact_Ventas por la fecha de venta

Todas son de uno a varios, con filtro en una sola dirección y activas.

La tabla de productos no traía el id de la categoría, solo el nombre. Para poder relacionarla lo agregué en Power Query combinando con Dim_Categorias.

La foto del modelo está en Modelo.jpeg.

## Calendario

Creé la tabla Dim_Fechas con DAX, desde la primera hasta la última venta, y la marqué como tabla de fechas. Le agregué año, número y nombre del mes, trimestre y semana. Los meses los ordené por número para que no salgan en orden alfabético.

Dim_Fechas = ADDCOLUMNS(
    CALENDAR(MIN(Fact_Ventas[fecha_venta]), MAX(Fact_Ventas[fecha_venta])),
    "Año", YEAR([Date]),
    "Mes Número", MONTH([Date]),
    "Mes Nombre", FORMAT([Date], "MMMM"),
    "Trimestre", "T" & QUARTER([Date]),
    "Semana", WEEKNUM([Date])
)

## Medidas

Las guardé todas en una tabla aparte llamada _Medidas.

Total Ventas = SUM(Fact_Ventas[total_venta])

Ventas Online = CALCULATE([Total Ventas], Fact_Ventas[canal] = "Online")

Ventas YTD = TOTALYTD([Total Ventas], Dim_Fechas[Date])

Ventas LY = CALCULATE([Total Ventas], SAMEPERIODLASTYEAR(Dim_Fechas[Date]))

% Crecimiento Anual =
VAR VentasActual = [Total Ventas]
VAR VentasAnterior = [Ventas LY]
RETURN
    DIVIDE(VentasActual - VentasAnterior, VentasAnterior)

## Control

En la página Validación armé una matriz para revisar que todo diera bien. En 2023 se vendieron 28.764 y en 2024 18.814 (hay datos hasta julio). El acumulado va sumando mes a mes y arranca de nuevo en enero de 2024. Ventas LY queda vacía en 2023 porque no hay año anterior. Contra el mismo período de 2023, las ventas de 2024 crecieron un 44,43 %.

La foto de la matriz está en Validación.jpeg.
