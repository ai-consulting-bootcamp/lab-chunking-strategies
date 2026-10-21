# Different Ways to Chunk Podcast and PDF Documents

## What it does

This lab compares document-chunking strategies for a podcast transcript and a PDF:

- Fixed-size character chunking with different chunk sizes and overlaps
- Recursive character chunking
- Token-based chunking
- Optional semantic chunking
- Chunk-size, boundary, and strategy comparisons with tables and plots

The notebook also evaluates whether chunks preserve sentence boundaries and records trade-offs for retrieval and context quality.

## Install

1. Use Python with Jupyter Notebook or Google Colab.
2. Open `chunking_strategies.ipynb`.
3. Run the first setup cell to install the notebook dependencies:

   ```bash
   pip install -q langchain-text-splitters pypdf openai tiktoken sentence-transformers matplotlib pandas python-dotenv
   ```

4. Add any required API keys to your environment before running cells that use external services.

## Usage

1. Open `chunking_strategies.ipynb`.
2. Load the included podcast and PDF source materials, or update the source paths in the notebook.
3. Run the cells in order to create fixed-size, recursive, and token-based chunks.
4. Compare chunk counts, sizes, overlaps, boundary quality, and visualizations.
5. Record recommendations for podcast transcripts and PDF documents.
6. Follow `instructions.md` to complete the lab proof and submission workflow.

Supporting files include `transcript.txt`, `_pieces/`, the source PDF, `instructions.md`, and `rubric.md`.

## License

No license is specified for this project.
