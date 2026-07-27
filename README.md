# Proyecto Júpiter · WhiteHosting

**Estrategia de inversión de 300 M€ en alojamiento turístico en Madrid, Barcelona y Valencia**

Análisis de datos de extremo a extremo: desde la construcción de las bases de datos propias mediante
web scraping hasta un cuadro de mando en Power BI que sostiene una recomendación de inversión concreta
y cuantificada.

**Autor:** Enrique García · Julio de 2026

---

## El caso

WhiteHosting es un fondo (caso ficticio, inspirado en operadores reales de *build-to-rent*) que plantea
invertir 300 millones de euros en vivienda para explotación turística en las tres principales plazas
españolas. El encargo: identificar dónde comprar, cuánto se puede facturar y cómo repartir el capital
entre las tres ciudades teniendo en cuenta el riesgo regulatorio.

Ninguno de los dos mercados existía como fuente de datos disponible al empezar. Salvo el dataset de
referencia facilitado por la escuela, **todos los datos del proyecto se extrajeron mediante sistemas
de scraping desarrollados desde cero**.

## Resultado

| Métrica | Valor |
|---|---|
| Capital desplegado | **299,6 M€** (99,86 % del presupuesto) |
| Inmuebles en cartera | **944** |
| Rentabilidad conjunta (ROI365) | **4,08 %** |
| Beneficio neto anual estimado | **12,23 M€** |
| Escenario recomendado | **E2 – Sostenible** |

La recomendación **no** maximiza la rentabilidad teórica, y es una decisión deliberada. El escenario E1
alcanza un 4,70 %, pero concentra el 45 % del capital en Barcelona, la única de las tres ciudades cuyas
licencias de vivienda de uso turístico se extinguen en noviembre de 2028. E2 reduce esa exposición del
45 % al 17 % a cambio de 0,62 puntos de rentabilidad y 1,84 M€ anuales: el precio de la prudencia,
cuantificado.

### Conclusiones de mercado

- **Precio por noche muy dispar:** 212 € en Barcelona, 128 € en Madrid, 97 € en Valencia.
- **ROI del inmueble típico en venta:** 3,1 % en Barcelona, 2,1 % en Valencia, 1,6 % en Madrid.
- **La gestión profesional es la palanca más accionable:** el distintivo Superhost eleva la ocupación
  en torno a un 50 % — entre 82 y 99 noches adicionales al año.
- **Prima de zona turística:** +32,7 % en Madrid, +12,4 % en Barcelona, +11,3 % en Valencia.
- **El lujo no es base de cartera:** solo el 4,9 % de la oferta de alojamiento completo.
- **Estacionalidad:** Barcelona es el mercado más estacional (36 % de amplitud entre junio y agosto);
  Madrid el más estable (18 %).

---

## Datos construidos

| Fuente | Registros | Método |
|---|---|---|
| Idealista | 31.435 anuncios | Scraping propio (Selenium) |
| Fotocasa | 24.826 anuncios | Scraping propio (Selenium) |
| **Compraventa consolidado** | **30.893 inmuebles** | Limpieza + armonización de barrios |
| Airbnb | 453.060 filas brutas | Scraping longitudinal propio (413 fechas de check-in) |
| Inside Airbnb | 24.153 anuncios | Fuente pública |
| Pontia | 164 extracciones (2017-2021) | Dataset de referencia de la escuela |
| Barrios oficiales | 282 barrios | Elaboración propia (INE + ayuntamientos) |

Validación externa contra estadísticas oficiales de ocupación hotelera (INE) y AirROI. Frente a AirROI,
las estimaciones del modelo son **inferiores** en Madrid y Valencia y superiores solo en Barcelona: el
sesgo del modelo es conservador por diseño.

---

## Estructura del repositorio

```
├── notebooks/
│   ├── 01_webscraping_idealista_fotocasa_airbnb.ipynb   Construcción de las 3 fuentes propias
│   ├── 02_mercado_compraventa_idealista_fotocasa.ipynb  Limpieza, armonización y EDA de compraventa
│   └── 03_alquiler_turistico_rentabilidad_inversion.ipynb  ROI365 y escenarios de inversión
├── data/
│   ├── raw/          CSV en bruto del scraping (no versionados)
│   ├── interim/      CSV combinados intermedios (no versionados)
│   └── processed/    15 tablas finales que alimentan Power BI
├── dashboard/
│   └── Dashboard WhiteHosting.pbix
├── docs/
│   ├── Memoria tecnica - Proyecto Jupiter - WhiteHosting.docx   Memoria completa (14 capítulos)
│   └── Informe ejecutivo - WhiteHosting.docx                    Resumen para comité
└── requirements.txt
```

### Los cuadernos

**01 · Web scraping.** Documenta en orden cronológico la evolución de los tres scrapers, incluidos los
callejones sin salida. Requests + BeautifulSoup → bloqueo HTTP 403 → migración a Selenium →
anti-detección y temporización aleatoria → extracción estructurada. Para Fotocasa, el bloqueo
sistemático en la página 2 obligó a rediseñar la arquitectura: **una sesión de Chrome por página**.
Para Airbnb, un bucle diario de 365 días resultó inviable en tiempo y se optimizó a muestreo semanal.

**02 · Mercado de compraventa.** Depuración de duplicados, errores de introducción y anuncios no
comparables (ocupados, subasta, ruina, reforma integral). Armonización de la nomenclatura de barrio y
distrito de ambos portales bajo un mismo criterio oficial. Resultado: 30.893 inmuebles con 14 variables
homogéneas, verificados fila a fila.

**03 · Alquiler turístico e inversión.** Cruce de Pontia, Airbnb propio e Inside Airbnb; cálculo del
ROI365 con jerarquía de asignación de precio y control de outliers en capas; construcción de los tres
escenarios de inversión mediante selección voraz por ROI descendente con presupuesto por ciudad.

### Decisiones metodológicas

- **Mediana para el precio, media para la ocupación.** El precio por noche tiene cola derecha larga;
  la ocupación no.
- **Describir el mercado ≠ decidir la inversión.** Los indicadores descriptivos usan el universo
  completo; la cartera aplica filtros de elegibilidad definidos *ex ante*.
- **Los cálculos viven en Python; Power BI muestra resultados certificados.** Ninguna métrica crítica
  se recalcula en DAX a partir de datos sin verificar.
- **El riesgo regulatorio se traduce a un parámetro, no a un filtro manual.** Barcelona computa 2 años
  de explotación (hasta noviembre de 2028) frente a 10 del resto; su peso en el reparto cae solo.

---

## Reproducir

```bash
git clone https://github.com/enriquegvda/proyecto-jupiter-whitehosting.git
cd proyecto-jupiter-whitehosting
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

Los cuadernos resuelven sus rutas de forma relativa al repositorio (celda *Configuración de rutas*), por
lo que no hace falta editar rutas absolutas. Las tablas de `data/processed/` están incluidas, así que los
cuadernos 02 y 03 son ejecutables sin volver a scrapear.

### Abrir el dashboard

`dashboard/Dashboard WhiteHosting.pbix` carga sus 15 tablas desde `data/processed/` a través de un único
parámetro de Power Query llamado **`RutaDatos`**. Para actualizarlo en otra máquina:

1. Abrir el `.pbix` en Power BI Desktop.
2. *Inicio → Transformar datos → Administrar parámetros*.
3. Poner en `RutaDatos` la ruta absoluta de la carpeta `data/processed` de tu copia del repositorio.
4. *Inicio → Actualizar*.

No hay que tocar ninguna consulta: las quince construyen su origen como `RutaDatos & "\fichero.csv"`.

**Sobre los scrapers:** requieren Chrome y ejecución local (Selenium no funciona bien en Colab). Se
incluyen como documentación del proceso; los portales cambian su HTML con frecuencia, por lo que los
selectores pueden haber quedado obsoletos.

### Stack

Python · pandas · numpy · Selenium · BeautifulSoup · matplotlib · seaborn · plotly · statsmodels · Power BI

---

## Nota sobre el uso de datos

Proyecto académico con fines formativos. Los datos se extrajeron para análisis agregado, con
temporización conservadora para no sobrecargar los servidores de origen, y no se redistribuyen en bruto
en este repositorio. WhiteHosting es una empresa ficticia; las cifras de inversión son un ejercicio de
modelización, no asesoramiento financiero.

## Licencia

Código bajo licencia MIT (ver [LICENSE](LICENSE)). La memoria, el informe ejecutivo y el cuadro de mando
se publican como muestra de trabajo, no para reutilización comercial.
