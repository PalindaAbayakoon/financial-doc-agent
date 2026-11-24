# Financial Document Analysis Agent

CSC3173 Artificial Intelligence - Programming Assignment 02

An agent that answers questions about Apple's revenue using the 10-K annual reports
for FY2023, FY2024 and FY2025. It is based on the Medical Document Agent from class.

## Files

```
financial-doc-agent/
├── data/                      Apple 10-K reports (2023, 2024, 2025)
├── outputs/                   charts and the PDF report
├── Financial_doc_Agent.ipynb  the notebook
└── requirements.txt
```

## Dataset

| File | Report |
|------|--------|
| `data/apple_10k_2023.pdf` | 10-K, fiscal year ended 30 Sep 2023 |
| `data/apple_10k_2024.pdf` | 10-K, fiscal year ended 28 Sep 2024 |
| `data/apple_10k_2025.pdf` | 10-K, fiscal year ended 27 Sep 2025 |

## How to run

1. Upload the three PDFs to `MyDrive/Demo/AI/Finance_AI_Agent/data/` in Google Drive.
2. Open the notebook in Colab and use a GPU runtime (T4).
3. Run all cells. The charts and `financial_analysis_report.pdf` are saved in
   `Finance_AI_Agent/outputs/`.

## How it works

- **A. Document processing**: text per page with PyMuPDF, fiscal year from the cover page, 250-word chunks
- **B. Semantic retrieval**: `all-MiniLM-L6-v2` embeddings + FAISS, can search one year only
- **C. Financial retrieval tool**: finds the net sales table (by category or by region) and reads the values
- **D. Calculation tool**: percentage change, absolute change, CAGR, share of total, growth ranking (Python)
- **E. Report generation**: answer, Qwen analysis (checked against the numbers), chart, PDF
- **F. Agent**: LangGraph loop `parse_question -> planner -> retrieve / calculate / report -> FINISH`

## Agentic components

| Component | Medical agent (class) | Financial agent |
|---|---|---|
| Goal | Health indicator trends from WHO reports | Revenue questions from Apple 10-Ks |
| State | question, indicator, context, values by year, result, report | + task, table, years, figures, source pages, planner log |
| Planner | Qwen picks the next action | Qwen picks, a rule check fixes wrong choices |
| Tools | retrieval, extraction, percent change, report | retrieval, table parser, calculator, report + PDF |
| Nodes | planner, retrieve, calculate, report | + parse_question |
| Routing / iteration | planner -> node -> planner | same, retrieve can retry a missing year |
| Termination | FINISH when the report is done | same, or after 10 iterations |

More detail is in section 0 of the notebook.

## Test questions

| | Question | Answer |
|---|---|---|
| Q1 | How did the company's total revenue change between 2023 and 2025? | $383,285M -> $416,161M, +$32,876M (+8.58%) |
| Q2 | Which product category contributed the most revenue in the latest year? | iPhone, $209,586M (50.4% of FY2025 sales) |
| Q3 | Which geographic segment grew the fastest between 2023 and 2025, and how did each region's net sales change? | Japan +18.33%, Europe +17.75%, Rest of Asia Pacific +13.78%, Americas +9.72%, Greater China -11.28% |

## Limitations

- Only the two net sales tables are supported, not margins, expenses or cash flow.
- The check on Qwen's analysis catches wrong numbers and the main statements, but not every wording mistake.
  The one-line answer is always built from the calculated values.
