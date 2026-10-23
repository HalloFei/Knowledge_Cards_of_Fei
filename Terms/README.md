# AI/ML Knowledge Base – Folder Structure

## Why three folders?

We started with four categories – `Concept`, `Method`, `Model`, `Tool` – but found high coupling (38% of terms belonged to ≥2 categories on a 50‑term test). The main overlaps were:

- **Concept ↔ Method** (e.g., “Attention”, “Data Augmentation”)
- **Method ↔ Model** (e.g., “RAG”, “MoE”, “Q‑Learning”)

To remove ambiguity, we merged **Method** and **Model** into a single class called **`Method`**.  
This reduces overlapping placements and keeps only one well‑defined boundary: *Concept vs. Method* (which is acceptable, ~26% overlap in practice).

The result is a clean, three‑folder system that is easy to use for both manual and automated term archiving.

---

## Folder definitions

| Folder     | Definition                                                                 | Examples                                                                 |
|------------|-----------------------------------------------------------------------------|--------------------------------------------------------------------------|
| **Concept** | Abstract idea, paradigm, theoretical construct, or task definition. No procedural or implementable specification. | `Agent`, `Loss Function`, `Markov Decision Process`, `Attention (as idea)`, `Embedding (as idea)` |
| **Method**  | Concrete algorithmic procedure, learning algorithm, model architecture, or any hybrid that can be implemented in code. | `Backpropagation`, `Transformer`, `RAG`, `Q‑Learning`, `k‑Means`, `PPO`, `Dropout` |
| **Tool**    | Software library, framework, SDK, programming language, or platform.       | `PyTorch`, `Rasa Open Source`, `PDDL`, `Docker`, `Hugging Face Transformers` |

---

## Edge cases & guidelines

- **If a term fits both `Concept` and `Method`** – prefer `Method` if it describes a concrete technique that can be coded (e.g., “Convolution” → `Method`). Use `Concept` only for purely non‑procedural ideas (e.g., “Loss Landscape” → `Concept`).

- **If a term is both a `Method` and a `Tool`** – assign `Tool` (e.g., `PDDL` is a planning language and a method – put it in `Tool` for consistency). The vast majority of tools are unambiguous.

- **Naming of stored files** – use lowercase with hyphens (e.g., `self-attention.md`).  
  Storage path: `KB/{folder}/{term}.md`

---

## Example usage

For the term `RLHF`:

```json
{
  "term": "RLHF",
  "full_form": "Reinforcement Learning from Human Feedback",
  "folder": "Method",
  "definition": "using human preferences as a reward signal to align language models",
  "tags": ["rl", "llm", "alignment"],
  "storage_path": "KB/Method/Reinforcement Learning from Human Feedback RLHF.md"
}