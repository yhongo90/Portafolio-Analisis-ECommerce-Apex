# 📊 Apex Retail Insights - Executive Sales & Financial Analytics Suite

## 💼 1. Escenario de Negocio e Identidad Corporativa
Este proyecto implementa una solución integral de Business Intelligence para la firma transaccional de comercio electrónico **Apex | Retail Insights**. El entregable consiste en un informe analítico interactivo de dos páginas diseñado bajo un estándar ejecutivo premium, optimizado para responder de forma inmediata a los requerimientos de la Dirección Comercial y la Dirección de Finanzas (CFO).

*   **Paleta de Colores Institucional:** Azul Marino Profundo (`#1E293B`) para componentes estructurales, Blanco Puro (`#FFFFFF`) para bloques modulares aislados y Oro/Ámbar (`#D97706`) como color de acento estratégico para KPIs de eficiencia.
*   **Diseño de Interfaz (UI):** Estructura en bloques limpios basados en el patrón de lectura en "Z", con bordes visuales suavizados (esquinas redondeadas a 8px) y sombras sutiles para maximizar la legibilidad y el enfoque analítico.

---

## 🛠️ 2. Proceso de ETL y Auditoría de Calidad (Data Quality)
La fuente original de datos (`Sales Data - Raw_Data.csv`) presentaba severas inconsistencias sintéticas detectadas durante la fase de perfilado. El proceso de extracción, transformación y limpieza se ejecutó mediante **Power Query (Lenguaje M)** bajo los siguientes criterios técnicos:

*   **Gobernanza de Vacíos:** Se aislaron filas nulas y registros fantasma en el extremo inferior del archivo mediante reglas de exclusión aplicadas sobre la llave primaria (`Order_ID`), garantizando un set con 100% de filas válidas.
*   **Tratamiento de Métricas Financieras:** Los campos de moneda contenían texto incrustado (`" INR"` y comillas dobles). Se programó un reemplazo de cadenas y una conversión forzada mediante **Configuración Regional (Inglés de Estados Unidos)** para asimilar de forma nativa los formatos de miles (comas) y habilitar las columnas como números decimales manipulables matemáticamente.
*   **Normalización de Texto y Homologación Semántica:** Se aplicaron transformaciones de formato (`Trim` y `Capitalize Each Word`) en los catálogos de texto para eliminar espacios espurios e inconsistencias de caja. Adicionalmente, se corrigieron errores de entrada manual mediante reemplazos directos estructurados (*Vacum Cleaner* ➔ *Vacuum Cleaner*, *Head Phones* ➔ *Headphones*, *Smart Watch* ➔ *Smartwatch*, *Make Up Kit* ➔ *Makeup Kit*).
*   **Resolución de Inconsistencias de Negocio (Cantidades Negativas):** Se detectó una anomalía crítica de integridad donde registros con `Quantity` negativa (`-3`, `-2`) presentaban montos de `Revenue` positivos elevados. Al concluirse que se trataba de un error de consistencia del sistema origen, se aplicó una función de **Valor Absoluto (`Number.Abs`)** para evitar la neutralización del volumen físico real del reporte.

### 💻 Código M de Limpieza Aplicado:
```powerquery
let
    Origen = Csv.Document(File.Contents("C:\Users\HP\Documents\analisis de datos\Portafolio_Ventas_Finanzas\01_Datos_Originales\Sales Data - Raw_Data.csv"),[Delimiter=",", Columns=9, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    #"Encabezados promovidos" = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Encabezados promovidos",{{"Order_ID", Int64.Type}, {"Order_Date", type date}, {"Customer_ID", type text}, {"Product", type text}, {"Category", type text}, {"Region", type text}, {"Quantity", Int64.Type}, {"Revenue", type text}, {"Profit", type text}}),
    #"Filas filtradas" = Table.SelectRows(#"Tipo cambiado", each ([Customer_ID] <> "") and ([Category] <> "") and ([Region] <> "")),
    #"Filas filtradas1" = Table.SelectRows(#"Filas filtradas", each [Category] <> null and [Category] <> ""),
    #"Filas filtradas2" = Table.SelectRows(#"Filas filtradas1", each [Quantity] <> null and [Quantity] <> ""),
    #"Filas filtradas3" = Table.SelectRows(#"Filas filtradas2", each [Revenue] <> null and [Revenue] <> ""),
    #"Filas filtradas4" = Table.SelectRows(#"Filas filtradas3", each [Profit] <> null and [Profit] <> ""),
    #"Valor reemplazado" = Table.ReplaceValue(#"Filas filtradas4"," INR","",Replacer.ReplaceText,{"Revenue", "Profit"}),
    #"Valor reemplazado1" = Table.ReplaceValue(#"Valor reemplazado","""","",Replacer.ReplaceText,{"Revenue", "Profit"}),
    #"Tipo cambiado con configuración regional" = Table.TransformColumnTypes(#"Valor reemplazado1", {{"Revenue", type number}}, "en-US"),
    #"Errores quitados" = Table.RemoveRowsWithErrors(#"Tipo cambiado con configuración regional", {"Revenue"}),
    #"Tipo cambiado1" = Table.TransformColumnTypes(#"Errores quitados",{{"Profit", type number}}),
    #"Errores quitados1" = Table.RemoveRowsWithErrors(#"Tipo cambiado1", {"Profit"}),
    #"Poner En Mayúsculas Cada Palabra" = Table.TransformColumns(#"Errores quitados1",{{"Customer_ID", Text.Proper, type text}, {"Region", Text.Proper, type text}, {"Product", Text.Proper, type text}, {"Category", Text.Proper, type text}}),
    #"Texto recortado" = Table.TransformColumns(#"Poner En Mayúsculas Cada Palabra",{{"Product", Text.Trim, type text}, {"Region", Text.Trim, type text}, {"Category", Text.Trim, type text}}),
    #"Valor reemplazado2" = Table.ReplaceValue(#"Texto recortado","Vacum Cleaner","Vacuum Cleaner",Replacer.ReplaceText,{"Product"}),
    #"Valor reemplazado3" = Table.ReplaceValue(#"Valor reemplazado2","Head Phones","Headphones",Replacer.ReplaceText,{"Product"}),
    #"Valor reemplazado4" = Table.ReplaceValue(#"Valor reemplazado3","Smart Watch","Smartwatch",Replacer.ReplaceText,{"Product"}),
    #"Valor reemplazado5" = Table.ReplaceValue(#"Valor reemplazado4","Make Up Kit","Makeup Kit",Replacer.ReplaceText,{"Product"}),
    #"Valor absoluto calculado" = Table.TransformColumns(#"Valor reemplazado5",{{"Quantity", Number.Abs, Int64.Type}})
in
    #"Valor absoluto calculado"
```

---

## 📐 3. Arquitectura del Modelo de Datos (Modelo en Estrella)
Para garantizar el máximo rendimiento del motor VertiPaq de Power BI, se rompió la estructura de tabla plana mediante la creación de consultas referenciadas, consolidando un **Modelo en Estrella optimizado (Relaciones 1:*)**:

*   **Tabla de Hechos (`Fact_Sales`):** Almacena las llaves relacionales y las métricas numéricas base (`Quantity`, `Revenue`, `Profit`).
*   **Dimensión Producto (`Dim_Product`):** Catálogo maestro único que contiene `Product` y `Category`.
*   **Dimensión Geografía (`Dim_Region`):** Atributos geográficos únicos basados en la columna `Region`.
*   **Dimensión Tiempo (`Dim_Calendar`):** Tabla de fechas pura autogenerada mediante DAX para asegurar el correcto funcionamiento de las funciones de inteligencia de tiempo. Está ordenada cronológicamente mediante el indexador `Mes Nro`.

---

## 🚀 4. Ingeniería de Métricas (Lógica de Negocio en DAX)
Todas las fórmulas fueron centralizadas dentro de un contenedor aislado de medidas (`_Medidas_Ventas`), implementando las siguientes expresiones de negocio:

*   **Ingresos Totales (Total Revenue):** Suma directa del campo monetario limpio.
    ```dax
    Total Revenue = SUM(Fact_Sales[Revenue])
    ```
*   **Volumen de Pedidos (Total Orders):** Conteo único de transacciones para mitigar duplicados.
    ```dax
    Total Orders = DISTINCTCOUNT(Fact_Sales[Order_ID])
    ```
*   **Ticket Promedio (Average Ticket / AOV):** Uso de la función segura `DIVIDE` para prevenir errores de división por cero (`NaN`) en entornos dinámicos filtrados.
    ```dax
    Average Ticket = DIVIDE([Total Revenue], [Total Orders], 0)
    ```
*   **Margen de Ganancia Comercial (Profit Margin %):** Indicador de rentabilidad neta por dólar ingresado.
    ```dax
    Profit Margin % = 
    VAR TotalProfit = SUM(Fact_Sales[Profit])
    RETURN DIVIDE(TotalProfit, [Total Revenue], 0)
    ```

---

## 📖 5. Storytelling y Diagnóstico Ejecutivo de Datos

### Página 1: Performance Comercial (Enfoque Ventas)
*   **El Diagnóstico Inicial:** Durante el año fiscal analizado, la firma consolidó una facturación macro de **$4 mil millones (Total Revenue)** apalancada por **49,000 órdenes de compra** independientes, con un ticket promedio de **$85.04 mil**.
*   **El Hallazgo Crítico:** El negocio exhibía un comportamiento mensual robusto y predecible por encima de los $500 millones hasta julio. Sin embargo, a partir de **Agosto se identifica una contracción catastrófica del 80%** en los ingresos, desplomándose la facturación mensual.
*   **Causa Raíz Analítica:** Al contrastar la tendencia con la estructura del catálogo, se visualiza una dependencia de riesgo estructural sobre la categoría de **Electronics** (la cual sostiene casi el 70% de los ingresos totales con más de $2.5 mil millones). La caída del negocio se asocia directamente a una parálisis de ventas en esta línea específica.

### Página 2: Salud Financiera y Control de Márgenes (Enfoque Finanzas)
*   **El Hallazgo de Blindaje:** A pesar del fuerte impacto en los volúmenes comerciales del segundo semestre, la auditoría de costos revela una excelente noticia: **el Margen de Ganancia Neto corporativo se mantuvo totalmente estable e intacto en un 16.01% global**.
*   **Eficiencia Operativa:** La matriz contable y el análisis de barras geográficas confirman una disciplina contable perfecta. Absolutamente todas las categorías de producto y todas las zonas operativas regionales (Delhi, Ahmedabad, Chennai, etc.) operan bajo la misma estructura simétrica de costos (COGS) y precios, manteniendo un margen individual del 16.09%. La empresa enfrenta una crisis de volumen de demanda o suministro, pero sus bases operativas y financieras están completamente sanas y controladas.
