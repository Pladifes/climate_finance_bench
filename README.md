# Climate Finance Bench - Retrieval‑Augmented QA over Corporate Climate Disclosures

**Climate Finance Bench** is an open benchmark and reference implementation for answering analyst‑style questions about corporate sustainability reports with *retrieval‑augmented generation* (RAG).  
It comes with:

* A curated dataset of 33 recent climate or ESG reports covering all 11 GICS sectors and **330 expert‑validated question–answer pairs**.
* Two ready‑to‑run Jupyter notebooks:
  * **`notebooks/vector_store_generation/sustainability-reports-vector-store-generation.ipynb`** – builds FAISS + BM25 indexes from the PDF reports with either **Docling** (HTML‑aware) or **LangChain** loaders.
  * **`notebooks/RAG_pipeline/sustainability-reports-rag-pipeline.ipynb`** – evaluates multiple RAG configurations (minimal vs. hybrid + rerank) across open‑source and API LLMs (Llama 3, Mistral‑NeMo, GPT‑4o, Claude 3.5, DeepSeek R1, Qwen 2.5).
* **Carbon‑footprint tracking** via *CodeCarbon* and *EcoLogits*.
* Reproducible experiments tested on **Kaggle → GCP P100 (16 GB GPU)** but runnable on any Linux machine with ≥16 GB GPU VRAM.

---

## Repository layout

```text
.
├── data/
│   ├── Company Reports/
│   │   ├── SP500/
│   │   │   └── Apple/
│   │   │       └── Apple_Environmental_Progress_Report_2024.pdf
│   │   └── … (other indices such as CAC40/, FTSE/, Other/)
│   └── Benchmark/
│       ├── Climate Finance Bench – Dataset.json
│       ├── Climate Finance Bench – Dataset.xlsx
│       └── …
└── notebooks/
    ├── vector_store_generation/
    │   ├── sustainability-reports-vector-store-generation.ipynb
    │   └── requirements.txt
    └── RAG_pipeline/
        ├── sustainability-reports-rag-pipeline.ipynb
        └── requirements.txt
```

The notebooks generate additional artefacts in the working directory:

```
FAISS_DB/
└── {docling, langchain}/
    ├── <Index‑name>/<Company>/   ← per‑company FAISS + BM25 stores
    └── GLOBAL_DB/                ← cross‑company store (shared)
```

---

## Quick‑start (Kaggle recommended)

1. **Fork the public Kaggle notebook**  
   Create a new Kaggle notebook (GPU → P100). Add this repository and the PDF dataset as *Dataset* inputs.

2. **Install dependencies**

   ```bash
   %pip install -r notebooks/vector_store_generation/requirements.txt
   %pip install -r notebooks/RAG_pipeline/requirements.txt
   ```

3. **Add API keys (optional but recommended)**  
   In *Add → Secrets* configure:
   - `HF_TOKEN` for gated Hugging Face models (e.g. Llama 3).
   - `OPENAI_API_KEY` if you plan to query GPT‑4o.
   - `ANTHROPIC_API_KEY` for Claude 3.5 Sonnet.
   - `NEBIUS_API_KEY` for DeepSeek R1 or Qwen 2.5 via Nebius.
   - `QUEPASA_API_TOKEN` if you want to run the *QuePasa* ingestion route.

4. **Build the vector stores**

   Open `sustainability-reports-vector-store-generation.ipynb` and run **all cells**.  
   *Parameters to note*
   - `CHUNK_SIZE` (default = 2048 tokens)  
   - `INDEXES` tuple if you added more sub‑indices under `data/Company Reports`.

5. **Run the RAG pipeline**

   Open `sustainability-reports-rag-pipeline.ipynb` and edit the **`main()`** call at the bottom:

   ```python
   main(
       vector_store_style="langchain",   # or "docling"
       store_scope="single",             # "single" (per‑company) or "shared"
       model_choice="openai",            # see options below
       retrieval_mode="hybrid",          # "minimal" or "hybrid"
       do_rerank=True                    # needs CrossEncoder weights
   )
   ```

   | `model_choice`        | Notes                                                                   |
   |-----------------------|-------------------------------------------------------------------------|
   | `nemo_4bit`           | Mistral‑NeMo‑12B quantised → fits in 16 GB GPU                          |
   | `llama_4bit`          | Llama 3.1‑8B quantised (preferred OSS)                                  |
   | `llama_full`          | Llama 3.1‑8B full precision (needs ≈24 GB)                               |
   | `openai`              | GPT‑4o via API                                                          |
   | `claude`              | Claude 3.5 Sonnet via API                                               |
   | `nebius_deepseek`     | DeepSeek R1 via Nebius                                                  |
   | `nebius_qwen`         | Qwen 2.5‑72B via Nebius                                                 |
   | `QuePasa`             | End‑to‑end ingestion & QA on <https://quepasa.ai>                       |

   The notebook appends answers and used context to **`Climate Finance Bench – Dataset_with_answers.xlsx`**.

---

## Citation

If you use **Climate Finance Bench** in academic work, please cite:

```bibtex
@misc{mankour2025cfb,
  title        = {Climate Finance Bench},
  author       = {R. Mankour, Y. Chafai, H. Saleh, G. Ben Hassine, T. Barreau and P. Tankov},
  year         = {2025},
  howpublished = {\\url{https://github.com/Pladifes/climate_finance_bench}},
  note         = {Working paper, PLADIFES}
}
```

---

## License

Code and benchmark annotations © 2025 PLADIFES, released under **Creative Commons BY‑NC‑SA 4.0**.  
Commercial use requires written permission.
 
> The PDF reports remain the property of their respective issuers and are redistributed here solely for research and non‑commercial use.

---

## Acknowledgements

We thank the ESG analysts at the *Institut Louis Bachelier* Labs and **Yuri Vorontosov** (QuePasa) for compute resources.