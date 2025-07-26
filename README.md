# Framework Avanzado para Evaluación de Pipelines RAG

Este repositorio presenta un framework integral para construir, comparar y evaluar sistemáticamente tres arquitecturas de **Retrieval-Augmented Generation (RAG)** de complejidad creciente. El objetivo es analizar cómo las decisiones de diseño—desde el *chunking* y los *embeddings* hasta el *re-ranking* y la *expansión de contexto*—impactan el rendimiento de los Modelos de Lenguaje Grandes (LLMs) en tareas de respuesta a preguntas.

## 📄 Paper de Investigación

Este proyecto se complementa con un paper de investigación que profundiza en el marco teórico, la metodología y el análisis de resultados. Para una comprensión más detallada, puedes consultar el documento completo.

**[Descargar el Paper en PDF](data/Documento_final_IA)**

---

## 🚀 Los Pipelines: Un Vistazo Técnico

Se implementaron y evaluaron tres pipelines, cada uno representando una evolución en sofisticación.

### 1. Pipeline RAG Simple (Baseline)

Este pipeline establece una línea base fundamental. Aunque no fue evaluado formalmente contra el dataset PeerQA, su código sirve como un excelente punto de partida y demuestra un rendimiento óptimo para tareas de extracción de información de documentos PDF individuales.

-   **Carga y Parseo**: Utiliza la librería `PyMuPDF (fitz)` para una extracción de texto directa y eficiente desde archivos PDF.
-   **Chunking**: Implementa una estrategia de división de texto de tamaño fijo (`fixed-size chunking`) con solapamiento.
-   **Embeddings**: Emplea el modelo `sentence-transformers/all-MiniLM-L6-v2`, un modelo ligero y rápido, ideal para una baseline.
-   **Búsqueda (Retrieval)**: Realiza una búsqueda semántica a través de una implementación manual de **similitud de coseno** en NumPy.
-   **Generación**: Es compatible con múltiples LLMs, incluyendo `llama3-70b-8192` (vía Groq), `gemini-2.0-flash` y `qwen` (vía OpenRouter).
-   **Notebook**: `src/simple_rag_evaluation.ipynb`

### 2. Pipeline RAG Intermedio (Evaluado como "Basic")

Este pipeline introduce técnicas estándar de la industria para mejorar la calidad de la recuperación de información, sirviendo como el primer nivel en nuestra evaluación formal.

-   **Carga y Parseo**: Usa la librería `unstructured` para un parseo más robusto que preserva metadatos estructurales del documento.
-   **Chunking**: Adopta `RecursiveCharacterTextSplitter` de LangChain, una técnica más avanzada que divide el texto respetando la estructura semántica (párrafos, frases).
-   **Embeddings**: Utiliza `sentence-transformers/all-mpnet-base-v2` a través de `HuggingFaceEmbeddings`, un modelo más potente que el de la baseline.
-   **Búsqueda (Retrieval)**: Implementa un **recuperador híbrido** (`EnsembleRetriever` de LangChain) que combina:
    -   **Búsqueda Densa/Vectorial**: Utilizando `ChromaDB` como vector store.
    -   **Búsqueda Lexical**: Utilizando `BM25Retriever` para capturar relevancia basada en palabras clave.
    - Los resultados de ambos se combinan con una ponderación de 0.7 (vectorial) y 0.3 (lexical).
-   **Automatización**: Introduce la clase `RAGEvaluationGenerator` para automatizar la generación de respuestas en lotes, gestionar APIs y preparar los datos para la evaluación final.
-   **Notebook**: `src/intermedia_rag_evaluation.ipynb`

### 3. Pipeline RAG Avanzado

Este es el pipeline más sofisticado, incorporando múltiples optimizaciones de vanguardia para maximizar la precisión y relevancia del contexto.

-   **Embeddings**: Emplea `BAAI/bge-m3`, un modelo de embeddings de alto rendimiento, diseñado para una comprensión semántica superior.
-   **Vector Store**: Utiliza `Qdrant` como base de datos vectorial, optimizada para búsquedas a gran escala.
-   **Estrategias de Búsqueda Avanzadas**:
    -   **Búsqueda Híbrida Optimizada**: Combina la búsqueda densa de Qdrant con un índice `BM25Okapi` construido dinámicamente.
    -   **Re-ranking**: Los resultados iniciales son re-ordenados usando un **Cross-Encoder** (`cross-encoder/ms-marco-MiniLM-L-12-v2`) para refinar la relevancia de los chunks.
-   **Ingeniería de Contexto y Consulta**:
    -   **Expansión de Contexto**: Enriquece los chunks recuperados añadiendo fragmentos de texto adyacentes para dar más contexto al LLM.
    -   **Expansión de Consulta**: Genera variaciones de la pregunta original para mejorar la cobertura de la búsqueda.
-   **Automatización**: La lógica está encapsulada en la clase `OptimizedRAGPipeline`, y la evaluación es gestionada por `AdvancedRAGEvaluationGenerator`.
-   **Notebook**: `src/advanced_rag_evaluation.ipynb`

## 📊 Framework de Evaluación

La evaluación de los pipelines **Intermedio** y **Avanzado** se realizó con un enfoque riguroso.

-   **Dataset**: Se utilizó el dataset **PeerQA** de Hugging Face, que contiene preguntas y respuestas basadas en artículos de investigación académica, proveyendo un benchmark realista y desafiante.
-   **Métricas (con `ragas`)**:
    -   `Faithfulness`: Fidelidad de la respuesta al contexto.
    -   `Answer Relevancy`: Relevancia de la respuesta a la pregunta.
    -   `Context Precision`: Precisión de los documentos recuperados.
    -   `Context Recall`: Cobertura de los documentos recuperados.
-   **Automatización**: Las clases `RAGEvaluationGenerator` y `AdvancedRAGEvaluationGenerator` son el corazón de la evaluación. Toman el dataset `PeerQA`, iteran sobre las preguntas, consultan el pipeline RAG correspondiente, generan respuestas con diferentes LLMs (Gemini, Llama, Qwen), y guardan los resultados en archivos `.json` estructurados, listos para ser analizados por el notebook `evaluacion.ipynb`.

### Resultados

El rendimiento comparativo de los modelos se visualiza en el siguiente heatmap:

![Heatmap de Resultados](src/heatmap_rag_models.png)

#### Fidelidad y Relevancia

| Modelo           | 🔍 Faithfulness | 🎯 Answer Relevancy |
|------------------|-----------------|---------------------|
| Llama Advanced   | 0.756           | 0.385               |
| Llama Basic      | 0.763           | 0.113               |
| Qwen Advanced    | 0.814           | 0.428               |
| Qwen Basic       | 0.789           | 0.209               |
| Gemini Advanced  | 0.863           | 0.370               |
| Gemini Basic     | 0.750           | 0.237               |

#### Precisión y Cobertura del Contexto

| Modelo           | 🎯 Context Precision | 🔁 Context Recall |
|------------------|----------------------|-------------------|
| Llama Advanced   | 0.602                | 0.505             |
| Llama Basic      | 0.277                | 0.168             |
| Qwen Advanced    | 0.711                | 0.553             |
| Qwen Basic       | 0.289                | 0.157             |
| Gemini Advanced  | 0.688                | 0.611             |
| Gemini Basic     | 0.272                | 0.141             |

## 📂 Estructura del Repositorio

```
.
├── data/
│   └── 1-s2.0-S0378517324005799-main.pdf  (Documento de ejemplo para pruebas)
├── src/
│   ├── simple_rag_evaluation.ipynb       (Pipeline Baseline)
│   ├── intermedia_rag_evaluation.ipynb   (Pipeline Intermedio, evaluado como "Basic")
│   ├── advanced_rag_evaluation.ipynb     (Pipeline Avanzado)
│   ├── evaluacion.ipynb                  (Análisis de resultados y generación de heatmap)
│   ├── rag_evaluation_results.json       (Ejemplo de salida de la evaluación intermedia)
│   └── advanced_rag_evaluation_results.json (Ejemplo de salida de la evaluación avanzada)
├── .env.example
│   └── Plantilla para variables de entorno (API Keys).
├── requirements.txt
│   └── Dependencias de Python.
└── README.md
```

## 🚀 Guía de Instalación y Ejecución

Esta guía detalla los pasos para configurar el entorno completo, incluyendo las bases de datos vectoriales en Docker y la evaluación con modelos locales.

### 1. Pre-requisitos

-   **Python**: Se recomienda encarecidamente usar **Python 3.10.6**, ya que el proyecto fue desarrollado y probado con esta versión. El uso de otras versiones podría causar conflictos con las dependencias.
-   **Docker**: Necesitarás Docker Desktop instalado y en ejecución para gestionar las bases de datos vectoriales.
-   **Git**: Para clonar el repositorio.

### 2. Configuración del Entorno

#### Paso 2.1: Clonar el Repositorio
```bash
git clone <URL-DEL-REPOSITORIO>
cd <NOMBRE-DEL-DIRECTORIO>
```

#### Paso 2.2: Entorno Virtual y Dependencias
```bash
# Crear el entorno virtual con la versión recomendada de Python
python3.10 -m venv nuevo_rag

# Activar el entorno
# En Windows:
.\nuevo_rag\Scripts\activate
# En macOS/Linux:
source nuevo_rag/bin/activate

# Instalar las dependencias
pip install -r requirements.txt
```

#### Paso 2.3: Variables de Entorno
Copia el archivo de ejemplo y añade tus claves de API para los modelos evaluados.
```bash
cp .env.example .env
```
Edita el archivo `.env` con tus credenciales:
```ini
# .env
GOOGLE_API_KEY="tu-api-key-de-google"
GROQ_API_KEY="tu-api-key-de-groq"
QWEN_API_KEY="tu-api-key-de-qwen"
OPENAI_API_KEY="tu-api-key-de-openai" # Necesaria para algunas evaluaciones de Ragas
```

### 3. Bases de Datos Vectoriales con Docker

Los pipelines intermedio y avanzado utilizan ChromaDB y Qdrant, respectivamente. La forma más sencilla de ejecutarlos es a través de Docker.

#### Paso 3.1: Iniciar Contenedor de ChromaDB (Pipeline Intermedio)
Abre una terminal y ejecuta el siguiente comando para iniciar ChromaDB:
```bash
docker run -d --name chroma-rag -p 8000:8000 -v chroma_rag_data:/chroma/chroma chromadb/chroma:latest
```
Esto iniciará un contenedor de ChromaDB que escuchará en el puerto `8000` y persistirá los datos en un volumen llamado `chroma_rag_data`.

#### Paso 3.2: Iniciar Contenedor de Qdrant (Pipeline Avanzado)
Abre otra terminal y ejecuta este comando para iniciar Qdrant:
```bash
docker run -d --name qdrant-rag -p 6333:6333 -p 6334:6334 \
    -v $(pwd)/qdrant_storage:/qdrant/storage \
    qdrant/qdrant:latest
```
Esto iniciará un contenedor de Qdrant que escuchará en el puerto `6333` y guardará los datos en una carpeta local `qdrant_storage`.

### 4. Ejecución de los Notebooks

Una vez que el entorno y los contenedores estén listos, puedes ejecutar las evaluaciones.

1.  **Inicia Jupyter Lab**:
    ```bash
    jupyter lab
    ```
2.  **Verifica las Rutas**:
    **¡Importante!** Antes de ejecutar las celdas, asegúrate de que las rutas a los archivos (como los PDFs en `data/` o los datasets generados en `resultados/`) sean correctas para tu sistema operativo. Es posible que necesites ajustar las rutas relativas o absolutas dentro de los notebooks.

3.  **Ejecuta los Notebooks en Orden**:
    -   **`simple_rag_evaluation.ipynb`**: Este notebook es autocontenido y no depende de Docker.
    -   **`intermedia_rag_evaluation.ipynb`**: Este notebook se conectará al contenedor de **ChromaDB**. La primera vez que lo ejecutes, creará y poblará la base de datos vectorial.
    -   **`advanced_rag_evaluation.ipynb`**: Se conectará al contenedor de **Qdrant**. Al igual que el anterior, la primera ejecución se encargará de la indexación.

### 5. Evaluación con Modelos Locales (Ollama)

El notebook `evaluation_via_local.ipynb` permite evaluar usando un modelo local a través de Ollama.

#### Paso 5.1: Instalar y Configurar Ollama
-   Descarga e instala Ollama desde su [sitio web oficial](https://ollama.com/).
-   Abre una terminal y descarga el modelo que se utiliza en el notebook (por ejemplo, `llama3`):
    ```bash
    ollama pull llama3
    ```
-   Una vez descargado, ejecuta el servidor de Ollama. Normalmente, se inicia automáticamente después de la instalación, pero puedes verificarlo con:
    ```bash
    ollama serve
    ```
    Esto dejará el servidor corriendo en segundo plano.

#### Paso 5.2: Ejecutar el Notebook de Evaluación Local
-   Abre `evaluation_via_local.ipynb`.
-   **Verifica el nombre del modelo**: Asegúrate de que el nombre del modelo instanciado en el notebook coincida exactamente con el que descargaste con Ollama (p. ej., `llama3`). Si usaste un modelo diferente, como `mistral`, cambia el nombre en el código.
-   Ejecuta las celdas para realizar la evaluación utilizando el modelo local.


## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor, abre un **issue** para discutir cambios o envía un **pull request**.

## 📄 Licencia

Este proyecto fue realizado para la asignatura de Inteligencia Artificial por el estudiante Daniel C. Blanco. Se permite su uso y distribución no comercial.
