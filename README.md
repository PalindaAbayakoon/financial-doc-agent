# Financial Document Analysis Agent

CSC3173 Artificial Intelligence - Programming Assignment 02

An agent that answers financial-analysis questions about Apple Inc. using its
annual reports (Form 10-K) for fiscal years 2023, 2024 and 2025. It is built on
the same architecture as the Medical Document (WHO statistics) Agent shown in
class: document processing, semantic retrieval, tool nodes and a LangGraph
planner loop.

## Project layout

```
financial-doc-agent/
├── data/                      Apple 10-K filings (FY2023, FY2024, FY2025)
├── outputs/                   generated charts and the final PDF report
├── Financial_doc_Agent.ipynb  the agent notebook
└── requirements.txt
```

## Dataset

| File | Filing |
|------|--------|
| `data/apple_10k_2023.pdf` | Form 10-K, fiscal year ended 30 Sep 2023 |
| `data/apple_10k_2024.pdf` | Form 10-K, fiscal year ended 28 Sep 2024 |
| `data/apple_10k_2025.pdf` | Form 10-K, fiscal year ended 27 Sep 2025 |

## How to run (Google Colab)

1. Create the folder `MyDrive/Demo/AI/Finance_AI_Agent/data/` in Google Drive and
   upload the three PDFs from `data/`.
2. Open `Financial_doc_Agent.ipynb` in Colab and select a GPU runtime (T4 is enough).
3. Run all cells. Charts and `financial_analysis_report.pdf` are written to
   `Finance_AI_Agent/outputs/`.

The notebook also runs locally (it then uses the folder it is opened from), but
Qwen2.5-1.5B is slow on a CPU.

## Pipeline

| Stage | What it does |
|---|---|
| A. Document processing | PyMuPDF text per page; fiscal year read from the 10-K cover page; 250-word chunks with 50-word overlap, tagged with file, page and year |
| B. Semantic retrieval | `all-MiniLM-L6-v2` embeddings in a FAISS inner-product index, optional fiscal-year filter |
| C. Financial retrieval tool | finds the *net sales by category* / *by reportable segment* table with semantic search, then parses the rows and matches them to the column years; Qwen extraction as a checked fallback |
| D. Calculation tool | percentage change, absolute change, CAGR, contribution share, growth ranking (all in Python) |
| E. Report generation | answer sentence from the calculated values, Qwen analysis paragraph checked number by number, charts, PDF export |
| F. Agent | LangGraph: `parse_question` → `planner` ⇄ `retrieve` / `calculate` / `report` → `FINISH` |

## Agentic components

| Component | Medical Document Agent (class) | Financial Document Agent |
|---|---|---|
| Goal | Trend of a health indicator across WHO reports | Revenue questions (total, category, region) across Apple 10-Ks with exact figures and citations |
| State | question, indicator, context, values by year, calc result, report, next action, steps | adds task type, table, target years, figures per year, evidence pages, retrieval attempts, facts, planner log, iteration counter |
| Planner | Qwen picks RETRIEVE / CALCULATE / REPORT / FINISH | Qwen proposes, a rule guard validates against the state, iteration cap |
| Tools | FAISS retrieval, Qwen + regex extraction, percent change, Qwen report | year-filtered retrieval, table parser, calculator, checked report writer, charts, PDF |
| Nodes | planner, retrieve, calculate, report | parse_question, planner, retrieve, calculate, report |
| Routing / iteration | conditional edges from the planner, all nodes loop back | same; RETRIEVE repeats with a wider search for missing years |
| Termination | FINISH when a report exists | FINISH when a report exists or the iteration limit is hit |

A longer comparison, including the problems seen in the class run and how they
were handled, is at the top of the notebook.

## Test questions

| | Question | Answer (from the 10-K tables) |
|---|---|---|
| Q1 | How did the company's total revenue change between 2023 and 2025? | $383,285M → $416,161M, +$32,876M (+8.58%) |
| Q2 | Which product category contributed the most revenue in the latest year? | iPhone, $209,586M = 50.4% of FY2025 net sales |
| Q3 | Which geographic segment grew the fastest between 2023 and 2025, and how did each region's net sales change? | Japan +18.33%, Europe +17.75%, Rest of Asia Pacific +13.78%, Americas +9.72%, Greater China -11.28% |

## Limitations

- The financial retrieval tool understands the two net sales tables. Questions
  about other statements (margins, expenses, cash flow) would need more table specs.
- The number check stops invented figures, but it cannot tell whether a correct
  number is described correctly (for example a growth rate called a "share").
- A 1.5B model often ignores the planner rules; the guard keeps the agent on a
  valid path and every override is logged.
