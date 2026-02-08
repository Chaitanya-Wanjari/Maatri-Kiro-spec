## Maatri

Maatri is a bilingual AI-powered maternal healthcare assistant that provides verified pregnancy guidance to expecting mothers in India. The system implements a dual-pipeline Retrieval-Augmented Generation (RAG) architecture on AWS, maintaining separate Hindi and English processing flows to preserve linguistic accuracy and cultural context.

The architecture prioritizes safety, accessibility, and trustworthiness through:
- Language-specific RAG pipelines with dedicated vector indexes
- Multi-stage retrieval (semantic search + cross-encoder re-ranking)
- Source citation for all factual claims
- Safety classification for high-risk symptom detection
- Voice and text interaction modes for low-literacy users
- Low-bandwidth optimization for rural connectivity
