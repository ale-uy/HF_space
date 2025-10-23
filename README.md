# Despliegue de RAG - Hugging Face Spaces

Este directorio contiene los artefactos que uso para desplegar la API RAG de mi proyecto principal. La arquitectura de producción es:

- Frontend: GitHub Pages (sitio estático bajo `ale-uy.github.io`) — interfaz del chat.
- Servicio LLM: Hosted en Hugging Face Spaces (este repositorio `HF_space` corre la API FastAPI que conecta el LLM con el retriever).
- Vector DB: Qdrant (instancia separada, mas información en [Tu_CV_Chatero](https://github.com/ale-uy/Tu_CV_Chatero)).

#### Archivos clave
 - `rag_api.py`: API FastAPI que expone endpoints `/ask` y `/` (health check). En el arranque conecta a Qdrant y configura la cadena RetrievalQA usando:
    - Embeddings: `GoogleGenerativeAIEmbeddings` (modelo: `models/text-embedding-004`).
    - LLM: `ChatGroq` (configurable vía `MODEL_NAME`, por defecto `openai/gpt-oss-120b`).
    - Vector store: `Qdrant` (con `QdrantClient`).
 - `Dockerfile`: imagen Python 3.11-slim preparada para correr Uvicorn en el puerto 7860 (estándar de HF Spaces).
 - `requirements.txt`: dependencias necesarias (langchain, qdrant-client, fastapi, uvicorn, langchain-groq, google-generativeai, etc.).

#### Variables de entorno (obligatorias / recomendadas)

Las variables que la API espera (puedes colocarlas en el entorno de HF Spaces o en un archivo `.env` que cargues en tu despliegue):

- QDRANT_URL: URL completa de la instancia de Qdrant (por ejemplo `http://mi-host-qdrant:6333` o la URL provista por el servicio).
- QDRANT_API_KEY: API key si tu instancia de Qdrant la requiere.
- COLLECTION_NAME: (opcional) nombre de la colección en Qdrant. Por defecto `carrera_profesional`.
- GROQ_API_KEY: clave para el servicio Groq (si usas ChatGroq como LLM).
- SYSTEM_PROMPT: prompt del sistema que se inyecta en el PromptTemplate para el QA.
- MODEL_NAME: (opcional) modelo de Groq a usar. Por defecto `openai/gpt-oss-120b` si no se sobreescribe.

#### Cómo preparar la base de vectores (Qdrant)

La carga de datos a Qdrant se realiza con un script del flujo de ingesta local. Para mas información dirigirse al proyecto especifico: [Tu_CV_Chatero](https://github.com/ale-uy/Tu_CV_Chatero)

#### Despliegue de la API en HF Spaces

HF Spaces espera escuchar en el puerto 7860. El `Dockerfile` incluido ya expone Uvicorn en ese puerto.

**Pasos mínimos para desplegar en HF Spaces (Docker):**
1. Empaqueta este directorio (`HF_space`) como un Space de tipo `Docker` en Hugging Face.
2. Añade las variables de entorno en la configuración del Space (GROQ_API_KEY, QDRANT_URL, QDRANT_API_KEY, SYSTEM_PROMPT, COLLECTION_NAME si quieres otro nombre).
3. Sube el repositorio del Space (o conéctalo a GitHub) y despliega. HF construirá la imagen a partir del `Dockerfile` y expondrá el servicio en el puerto 7860.

#### Ejemplo con curl (ajusta URL y puerto):

```bash
curl -X POST "https://mi-space.hf.space/ask" -H "Content-Type: application/json" -d '{"query":"¿Qué proyectos tengo relacionados con ML?"}'
```

#### Notas de seguridad y costes:

- Asegúrate de no exponer claves (GROQ_API_KEY, QDRANT_API_KEY, GOOGLE_API_KEY) en repositorios públicos. Usa los secrets/variables de entorno del servicio de despliegue.
- Google Gemini / Google Generative AI y Groq pueden tener costes por uso; revisa tu plan y limita el tráfico si es necesario.

#### Checklist rápida antes de desplegar

- [ ] Confirmar que Qdrant está accesible desde HF Spaces (puede requerir IP pública, VPN o un servicio gestionado).
- [ ] Poner todas las variables de entorno en el Space.
- [ ] Ejecutar localmente con las mismas variables para verificar flujo completo: ingesta -> health check -> /ask.

#### Enlaces útiles (referencias externas)

- Código fuente de la ingesta (ejemplos y más contexto): https://github.com/ale-uy/Tu_CV_Chatero
- Interfaz pública / despliegue en GitHub Pages: https://ale-uy.github.io