# Análisis y Visualización — Entrega 1

Análisis exploratorio de una base de **694 apartamentos** en Medellín y su área
metropolitana, con datos de precio, área, estrato, zona y características de cada
inmueble. Todo el trabajo está en [`entrega_1.ipynb`](entrega_1.ipynb).

---

## Requisitos

- **Python 3.14** (se instala automáticamente, ver abajo)
- **[uv](https://docs.astral.sh/uv/)** como gestor de dependencias y entornos

## 1. Instalar uv

`uv` es un gestor de paquetes y entornos virtuales para Python. Reemplaza a `pip`,
`venv` y `pyenv` en un solo comando y resuelve las dependencias mucho más rápido.

**macOS / Linux**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**macOS con Homebrew**

```bash
brew install uv
```

**Windows (PowerShell)**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Verifica la instalación:

```bash
uv --version
```

## 2. Crear el entorno virtual y sincronizar las librerías

Desde la raíz del proyecto, un solo comando hace todo:

```bash
uv sync
```

Ese comando:

1. Lee la versión de Python en [`.python-version`](.python-version) (3.14) y **la
   descarga si no la tienes** — no necesitas instalarla por tu cuenta.
2. Crea el entorno virtual en `.venv/`.
3. Instala las dependencias declaradas en [`pyproject.toml`](pyproject.toml), con
   las **versiones exactas** fijadas en `uv.lock`.

El archivo `uv.lock` está versionado en el repositorio a propósito: garantiza que
todos obtengan exactamente las mismas versiones de cada paquete y que los
resultados del notebook sean reproducibles.

## 3. Ejecutar el notebook

**Opción A — JupyterLab desde la terminal**

```bash
uv run jupyter lab
```

`uv run` ejecuta el comando dentro del entorno del proyecto, así que **no hace
falta activar el `.venv` manualmente**.

**Opción B — VS Code**

Abre `entrega_1.ipynb`, haz clic en el selector de kernel (arriba a la derecha) y
elige el intérprete de `.venv/bin/python`.

### Comandos útiles

| Comando | Qué hace |
|---|---|
| `uv sync` | Crea/actualiza el entorno según `uv.lock` |
| `uv run <cmd>` | Ejecuta un comando dentro del entorno |
| `uv add <paquete>` | Agrega una dependencia y actualiza `pyproject.toml` y `uv.lock` |
| `uv remove <paquete>` | Elimina una dependencia |
| `uv tree` | Muestra el árbol de dependencias |

### Solución de problemas

Si ves el aviso `VIRTUAL_ENV=... does not match the project environment path .venv`,
significa que tienes otro entorno activo (por ejemplo de `pyenv` o `conda`).
Es solo una advertencia — `uv` ignora ese entorno y usa el del proyecto. Para
silenciarla, ejecuta `deactivate` antes de trabajar.

---

## Estructura del proyecto

```
.
├── data/
│   └── Apartamentos.csv      # Dataset original (694 registros, 12 columnas)
├── slides/                   # Material de clase (PDF)
├── entrega_1.ipynb           # Notebook con el análisis
├── pyproject.toml            # Dependencias del proyecto
├── uv.lock                   # Versiones exactas (no editar a mano)
└── .python-version           # Versión de Python del proyecto
```

### Dependencias principales

| Paquete | Uso |
|---|---|
| `pandas` | Carga, limpieza y agregación de los datos |
| `seaborn` | Gráficas estadísticas |
| `jupyterlab` | Entorno de ejecución del notebook |

---

## Resumen del notebook

`entrega_1.ipynb` está organizado en cinco secciones, precedidas por una celda de
configuración que fija una paleta y un estilo consistentes para todas las gráficas.

### 1. Tareas de limpieza

Preparación del dataset antes de analizarlo:

- **Fechas.** Se detectó `29/02/23`, una fecha inexistente (2023 no fue bisiesto);
  se corrigió a `28/02/23` y la columna se convirtió de texto a `datetime`.
- **Nombres de columnas.** Se renombraron las columnas largas a identificadores
  cortos: `precio`, `area`, `admon`, `avaluo`.
- **Categorías de texto.** La columna `zona` tenía espacios sobrantes, mayúsculas
  inconsistentes y errores de digitación (`belen  guayabal`, `aburra s`, `surur`).
  Se estandarizó a minúsculas sin espacios extra, consolidando las **7 zonas**
  reales.
- **Datos faltantes.** Se encontraron 2 valores nulos en `precio` y 1 en `admon`.
  Se imputaron con la **mediana del grupo** definido por estrato, zona y número de
  habitaciones — se usó la mediana en lugar de la media porque los precios de
  vivienda tienen una distribución sesgada a la derecha.

### 2. Oferta de apartamentos por zona

Conteo de inmuebles por zona en un gráfico de barras horizontales ordenado por
magnitud, con el valor absoluto y el porcentaje sobre el total etiquetados en cada
barra. La oferta está fuertemente concentrada: **El Poblado reúne el 39%** de los
registros y, junto con Aburrá Sur, supera el 60% del total.

### 3. Precio promedio del metro cuadrado por zona

Se construye la variable derivada precio por m² y se compara entre zonas con dos
vistas complementarias: un gráfico de barras con el promedio por zona, y un
**boxplot** que muestra la distribución completa. El boxplot es el que sostiene la
conclusión, porque la dispersión *dentro* de cada zona es mayor que la diferencia
entre varias zonas vecinas.

### 4. Relación entre área y precio

- **Diagrama de dispersión con recta de regresión** (`regplot`) entre área y precio.
  La relación es fuerte y positiva: cada m² adicional suma cerca de 2.84 millones
  de pesos, y el área explica aproximadamente el 74% de la varianza del precio
  (R² = 0.735).
- **Mapa de calor de correlaciones** entre todas las variables numéricas. Además
  del área, el avalúo (0.79) y la administración (0.75) son los predictores más
  correlacionados con el precio.

### 5. Porcentaje de avalúo por estrato

Se calcula el avalúo como porcentaje del precio de venta de cada inmueble y se
compara su promedio entre estratos, para observar cómo varía la brecha entre el
valor catastral y el valor comercial a lo largo de la escala socioeconómica.

---

## Datos

`data/Apartamentos.csv` — 694 registros, 12 columnas.

| Columna | Descripción |
|---|---|
| `fecha` | Fecha del registro |
| `precio(millones de pesos)` | Precio de venta |
| `area(mt2)` | Área construida en metros cuadrados |
| `zona` | Zona de la ciudad (7 categorías) |
| `estrato` | Estrato socioeconómico (2 a 6) |
| `habitaciones` | Número de habitaciones |
| `baños` | Número de baños |
| `balcon` | Tiene balcón (sí / no) |
| `parqueadero` | Tiene parqueadero (sí / no) |
| `administracion (millones de pesos)` | Cuota de administración |
| `avaluo (millones de pesos)` | Avalúo catastral |
| `remodelado` | Está remodelado (sí / no) |
