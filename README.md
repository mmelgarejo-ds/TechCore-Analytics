# TechCore Analytics — Dashboard de Inteligencia de Negocio

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Excel](https://img.shields.io/badge/Excel-217346?style=flat&logo=microsoft-excel&logoColor=white)

Dashboard de BI sobre ventas de una empresa tecnológica ficticia (TechCore).
Módulo 3 — Soy Henry Data Science AI.

---

## Problema de negocio

TechCore necesita transformar sus datos de ventas crudos en inteligencia
accionable: identificar sucursales con mejor desempeño, marcas más vendidas,
tendencias temporales y comportamiento de clientes para apoyar decisiones comerciales.

---

## Dataset

- 30.307 filas crudas · 30 columnas
- Período: septiembre 2014 — septiembre 2025
- Después de limpieza: **30.000 filas · 32 columnas**

---

## Pipeline

| Avance | Herramienta | Contenido |
|---|---|---|
| 1 — Limpieza y Transformación | Power Query · Power BI | Estandarización, deduplicación, corrección de tipos, tratamiento de nulos, columnas derivadas |
| 2 — Modelo Relacional | Python · Excel | Modelado dimensional, separación en tablas de hechos y dimensiones |
| 3 — Dashboard | Power BI · DAX | Dashboard interactivo con métricas de ventas, sucursales y productos |

---

## Hallazgos del dataset crudo

- 307 facturas duplicadas detectadas y eliminadas
- 11 variantes mal escritas en columnas de marca (Aple → Apple, Lenvo → Lenovo, etc.)
- 6.014 nulos en DescuentoVenta (20% de ventas sin descuento)
- 431 nulos en CiudadSucursal — reconstruidos desde SucursalNombre

---

## Stack

Power BI · Power Query · DAX · Python · Pandas · Excel · Matplotlib · Seaborn

---

## Autor

Mauricio Melgarejo · [LinkedIn](https://www.linkedin.com/in/mauricio-daniel-melgarejo-amaro-482596105/) · [GitHub](https://github.com/mmelgarejo-ds)
