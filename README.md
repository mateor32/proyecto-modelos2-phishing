<div align="center">

# 🎣 Detección de URLs de *Phishing* con Machine Learning

### Clasificación binaria supervisada sobre el conjunto de datos PhiUSIIL

*Proyecto del curso **Introducción al Machine Learning (Modelos II)** — Universidad de Antioquia*

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.9-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat&logo=jupyter&logoColor=white)
![Estado](https://img.shields.io/badge/estado-entrega%20final-brightgreen)

</div>

---

## 📑 Tabla de contenidos
- [Descripción](#-descripción)
- [Resultados principales](#-resultados-principales)
- [Conjunto de datos](#-conjunto-de-datos)
- [Metodología](#-metodología)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Cómo reproducir](#-cómo-reproducir)
- [Autores](#-autores)
- [Referencias](#-referencias)

---

## 🎯 Descripción

El **phishing** es un ataque en el que un sitio web fraudulento suplanta a una entidad legítima
para robar credenciales o información financiera. Las listas negras tradicionales son **reactivas**:
solo bloquean URLs ya reportadas y fallan ante los sitios nuevos que aparecen cada día.

Este proyecto construye y valida modelos de **aprendizaje supervisado** que deciden si una URL es
**legítima** o de **phishing**. El informe completo, en formato IEEE Transactions, está en
[`informe.pdf`](informe.pdf).

---

## 🏆 Resultados principales

**Escenario Solo URL** (19 variables calculadas a partir de la URL). MCC en el conjunto de prueba
(47.074 URLs de dominios que no aparecen en entrenamiento):

| Modelo | MCC validación | MCC prueba | FP | FN |
|---|---|---|---|---|
| Regresión logística | 0,9091 ± 0,0015 | 0,9090 ± 0,0038 | 622 | 1.477 |
| KNN | 0,9653 ± 0,0026 | 0,9678 ± 0,0023 | 296 | 446 |
| Random Forest | 0,9911 ± 0,0015 | 0,9928 ± 0,0011 | 55 | 112 |
| **MLP** | **0,9956 ± 0,0014** | **0,9960 ± 0,0008** | 2 | 91 |
| **SVM (RBF)** | **0,9949 ± 0,0041** | **0,9961 ± 0,0008** | 6 | 85 |

**Hallazgos:**
- ⚠️ **Fuga de información.** Con las 50 variables numéricas, incluso la regresión logística llega a
  MCC 0,9998. Variables como `URLSimilarityIndex` e `IsHTTPS` separan las clases casi de forma
  determinista por cómo se construyó el conjunto, así que el escenario principal las excluye.
- ✂️ **Selección de variables.** Se pueden eliminar 10 de las 19 variables (52,6 %) sin pérdida
  apreciable: MLP obtiene MCC de prueba 0,9958 con 9 variables.
- 📉 **PCA y UMAP.** PCA necesita 14 componentes (99 % de la varianza) para conservar el desempeño;
  UMAP pierde más de 0,15 en MCC incluso con 10 dimensiones.

---

## 📊 Conjunto de datos

**PhiUSIIL Phishing URL Dataset** — Prasad & Chandra (2024), *Computers & Security*, disponible en el
[UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+dataset).

| Propiedad | Valor |
|---|---|
| Muestras | 235.795 URLs (235.370 sin URLs repetidas) |
| Características | 54 (+ etiqueta): léxicas de la URL, derivadas y del código HTML |
| URLs legítimas | 134.850 (57,2 %) |
| URLs de phishing | 100.945 (42,8 %) |
| Valores faltantes | Ninguno |

En el código la clase positiva es **phishing** (`label = 0` en el archivo original).

---

## 🔬 Metodología

- **Partición:** 80 % entrenamiento / 20 % prueba, estratificada y **agrupada por dominio**
  (`StratifiedGroupKFold`), de modo que ningún dominio aparece en ambos conjuntos.
- **Validación:** validación cruzada estratificada y agrupada de 5 pliegues; intervalos como
  media ± 2 desviaciones estándar.
- **Preprocesamiento:** `log(1 + x)` y estandarización dentro de un `Pipeline`.
- **Métrica principal:** coeficiente de correlación de Matthews (MCC); también exactitud, F1, AUC y FPR.
- **Modelos:** regresión logística, KNN, Random Forest, MLP y SVM, cada uno con su malla de hiperparámetros.
- **Reducción de dimensión** sobre los dos mejores modelos: análisis individual (Fisher, AUC,
  información mutua, correlación), selección de variables, PCA y UMAP, repitiendo la búsqueda de
  hiperparámetros en cada espacio.

---

## 📁 Estructura del repositorio

```
proyecto-modelos2-phishing/
├── README.md
├── informe.pdf                     # informe final (IEEE Transactions)
├── informe.tex                     # fuente LaTeX del informe
├── requirements.txt
├── 01_EDA_phishing.ipynb           # Sección 2: análisis exploratorio
├── 02_modelos.ipynb                # Sección 4: entrenamiento y evaluación de 5 modelos
├── 03_reduccion_dimension.ipynb    # Sección 5: selección de variables, PCA y UMAP
├── figures/                        # figuras usadas en el informe
├── results/                        # mallas, mejores configuraciones y tablas (CSV/JSON)
└── data/                           # (no versionado) CSV de PhiUSIIL
```

Los notebooks 02 y 03 guardan sus resultados en `results/`. Si un archivo ya existe, lo reutilizan en
lugar de recalcularlo, así que se pueden volver a ejecutar sin repetir el entrenamiento.

---

## ⚙️ Cómo reproducir

```bash
git clone https://github.com/mateor32/proyecto-modelos2-phishing.git
cd proyecto-modelos2-phishing
pip install -r requirements.txt
jupyter notebook
```

- Los datos se descargan solos con `ucimlrepo` si no existe `data/PhiUSIIL_Phishing_URL_Dataset.csv`.
- Ejecutar los notebooks en orden: **01 → 02 → 03**. El 03 usa los resultados del 02.
- El entrenamiento completo es pesado (varias horas y bastante memoria). Se ejecutó en
  **Google Colab**: los notebooks 02 y 03 incluyen una celda inicial que monta Google Drive y trabaja en
  la carpeta `proyecto-modelos2-phishing`.
- Para recalcular todo desde cero, cambiar `RECALCULAR = False` a `True` en la celda de parámetros.

---

## 👥 Autores

| Nombre | Correo |
|---|---|
| Julián Andrés Agudelo Echeverri | julian.agudelo1@gmail.com |
| Mateo Reyes Uribe | mateo.reyesu@udea.edu.co |
| Cristian Lizarazo Camaño | cristian.lizarazoc@udea.edu.co |

Universidad de Antioquia — Facultad de Ingeniería · Curso: Introducción al Machine Learning (Modelos II)

---

## 📖 Referencias

1. A. Prasad y S. Chandra, "PhiUSIIL: A diverse security profile empowered phishing URL detection framework based on similarity index and incremental learning," *Computers & Security*, vol. 136, art. 103545, 2024. https://doi.org/10.1016/j.cose.2023.103545
2. R. Gobinath y S. Manikandan, "Deep learning-based phishing classification framework for accurate detection using optimized URL intelligence," *Scientific Reports*, vol. 16, art. 15794, 2026. https://doi.org/10.1038/s41598-026-46481-2
3. T. M. Bao, K. Shashvat, N. G. Nhu y D.-N. Le, "A comparative analysis of machine learning algorithms for spam and phishing URL classification," *Computers, Materials & Continua*, vol. 87, no. 2, 2026. https://doi.org/10.32604/cmc.2025.075161
4. F. Yulianti y J. Forenziana, "A comparative analysis of feature selection methods for machine learning-based phishing URL detection," *Formosa Journal of Computer and Information Science*, vol. 5, no. 2, pp. 243–258, 2026. https://doi.org/10.55927/fjcis.v5i2.16783

---

<div align="center">
<sub>Proyecto académico · Universidad de Antioquia · 2026</sub>
</div>
