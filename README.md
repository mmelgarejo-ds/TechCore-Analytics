# TechCore Analytics — Dashboard de Inteligencia de Negocio

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Excel](https://img.shields.io/badge/Excel-217346?style=flat&logo=microsoft-excel&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-F2C811?style=flat&logo=powerbi&logoColor=black)

Dashboard de BI end-to-end sobre ventas de una empresa tecnológica ficticia (TechCore): desde la limpieza del dato crudo hasta un dashboard interactivo de 6 páginas con seguridad a nivel de fila. Módulo 3 — Soy Henry Data Science AI.

---

## Problema de negocio

TechCore necesita transformar sus datos de ventas crudos en inteligencia accionable: identificar sucursales con mejor desempeño, marcas más vendidas, tendencias temporales y comportamiento de clientes para apoyar decisiones comerciales.

---

## Dataset

- 30.307 filas crudas · 30 columnas
- Período: septiembre 2014 — septiembre 2025
- 6 sucursales en 4 ciudades de Colombia
- Después de limpieza: **30.000 filas · 32 columnas**

---

## Pipeline — 3 etapas, cada una sobre la anterior

| Etapa | Herramienta | Contenido |
|---|---|---|
| 1 — Limpieza y Transformación | Power Query · Power BI Desktop | Estandarización de encabezados, tipado regional correcto, deduplicación, corrección de typos de marca, tratamiento de nulos, columnas derivadas |
| 2 — Modelo Relacional | Python · pandas · Excel | Tabla plana descompuesta en 8 tablas relacionadas con claves primarias/foráneas, validación de integridad referencial |
| 3 — Dashboard + RLS | Power BI Desktop · DAX | Dashboard interactivo de 6 páginas, 6 medidas DAX, Row Level Security con 3 roles |

---

## Etapa 1 — Limpieza y Transformación (Power Query)

Trabajé sobre `ventas.csv` (30.307 filas, 30 columnas) en Power Query dentro de Power BI Desktop.

**Hallazgos en el dato crudo:**
- **307 facturas duplicadas** — mismo `VentaID` repetido con datos idénticos
- **431 nulos en `CiudadSucursal`** (1.4%) — reconstruidos extrayendo la ciudad desde `SucursalNombre` con columna condicional, en vez de rellenar con "No especificado"
- **`CantidadProducto2`/`3` con valores 10x inflados** (10 y 20 en vez de 1 y 2) — causado por el separador decimal: el CSV usa punto como decimal (formato anglosajón) y sin la configuración regional `en-US`, Power BI interpretaba "1.0" como mil. Lo detecté al notar que `Subtotal` no cuadraba con `Precio × Cantidad`
- **11 variantes mal escritas de marca** (Acr→Acer, Aple→Apple, Delll→Dell, Lenvo→Lenovo, Razor→Razer, Samsng→Samsung, entre otras) corregidas simultáneamente en las 3 columnas de producto
- **6.014 nulos en `DescuentoVenta`** (20%) — reemplazados por 0 porque representan ventas *sin* descuento, no datos faltantes (distinto tratamiento que los nulos de texto, que se marcaron como "No especificado")

**Salida:** `ventasTransformed.csv` — 30.000 filas · 32 columnas (30 originales + `Año` + `Mes`) · 0 nulos en columnas obligatorias.

---

## Etapa 2 — Modelo Relacional (Python)

Descompuse la tabla plana limpia en **8 tablas relacionadas**: `Ciudades` (4) · `Sucursales` (6) · `Vendedores` (30) · `Clientes` (17.453) · `MetodosPago` (6) · `Productos` (40) · `Facturas` (30.000, tabla central) · `DetalleFacturas` (60.059, una fila por línea de producto vendido).

**Decisiones de diseño:**
- **Identificación de clientes:** el dataset no trae `ClienteID`, así que identifiqué clientes únicos por `ClienteNombre + EmailCliente`. Encontré 4 nombres duplicados (ej. "Lucio del Hurtado" x2); verificando edad, email, teléfono y dirección confirmé que eran personas distintas con nombre coincidente, no el mismo cliente
- **Descuento en `Facturas`, no en `DetalleFacturas`:** la consigna sugería lo segundo, pero el dato real (`DescuentoVenta`) viene a nivel de factura completa, no por producto — coloqué el campo donde el dato realmente vive
- **Ciudad por sucursal:** usé la ciudad más frecuente registrada por sucursal (ignorando "No especificado") para evitar que 6 sucursales se partieran en 12 filas falsas
- **Validación de integridad referencial:** 6 controles de FK huérfanas, los 6 en 0

**Salida:** `modeloVentas.xlsx` — 8 hojas, generado por `2_modelo_relacional.ipynb`.

---

## Etapa 3 — Dashboard + RLS (Power BI)

Dashboard interactivo de **6 páginas** (Home · KPIs Globales · Ventas en el Tiempo · Clientes · Productos · Vendedores) con 6 medidas DAX (`Ventas Totales`, `Ticket Promedio`, `Margen Estimado`, `% Participación Ciudad`, entre otras).

**Decisiones de diseño:**
- **Margen Estimado:** el dataset no tiene costo unitario. Descarté el descuento como proxy (sin correlación con marca/ciudad/precio) y estimé el margen por tramos de precio (hasta $4M→30%, hasta $6M→25%, hasta $8M→20%, más de $8M→15%), documentado explícitamente como supuesto, no como dato real
- **Tabla Calendario:** creada con una fila por día (2014–2025) para que los cálculos de tiempo en DAX no fallen en días sin ventas
- **Medida específica para marca:** `Ventas Totales` no filtraba bien a través de `Productos → DetalleFacturas → Facturas`; resolví con `SUMX(DetalleFacturas, Subtotal)` para ese visual puntual
- **Extra credit — Row Level Security:** 3 roles configurados (Gerente Nacional sin filtro, Gerente Regional filtrado por ciudad, Gerente Sucursal filtrado por sucursal), con una tabla de 11 usuarios simulados mapeando roles a ciudades/sucursales

---

## Hallazgos de negocio

- **Concentración geográfica:** Medellín concentra el **43%** de las ventas, duplicando a la sucursal de menor rendimiento (Cali)
- **Perfil de clientes:** 65% hombres (11.366) vs 35% mujeres (6.087); edad distribuida de forma pareja entre 18 y 51 años
- **Producto estrella:** HP Spectre x360 lidera con 7.726 unidades, 30% por encima del segundo lugar
- **Marca líder:** Lenovo domina en volumen (21.697 u., 24%) y en margen estimado
- **Sin estacionalidad:** ventas estables ($3-4M mensuales) durante los 11 años, sin meses pico consistentes
- **Vendedores con perfiles complementarios:** Ana Sofía lidera en ingresos (tickets grandes, menos clientes); Liliana Jordá lidera en facturas atendidas (más clientes, tickets chicos)
- **Fidelización:** 17.453 clientes únicos sobre 30.000 ventas → ~1,7 compras promedio por cliente
- **Descuentos sin estrategia:** la correlación entre descuento y marca/ciudad/precio es prácticamente cero — la herramienta comercial no se está usando de forma dirigida

## Recomendaciones

1. Desarrollar estrategia de crecimiento comercial en Cali y Pereira (13% cada una vs 43% de Medellín, sin diferencia demográfica que lo explique)
2. Potenciar el segmento femenino (35% de clientes) con campañas dirigidas
3. Priorizar stock y comunicación del HP Spectre x360 como producto ancla
4. Definir criterios claros de aplicación de descuentos (hoy no correlacionan con ninguna variable de negocio)
5. Evaluar vendedores por ingresos *y* por volumen de clientes atendidos, no solo por un criterio
6. Incorporar costo unitario real en una futura iteración para reemplazar la estimación de margen por tramos de precio

---

## Preguntas de negocio que responde el dashboard

- **¿Dónde está concentrado el negocio y dónde hay margen para crecer?** La página "Ventas en el Tiempo", filtrada por ciudad o sucursal, muestra que Medellín concentra el 43% de las ventas frente al 13% de Cali o Pereira — sin que la demografía de clientes (edad, género) explique la diferencia.
- **¿Qué productos y marcas priorizar en stock y comunicación?** La página "Productos" aísla al HP Spectre x360 (7.726 unidades, 30% sobre el segundo lugar) y a Lenovo como marca líder en volumen y margen estimado.
- **¿Cómo evaluar el desempeño real de cada vendedor?** La página "Vendedores" cruza ingresos totales con cantidad de facturas atendidas por persona — necesario porque el equipo tiene perfiles complementarios (ticket alto/pocos clientes vs. ticket bajo/muchos clientes) que un solo ranking por ingresos esconde.

## Ejemplo de decisión que apoya

Un Gerente Regional de Cali, con su rol de RLS activo (solo ve su propia ciudad), entra a "KPIs Globales" y ve que su sucursal genera 13% de las ventas totales. Compara contra Medellín en "Ventas en el Tiempo" sin filtro y confirma que la diferencia no viene de una base de clientes distinta — la distribución por edad y género es similar entre ciudades. Esa comparación, posible directamente en el dashboard sin pedirle un reporte ad hoc a nadie, es lo que sostiene la recomendación #1 de más abajo: el problema de Cali no es de demanda, es de presencia o capacidad comercial local, así que la inversión debería ir ahí y no en más estudios de mercado.

---

**Informe ejecutivo** (hallazgos y recomendaciones de negocio, pensado para lectores no técnicos): [`INFORME_EJECUTIVO.md`](INFORME_EJECUTIVO.md).

---

## Estructura del repositorio

| Carpeta/Archivo | Contenido |
|---|---|
| `Avance_2/2_modelo_relacional.ipynb` | Notebook que construye el modelo relacional desde el CSV limpio |
| `Avance_2/modeloVentas.xlsx` | Modelo relacional final — 8 tablas |
| `Avance_3/Avance_3_Dashboard_PowerBI.pbix` | Dashboard completo (abrir con Power BI Desktop) |

---

## Stack

Power BI Desktop · Power Query · DAX · Python · Pandas · Excel · Matplotlib · Seaborn

---

## Cómo reproducirlo

```bash
git clone https://github.com/mmelgarejo-ds/TechCore-Analytics.git
cd TechCore-Analytics
pip install -r requirements.txt
```

`Avance_2/2_modelo_relacional.ipynb` reconstruye el modelo relacional desde el CSV limpio (no incluido en el repo por tamaño; se genera siguiendo los pasos de limpieza documentados en el informe ejecutivo). `Avance_3/Avance_3_Dashboard_PowerBI.pbix` se abre directamente con Power BI Desktop.

---

## Autor

Mauricio Melgarejo · [LinkedIn](https://www.linkedin.com/in/mauricio-daniel-melgarejo-amaro-482596105/) · [GitHub](https://github.com/mmelgarejo-ds)
