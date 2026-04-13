# SimulacionFin

Proyecto de preparacion, limpieza y analisis exploratorio de datos financieros para estudiar impagos empresariales. El repositorio combina estados financieros historicos, una variable objetivo de impago y un notebook de trabajo para construir datasets listos para analisis estadistico y modelos Logit.

## Objetivo

El objetivo principal es transformar datos financieros empresariales en una base analitica usable para estudiar la probabilidad de impago. Para ello se han realizado tareas de limpieza, normalizacion, tratamiento de valores nulos, fusion de fuentes, creacion de variables financieras y primeras pruebas de modelizacion.

## Estructura del proyecto

```text
.
|-- README.md
|-- tratamiento_valores_nulos.md
`-- Datasets/
    |-- EEFF base de datos.csv
    |-- EEFF_con_variable_Y.csv
    |-- Variable Y.csv
    |-- Variable_Y_normalizada.csv
    |-- fusionado_features_impagos.csv
    |-- fusionado.csv
    `-- normalizacion_y_eda_impagos.ipynb
```

## Archivos principales

| Archivo | Descripcion |
|---|---|
| `Datasets/Variable Y.csv` | Archivo original de la variable objetivo de impago por empresa y periodo mensual. Contiene columnas extra sin nombre que se limpian en el notebook. |
| `Datasets/Variable_Y_normalizada.csv` | Version normalizada de la variable objetivo, con valores booleanos convertidos a `1` y `0`, y valores invalidos tratados como ausentes. |
| `Datasets/EEFF base de datos.csv` | Base original de estados financieros, con informacion contable, sectorial y geografica de empresas por ejercicio. |
| `Datasets/EEFF_con_variable_Y.csv` | Dataset de estados financieros enriquecido con la variable objetivo `Y`. |
| `Datasets/fusionado.csv` | Dataset final en formato ancho por empresa, con variables de los ejercicios 2008, 2009, 2010 y 2011 en columnas separadas. |
| `Datasets/fusionado_features_impagos.csv` | Dataset de modelizacion generado desde el notebook con una fila por empresa, una unica variable objetivo `Y` y variables financieras avanzadas para prediccion de impagos. |
| `Datasets/normalizacion_y_eda_impagos.ipynb` | Notebook principal de limpieza, normalizacion, EDA, tratamiento de nulos, fusion y modelizacion Logit. |
| `tratamiento_valores_nulos.md` | Documento metodologico con la estrategia definida para imputar valores nulos en variables financieras. |

## Trabajo realizado

### 1. Normalizacion de la variable objetivo

En el notebook `normalizacion_y_eda_impagos.ipynb` se carga `Variable Y.csv` y se realiza la limpieza inicial:

- Eliminacion de columnas sin nombre o completamente vacias.
- Eliminacion de filas completamente vacias.
- Conversion de `VERDADERO` a `1`.
- Conversion de `FALSO` a `0`.
- Tratamiento de `#¡NULO!`, cadenas vacias y valores no validos como ausentes.
- Conversion de `ID` a valor numerico.
- Conversion de columnas mensuales a formato numerico.
- Exportacion del resultado a `Variable_Y_normalizada.csv`.

Resultado observado en el notebook:

- `Variable Y.csv`: 33.256 filas y 46 columnas antes de la limpieza estructural.
- Dataset normalizado en notebook: 3.642 filas y 16 columnas.
- Porcentaje total de impagos calculado: `6,96%`.

### 2. Analisis exploratorio de impagos

Se construye una tabla en formato largo mediante `melt`, donde cada fila representa una combinacion empresa-periodo. A partir de esa tabla se calculan:

- Porcentaje total de impagos.
- Porcentaje de impagos por mes y año.
- Porcentaje de impagos por año.
- Graficos de barras para visualizar la evolucion temporal de la tasa de impago.

Tambien se parsean los nombres de periodos mensuales en español para construir una columna de fecha (`fecha`) con formato temporal.

### 3. Carga y tipado de estados financieros

Se carga `EEFF base de datos.csv` y se aplican reglas de limpieza:

- Sustitucion de marcadores de ausentes (`n.d.`, `N.D.`, cadenas vacias) por valores nulos.
- Parseo de numeros con separadores europeos, incluyendo coma decimal y punto de miles.
- Conversion automatica de columnas de texto a numericas cuando al menos el 95% de sus valores no nulos son convertibles.
- Clasificacion de variables por tipo:
  - Identificador.
  - Numerica continua.
  - Numerica entera.
  - Numerica discreta.
  - Binaria numerica.
  - Binaria categorica.
  - Categorica.
- Calculo de porcentaje de nulos y numero de valores unicos por variable.
- Visualizacion de la distribucion de tipos de variables.

Resultado observado:

- `EEFF base de datos.csv`: 11.298 filas y 91 columnas.

### 4. Fusion de estados financieros con la variable objetivo

El proyecto incluye `EEFF_con_variable_Y.csv`, que añade la variable objetivo `Y` a la base de estados financieros.

En el notebook se filtran las empresas que tienen exactamente cuatro registros historicos. Esto permite trabajar con empresas que cuentan con informacion para los ejercicios 2008, 2009, 2010 y 2011.

Resultado observado:

- Subconjunto `df_4`: 7.116 filas.
- Empresas unicas con cuatro ejercicios: 1.779.

### 5. Tratamiento de valores nulos

La estrategia esta documentada en `tratamiento_valores_nulos.md` y aplicada en el notebook sobre el subconjunto de empresas con cuatro ejercicios.

Decisiones implementadas:

- Reconstruccion de totales contables cuando se pueden calcular desde componentes:
  - `Total activo (A + B)` = `Activo no corriente` + `Activo corriente`.
  - `Total patrimonio neto y pasivo (A + B + C)` = `Patrimonio neto` + `Pasivo no corriente` + `Pasivo corriente`.
- Imputacion de partidas financieras clave mediante medianas jerarquicas:
  - Mediana por empresa y año.
  - Mediana por año y CNAE.
  - Mediana por año.
- Imputacion especifica de `Gastos de personal` por año y tramo de empleados.
- Tratamiento diferenciado de `Existencias`:
  - Empresas de servicios: nulos imputados a `0`.
  - Resto de empresas: mediana por año y CNAE.
- Imputacion a `0` de partidas financieras donde el nulo representa ausencia de actividad.

### 6. Variables indicadoras creadas

Antes de imputar a cero ciertas partidas, se crean indicadores binarios para conservar informacion sobre la existencia de conceptos financieros relevantes:

- `recibe_subvenciones`: indica si la empresa tiene subvenciones, donaciones o legados recibidos.
- `tiene_inversiones_grupo`: indica si la empresa tiene inversiones en empresas del grupo y asociadas a largo plazo.
- `tiene_arrendamiento_financiero`: indica si la empresa presenta acreedores por arrendamiento financiero.

En `fusionado.csv`, estos indicadores aparecen separados por ejercicio:

- `recibe_subvenciones_2008`, `recibe_subvenciones_2009`, `recibe_subvenciones_2010`, `recibe_subvenciones_2011`.
- `tiene_inversiones_grupo_2008`, `tiene_inversiones_grupo_2009`, `tiene_inversiones_grupo_2010`, `tiene_inversiones_grupo_2011`.
- `tiene_arrendamiento_financiero_2008`, `tiene_arrendamiento_financiero_2009`, `tiene_arrendamiento_financiero_2010`, `tiene_arrendamiento_financiero_2011`.

### 7. Ratios financieros creados en el notebook

El notebook calcula ratios de liquidez a partir de partidas del balance:

- `Current Ratio` = `Activo corriente` / `Pasivo corriente`.
- `Quick Ratio` = (`Activo corriente` - `Existencias`) / `Pasivo corriente`.
- `Cash Ratio` = `Efectivo y otros activos liquidos equivalentes` / `Pasivo corriente`.

Tambien calcula ratios de endeudamiento y caja usando proxies contables disponibles:

- `DTI total` = (`Pasivo no corriente` + `Pasivo corriente`) / (`Importe neto de la cifra de negocios` + `Otros ingresos de explotacion` + `Ingresos financieros`).
- `Deuda total / Activos totales` = (`Pasivo no corriente` + `Pasivo corriente`) / `Total activo`.
- `Deuda neta / EBITDA` = (`Deudas a largo plazo` + `Deudas a corto plazo` + `Acreedores por arrendamiento financiero` - `Efectivo y otros activos liquidos equivalentes`) / (`Resultado de explotacion` + `Amortizacion del inmovilizado`).
- `Deuda financiera / Patrimonio neto` = (`Deudas a largo plazo` + `Deudas a corto plazo` + `Acreedores por arrendamiento financiero`) / `Patrimonio neto`.
- `Days Cash on Hand` = `Efectivo y otros activos liquidos equivalentes` / ((`Aprovisionamientos` + `Gastos de personal` + `Otros gastos de explotacion`) / 365).

Despues de transformar los datos a formato ancho, tambien se calculan agregados por empresa:

- Mediana de cuatro años para cada ratio.
- Version normalizada de la mediana de cuatro años:
  - `Current Ratio_mediana_4_anios_norm`.
  - `Quick Ratio_mediana_4_anios_norm`.
  - `Cash Ratio_mediana_4_anios_norm`.
  - `DTI total_mediana_4_anios_norm`.
  - `Deuda total / Activos totales_mediana_4_anios_norm`.
  - `Deuda neta / EBITDA_mediana_4_anios_norm`.
  - `Deuda financiera / Patrimonio neto_mediana_4_anios_norm`.
  - `Days Cash on Hand_mediana_4_anios_norm`.

Para `Days Cash on Hand_mediana_4_anios_norm`, el notebook crea antes de imputar la variable binaria `days_cash_missing`, que marca los casos en los que el ratio era incalculable. Despues rellena esos nulos con la mediana del sector CNAE y, si no existe mediana sectorial disponible, con la mediana global.

Nota: el archivo `fusionado.csv` actual contiene el dataset ancho y los indicadores por año. Los ratios agregados y normalizados estan definidos en el notebook como parte del flujo de trabajo de modelizacion.

### 8. Transformacion a formato ancho

Se transforma el subconjunto tratado a una fila por empresa (`ID`) y columnas por ejercicio. Cada variable financiera queda expandida con sufijos:

- `_2008`
- `_2009`
- `_2010`
- `_2011`

La variable objetivo se valida para comprobar que `Y_2008`, `Y_2009`, `Y_2010` y `Y_2011` sean consistentes antes de fusionarlas en una unica variable `Y` dentro del notebook.

Resultado del archivo final existente:

- `Datasets/fusionado.csv`: 1.779 filas y 374 columnas.
- Incluye una columna `Unnamed: 0` generada como indice de exportacion.

### 9. Feature engineering para prediccion de impagos

El notebook crea una base de modelizacion llamada `df_modelo_impagos` y la exporta a `Datasets/fusionado_features_impagos.csv`.

Resultado generado:

- `Datasets/fusionado_features_impagos.csv`: 1.773 filas y 778 columnas.
- Variable objetivo unica: `Y`.
- Tasa de impago en la base modelable: `9,42%`.
- Se eliminan las empresas sin variable objetivo disponible.

Grupos de variables implementadas:

- Variables de tamano: `log_total_assets`, `log_sales`, `log_employees`.
- Liquidez y capital circulante: `current_ratio`, `quick_ratio`, `cash_ratio`, `working_capital_to_assets`, `cash_to_assets`.
- Endeudamiento y solvencia: `total_liabilities_to_assets`, `financial_debt_to_assets`, `financial_debt_to_ebitda`, `net_debt_to_ebitda`, `debt_to_equity`, `financial_debt_to_equity`, `short_term_debt_share`, `bank_debt_share`.
- Cobertura de intereses y carga financiera: `interest_coverage_ebit`, `interest_coverage_ebitda`, `financial_expense_to_sales`, `financial_expense_to_revenue`.
- Rentabilidad: `ebit_margin`, `ebitda_margin`, `net_margin`, `roa`, `roe`.
- Ciclo de cobro, inventario y pago: `dso`, `dio`, `dpo`, `cash_conversion_cycle`, `inventory_to_sales`, `receivables_to_sales`, `payables_to_sales`, `payables_to_assets`.
- Costes: `operating_cost_to_sales`, `personnel_cost_to_sales`.
- Flags de deterioro financiero: `loss_flag`, `negative_ebit_flag`, `negative_equity_flag`, `liabilities_greater_than_assets_flag`, `high_leverage_flag`, `zero_sales_flag`, `zero_equity_flag`, `zero_financial_expense_flag`.
- Flags de partidas estructurales: `has_financial_investments_lp`, `has_financial_investments_cp`, `has_deferred_tax_assets`, `has_deferred_tax_liabilities`, `has_provisions_lp`, `has_provisions_cp`, `has_exchange_differences`, `has_special_debt`, `has_group_debt`, `has_group_receivables`.

Para cada variable anual relevante se generan versiones por ejercicio (`2008`, `2009`, `2010`, `2011`) y agregados temporales:

- Media de cuatro anos: `_mean_4y`.
- Mediana de cuatro anos: `_median_4y`.
- Minimo de cuatro anos: `_min_4y`.
- Maximo de cuatro anos: `_max_4y`.
- Desviacion tipica de cuatro anos: `_std_4y`.
- Cambio absoluto 2011-2008: `_delta_2011_2008`.
- Cambio porcentual 2011-2008: `_pct_change_2011_2008`.
- Pendiente anual aproximada 2008-2011: `_slope_2008_2011`.

Tambien se crean variables de tendencia:

- `sales_declined_2_or_more_years`.
- `assets_declined_2_or_more_years`.
- `cash_declined_2_or_more_years`.
- `equity_declined_2_or_more_years`.
- `debt_increased_2_or_more_years`.
- `consecutive_loss_years`.
- Conteos de cuatro anos para flags de perdidas, EBIT negativo, patrimonio neto negativo, pasivo superior al activo, ventas cero, patrimonio cero, gastos financieros cero y alto endeudamiento.

Variables de auditoria implementadas:

- `audit_worse_than_approved_flag_2011`.
- `audit_denied_flag_2011`.
- `audit_changed_flag_2008_2011`.
- `audit_worsened_flag_2008_2011`.
- `audit_severity_2011`.
- One-hot encoding de la calificacion auditora de 2011.

Variables sectoriales y geograficas:

- Target encoding out-of-fold para `CNAE`, `Provincia`, `CNAE` a dos digitos y combinacion `Provincia + CNAE`.
- Variables relativas a la mediana sectorial CNAE: `current_ratio_vs_cnae_median`, `debt_assets_vs_cnae_median`, `roa_vs_cnae_median`, `sales_growth_vs_cnae_median`, `cash_assets_vs_cnae_median`.

Para ratios incalculables por denominadores cero o ausentes, el notebook crea indicadores `_was_invalid_flag` antes de dejar el valor como nulo. Esto permite al modelo capturar que el ratio no era calculable sin confundirlo con un valor financiero real.

### 10. Modelizacion Logit

El notebook incluye primeras pruebas de regresion logistica con `statsmodels`:

- Modelo Logit general usando variables numericas disponibles.
- Modelo Logit especifico de liquidez usando:
  - `Current Ratio_mediana_4_anios_norm`.
  - `Quick Ratio_mediana_4_anios_norm`.
  - `Cash Ratio_mediana_4_anios_norm`.

Estos modelos sirven como primera validacion de la utilidad predictiva de las variables financieras creadas.

## Funcionalidades creadas

- Limpieza automatica de columnas vacias y columnas `Unnamed`.
- Normalizacion binaria de la variable objetivo de impago.
- Exportacion de la variable objetivo normalizada.
- Calculo de tasa total de impago.
- Calculo de tasa de impago por mes/año.
- Calculo de tasa de impago por año.
- Visualizaciones exploratorias con `matplotlib` y `seaborn`.
- Parseo robusto de numeros financieros con formato europeo.
- Clasificacion automatica de tipos de variables.
- Resumen de nulos, valores unicos y ejemplos por variable.
- Documentacion metodologica del tratamiento de nulos.
- Reconstruccion de totales contables desde componentes.
- Imputacion jerarquica mediante medianas.
- Imputacion sectorial para existencias.
- Creacion de indicadores binarios financieros.
- Calculo de ratios de liquidez.
- Transformacion de datos longitudinales a formato ancho por empresa.
- Preparacion de variables agregadas de cuatro años.
- Normalizacion de ratios agregados.
- Creacion de `df_modelo_impagos` con variables avanzadas para prediccion de impagos.
- Exportacion de `fusionado_features_impagos.csv`.
- Creacion de ratios de liquidez, solvencia, rentabilidad, cobertura, ciclo de caja y costes.
- Creacion de agregados temporales, tendencias y flags de deterioro.
- Target encoding out-of-fold para CNAE y provincia.
- Creacion de variables relativas a la mediana sectorial.
- Creacion de indicadores de ratios incalculables.
- Primera modelizacion Logit general.
- Primera modelizacion Logit centrada en liquidez.

## Requisitos

El notebook utiliza Python y las siguientes librerias:

```bash
pandas
numpy
matplotlib
seaborn
statsmodels
```

En el propio notebook se incluye una celda para instalar las dependencias principales:

```python
%pip install pandas numpy matplotlib seaborn
```

Para ejecutar los modelos Logit tambien es necesario tener instalado `statsmodels`:

```bash
pip install statsmodels
```

## Como ejecutar el proyecto

1. Abrir `Datasets/normalizacion_y_eda_impagos.ipynb` en Jupyter Notebook, JupyterLab o VS Code.
2. Ejecutar las celdas en orden.
3. Revisar la exportacion de `Variable_Y_normalizada.csv`.
4. Revisar los analisis de impagos y estados financieros.
5. Ejecutar las secciones de tratamiento de nulos y fusion.
6. Ejecutar la seccion de feature engineering para generar `fusionado_features_impagos.csv`.
7. Ejecutar las secciones de Logit.

## Notas de calidad de datos

- Los CSV originales usan distintos separadores (`;` y `,`), por lo que el notebook especifica el separador correcto en cada carga.
- Algunos nombres de columnas financieras contienen saltos de linea internos, por ejemplo `mil EUR`.
- La base `Variable Y.csv` contiene columnas sin nombre que se eliminan antes del analisis.
- `fusionado.csv` conserva una columna `Unnamed: 0`, generada al exportar el indice. Puede eliminarse en futuros pasos con `index=False` al exportar.
- Algunos textos del notebook muestran caracteres con codificacion incorrecta en ciertas salidas, aunque los datos se leen con `utf-8-sig` en las cargas principales.

## Estado actual

El proyecto ya cuenta con un flujo completo desde datos brutos hasta dataset preparado para analisis:

- Datos originales cargados y limpiados.
- Variable objetivo normalizada.
- Estados financieros enriquecidos con `Y`.
- Empresas con cuatro ejercicios seleccionadas.
- Valores nulos tratados con criterios contables y estadisticos.
- Variables financieras e indicadores creados.
- Dataset ancho final disponible.
- Dataset de features avanzadas disponible en `Datasets/fusionado_features_impagos.csv`.
- Modelos Logit iniciales definidos en el notebook.
