<div align="center">

# 🎣 Detección de URLs de *Phishing* con Machine Learning

### Clasificación binaria supervisada sobre el conjunto de datos PhiUSIIL

*Proyecto del curso **Introducción al Machine Learning (Modelos II)** — Universidad de Antioquia*

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat&logo=jupyter&logoColor=white)
![Estado](https://img.shields.io/badge/estado-avance%20parcial-yellow)
![Licencia](https://img.shields.io/badge/licencia-MIT-green)

</div>

---

## 📑 Tabla de contenidos
- [Descripción](#-descripción)
- [El problema](#-el-problema)
- [Conjunto de datos](#-conjunto-de-datos)
- [Análisis exploratorio (EDA)](#-análisis-exploratorio-eda)
- [Paradigma de aprendizaje](#-paradigma-de-aprendizaje)
- [Estado del arte](#-estado-del-arte)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Cómo reproducir](#-cómo-reproducir)
- [Hoja de ruta](#-hoja-de-ruta)
- [Autores](#-autores)
- [Referencias](#-referencias)

---

## 🎯 Descripción

El **phishing** es un ataque en el que un sitio web fraudulento suplanta a una entidad legítima
para robar credenciales o información financiera. Las listas negras tradicionales son **reactivas**:
solo bloquean URLs ya reportadas y fallan ante los miles de sitios nuevos que aparecen cada día.

Este proyecto construye y valida un modelo de **aprendizaje supervisado** que decide,
de forma automática, si una URL es **legítima** o de **phishing**, a partir de características
de la propia URL y del código de la página.

> **Avance parcial:** este repositorio contiene la *descripción del problema* (análisis exploratorio)
> y el *estado del arte*. El modelado y la evaluación corresponden a la entrega final.

---

## 🧩 El problema

| | |
|---|---|
| **Tarea** | Clasificación binaria (legítima vs. phishing) |
| **Entrada** | 54 características de la URL y de la página web |
| **Salida** | `1` = URL legítima · `0` = URL de phishing |
| **Reto** | Alta tasa de detección con **baja tasa de falsos positivos** (bloquear un sitio legítimo molesta al usuario) |
| **Utilidad** | Integrable en navegadores, clientes de correo o *gateways* de red, operando en tiempo casi real |

---

## 📊 Conjunto de datos

**PhiUSIIL Phishing URL Dataset** — Prasad & Chandra (2024), publicado en *Computers & Security* (Elsevier)
y disponible en el [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+dataset).

| Propiedad | Valor |
|---|---|
| Muestras | **235.795** URLs |
| Características | **54** (+ etiqueta) |
| URLs legítimas | 134.850 (**57,2 %**) |
| URLs de phishing | 100.945 (**42,8 %**) |
| Valores faltantes | **Ninguno** |
| Filas duplicadas | Ninguna |

Las características se agrupan en tres familias:

| Grupo | Ejemplos | Qué miden |
|---|---|---|
| **Léxicas / estructurales de la URL** | `URLLength`, `DomainLength`, `NoOfSubDomain`, `SpacialCharRatioInURL`, `IsDomainIP` | Forma y composición de la cadena de la URL |
| **Derivadas** | `URLSimilarityIndex`, `CharContinuationRate`, `TLDLegitimateProb`, `URLCharProb` | Variables calculadas por los autores (índices y probabilidades) |
| **Del código HTML** | `LineOfCode`, `HasTitle`, `HasFavicon`, `IsResponsive`, `HasPasswordField`, `NoOfJS`, `HasSocialNet` | Atributos extraídos de la página descargada |

Además hay 4 variables de texto (`URL`, `Domain`, `TLD`, `Title`). De las 54 características, **50 son
numéricas** (19 de ellas binarias) y **4 son de texto/categóricas**.

---

## 🔎 Análisis exploratorio (EDA)

Todo el análisis es reproducible en [`notebooks/01_EDA_phishing.ipynb`](notebooks/01_EDA_phishing.ipynb).

### Balance de clases
El problema está **moderadamente balanceado**, por lo que se usará una partición *estratificada* y
métricas robustas al desbalance además de la exactitud.

<p align="center"><img src="figures/01_class_balance.png" width="420"></p>

### Correlación de las características con la etiqueta
Varias características del código HTML y derivadas separan bien las clases.

<p align="center"><img src="figures/02_corr_label.png" width="560"></p>

### Distribuciones por clase
Diferencias claras entre URLs legítimas y de phishing en variables como `URLSimilarityIndex`.

<p align="center"><img src="figures/03_distributions.png" width="760"></p>

### Cardinalidad de las variables de texto
`URL`, `Domain` y `Title` son casi identificadores (cardinalidad ≈ nº de muestras) → **no** se codifican
con *one-hot*. `TLD` (695 categorías) se codifica mediante `TLDLegitimateProb` o por frecuencia.

<p align="center"><img src="figures/04_cardinality.png" width="520"></p>

### ⚠️ Hallazgo importante: posible *data leakage*
Se detectaron señales que podrían inflar artificialmente el desempeño y que se vigilarán en el modelado:
- `URLSimilarityIndex` tiene una correlación de **≈ 0,86** con la etiqueta.
- **Todas** las URLs con `IsHTTPS = 1` están etiquetadas como legítimas.

Esto es coherente con los desempeños superiores al 99 % reportados en la literatura y refuerza la
necesidad de un protocolo de validación cuidadoso.

---

## 🧠 Paradigma de aprendizaje

Es un problema de **aprendizaje supervisado** (hay pares etiquetados) y de **clasificación binaria**
(la salida es categórica: legítima / phishing). Es el paradigma apropiado porque el objetivo es predecir
una etiqueta conocida a partir de ejemplos previos, no descubrir estructura latente.

**Plan de evaluación:** partición estratificada + métricas **Exactitud, F1, *Balanced Accuracy*, MCC y AUC**.

---

## 📚 Estado del arte

Trabajos que emplean el conjunto de datos PhiUSIIL:

| Trabajo | Técnica(s) | Validación | Resultados |
|---|---|---|---|
| **Prasad & Chandra, 2024** · *Computers & Security* | Índice de similitud de URL + aprendizaje incremental | Entrenamiento/prueba | **99,24 %** (incremental) · **99,79 %** (pre-entrenamiento) |
| **Gobinath & Manikandan, 2026** · *Scientific Reports* | Red neuronal profunda con atención *multi-head* (ADUIN) | Estratificada 70/15/15 | 95 % exactitud · 93 % precisión · FPR 3,5 % |
| **Bao et al., 2026** · *Computers, Materials & Continua* | Random Forest, SVM, Naïve Bayes, MLP | Hold-out 80/20 (+ k-fold) | RF 99,99 % · MLP 99,98 % · SVM 99,97 % |
| **Yulianti & Forenziana, 2026** · *Formosa J. of Comp. and Inf. Science* | Selección de características (LASSO) | Validación cruzada | LASSO (k=5): F1 0,9975 |

---

## 📁 Estructura del repositorio

```
proyecto-modelos2-phishing/
├── README.md                      # este archivo
├── reporte_avance.tex             # informe en plantilla IEEE (Overleaf)
├── requirements.txt               # dependencias de Python
├── notebooks/
│   └── 01_EDA_phishing.ipynb      # análisis exploratorio (Parte II)
├── figures/                       # gráficas generadas por el notebook
│   ├── 01_class_balance.png
│   ├── 02_corr_label.png
│   ├── 03_distributions.png
│   └── 04_cardinality.png
└── data/                          # (no versionado) ver nota abajo
```

> El CSV (~54 MB) **no se versiona** en Git. El notebook descarga los datos automáticamente con `ucimlrepo`.

---

## ⚙️ Cómo reproducir

```bash
# 1. Clonar el repositorio
git clone https://github.com/mateor32/proyecto-modelos2-phishing.git
cd proyecto-modelos2-phishing

# 2. Instalar dependencias
pip install -r requirements.txt

# 3. Abrir el notebook
jupyter notebook notebooks/01_EDA_phishing.ipynb
```

Los datos se cargan solos con:
```python
from ucimlrepo import fetch_ucirepo
ds = fetch_ucirepo(id=967)
X, y = ds.data.features, ds.data.targets
```

También puedes abrirlo directamente en **Google Colab** subiendo el `.ipynb`.

---

## 🗺️ Hoja de ruta

- [x] **Parte I** — Descripción del problema y análisis exploratorio
- [x] **Parte II** — Estado del arte
- [ ] **Parte III** — Preprocesamiento y codificación de variables
- [ ] **Parte IV** — Entrenamiento y comparación de modelos
- [ ] **Parte V** — Validación, métricas y resultados finales

---

## 👥 Autores

| Nombre | Correo |
|---|---|
| *<<Nombre Apellido>>* | *<<usuario>>@udea.edu.co* |
| *<<Nombre Apellido>>* | *<<usuario>>@udea.edu.co* |

Universidad de Antioquia — Facultad de Ingeniería
Curso: Introducción al Machine Learning (Modelos II)

---

## 📖 Referencias

1. A. Prasad y S. Chandra, "PhiUSIIL: A diverse security profile empowered phishing URL detection framework based on similarity index and incremental learning," *Computers & Security*, vol. 136, art. 103545, 2024. https://doi.org/10.1016/j.cose.2023.103545
2. R. Gobinath y S. Manikandan, "Deep learning-based phishing classification framework for accurate detection using optimized URL intelligence," *Scientific Reports*, vol. 16, art. 15794, 2026. https://doi.org/10.1038/s41598-026-46481-2
3. T. M. Bao, K. Shashvat, N. G. Nhu y D. Le, "A comparative analysis of machine learning algorithms for spam and phishing URL classification," *Computers, Materials & Continua*, vol. 87, no. 2, 2026. https://doi.org/10.32604/cmc.2025.075161
4. F. Yulianti y J. Forenziana, "A comparative analysis of feature selection methods for machine learning-based phishing URL detection," *Formosa Journal of Computer and Information Science*, vol. 5, no. 2, pp. 243–258, 2026. https://doi.org/10.55927/fjcis.v5i2.16783

---

<div align="center">
<sub>Proyecto académico · Universidad de Antioquia · 2026</sub>
</div>
