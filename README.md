# RAG-lo-que-realmente-debes-recordar
piensa en esto como un mapa mental para volver a desarrollar, no como teoría desde cero.


Claro. Como ya conoces RAG, piensa en esto como un **mapa mental para volver a desarrollar**, no como teoría desde cero.

## 1. RAG: lo que realmente debes recordar

**RAG = Retrieval-Augmented Generation.**

La idea práctica:

**Pregunta → buscar información relevante → dar contexto al LLM → generar respuesta**

```text
Usuario
   ↓
Pregunta
   ↓
Retrieval
   ↓
Documentos relevantes
   ↓
Prompt + contexto
   ↓
LLM
   ↓
Respuesta + fuentes
```

Ejemplo: tienes 500 PDFs de una empresa y preguntas:

> "¿Cuál es la política de vacaciones?"

El LLM **no debería inventar ni depender solo de su conocimiento**. Primero recuperas las partes relevantes de los documentos y luego se las pasas como contexto.

---

# 2. Pipeline RAG que debes memorizar

Hay dos fases diferentes.

### A. Indexación

```text
PDF / Word / HTML
        ↓
Extraer texto
        ↓
Chunking
        ↓
Embeddings
        ↓
Vector Database
```

### B. Consulta

```text
Pregunta
   ↓
Embedding / búsqueda
   ↓
Retrieval
   ↓
Top-K chunks
   ↓
Opcional: Reranking
   ↓
Prompt + contexto
   ↓
LLM
   ↓
Respuesta
```

Si entiendes bien estas dos fases, tienes la base de casi cualquier sistema RAG.

---

# 3. Chunking

No guardamos normalmente un documento entero. Lo dividimos en **chunks**.

Por ejemplo:

```text
manual.pdf

Chunk 1 → Introducción
Chunk 2 → Instalación
Chunk 3 → Configuración
Chunk 4 → Errores
...
```

Debes recordar tres cosas:

**chunk_size** → tamaño del fragmento.

**chunk_overlap** → texto compartido entre chunks consecutivos.

**metadata** → información sobre el chunk.

```json
{
  "text": "Para resetear la contraseña...",
  "source": "manual.pdf",
  "page": 14,
  "section": "Password"
}
```

No caigas en la regla simplista de "chunks de 500 caracteres". El mejor chunking depende del documento. Para proyectos reales suelen ser mejores estrategias que respeten **párrafos, títulos, secciones o estructura semántica**.

---

# 4. Embeddings

Convierte texto en un vector numérico que representa aproximadamente su significado.

```text
"reset password"
      ↓
Embedding model
      ↓
[0.12, -0.84, 0.31, ...]
```

Una consulta parecida:

```text
"change my password"
```

produce un vector cercano.

Eso permite hacer **semantic search**.

No necesitas memorizar matemáticas de embeddings. Recuerda:

> embedding = representación vectorial usada para comparar significado.

---

# 5. Vector Database

Guarda principalmente:

```text
vector
texto
metadata
ID
```

Ejemplos:

- Chroma
- Qdrant
- Pinecone
- Weaviate
- Milvus
- pgvector

Para aprender y hacer un MVP, **Chroma** es suficiente.

Para sistemas reales, la elección depende de escala, filtros, infraestructura, latencia y coste. No existe una vector DB universalmente "mejor".

---

# 6. Retrieval

El **retriever** decide qué información enviar al LLM.

```text
Pregunta
   ↓
Retriever
   ↓
Chunk 7
Chunk 23
Chunk 41
```

Concepto importante:

### Top-K

```python
k = 5
```

significa recuperar, por ejemplo, los 5 resultados más relevantes.

Pero **más contexto no significa mejor respuesta**.

Si recuperas 30 chunks irrelevantes, puedes empeorar la respuesta.

---

# 7. Semantic Search vs BM25

### Semantic Search

Usa embeddings.

Pregunta:

```text
How can I recover my account?
```

Puede encontrar:

```text
Password reset procedure
```

aunque no tengan exactamente las mismas palabras.

### BM25

Búsqueda basada principalmente en palabras/términos.

Es especialmente buena para:

```text
AGLVERT2-4710
ERR-503
product IDs
nombres
códigos
keywords exactas
```

Por eso, en sistemas reales aparece mucho:

## Hybrid Search

```text
BM25
  +
Vector Search
  ↓
Resultados combinados
```

Este concepto sí merece la pena recordarlo.

---

# 8. Reranking

El retrieval puede encontrar 20 candidatos.

Después, un **reranker** los vuelve a ordenar según relevancia.

```text
Query
 ↓
Retrieval → 20 chunks
 ↓
Reranker
 ↓
Top 5 realmente relevantes
 ↓
LLM
```

Una arquitectura potente puede ser:

```text
Hybrid Search
      ↓
20 candidatos
      ↓
Reranker
      ↓
5 chunks
      ↓
LLM
```

Recuerda la diferencia:

**Retriever = encuentra candidatos.**

**Reranker = mejora el orden/selección de esos candidatos.**

---

# 9. Context

El contexto es la información recuperada que introduces en el prompt.

Ejemplo conceptual:

```text
SYSTEM:
Responde únicamente usando el contexto.

CONTEXT:
{retrieved_chunks}

QUESTION:
{user_question}
```

Un problema común es pensar:

> "Tengo una ventana de contexto enorme, entonces puedo meter todo."

No es una buena arquitectura. Aumenta coste, ruido y puede reducir precisión.

RAG intenta entregar **poco contexto, pero muy relevante**.

---

# 10. Metadata filtering

Muy importante en aplicaciones reales.

Supón documentos de:

```text
HR
Finance
IT
Legal
```

Antes de hacer búsqueda puedes filtrar:

```python
department = "IT"
language = "EN"
year = 2026
```

Entonces:

```text
Pregunta
 ↓
Metadata filter
 ↓
Semantic/Hybrid search
 ↓
Resultados
```

Esto puede mejorar muchísimo el retrieval.

---

# 11. Query rewriting

El usuario puede preguntar mal:

```text
"how vacation works?"
```

El sistema puede transformarlo internamente:

```text
"employee annual leave policy, entitlement and approval process"
```

Y usar esa query para retrieval.

Conceptos relacionados que basta con reconocer:

**Multi-query retrieval** → generar varias búsquedas.

**Query expansion** → añadir términos relacionados.

**HyDE** → generar una respuesta/documento hipotético y usarlo para retrieval.

No necesitas memorizarlos todos ahora.

---

# 12. Evaluación RAG

Este punto se olvida mucho y es crítico.

No evalúes un RAG diciendo simplemente:

> "Parece que responde bien."

Separa dos problemas:

### Retrieval

¿Encontramos la información correcta?

Conceptos:

```text
Recall
Precision
Hit Rate
MRR
NDCG
```

No necesitas memorizar todas las fórmulas.

### Generation

¿El LLM respondió correctamente usando esa información?

Mira:

```text
Correctness
Faithfulness
Groundedness
Relevance
Citation accuracy
```

Idea fundamental:

```text
Mala respuesta
   ↓
¿Retrieval malo?
      OR
¿LLM usó mal un buen contexto?
```

Debes distinguirlos antes de intentar arreglar el sistema.

---

# 13. De RAG a Agentic RAG

RAG tradicional:

```text
Question
 ↓
Retrieve
 ↓
LLM
 ↓
Answer
```

Agentic RAG:

```text
Question
 ↓
Agent
 ↓
Decide qué hacer
 ├─ Search vector DB
 ├─ Search SQL
 ├─ Call API
 ├─ Search web
 └─ Ask another tool
 ↓
Analizar resultado
 ↓
Puede ejecutar otra acción
 ↓
Answer
```

Aquí empieza la parte **agéntica**.

---

# 14. Tool Calling

Un LLM no debería "simular" que llamó a Jira, SQL o una API.

Le proporcionas herramientas:

```text
search_documents()
get_customer()
create_ticket()
query_database()
send_email()
```

El modelo decide algo parecido a:

```text
Necesito información del cliente.
→ llamar get_customer()
```

Recibe:

```json
{
  "name": "Zakaria",
  "status": "active"
}
```

y continúa.

Recuerda:

> **LLM decide qué herramienta necesita; tu aplicación ejecuta realmente la herramienta.**

---

# 15. Agent

Una definición útil para desarrollo:

> **Agent = LLM + tools + instrucciones + estado + lógica para decidir la siguiente acción.**

Por ejemplo:

```text
Usuario:
"Busca mi factura y dime si está pagada."

Agent
 ↓
get_customer()
 ↓
get_invoices()
 ↓
check_payment()
 ↓
LLM
 ↓
Respuesta
```

Pero no conviertas todo en agente.

Si el proceso siempre es:

```text
A → B → C → D
```

un workflow determinista suele ser mejor: más barato, fácil de probar y predecible.

Usa un agente cuando existe una decisión real:

```text
        ┌→ Tool A
Input → Decision → Tool B
        └→ Tool C
```

---

# 16. Workflow

Un workflow define pasos controlados.

Ejemplo:

```text
START
 ↓
Classify Question
 ↓
 ┌─────────────────┐
 │                 │
HR               Technical
 │                 │
 ↓                 ↓
HR RAG           IT RAG
 │                 │
 └────────┬────────┘
          ↓
       Answer
          ↓
         END
```

Esto es justamente donde **LangGraph** resulta útil.

---

# 17. Routing

Routing significa:

> decidir qué camino debe seguir la petición.

Ejemplo:

```text
"vacation policy"
      ↓
HR Retriever

"server error 503"
      ↓
Technical Retriever

"show my invoices"
      ↓
SQL/API Tool
```

El router puede ser:

```text
reglas
clasificador
LLM
```

Para decisiones simples, no gastes un LLM innecesariamente. Una regla determinista puede ser mejor.

---

# 18. State

En sistemas agénticos necesitas recordar qué está ocurriendo **durante la ejecución**.

Ejemplo:

```python
state = {
    "question": "...",
    "documents": [...],
    "tool_results": [...],
    "answer": "...",
    "retry_count": 1
}
```

En LangGraph puedes imaginarlo así:

```text
Node A
  ↓
STATE
  ↓
Node B
  ↓
STATE actualizado
  ↓
Node C
```

El state permite que los nodos compartan información.

---

# 19. Memory ≠ State

Distinción importante.

### State

Información de la ejecución actual.

```text
¿Qué herramienta llamé?
¿Qué documentos encontré?
¿Qué resultado obtuve?
```

### Memory

Información que quieres conservar entre interacciones cuando realmente aporta valor.

```text
Conversation 1
   ↓
Memory
   ↓
Conversation 2
```

Y cuidado: guardar todo el chat como "memoria" suele ser una mala solución. Necesitas decidir **qué conservar, cuánto tiempo y por qué**.

---

# 20. Guardrails

Controlan lo que el sistema puede hacer.

Ejemplos:

```text
No revelar datos privados
No ejecutar DELETE sin autorización
Validar parámetros antes de llamar APIs
Limitar herramientas disponibles
Validar structured output
Bloquear prompt injection cuando sea posible
```

Para agentes con acciones reales, esto es fundamental.

Ejemplo:

```text
Agent
 ↓
"Delete customer"
 ↓
Guardrail
 ↓
Require human approval
 ↓
API
```

---

# 21. Human-in-the-loop

No dejes que un agente ejecute automáticamente acciones críticas.

```text
Agent
 ↓
Preparar transferencia de 10.000 €
 ↓
STOP
 ↓
Humano aprueba
 ↓
Ejecutar
```

Especialmente importante para:

```text
DELETE
pagos
emails sensibles
cambios de producción
modificaciones de datos
```

---

# 22. Structured Output

En lugar de pedir al modelo texto libre:

```text
"Priority is probably high..."
```

pide estructura:

```json
{
  "priority": "high",
  "category": "technical",
  "requires_human": false
}
```

Esto hace mucho más fiables los workflows.

Para desarrollo real, **structured outputs + validación** es una combinación que debes dominar.

---

# 23. LangGraph

No pienses en LangGraph simplemente como "framework para agentes".

Piensa:

> **LangGraph = construir workflows con estado, nodos, decisiones y ciclos.**

Modelo mental:

```text
          ┌→ retrieve ──→ rerank ──┐
START → router                     → generate → END
          └→ tool ────────────────┘
```

Conceptos que debes recordar:

```text
State
Node
Edge
Conditional Edge
START
END
Checkpoint
Human-in-the-loop
```

Ejemplo:

```text
START
 ↓
Router
 ↓
Retrieve
 ↓
¿Información suficiente?
 ↓                 ↓
YES                NO
 ↓                  ↓
Generate          Web Search
                    ↓
                  Retrieve
                    ↓
                  Generate
 ↓
END
```

Ahí LangGraph tiene mucho sentido.

---

# 24. Arquitectura que te recomiendo tener en la cabeza

Para un **RAG profesional**:

```text
Documents
   ↓
Parsing
   ↓
Chunking
   ↓
Embeddings
   ↓
Vector DB
   ↓
────────────────────
User Query
   ↓
Query processing
   ↓
Hybrid Search
   ↓
Metadata filtering
   ↓
Reranking
   ↓
Context
   ↓
LLM
   ↓
Answer + Sources
   ↓
Evaluation / Observability
```

Para **Agentic RAG**:

```text
                 ┌→ Vector DB
                 │
User → Router → Agent → API
                 │
                 ├→ SQL
                 │
                 └→ Other Tools
                       ↓
                     State
                       ↓
                  Guardrails
                       ↓
                      LLM
                       ↓
                    Answer
```

---

## 25. Lo que yo priorizaría para volver a practicar

No intentes reaprender 50 conceptos antes de programar. Para proyectos reales, tu núcleo debería ser:

**RAG:** `chunking → embeddings → vector DB → retrieval → hybrid search → reranking → context → generation → evaluation`

**Agentic AI:** `tool calling → structured output → state → routing → workflows → guardrails → human-in-the-loop`

**LangGraph:** `State → Nodes → Edges → Conditional Edges → Tools → Checkpoints`

Después puedes profundizar cuando aparezca una necesidad real en **multi-query, HyDE, GraphRAG, contextual retrieval, multi-agent, MCP, advanced memory, observability**, etc.

La mejor forma de consolidarlo ahora no es seguir estudiando teoría: es construir un pequeño **RAG normal**, medir dónde falla y después convertirlo gradualmente en **Agentic RAG**.
