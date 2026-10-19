LumaHome: Customer Support RAG with Hybrid Search

A low-code n8n Cloud demo using five synthetic English documents: shipping,
returns, payments FAQ, GlowMini manual, and sample resolved tickets.
English, Hindi, and Romanized Hindi are supported by the implemented flow.
No real customer records are included. Not affiliated with a real store.

FILES
knowledge-ingestion.json / support-chat.json: sanitized n8n workflow exports.
synthetic-corpus.json: exact five documents extracted from ingestion export.
evaluation.csv: all 20 planned cases, including ten explicitly not evaluated.
METRICS.txt: measured results and caveats.
REVIEW.txt: review findings and limitations.
DEMO_SCRIPT.txt: recording outline. A recording is not included.

SETUP
1. Import the two workflow JSON files in n8n. They are inactive drafts.
2. Create OpenAI and Pinecone credentials; select the appropriate credential
   in every HTTP Request node. Credential references were removed for sharing.
3. Create dense Pinecone index lumahome-support: dimension 1536, cosine.
   Create sparse index lumahome-keyword: dotproduct. Use an available region.
4. Replace REPLACE_DENSE_INDEX_HOST and REPLACE_SPARSE_INDEX_HOST in the four
   Pinecone index URLs (two ingestion upserts, two support searches). Retain paths.
5. Create n8n Data Table LumaHome Escalations with String columns ticket_id,
   session_id, question, reason, status, assigned_to, source_ids; Boolean column
   validation_passed. Keep built-in ID/timestamps. Select this table in Create
   Escalation Record, replacing REPLACE_ESCALATIONS_TABLE_ID; recheck mappings.
6. Check Generate Dense Embeddings body: exported name '=input' is an n8n
   expression field. Confirm its resolved parameter name is input; if needed,
   change the field name to fixed input. The supplied export is preserved here.
7. Run ingestion. Confirm eight records in namespace kb-v1 in EACH index.
   Upsert overwrites matching IDs; changed corpus/chunking may leave old IDs.
   Use a new namespace in both ingestion and search when changing versions.
8. Test support chat in the editor. Confirm replies and successful review inserts.
   Publish only when configured and ready; no public deployment is included.

PIPELINE
Ingestion: documents -> sentence-based chunks (~900 characters, one-sentence
  overlap) -> OpenAI dense and Pinecone sparse embeddings -> two indexes.
Chat: original question -> dense embedding/search; English translation ->
  sparse embedding/search -> Merge (semantic Input 1, keyword Input 2) ->
  weighted RRF (0.6 dense, 0.4 sparse; k=60; retain top five unique chunk IDs)
  -> cited JSON draft -> validation -> reply or saved escalation record.

MODELS
text-embedding-3-small (1536 dimensions), pinecone-sparse-english-v0,
gpt-4.1-mini for translation and response drafting. API calls require quota.
Ollama was discussed but is NOT used in these exported workflows.

REVIEW QUEUE
A record is saved for escalations, including validation fallbacks. It contains
retrieved chunk IDs, which are not necessarily verified supporting citations.
No support agent notification, helpdesk integration, live order access, refund
execution, or response-time guarantee is implemented.
