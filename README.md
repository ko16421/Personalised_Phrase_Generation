# Personalised Retrieval-Based Personalised Phrase Generation for AAC

A privacy-oriented proof-of-concept exploring whether a locally deployed large language model (LLM) and retrieval-augmented generation (RAG) can produce AAC phrases that preserve a user’s intended meaning while better reflecting their individual communication style.

Developed for my MSc Computer Science (Conversion) project at Queen Mary University of London, drawing on my previous clinical experience as a Speech and Language Therapist.

## The problemn

Augmentative and Alternative Communication (AAC) systems can support people who have difficulty producing spoken language, but their outputs may not reflect the way an individual naturally communicates.

For people with acquired communication difficulties such as aphasia, this creates two related challenges:

1. **Meaning reconstruction** — turning a short or agrammatic input into a complete sentence without changing what the user intended to say.
2. **Linguistic personalisation** — making the resulting sentence
sound more like the individual rather than producing generic
language.

This project investigates whether these tasks can be combined using a local LLM and examples of a user’s previous written communication.

## System overview

The final pipeline deliberately separates meaning reconstruction from personalisation:

Short / agrammatic input
          │
          ▼
 Gemma 4:12B (local)
 meaning reconstruction
          │
          ▼
Complete grammatical sentence ─────────────► LLM-only output
          │
          ▼
all-MiniLM-L6-v2 embedding
          │
          ▼
Participant-specific ChromaDB
          │
          ▼
5 semantically similar messages
          │
          ▼
Gemma + retrieved style examples
          │
          ▼
Personalised LLM + RAG output

Early experiments found that retrieving personal language samples before the input’s meaning had been reconstructed could distract the model and introduce unrelated information. The final architecture therefore establishes the intended meaning first and only then introduces retrieved examples for stylistic personalisation.

## Tech stack

- **Python**
- **Ollama** for local LLM inference
- **Gemma 4:12B** for grammatical reconstruction and personalised
generation
- **Sentence Transformers**
- **all-MiniLM-L6-v2** for 384-dimensional sentence embeddings
- **ChromaDB** for participant-specific vector storage and semantic
retrieval
- **Jupyter Notebook**
- **pandas / NumPy / SciPy** for data handling and analysis

All language-model processing was performed locally rather than sending
personal communication data to an external cloud LLM.

## How it works

### 1. Build the language sample

Previous messages provide examples of the user’s natural communication style. Messages are embedded using `all-MiniLM-L6-v2` and stored in a participant-specific ChromaDB collection.

### 2. Reconstruct the intended meaning

A short or agrammatic input is passed to the local LLM. The prompt is designed to preserve literal meaning, retain negation and speaker perspective, reconstruct rather than answer questions, avoid unsupported information, and return a complete grammatical sentence.

This forms the **LLM-only** output.

### 3. Retrieve relevant examples

The reconstructed sentence is embedded and used as the retrieval query. The five most semantically similar messages are retrieved from the user’s ChromaDB collection.

### 4. Personalise the sentence

The reconstructed sentence and retrieved messages are passed to Gemma in a new prompt. Retrieved messages are explicitly treated as **style examples** — evidence of wording, punctuation, emphasis and phrasing rather than new factual content.

This forms the **LLM + RAG** output.

## Model selection

Local inference was a core design choice because personal communication data can be highly sensitive.

| Model | Development outcome |
| --- | --- |
| `llama3.1:8b` | Useful for early experimentation
but produced more meaning-preservation and instruction-following errors |
| `qwen3.6` | Some generations took more than 15
minutes on the study hardware, making it unsuitable for interactive communication |
| `gemma4:12b` | More consistently preserved intended meaning and followed the project requirements; selected for the final pipeline |

Disabling Gemma’s thinking mode reduced generation time during
development to under five seconds while retaining stronger performance in the project-specific tests.

## Evaluation

The proof-of-concept was evaluated with **14 English-speaking adults without aphasia**. Participants supplied personal language samples and deliberately reduced their language for three communication scenarios.

Three conditions were compared:

| Condition | Description |
| --- | --- |
| **AAC only** | Input reproduced without AI reconstruction |
| **LLM only** | Grammatical reconstruction produced by the local LLM |
| **LLM + RAG** | Personalised version generated using semantically retrieved examples from the participant’s language sample |

Participants evaluated meaning fidelity, personalisation, liking and naturalness, comparatively ranked the outputs, and took part in a semi-structured interview.

## Results

Participant-level median ratings were highest for **LLM + RAG** across all four measures:

| Outcome | AAC only | LLM only | LLM + RAG |
| --- | --- | --- | --- |
| Meaning fidelity | 3.5 | 6.0 | **8.0** |
| Personalisation | 1.0 | 4.5 | **7.0** |
| Liking | 1.5 | 5.5 | **7.0** |
| Naturalness | 1.5 | 5.0 | **8.0** |

Condition effects were statistically significant for all four outcomes (`p < .001` in Friedman tests), with significant Holm-adjusted pairwise differences between all three conditions.

Qualitative findings added an important qualification: participants valued personalisation, particularly its potential to preserve identity, but this depended on **meaning accuracy, contextual appropriateness, reliability and user control**.

These findings support further investigation of retrieval-based
personalisation rather than establishing clinical effectiveness.

## Key development lesson

The final architecture emerged through iterative testing rather than working as intended on the first attempt.

Applying retrieval directly to incomplete input sometimes allowed
retrieved language to dominate generation and change the intended
meaning. This led to the final two-stage approach:

```
reconstruct meaning → retrieve relevant examples → personalise style
```

The project therefore highlighted an important design principle for AI-assisted AAC: **personalisation is only useful if the system first communicates what the user actually intended to say.**

## Important limitation

The system was designed with aphasia-oriented AAC in mind, but
recruitment difficulties meant that the final evaluation involved
participants **without aphasia**, who deliberately produced reduced inputs.

The study therefore does **not** demonstrate that the system is
effective for people with aphasia. Evaluation with the intended
population is required before conclusions can be drawn about clinical usefulness.

This implementation should be considered a research proof-of-concept, not a clinical AAC system.

## Repository structure

```
.
├── FINAL.ipynb
├── AB123_public_example_200.csv
├── requirements.txt
└── README.md
```

- `FINAL.ipynb` — main pipeline implementation and demonstration.
- `AB123_public_example_200.csv` — reduced example language sample for demonstrating retrieval.
- `requirements.txt` — Python dependencies.
- `README.md` — project documentation.

The complete research datasets and persisted research ChromaDB
collections are intentionally excluded because they contain private communication data.

## Running the project

### 1. Clone the repository

```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Install and run Ollama

Install Ollama separately, then pull the model used by the final
pipeline:

```bash
ollama pull gemma4:12b
```

### 5. Open the notebook

```bash
jupyter notebook FINAL.ipynb
```

Run the cells in order to create the embeddings/vector collection and
test the reconstruction and personalisation pipeline.

> **Note:** exact model availability and Ollama model names may change.
This repository reflects the environment used during development.
> 

## Data privacy

The original research data is **not included** in this repository.

The research version used personal conversational language samples.

Local inference and participant-specific vector collections were used to reduce external transmission of this data, and identifying information was removed during preprocessing.

Any example data in this public repository should not be interpreted as the complete study dataset.

## Future work

The most important next step is evaluation with people with aphasia and co-design with intended AAC users. Further development could investigate automated meaning-preservation safeguards, better separation of semantic content from linguistic style, relationship/context effects, more deterministic generation, deployment latency, user control over
personalisation, and integration into an accessible AAC interface.

## Project context

**MSc Computer Science (Conversion)**

Queen Mary University of London

**Project:** *Evaluating Retrieval-Based Personalised Phrase Generation Using a Local Language Model for Aphasia-Oriented AAC*

This project combines my background in Speech and Language Therapy with interests in NLP, human-centred AI and privacy-conscious health technology.

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