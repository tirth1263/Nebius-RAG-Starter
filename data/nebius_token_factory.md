# Nebius Token Factory

Nebius Token Factory is a hosted inference platform that serves open-source models
through an OpenAI-compatible API. You send a request with an API key, you get tokens
back. There is no infrastructure to provision and no model weights to download.

## What it serves

Token Factory hosts two families of models that matter for a RAG pipeline.

**Generative models** produce the final answer. The default used in this project is
`deepseek-ai/DeepSeek-V3`, a strong general-purpose open-weights model with a large
context window. Other served families include Meta's Llama models, Qwen, and Mistral.

**Embedding models** turn text into vectors for retrieval. The default here is
`BAAI/bge-en-icl`, an English embedding model that performs well on retrieval
benchmarks. Multilingual alternatives such as the `intfloat/e5` family are also
available.

## Why it fits a starter RAG project

**No local GPU.** Both embedding and generation happen server-side. The notebook runs
comfortably on a laptop with no accelerator.

**One credential.** A single `NEBIUS_API_KEY` authenticates both the embedding model
and the LLM, so there is one secret to manage rather than two.

**OpenAI-compatible surface.** The API mirrors the OpenAI request and response shapes,
so existing tooling and client libraries work with a changed base URL.

**First-class LlamaIndex integration.** The `llama-index-llms-nebius` and
`llama-index-embeddings-nebius` packages expose `NebiusLLM` and `NebiusEmbedding`,
which drop directly into LlamaIndex's `Settings` object.

## Getting a key

Sign up at the Nebius Token Factory console and create an API key from the dashboard.
Treat the key like a password: keep it in an environment variable or a `.env` file,
never in source control, and rotate it if it is ever exposed.

## Cost model

Billing is per token, metered separately for input and output. Embedding calls are
substantially cheaper than generation calls, which is why re-embedding a document
collection is cheap while regenerating answers over a large context is not.

Two practical consequences for RAG:

- Persist your index. Re-embedding unchanged documents on every run is pure waste.
- Tune `similarity_top_k` deliberately. Every extra retrieved chunk becomes input
  tokens on every single query.
