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

- `Datasets/fusionado_features_impagos.csv`: 1.773 filas y 822 columnas.
- Variable objetivo unica: `Y`.
- Tasa de impago en la base modelable: `9,42%`.
- Se eliminan las empresas sin variable objetivo disponible.

Detalle de variables implementadas:

| Variable base | Tipo | Calculo / interpretacion |
| --- | --- | --- |
| `log_total_assets` | Continua transformada | `log1p(Total activo)`. Solo se aplica a valores no negativos; reduce asimetria de escala. |
| `log_sales` | Continua transformada | `log1p(Importe neto de la cifra de negocios)`. Solo se aplica a valores no negativos. |
| `log_employees` | Continua transformada | `log1p(Numero empleados)`. Solo se aplica a valores no negativos. |
| `equity` | Continua monetaria | Patrimonio neto sin normalizar. |
| `current_ratio` | Continua, ratio | `Activo corriente / Pasivo corriente`. Mide liquidez corriente. |
| `quick_ratio` | Continua, ratio | `(Activo corriente - Existencias) / Pasivo corriente`. Mide liquidez sin inventarios. |
| `cash_ratio` | Continua, ratio | `Efectivo y otros activos liquidos equivalentes / Pasivo corriente`. |
| `working_capital_to_assets` | Continua, ratio | `(Activo corriente - Pasivo corriente) / Total activo`. |
| `cash_to_assets` | Continua, ratio | `Efectivo / Total activo`. |
| `inventory_to_sales` | Continua, ratio | `Existencias / Ventas`. |
| `receivables_to_sales` | Continua, ratio | `Deudores comerciales y otras cuentas a cobrar / Ventas`. |
| `payables_to_sales` | Continua, ratio | `Acreedores comerciales y otras cuentas a pagar / Ventas`. |
| `payables_to_assets` | Continua, ratio | `Acreedores comerciales y otras cuentas a pagar / Total activo`. |
| `dso` | Continua, dias | `(Deudores / Ventas) * 365`. Dias aproximados de cobro. |
| `dio` | Continua, dias | `(Existencias / abs(Aprovisionamientos)) * 365`. Dias aproximados de inventario. |
| `dpo` | Continua, dias | `(Acreedores / abs(Aprovisionamientos)) * 365`. Dias aproximados de pago. |
| `cash_conversion_cycle` | Continua, dias | `dso + dio - dpo`. Ciclo de conversion de caja. |
| `total_liabilities_to_assets` | Continua, ratio | `(Pasivo no corriente + Pasivo corriente) / Total activo`. |
| `financial_debt_to_assets` | Continua, ratio | Deuda financiera / Total activo. La deuda financiera suma deudas a largo plazo, deudas a corto plazo y leasing. |
| `financial_debt_to_ebitda` | Continua, ratio | Deuda financiera / EBITDA proxy. |
| `net_debt_to_ebitda` | Continua, ratio | `(Deuda financiera - Efectivo) / EBITDA proxy`. |
| `debt_to_equity` | Continua, ratio | Pasivo total / Patrimonio neto. |
| `financial_debt_to_equity` | Continua, ratio | Deuda financiera / Patrimonio neto. |
| `short_term_debt_share` | Continua, proporcion | Deuda a corto plazo / deuda total de corto y largo plazo. |
| `bank_debt_share` | Continua, proporcion | Deuda con entidades de credito / deuda total de corto y largo plazo. |
| `ebit_margin` | Continua, ratio | EBIT / Ventas. EBIT se aproxima con `Resultado de explotacion`. |
| `ebitda_margin` | Continua, ratio | EBITDA proxy / Ventas. EBITDA proxy = EBIT + `abs(Amortizacion del inmovilizado)`. |
| `net_margin` | Continua, ratio | Resultado del ejercicio / Ventas. |
| `roa` | Continua, ratio | Resultado del ejercicio / Total activo. |
| `roe` | Continua, ratio | Resultado del ejercicio / Patrimonio neto. |
| `interest_coverage_ebit` | Continua, ratio | EBIT / `abs(Gastos financieros)`. |
| `interest_coverage_ebitda` | Continua, ratio | EBITDA proxy / `abs(Gastos financieros)`. |
| `financial_expense_to_sales` | Continua, ratio | `abs(Gastos financieros) / Ventas`. |
| `financial_expense_to_revenue` | Continua, ratio | `abs(Gastos financieros) / Ingresos totales`. |
| `operating_cost_to_sales` | Continua, ratio | Costes operativos / Ventas. |
| `personnel_cost_to_sales` | Continua, ratio | `abs(Gastos de personal) / Ventas`. |
| `free_cash_flow` | Continua monetaria | Proxy contable de flujo de caja libre. Se detalla debajo. |
| `free_cash_flow_to_sales` | Continua, ratio | `free_cash_flow / Ventas`. |
| `free_cash_flow_to_assets` | Continua, ratio | `free_cash_flow / Total activo`. |

Variables binarias financieras:

| Variable base | Tipo | Valor 1 cuando... |
| --- | --- | --- |
| `loss_flag` | Binaria 0/1 | El resultado del ejercicio es negativo. |
| `negative_ebit_flag` | Binaria 0/1 | El EBIT es negativo. |
| `negative_equity_flag` | Binaria 0/1 | El patrimonio neto es negativo. |
| `liabilities_greater_than_assets_flag` | Binaria 0/1 | El pasivo total supera el total activo. |
| `zero_sales_flag` | Binaria 0/1 | Las ventas son cero o practicamente cero (`abs(ventas) <= 1e-9`). |
| `zero_equity_flag` | Binaria 0/1 | El patrimonio neto es cero o practicamente cero. |
| `zero_financial_expense_flag` | Binaria 0/1 | Los gastos financieros son cero o practicamente cero. |
| `high_leverage_flag` | Binaria 0/1 | `total_liabilities_to_assets > 0.8`. |
| `has_financial_investments_lp` | Binaria 0/1 | Hay inversiones financieras a largo plazo. |
| `has_financial_investments_cp` | Binaria 0/1 | Hay inversiones financieras a corto plazo. |
| `has_deferred_tax_assets` | Binaria 0/1 | Hay activos por impuesto diferido. |
| `has_deferred_tax_liabilities` | Binaria 0/1 | Hay pasivos por impuesto diferido. |
| `has_provisions_lp` | Binaria 0/1 | Hay provisiones a largo plazo. |
| `has_provisions_cp` | Binaria 0/1 | Hay provisiones a corto plazo. |
| `has_exchange_differences` | Binaria 0/1 | Hay diferencias de cambio. |
| `has_special_debt` | Binaria 0/1 | Hay deuda con caracteristicas especiales a corto o largo plazo. |
| `has_group_debt` | Binaria 0/1 | Hay deudas con empresas del grupo a corto o largo plazo. |
| `has_group_receivables` | Binaria 0/1 | Hay inversiones en empresas del grupo a corto o largo plazo. |

Para cada variable anual se generan columnas por ejercicio con prefijo `2008_`, `2009_`, `2010_` y `2011_`. Ejemplo: `2011_current_ratio`.

Tambien se generan agregados temporales por variable base:

| Sufijo | Tipo | Calculo |
| --- | --- | --- |
| `_mean_4y` | Continua agregada | Media de los cuatro ejercicios. |
| `_median_4y` | Continua agregada | Mediana de los cuatro ejercicios. |
| `_min_4y` | Continua agregada | Minimo observado entre 2008 y 2011. |
| `_max_4y` | Continua agregada | Maximo observado entre 2008 y 2011. |
| `_std_4y` | Continua agregada | Desviacion tipica entre 2008 y 2011. |
| `_delta_2011_2008` | Continua agregada | Valor de 2011 menos valor de 2008. |
| `_pct_change_2011_2008` | Continua agregada | Cambio porcentual entre 2008 y 2011. |
| `_slope_2008_2011` | Continua agregada | Pendiente anual aproximada: `(valor_2011 - valor_2008) / 3`. |
| `_count_4y` | Conteo entero | Para variables binarias, numero de ejercicios con valor `1`, de 0 a 4. |

Variables de tendencia y deterioro:

| Variable | Tipo | Calculo / interpretacion |
| --- | --- | --- |
| `sales_declined_2_or_more_years` | Binaria 0/1 | Vale 1 si `log_sales` desciende en al menos dos transiciones anuales. |
| `assets_declined_2_or_more_years` | Binaria 0/1 | Vale 1 si `log_total_assets` desciende en al menos dos transiciones anuales. |
| `cash_declined_2_or_more_years` | Binaria 0/1 | Vale 1 si `cash_to_assets` desciende en al menos dos transiciones anuales. |
| `equity_declined_2_or_more_years` | Binaria 0/1 | Vale 1 si `equity` desciende en al menos dos transiciones anuales. |
| `debt_increased_2_or_more_years` | Binaria 0/1 | Vale 1 si `financial_debt_to_assets` aumenta en al menos dos transiciones anuales. |
| `consecutive_loss_years` | Conteo entero | Racha maxima de ejercicios consecutivos con `loss_flag = 1`. |

Variables de auditoria:

| Variable | Tipo | Calculo / interpretacion |
| --- | --- | --- |
| `audit_worse_than_approved_flag_2011` | Binaria 0/1 | Vale 1 si la calificacion auditora de 2011 no es `aprobado`. |
| `audit_denied_flag_2011` | Binaria 0/1 | Vale 1 si la auditoria de 2011 es `denegado`. |
| `audit_changed_flag_2008_2011` | Binaria 0/1 | Vale 1 si cambia la calificacion auditora entre 2008 y 2011. |
| `audit_worsened_flag_2008_2011` | Binaria 0/1 | Vale 1 si la severidad auditora empeora entre 2008 y 2011. |
| `audit_severity_2011` | Ordinal numerica | Codifica severidad: `aprobado = 0`, `salvedades = 1`, `desfavor. = 2`, `denegado = 3`; ausente queda como nulo. |
| `audit_opinion_2011_*` | Binaria one-hot 0/1 | Dummies de la calificacion auditora de 2011, una columna por categoria. |

Variables sectoriales y geograficas:

| Variable | Tipo | Calculo / interpretacion |
| --- | --- | --- |
| `cnae_default_rate_oof` | Continua codificada | Target encoding out-of-fold de la tasa de impago por CNAE 2011. Es una proporcion entre 0 y 1. |
| `province_default_rate_oof` | Continua codificada | Target encoding out-of-fold de la tasa de impago por provincia 2011. Es una proporcion entre 0 y 1. |
| `cnae_2digit_default_rate_oof` | Continua codificada | Target encoding out-of-fold de la tasa de impago por los dos primeros digitos del CNAE. Es una proporcion entre 0 y 1. |
| `province_cnae_default_rate_oof` | Continua codificada | Target encoding out-of-fold de la tasa de impago por combinacion provincia + CNAE a dos digitos. Es una proporcion entre 0 y 1. |
| `current_ratio_vs_cnae_median` | Continua relativa | `2011_current_ratio` menos mediana del CNAE. |
| `debt_assets_vs_cnae_median` | Continua relativa | `2011_total_liabilities_to_assets` menos mediana del CNAE. |
| `roa_vs_cnae_median` | Continua relativa | `2011_roa` menos mediana del CNAE. |
| `sales_growth_vs_cnae_median` | Continua relativa | `log_sales_pct_change_2011_2008` menos mediana del CNAE. |
| `cash_assets_vs_cnae_median` | Continua relativa | `2011_cash_to_assets` menos mediana del CNAE. |

Variables de validez de ratios:

| Variable | Tipo | Calculo / interpretacion |
| --- | --- | --- |
| `*_was_invalid_flag` | Binaria 0/1 | Vale 1 si el ratio original era incalculable por denominador cero, ausente o resultado infinito. El ratio queda como nulo y este flag conserva la informacion de invalidez. |

Normalizacion y escalado:

- Las variables `log_*` estan transformadas con `log1p`, pero no estandarizadas.
- Los ratios, margenes, proporciones, importes y conteos se exportan en su escala natural.
- En el bloque anterior de ratios de `fusionado.csv` se crean columnas `*_mediana_4_anios_norm`, que si estan normalizadas como z-score: `(valor - media) / desviacion_tipica`.
- En la modelizacion Logit, el escalado se aprende despues sobre train con `StandardScaler`; ese escalado no se guarda en `fusionado_features_impagos.csv`.

#### Free Cash Flow

El notebook incorpora `free_cash_flow` como proxy contable porque la base no incluye un estado de flujos de efectivo directo. Se calcula desde las partidas disponibles de balance y cuenta de resultados:

```text
free_cash_flow = EBIT + abs(Amortizacion del inmovilizado)
                 - variacion_capital_circulante
                 - capex_proxy
```

Donde:

- `EBIT` = `Resultado de explotacion`.
- `variacion_capital_circulante` = (`Activo corriente` - `Pasivo corriente`) del ejercicio actual menos el mismo capital circulante del ejercicio anterior.
- `capex_proxy` = variacion interanual de (`Inmovilizado intangible` + `Inmovilizado material` + `Inversiones inmobiliarias`) + `abs(Amortizacion del inmovilizado)`.

La feature se calcula para 2009, 2010 y 2011 porque necesita datos del ejercicio anterior; en 2008 queda como nula. A partir de `free_cash_flow` tambien se crean `free_cash_flow_to_sales` y `free_cash_flow_to_assets`, y el flujo general del notebook genera sus agregados temporales: media, mediana, minimo, maximo, desviacion tipica, cambio 2011-2008, cambio porcentual y pendiente.

### 10. Modelizacion Logit con seleccion supervisada

El notebook incluye un modelo Logit general con seleccion supervisada de variables. El objetivo es usar las features financieras creadas y documentadas en este README evitando el PCA no supervisado, porque este reduce dimensionalidad segun varianza explicada y no necesariamente segun capacidad predictiva sobre el impago.

Pipeline propuesto:

```text
140 features (train+test)
        |
Imputacion + escalado
        |
Seleccion supervisada
LASSO / RFE sobre target
        |
Modelo + desbalanceo
class_weight / SMOTE / umbral
        |
Calibracion de umbral
        |
Objetivo
ROC-AUC > 0,75 · Recall > 0,4 · F1 equilibrado
```

Metodologia aplicada:

- Se parte de `df_modelo_impagos`, generado en la seccion de feature engineering.
- Se seleccionan explicitamente las familias de features documentadas en este README:
  - valor reciente de 2011 (`2011_...`);
  - mediana temporal de cuatro anos (`..._median_4y`);
  - conteos de cuatro anos para flags financieros (`..._count_4y`);
  - variables de tendencia;
  - variables de auditoria;
  - one-hot encoding de la calificacion auditora de 2011;
  - target encoding out-of-fold sectorial y geografico;
  - variables relativas a la mediana sectorial CNAE;
  - indicadores `_was_invalid_flag` asociados a las features usadas.
- Se separa la muestra en train y test con `train_test_split`, usando particion estratificada para conservar la proporcion de impagos.
- La imputacion de nulos se aprende solo con train mediante medianas y despues se aplica a test.
- Se eliminan columnas vacias o constantes calculadas sobre train.
- El escalado se aprende solo con train y despues se aplica a test.
- La seleccion supervisada se aprende solo con train para evitar fuga de informacion.
- La primera seleccion usa regularizacion L1 tipo LASSO sobre el target; si no retiene variables, se conserva un bloque reducido por importancia supervisada.
- El modelo Logit se entrena con penalizacion y `class_weight='balanced'` para compensar el desbalanceo de impagos.
- El umbral de clasificacion se calibra con las probabilidades de train, maximizando un F1 con restriccion minima de `recall`.
- Las predicciones se generan sobre test como probabilidad de impago.

El notebook muestra:

- numero de observaciones en train y test;
- tasa de impago en train y test;
- numero de features originales usadas antes de la seleccion;
- numero de features retenidas por la seleccion supervisada;
- umbral calibrado;
- metricas de test: `accuracy`, `balanced_accuracy`, `precision`, `recall`, `f1`, `roc_auc`, `average_precision` y `brier_score`;
- matriz de confusion;
- tabla con las mayores probabilidades predichas de impago en test.

La estructura anterior basada en PCA no supervisado y umbral fijo `0,5` producia esta referencia de partida:

```text
Train: 1329 observaciones
Test: 444 observaciones
Tasa de impago train: 9,41%
Tasa de impago test: 9,46%
Features originales antes de seleccion: 140
Seleccion anterior PCA: 30 componentes
Varianza explicada anterior: 73,79%
```

Metricas en test con umbral de clasificacion fijo `0,5`:

```text
accuracy             0,9099
balanced_accuracy    0,5345
precision            0,7500
recall               0,0714
f1                   0,1304
roc_auc              0,7182
average_precision    0,3084
brier_score          0,0776
```

Matriz de confusion en test:

```text
        pred_0  pred_1
real_0     401       1
real_1      39       3
```

La lectura principal es que el pipeline anterior ordenaba razonablemente el riesgo (`roc_auc` superior a 0,7), pero con umbral `0,5` clasificaba pocos impagos. Por eso la nueva estructura sustituye PCA por seleccion supervisada, incorpora tratamiento del desbalanceo y calibra el umbral para orientar el modelo a deteccion de riesgo.

El notebook mantiene ademas un Logit especifico de liquidez usando:

- `Current Ratio_mediana_4_anios_norm`.
- `Quick Ratio_mediana_4_anios_norm`.
- `Cash Ratio_mediana_4_anios_norm`.

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
scikit-learn
```

En el propio notebook se incluye una celda para instalar las dependencias principales:

```python
%pip install pandas numpy matplotlib seaborn
```

Para ejecutar los modelos Logit tambien es necesario tener instalado `statsmodels` y `scikit-learn`:

```bash
pip install statsmodels scikit-learn
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
