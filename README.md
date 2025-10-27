fluj# 🧠 SentimentIA – Flujo de Análisis Emocional y Generación de Post para Redes Sociales (LinkedIn / X)

## 📋 Descripción General

**SentimentIA** es un flujo automatizado en **n8n** que combina análisis de sentimientos, minería de comentarios y generación de contenido estratégico mediante inteligencia artificial.  
El sistema extrae comentarios desde **YouTube**, los analiza con modelos de **OpenAI** para identificar emociones, temas y riesgos reputacionales, y genera automáticamente un **post profesional** para **LinkedIn o X (Twitter)**.  
Finalmente, guarda el resultado en una base de datos de **Notion** para su gestión y revisión.

El flujo está diseñado para **community managers, estrategas digitales o analistas de reputación** que deseen convertir conversaciones en línea en **insights narrativos y contenido de alto valor**.

---

## 🧩 Estructura del Flujo

### 1. **Start**
- **Función:** Nodo inicial que activa el flujo.
- **Salida:** Dispara la obtención de comentarios desde YouTube.

---

### 2. **Get YouTube Comments3**
- **Tipo:** `HTTP Request`
- **Función:** Conecta con la API de YouTube para obtener los comentarios del video especificado.
- **Parámetros clave:**
  - `videoId`: `5KmpT-BoVf4`
  - `maxResults`: `100`
  - `part`: `snippet`
  - `key`: Clave de API de YouTube.
- **Salida:** Array con los comentarios del video.

---

### 3. **Limpiar Comentarios3**
- **Tipo:** `Function`
- **Función:** Extrae y normaliza los datos relevantes de cada comentario.
- **Campos resultantes:**
  - `comentario_original`
  - `autor`
  - `fecha`
  - `likes`

Código principal:
```js
const resultados = [];
for (const item of $json["items"] || []) {
  const data = item.snippet.topLevelComment.snippet;
  resultados.push({
    comentario_original: data.textOriginal,
    autor: data.authorDisplayName,
    fecha: data.publishedAt,
    likes: data.likeCount
  });
}
return resultados;
```

---

### 4. **Guardar CSV3**
- **Tipo:** `Spreadsheet File`
- **Función:** Exporta los comentarios procesados en formato CSV como respaldo.

---

### 5. **Extract from File3**
- **Tipo:** `Extract from File`
- **Función:** Extrae los datos del CSV generado para procesarlos posteriormente en el flujo.

---

### 6. **Limit3**
- **Tipo:** `Limit`
- **Función:** Limita el número de comentarios a analizar (por defecto: **10**) para optimizar costos y tiempos de ejecución.

---

### 7. **Edit Fields3**
- **Tipo:** `Set`
- **Función:** Ajusta los nombres de campos o elimina caracteres no deseados antes de enviar a la IA.

---

### 8. **Analizador de Sentimiento (GPT-4.1-mini)2**
- **Tipo:** `OpenAI – n8n Langchain`
- **Modelo:** `GPT-4.1-mini`
- **Función:** Analiza cada comentario y devuelve un JSON estructurado con los siguientes campos:
  - `comentario_original`
  - `sentimiento` (positivo | negativo | neutral)
  - `intensidad_sentimiento`
  - `emociones_detectadas`
  - `justificacion`
  - `tema_detectado`
  - `posible_reaccion_comunidad`
  - `riesgo_reputacional`
  - `tipo_de_usuario`
  - `nivel_de_influencia`
  - `accion_recomendada`

---

### 9. **Code in JavaScript**
- **Tipo:** `Code`
- **Función:** Limpia la salida de OpenAI y deja solo los campos JSON relevantes.

---

### 10. **Generador de Post (GPT-5 – Community Manager Polémico)1**
- **Tipo:** `OpenAI – n8n Langchain`
- **Modelo:** `GPT-5`
- **Rol asignado:** Community Manager estratégico con tono reflexivo y humano.
- **Objetivo:** Transformar los análisis de comentarios en un post para redes sociales (LinkedIn o X) con:
  - Un **gancho inicial (scroll stopper)**.
  - **Contexto** breve.
  - **Reflexión o aprendizaje.**
  - **Cierre** que invita al diálogo.
  - **Hashtags relevantes.**

- **Output:** Post completo + resumen analítico + insight central.

#### 🧩 Variación entre plataformas
Este nodo puede adaptarse fácilmente para publicar en **LinkedIn** o **X (Twitter)** modificando el *prompt* del modelo:

- **Modo LinkedIn:** tono profesional, reflexivo, orientado al liderazgo o innovación.  
- **Modo X (Twitter):** tono directo, provocador, emocional o de debate público (280 caracteres).

Solo se requiere cambiar el prompt dentro del nodo “Generador de Post” para ajustar la salida al estilo de cada plataforma.

---

### 11. **Code in JavaScript1**
- **Tipo:** `Code`
- **Función:** Prepara el contenido final para ser guardado en una base de datos de Notion.
- **Crea un objeto JSON con la siguiente estructura:**
```json
{
  "parent": { "database_id": "290d136c07c68015b7a0fcf7091d8418" },
  "properties": {
    "Content": {
      "title": [
        { "text": { "content": "Texto del post generado" } }
      ]
    }
  }
}
```

---

### 12. **Enviar a Notion**
- **Tipo:** `HTTP Request`
- **Función:** Envía el contenido final al endpoint `https://api.notion.com/v1/pages` para almacenarlo en la base de datos.
- **Autenticación:** `notionApi` (credencial predefinida).
- **Notas:** “Guarda el post final en tu base de datos de Notion.”

---

## ⚙️ Flujo de Datos

Start → Get YouTube Comments  
→ Limpiar Comentarios  
→ Guardar CSV  
→ Extract from File  
→ Limit  
→ Edit Fields  
→ Analizador de Sentimiento (GPT-4-mini)  
→ Code JS  
→ Generador de Post (GPT-5)  
→ Code JS Final  
→ Enviar a Notion

---

## 🔐 Requisitos Previos

1. **Credenciales necesarias:**
   - API Key de **YouTube Data v3**
   - API Key de **OpenAI**
   - Token de **Notion Integration**

2. **Base de datos en Notion:**
   - Crea una base con al menos una propiedad tipo `title` llamada `Content`.
   - Copia su `database_id` y reemplázalo en el flujo.

3. **Instancia de n8n:**
   - Puede ejecutarse en local, n8n.cloud o Docker.

---

## 🚀 Cómo Ejecutarlo

1. Importa el archivo `sentimentIA.json` en tu instancia de **n8n**.
2. Configura las credenciales:
   - `YouTube API`
   - `OpenAI`
   - `Notion`
3. Ajusta el ID del video y la base de datos Notion.
4. Si deseas generar publicaciones para **X (Twitter)**, modifica el *prompt* del nodo *Generador de Post* para usar un estilo más breve, emocional y directo.
5. Ejecuta el flujo y observa:
   - Los comentarios extraídos.
   - El análisis emocional generado.
   - El post final guardado automáticamente en Notion.

---

## 🧭 Objetivo Final

Transformar conversaciones reales (comentarios) en **insights narrativos y contenido estratégico para redes sociales**, optimizando el trabajo de community management con inteligencia artificial.

---

## 📦 Archivo

- **Nombre:** `sentimentIA.json`
- **Versión:** 1.1
- **Creado por:** 🧑‍💻 Juan Gonzalez(UNIVERSIDAD TECNOLOGICA DE PEREIRA)
- **Compatibilidad:** n8n ≥ v1.8

---

## 🧩 Posibles Extensiones

- Añadir análisis de sentimiento multilingüe.
- Conectar a X (Twitter), Reddit o TikTok como nuevas fuentes.
- Automatizar la publicación del post directamente en LinkedIn o X.
- Enviar alertas de alto riesgo reputacional por correo o Slack.

---

## 🧠 Ejemplo de Output en Notion

| Comentario Original | Sentimiento | Tema Detectado | Riesgo | Acción Recomendada | Post Generado |
|----------------------|--------------|----------------|--------|--------------------|----------------|
| “No confío en esa IA…” | Negativo | Ética en IA | Medio | Contención | “La confianza en la IA no se construye con código, sino con diálogo. ¿Qué opinas tú?” |

---

> 💡 **Insight:** Este flujo convierte ruido digital en aprendizaje humano — automatizando la empatía estratégica para LinkedIn y X.
