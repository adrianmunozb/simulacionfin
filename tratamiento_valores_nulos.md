# Tratamiento de Valores Nulos (`n.d.`) — EEFF Base de Datos

## Carga del archivo

```python
import pandas as pd

df = pd.read_csv('EEFF_base_de_datos.csv', sep=';', na_values=['n.d.'], low_memory=False)
```

> Al especificar `na_values=['n.d.']`, pandas convierte automáticamente todas las celdas con ese valor en `NaN`, facilitando el tratamiento posterior.

---

## Variables sin nulos — Sin acción necesaria ✅

Las siguientes variables identificadoras y categóricas no presentan valores nulos:

| Variable | Descripción |
|---|---|
| `Id` | Identificador de empresa |
| `Año` | Ejercicio contable |
| `Número empleados` | Plantilla |
| `Calificación auditor` | Opinión de auditoría |
| `Código primario CNAE 2009` | Sector de actividad |
| `Provincia` | Localización |

---

## Grupo 1 — Variables financieras clave (< 5% nulos) → Imputar

Son partidas esenciales del balance y la cuenta de resultados. El bajo porcentaje de missings indica errores puntuales de reporte, no ausencia estructural.

| Variable | % Nulos | Estrategia |
|---|---|---|
| Activo corriente / no corriente | 0.1–0.5% | Mediana por empresa (`Id`) y año |
| Total activo / Total PN y Pasivo | 0.1% | Reconstruir como suma de componentes* |
| Resultado del ejercicio | 0.4–0.5% | Mediana por año y sector CNAE |
| Gastos de personal | 1.4% | Mediana por año y tramo de empleados |
| Amortización del inmovilizado | 1.8% | Mediana por año y sector CNAE |
| Importe neto de la cifra de negocios | 1.4% | Mediana por año y sector CNAE |

\* **Reconstrucción del total activo:**

```python
df['Total activo (A + B)\nmil EUR'] = df['Total activo (A + B)\nmil EUR'].fillna(
    df['Activo no corriente\nmil EUR'] + df['Activo corriente\nmil EUR']
)
```

**Imputación por mediana para las demás:**

```python
cols_clave = [
    'Activo corriente\nmil EUR',
    'Resultado del ejercicio\nmil EUR',
    'Gastos de personal\nmil EUR',
    'Amortización del inmovilizado\nmil EUR',
    'Importe neto de la cifra de negocios\nmil EUR',
]

for col in cols_clave:
    mediana = df.groupby(['Id', 'Año'])[col].transform('median')
    df[col] = df[col].fillna(mediana)
```

---

## Grupo 2 — Variables financieras con missings moderados (5%–55%) → Imputar a 0 o crear indicador

Muchas empresas no tienen estas partidas. El nulo puede significar **cero** (no tiene esa actividad) o **no aplica**.

| Variable | % Nulos | Estrategia |
|---|---|---|
| Existencias | 11% | → 0 si empresa de servicios (según CNAE); mediana si industrial |
| Inversiones financieras LP/CP | 11–33% | → 0 (ausencia = sin inversiones) |
| Activos / Pasivos por impuesto diferido | 41–51% | → 0 |
| Inversiones en grupo LP/CP | 47–61% | → 0 o crear variable binaria `tiene_inversiones_grupo` |
| Diferencias de cambio | 53% | → 0 (sin operativa internacional) |
| Variación de existencias | 53% | → 0 si empresa sin stock |
| Impuesto sobre beneficios | 9.6% | → 0 (puede tener pérdidas o exenciones) |

```python
# Partidas que representan ausencia de actividad = 0
cols_a_cero = [
    'Existencias\nmil EUR',
    'Inversiones financieras a largo plazo\nmil EUR',
    'Inversiones financieras a corto plazo\nmil EUR',
    'Activos por impuesto diferido\nmil EUR',
    'Pasivos por impuesto diferido\nmil EUR',
    'Inversiones en empresas del grupo y asociadas a largo plazo\nmil EUR',
    'Inversiones en empresas del grupo y asociadas a corto plazo\nmil EUR',
    'Diferencias de cambio\nmil EUR',
    'Impuestos sobre beneficios\nmil EUR',
]

df[cols_a_cero] = df[cols_a_cero].fillna(0)

# Opcional: crear variable indicadora antes de imputar
df['tiene_inversiones_grupo'] = df[
    'Inversiones en empresas del grupo y asociadas a largo plazo\nmil EUR'
].notna().astype(int)
```

---

## Grupo 3 — Variables con > 70% nulos → Imputar a 0 o eliminar

Son partidas estructuralmente poco comunes. El nulo no es un dato perdido, es **información en sí misma** (la empresa no tiene esa partida).

| Variable | % Nulos | Estrategia |
|---|---|---|
| Clientes LP/CP, Proveedores LP/CP | 96–99% | → 0 (o eliminar si no se usa en el modelo) |
| (Capital no exigido), (Dividendo a cuenta) | 95–99% | → 0 |
| Ajustes por cambios de valor | 87% | → 0 |
| Subvenciones, donaciones y legados | 72% | → 0 o crear `recibe_subvenciones` (0/1) |
| Trabajos realizados para su propio activo | 87% | → 0 |
| Provisiones LP/CP | 75–80% | → 0 |
| Excesos de provisiones | 92.5% | → 0 |
| Acreedores arrendamiento financiero | 67–70% | → 0 |
| Periodificaciones LP/CP | 41–84% | → 0 |

```python
# Todas las columnas financieras restantes → 0
cols_financieras = [c for c in df.columns if 'mil EUR' in c]
df[cols_financieras] = df[cols_financieras].fillna(0)

# Indicador binario para subvenciones (antes de imputar a 0)
df['recibe_subvenciones'] = df[
    'Subvenciones, donaciones y legados recibidos\nmil EUR'
].notna().astype(int)
```

---

## Script completo de tratamiento

```python
import pandas as pd

# 1. Carga
df = pd.read_csv('EEFF_base_de_datos.csv', sep=';', na_values=['n.d.'], low_memory=False)

# 2. Reconstruir totales desde componentes
df['Total activo (A + B)\nmil EUR'] = df['Total activo (A + B)\nmil EUR'].fillna(
    df['Activo no corriente\nmil EUR'] + df['Activo corriente\nmil EUR']
)
df['Total patrimonio neto y pasivo (A + B + C)\nmil EUR'] = (
    df['Total patrimonio neto y pasivo (A + B + C)\nmil EUR'].fillna(
        df['Patrimonio neto\nmil EUR'] + df['Pasivo no corriente\nmil EUR'] + df['Pasivo corriente\nmil EUR']
    )
)

# 3. Imputar clave con mediana por empresa y año
cols_clave = [
    'Activo corriente\nmil EUR',
    'Activo no corriente\nmil EUR',
    'Resultado del ejercicio\nmil EUR',
    'Gastos de personal\nmil EUR',
    'Amortización del inmovilizado\nmil EUR',
    'Importe neto de la cifra de negocios\nmil EUR',
    'Resultado de explotación (1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12)\nmil EUR',
    'Resultado antes de impuestos (A + B)\nmil EUR',
]
for col in cols_clave:
    mediana = df.groupby(['Id', 'Año'])[col].transform('median')
    df[col] = df[col].fillna(mediana)

# 4. Crear indicadores binarios antes de imputar
df['recibe_subvenciones'] = df['Subvenciones, donaciones y legados recibidos\nmil EUR'].notna().astype(int)
df['tiene_inversiones_grupo'] = df['Inversiones en empresas del grupo y asociadas a largo plazo\nmil EUR'].notna().astype(int)
df['tiene_arrendamiento_financiero'] = df['Acreedores por arrendamiento financiero\nmil EUR'].notna().astype(int)

# 5. Resto de columnas financieras → 0
cols_financieras = [c for c in df.columns if 'mil EUR' in c]
df[cols_financieras] = df[cols_financieras].fillna(0)

# 6. Verificación final
print("Nulos restantes:", df.isnull().sum().sum())
print("Shape final:", df.shape)
```

---

## Resumen de decisiones

| Situación | Criterio | Acción |
|---|---|---|
| Total calculable desde partes | Coherencia contable | Reconstruir aritméticamente |
| < 5% nulos en partida esencial | Error de reporte puntual | Mediana por empresa/año |
| 5–55% nulos en partida opcional | Empresa sin esa actividad | Imputar a 0 |
| > 70% nulos | Partida estructuralmente ausente | Imputar a 0 (+ indicador binario si útil) |
