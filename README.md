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
