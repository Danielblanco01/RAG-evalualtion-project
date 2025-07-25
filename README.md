# Framework de Evaluación Comparativa de Pipelines RAG

Este repositorio contiene un framework avanzado para la implementación, evaluación y comparación de múltiples estrategias de **Retrieval-Augmented Generation (RAG)**. El objetivo es analizar de forma sistemática cómo diferentes componentes, como los modelos de lenguaje (LLMs) y las técnicas de procesamiento de texto, impactan en la calidad y precisión de las respuestas generadas.

## 🎯 Metodología

El núcleo del proyecto es la evaluación de tres pipelines de RAG con complejidad creciente:

1.  **RAG Simple**: Un pipeline base que utiliza un chunking de tamaño fijo y un modelo de embedding estándar. Sirve como punto de referencia (baseline).
2.  **RAG Intermedio**: Introduce técnicas de pre-procesamiento de texto más sofisticadas y metadatos para mejorar la relevancia de los chunks recuperados.
3.  **RAG Avanzado**: Implementa un enrutador de consultas (Query Router) que selecciona dinámicamente la mejor estrategia de búsqueda (vectorial o de texto completo) según la naturaleza de la pregunta del usuario.

Para cada uno de estos pipelines, se evalúa el rendimiento de varios LLMs (incluyendo **Gemini**, **llama** y **Qwen**) utilizando un conjunto de métricas de evaluación estandarizadas.

## 📊 Visualización de Resultados

El rendimiento comparativo de los modelos en las distintas configuraciones de RAG se resume en el siguiente heatmap, permitiendo una rápida identificación de las combinaciones más efectivas.

![Heatmap de Resultados](src/heatmap_rag_models.png)

## 📊 Evaluación comparativa de modelos/Configuraciones RAG

### 🔍 Fidelidad y Relevancia

| Modelo           | 🔍 Faithfulness | 🎯 Answer Relevancy |
|------------------|-----------------|---------------------|
| Llama Advanced   | 0.756           | 0.385               |
| Llama Basic      | 0.763           | 0.113               |
| Qwen Advanced    | 0.814           | 0.428               |
| Qwen Basic       | 0.789           | 0.209               |
| Gemini Advanced  | 0.863           | 0.370               |
| Gemini Basic     | 0.750           | 0.237               |

### 🎯 Precisión y Cobertura del Contexto

| Modelo           | 🎯 Context Precision | 🔁 Context Recall |
|------------------|----------------------|-------------------|
| Llama Advanced   | 0.602                | 0.505             |
| Llama Basic      | 0.277                | 0.168             |
| Qwen Advanced    | 0.711                | 0.553             |
| Qwen Basic       | 0.289                | 0.157             |
| Gemini Advanced  | 0.688                | 0.611             |
| Gemini Basic     | 0.272                | 0.141             |


## 📂 Estructura del Repositorio

El proyecto está organizado para separar los datos, el código fuente, los resultados y la configuración.

```
.
├── data/
│   └── 1-s2.0-S0378517324005799-main.pdf  (Documento fuente de ejemplo)
├── src/
│   ├── simple_rag_evaluation.ipynb       (Notebook para RAG Básico)
│   ├── intermedia_rag_evaluation.ipynb   (Notebook para RAG Intermedio)
│   ├── advanced_rag_evaluation.ipynb     (Notebook para RAG Avanzado)
│   ├── evaluacion.ipynb                  (Notebook para análisis de resultados)
│   └── heatmap_rag_models.png            (Imagen del heatmap de resultados)
├── .env.example
│   └── Plantilla para las variables de entorno (API Keys).
├── .gitignore
│   └── Archivos y directorios a ignorar por Git.
├── requirements.txt
│   └── Dependencias de Python para el proyecto.
└── README.md
```

**Directorios y archivos clave (no versionados):**

-   `resultados/`: Contiene los datasets de evaluación (`.json`) y reportes (`.csv`) generados por los notebooks. Ignorado por Git.
-   `vector_db/`: Almacena las bases de datos vectoriales de ChromaDB. Ignorado por Git.
-   `nuevo_rag/`: Entorno virtual de Python. Ignorado por Git.
-   `.env`: Archivo local con las API keys. Ignorado por Git.

## 🚀 Guía de Instalación

Sigue estos pasos para configurar y ejecutar el proyecto en tu entorno local.

### 1. Clonar el Repositorio

```bash
git clone <URL-DEL-REPOSITORIO>
cd <NOMBRE-DEL-DIRECTORIO>
```

### 2. Configurar el Entorno Virtual

Es fundamental utilizar un entorno virtual para aislar las dependencias del proyecto.

```bash
# Crear el entorno virtual
python -m venv nuevo_rag

# Activar en Windows
.\nuevo_rag\Scripts\activate

# Activar en macOS/Linux
source nuevo_rag/bin/activate
```

### 3. Instalar Dependencias

Instala todas las librerías de Python requeridas con el siguiente comando:

```bash
pip install -r requirements.txt
```

### 4. Configurar Variables de Entorno

Crea un archivo `.env` a partir del ejemplo proporcionado y añade tus claves de API.

```bash
# Copia el archivo de ejemplo
cp .env.example .env
```

Luego, edita el archivo `.env` con tus credenciales:

```ini
# .env
GOOGLE_API_KEY="tu-api-key-de-google"
GROQ_API_KEY="tu-api-key-de-groq"
QWEN_API_KEY="tu-api-key-de-qwen"
```

## 🛠️ Ejecución de las Evaluaciones

El proceso de evaluación se realiza a través de los Jupyter Notebooks en el directorio `src/`.

1.  **Iniciar el Servidor de Jupyter:**

    ```bash
    jupyter lab
    ```

2.  **Ejecutar los Notebooks en Orden:**

    Se recomienda seguir la secuencia de complejidad para construir las bases de datos vectoriales y generar los resultados de forma incremental:

    -   **`simple_rag_evaluation.ipynb`**: Ejecuta este notebook primero para crear la base de datos vectorial inicial y evaluar el pipeline de RAG simple.
    -   **`intermedia_rag_evaluation.ipynb`**: Continúa con este para aplicar técnicas de chunking mejoradas y evaluar el segundo nivel de RAG.
    -   **`advanced_rag_evaluation.ipynb`**: Finalmente, ejecuta este notebook para implementar y evaluar el pipeline avanzado con enrutamiento de consultas.

3.  **Analizar los Resultados:**

    -   El notebook **`evaluacion.ipynb`** contiene el código para cargar los resultados generados, calcular métricas agregadas y crear las visualizaciones como el heatmap.

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Si deseas proponer mejoras, añadir nuevos modelos de evaluación o corregir errores, por favor, abre un **issue** para discutirlo o envía un **pull request** con tus cambios.

## 📄 Licencia

Este proyecto de aula se realizó para la asignatura de inteligencia artificial por el estudiante Daniel C. Blanco y está permitido el uso y la distribución de manera no comercial.