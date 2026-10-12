# PMD1 2026-02 — Fundamentos de Gestión de Datos

Este repositorio contiene diez casos **sintéticos** para el PMD1. Cada grupo trabaja únicamente con el número de caso que le asigne la docente. Cada Excel conserva las hojas `Operaciones`, `Clientes`, `Catalogo` y `Sedes` separadas; la hoja `LEEME` explica el caso. Los datos contienen incidencias didácticas de calidad que el equipo debe identificar y tratar.

| Número | Caso | Archivo |
|---:|---|---|
| 01 | Delivery | [01.xlsx](casos/01.xlsx) |
| 02 | Restaurante | [02.xlsx](casos/02.xlsx) |
| 03 | Farmacia | [03.xlsx](casos/03.xlsx) |
| 04 | Hotel | [04.xlsx](casos/04.xlsx) |
| 05 | Agro | [05.xlsx](casos/05.xlsx) |
| 06 | Energía | [06.xlsx](casos/06.xlsx) |
| 07 | Taller | [07.xlsx](casos/07.xlsx) |
| 08 | Ferretería | [08.xlsx](casos/08.xlsx) |
| 09 | Clínica | [09.xlsx](casos/09.xlsx) |
| 10 | Logística | [10.xlsx](casos/10.xlsx) |

## Cargar el caso asignado en Google Colab

**Cambien únicamente el número de `NUMERO_CASO`**. La celda descarga el Excel directamente desde GitHub, lee las hojas por separado y las carga en una base SQLite temporal. No requiere subir archivos manualmente a Colab.

```python
import io
import sqlite3
import requests
import pandas as pd

NUMERO_CASO = 1  # Reemplazar solo este número por el caso asignado (1 a 10)
assert 1 <= NUMERO_CASO <= 10, "El número de caso debe estar entre 1 y 10"

url = (
    "https://raw.githubusercontent.com/Rociosayan/FDG_PMD1_2026_02/"
    f"main/casos/{NUMERO_CASO:02d}.xlsx"
)
respuesta = requests.get(url, timeout=30)
respuesta.raise_for_status()
hojas = pd.read_excel(io.BytesIO(respuesta.content), sheet_name=None)

conn = sqlite3.connect(":memory:")
tablas = ["Operaciones", "Clientes", "Catalogo", "Sedes"]
for nombre in tablas:
    if nombre not in hojas:
        raise ValueError(f"Falta la hoja {nombre} en el caso {NUMERO_CASO}")
    hojas[nombre].to_sql(nombre, conn, if_exists="replace", index=False)
    print(f"{nombre}: {len(hojas[nombre])} filas")

print("Caso cargado desde:", url)
print("Hojas disponibles:", list(hojas))
```

Ejemplo de consulta para comprobar las relaciones. El equipo debe analizar las claves y los duplicados antes de construir su tabla final:

```python
consulta = """
SELECT o.id_operacion, o.id_cliente, c.id_cliente AS cliente_encontrado,
       o.id_catalogo, p.nombre_item, o.id_sede
FROM Operaciones AS o
LEFT JOIN Clientes AS c ON o.id_cliente = c.id_cliente
LEFT JOIN Catalogo AS p ON o.id_catalogo = p.id_catalogo
LIMIT 5
"""
pd.read_sql_query(consulta, conn)
```

Los precios y demás cifras de estos casos son sintéticos. No representan precios de mercado ni datos personales reales.
