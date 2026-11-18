# TP4 – Redes convolucionales para clasificar lesiones de piel (ISIC)

Redes Neuronales – ITBA, 2C 2026.

En este TP se clasifican imágenes de lesiones de piel en 9 clases con una red convolucional (CNN). El dataset es ISIC (`Split_smol`) y tiene 878 imágenes, 844 después de eliminar duplicados. La CNN se compara con una MLP de referencia y se estudian el efecto de la arquitectura, de la resolución de entrada y de la data augmentation. El notebook se entrega **ejecutado, con todos los outputs**, y el análisis está en las celdas marcadas con ✏️.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `CNN_en_Skin_Dataset.ipynb` | Notebook ejecutado de punta a punta, con outputs y análisis. |
| `requirements.txt` | Dependencias de Python. |
| `README.md` | Este archivo. |

El dataset y los modelos entrenados **no** se incluyen en el repositorio (ver `.gitignore`).

## Cómo reproducirlo

1. **Clonar el repositorio**
   ```bash
   git clone <URL-de-este-repo>
   cd <carpeta-del-repo>
   ```

2. **Agregar el dataset.** Copiar la carpeta `data/Split_smol` (la que se distribuye con la consigna, con `train/` y `val/` y una subcarpeta por clase) dentro del repositorio. La estructura tiene que quedar así:
   ```
   .
   ├── CNN_en_Skin_Dataset.ipynb
   ├── requirements.txt
   └── data/
       └── Split_smol/
           ├── train/<clase>/*.jpg
           └── val/<clase>/*.jpg
   ```
   No hace falta crear un conjunto de test: el notebook elimina los duplicados entre `train/` y `val/` y vuelve a dividir todas las imágenes en train/val/test (60/20/20), de forma estratificada y con semilla fija.

3. **Crear el entorno e instalar las dependencias**
   ```bash
   python -m venv .venv
   source .venv/bin/activate        # en Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

4. **Ejecutar el notebook** desde la carpeta que contiene `data/`:
   ```bash
   jupyter notebook CNN_en_Skin_Dataset.ipynb
   ```
   y luego *Kernel → Restart & Run All*. También se puede ejecutar sin abrir la interfaz:
   ```bash
   jupyter nbconvert --to notebook --execute --inplace CNN_en_Skin_Dataset.ipynb
   ```
   En Google Colab hay que subir o clonar el repositorio junto con la carpeta `data/` y ejecutar desde esa carpeta.

## Configuración usada

Se usaron las constantes por defecto de la sección 0, **sin modificaciones**:

| Constante | Valor |
|---|---|
| `SIZES` | [32, 64, 96] |
| `SEEDS` | [0, 1, 2] |
| `SEEDS_FINAL` | [0, 1, 2, 3, 4] |
| `MAX_EPOCHS` / `PATIENCE` | 60 / 10 |
| `BATCH` | 32 |
| `AUG_COPIES` | 2 |
| `SEED_SPLIT` | 42 (no se modifica) |

- **Hardware de la corrida entregada:** CPU de 2 núcleos, sin GPU. La ejecución completa tardó unos 26 minutos. Con más núcleos o con GPU tarda menos. Los tiempos que aparecen en las tablas dependen de la máquina, así que conviene compararlos solo entre sí.
- **Reproducibilidad:** el split es determinístico (`SEED_SPLIT=42`) y cada entrenamiento fija su semilla. Aun así, en otro hardware o con otra versión de PyTorch los accuracies pueden variar levemente (del orden del ruido entre semillas). El análisis escrito corresponde a los outputs guardados en el notebook.

## Resumen de resultados

| Modelo (test, 5 semillas) | Parámetros | Accuracy | F1 macro |
|---|---|---|---|
| CNN final (3 bloques, k=3, 16 filtros, 32 px, sin augmentation) | 156.057 | 0,673 ± 0,035 | 0,671 ± 0,035 |
| MLP de referencia (64 px) | 6.358.793 | 0,514 ± 0,053 | 0,496 ± 0,057 |

- El error estándar binomial del test es ±0,036 (169 imágenes) y el azar es 0,111.
- La CNN supera a la MLP con unas 40 veces menos parámetros. Las variantes de arquitectura y de augmentation no mostraron diferencias mayores al ruido entre semillas.
- El recall de melanoma (0,35) y el sesgo de fuente del dataset (la resolución y el tipo de foto dependen de la clase) son las principales limitaciones. Se discuten en el análisis 4.
