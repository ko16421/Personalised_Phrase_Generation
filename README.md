# Final project: Retrieval-Based Personalised Phrase Generation for AAC

This repository contains the code developed for my MSc Computer Science dissertation.

## System overview

The system compares three conditions:

1. AAC-only: the participant’s input displayed through Proloquo4Text.
2. LLM-only: grammatical reconstruction using a locally deployed Gemma model.
3. LLM+RAG: personalised generation using participant-specific conversational messages retrieved from ChromaDB.

All language-model processing was conducted locally using Ollama.
CSV file 'AB123' is a test file and is not from participants.

## Requirements

- Python version 3
- Ollama (I used version 0.30.7)
- Gemma 4:12b
- sentence-transformers (model name is "all-MiniLM-L6-v2")
- ChromaDB
- pandas

Install the required Python packages using:

```bash
pip install -r requirements.txt
```