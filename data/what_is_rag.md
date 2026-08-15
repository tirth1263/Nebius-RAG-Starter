# What is Retrieval-Augmented Generation?

Retrieval-Augmented Generation (RAG) is a technique for grounding a large language
model's answers in a specific body of text that the model was never trained on.

Instead of relying on whatever the model memorised during pre-training, a RAG system
looks up relevant passages at question time and hands them to the model as context.
The model's job shrinks from "recall a fact" to "read this passage and answer the
question" — a task language models are dramatically better at.

## The three stages of a RAG pipeline

Every RAG system, no matter how elaborate, is built from the same three stages.

### 1. Indexing

Documents are loaded, split into chunks, and converted into vectors by an embedding
model. Each vector is a numeric fingerprint of a chunk's meaning. The vectors are
stored in a vector index, either in memory for small collections or in a dedicated
vector database for large ones.

Chunk size is the most consequential knob at this stage. Chunks that are too small
lose the context needed to make sense; chunks that are too large dilute the signal
and waste tokens.

### 2. Retrieval

The user's question is embedded with the same model used during indexing, then
compared against every stored vector. The closest matches — usually measured by
cosine similarity — are returned as the top-k results.

Retrieval quality sets the ceiling on answer quality. If the correct passage is not
retrieved, no amount of prompt engineering will produce a correct answer.

### 3. Generation

The retrieved chunks are inserted into a prompt alongside the original question and
sent to the language model. The model synthesises an answer using the supplied
passages as its source of truth.

Because the passages are known, the answer can be cited back to specific documents.
This traceability is one of RAG's most valuable properties.

## Why use RAG instead of fine-tuning?

Fine-tuning adjusts a model's weights on your data. RAG leaves the model untouched
and changes what you put in front of it. They solve different problems, and for
most knowledge-oriented applications RAG is the better fit.

**Freshness.** A RAG index is updated by re-indexing a folder. A fine-tuned model is
updated by running another training job. When your knowledge changes weekly, that
difference compounds fast.

**Cost.** Indexing a document collection costs a few embedding calls. Fine-tuning
costs GPU hours, and you pay them again with every meaningful data change.

**Traceability.** A RAG answer arrives with the passages that produced it, so a human
can verify it. A fine-tuned model gives you a confident sentence and no receipts.

**Access control.** Retrieval can be filtered per user before generation happens.
Knowledge baked into weights cannot be un-baked for a user who should not see it.

**Hallucination pressure.** Grounding the model in retrieved text measurably reduces
fabrication, because the answer is constrained by what was actually supplied.

Fine-tuning still wins when you need to change a model's *behaviour* — its tone, its
output format, its adherence to a domain-specific style. The two techniques compose
well: fine-tune for behaviour, retrieve for knowledge.

## Common failure modes

- **Bad chunking.** Splitting mid-sentence or mid-table destroys meaning.
- **Embedding mismatch.** Using different models to index and to query produces
  vectors that live in incompatible spaces and retrieval collapses.
- **Top-k set too low.** The right passage exists in the index but never reaches
  the model.
- **Top-k set too high.** The right passage reaches the model buried in noise.
- **No reranking.** Vector similarity is a coarse signal; a reranker applied to a
  larger candidate set consistently improves precision.
