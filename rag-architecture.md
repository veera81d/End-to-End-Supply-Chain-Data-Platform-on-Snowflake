# RAG Architecture

This document describes the Retrieval-Augmented Generation layer of the **End-to-End Supply Chain Data Platform on Snowflake**.

## Purpose

The RAG layer gives users conversational access to supply chain knowledge by combining semantic search with generative answers. It sits on top of the curated analytics layer and turns structured business data into question-answering experiences.

## RAG Flow

```text
Knowledge Base Construction
  -> Cortex Search Service
  -> SP_RAG_QUERY
  -> Natural Language Answer
```

## Step 1: Knowledge Base Construction

The first step is building an AI-enriched knowledge base.

### Inputs
- 50 products
- 30 facilities
- 70 customers

### Output
- 150 knowledge entries
- Rich textual descriptions for each entity
- Context that can be indexed and retrieved semantically

### Purpose

This converts structured master data into a retrieval-friendly knowledge layer that supports natural-language questions.

## Step 2: Cortex Search Service

**Service name:** `SUPPLY_CHAIN_SEARCH`

The knowledge base is indexed using Cortex Search for semantic retrieval.

### Indexed attributes
- `ENTITY_TYPE`
- `ENTITY_NAME`
- `CATEGORY`
- `BUSINESS_LINE`
- `METADATA`

### Target lag
- 1 hour

### Purpose

This layer identifies the most relevant knowledge entries for a user question before generation happens.

## Step 3: RAG Query Procedure

**Procedure name:** `SP_RAG_QUERY`

### Flow
1. Accept a natural language question.
2. Retrieve the top 5 semantically relevant entries.
3. Pass the question and context to `AI_COMPLETE`.
4. Return a 3-4 sentence answer.

### Model usage
- `AI_COMPLETE` with `llama3.1-8b`

## Why RAG Is Useful

RAG allows the platform to answer business questions that are not well served by dashboards alone. Instead of forcing users to search tables or write SQL, the system can return contextual explanations, summaries, and recommendations.

## Example Use Cases

- Explain a product, facility, or customer in plain language
- Summarize a business entity using indexed context
- Answer analytical questions with supporting context
- Provide conversational access to supply chain knowledge

## Relationship to the Analytics Layer

The RAG layer complements the semantic and AI view layers.

- **Semantic views** support governed KPI analysis
- **AI views** support classification, summarization, and recommendations
- **RAG** supports conversational retrieval and generation

## Design Notes

- The knowledge base should stay aligned with curated business entities.
- Search quality depends on clean metadata and consistent descriptions.
- RAG works best when the underlying curated data and semantic models are trustworthy.
- The 1-hour search lag is acceptable for near-real-time conversational analytics, but can be tuned if needed.
