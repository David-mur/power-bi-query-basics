# power-bi-query-basics

Documentación de Transformación en Power Query de ventaslegacy.pbix

Se recibió un archivo crudo de ventas en formato de Excel para procesar en Power BI, y el primer paso es realizar varias transformaciones mediante Power Query, las cuales se detallan a continuación:

## 1. Cambio de nombre de columnas:
| Nombre Original (Legacy) | Nombre Renombrado |
| :--- | :--- |
| `COD_OP` | `ID Operacion` |
| `COD_CLI` | `ID Cliente` |
| `NOM_CLI` | `Nombre Cliente` |
| `MAIL_CLI` | `Correo Cliente` |
| `TEL_CLI` | `Telefono Cliente` |
| `CIU_CLI` | `Ciudad` |
| `PROV_CL` | `Provincia` |
| `SEG_CL` | `Segmento Cliente` |
| `FLG_ACT` | `Cliente Activo` |
| `F_ALTA_CLI` | `Fecha Alta Cliente` |
| `F_VTA` | `Fecha Venta` |
| `COD_PROD` | `ID Producto` |
| `DESC_PROD` | `Producto` |
| `RUBRO_PROD` | `Categoria Producto` |
| `CANT` | `Cantidad` |
| `PU_VENTA` | `Precio Unitario` |
| `DTO_PCT` | `Porcentaje Descuento` |
| `TOT_VTA` | `Total Venta` |
| `COD_MON` | `Moneda` |
| `CANAL_VTA` | `Canal Venta` |

## 2. Justificación de los Tipos de Datos Elegidos

- **Texto (`Text`):** Asignado a `ID Operacion`, `ID Cliente` e `ID Producto`. Aunque contengan dígitos, son identificadores categóricos que no representan cantidades medibles ni sumables. Clasificarlos como texto evita sumas automáticas en Power BI. También se aplicó a campos cualitativos como nombres, correos, ciudades, provincias, canales y segmentos.
- **Fecha (`Date`):** Aplicado a `Fecha Venta` y `Fecha Alta Cliente`. Se eliminó la estampa de hora (`00:00:00`) para reducir la cardinalidad de la columna, optimizar la compresión en memoria y habilitar funciones de Inteligencia de Tiempo en DAX.
- **Número Entero (`Whole Number`):** Asignado a `Cantidad`, correspondiente a unidades físicas discretas vendidas.
- **Número Decimal Fijo (`Fixed Decimal Number / Currency`):** Aplicado a `Precio Unitario` y `Total Venta` para almacenar valores monetarios con precisión exacta y evitar errores de redondeo flotante.
- **Número Decimal (`Decimal Number`):** Asignado a `Porcentaje Descuento` para operar valores continuos en escala proporcional (0.0 a 1.0).

---

## 3. Resolución de Valores Nulos y Duplicados

- **Duplicados:** Se eliminaron en dos momentos. Primero a nivel transaccional en la tabla base utilizando `ID Operacion` para remover registros duplicados por fallas del sistema legacy. Segundo, al crear la dimensión `D_Clientes` aplicando *Quitar duplicados* sobre `ID Cliente` para garantizar unicidad.
- **Nulos en Contacto:** En `Telefono Cliente` y `Correo Cliente`, los valores `null` se sustituyeron por `"Sin registro"` para evitar categorías `(En blanco)` en visualizaciones y filtros.
- **Nulos en Descuentos:** En `Porcentaje Descuento`, los `null` se reemplazaron por `0` para mantener consistencia matemática.
- **Recálculo de Totales Nulos:** En las filas con `Total Venta` faltante, el importe se reconstruyó mediante la columna personalizada: `[Cantidad] * [Precio Unitario] * (1 - [Porcentaje Descuento])`.

---

## 4. Criterio de Separación de Tablas (Modelo en Estrella)

Para estructurar los datos según las mejores prácticas de inteligencia de negocios, se implementó un **Modelo en Estrella (*Star Schema*)**:

- **Tabla Dimensión (`D_Clientes`):** Contiene 120 registros únicos con atributos descriptivos y estáticos del cliente (`ID Cliente`, `Nombre Cliente`, `Correo Cliente`, `Telefono Cliente`, `Ciudad`, `Provincia`, `Segmento Cliente`, `Cliente Activo`, `Fecha Alta Cliente`).
- **Tabla de Hechos (`F_Ventas`):** Registra 901 eventos transaccionales con importes, cantidades, fechas e identificadores, conservando `ID Cliente` como *Clave Foránea*
- (*Foreign Key*) para vincular ambas tablas en una relación **1 a Varios (1:*)**.
- **Beneficio:** Reduce drásticamente la redundancia de datos, minimiza el tamaño del archivo `.pbix` y optimiza el rendimiento de consulta.

---

## 5. Evidencias de los cambios realizados en Power Query
<img width="1600" height="900" alt="WhatsApp Image 2026-10-17 at 1 51 47 AM" src="https://github.com/user-attachments/assets/9ed1524d-9718-462c-9d7c-7c06e1054cc3" />
<img width="1600" height="900" alt="WhatsApp Image 2026-10-17 at 1 51 31 AM" src="https://github.com/user-attachments/assets/fca66953-d55c-49d8-990d-56f240b4ba5e" />
<img width="1600" height="900" alt="WhatsApp Image 2026-10-17 at 1 50 34 AM" src="https://github.com/user-attachments/assets/72d589f4-5582-4d26-9b68-783eb71ba29f" />
<img width="1600" height="900" alt="WhatsApp Image 2026-10-17 at 1 50 19 AM" src="https://github.com/user-attachments/assets/7db3381d-64f5-4acd-a92e-31deb58dd06b" />

   
   
