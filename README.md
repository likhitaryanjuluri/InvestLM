# RiskAware-InvestLM-Lite

**A structured, risk-aware financial language model for SEC filing analysis and investment-oriented insights.**

RiskAware-InvestLM-Lite is a research project focused on adapting a large language model to understand financial disclosures and produce structured risk assessments. The paper describes instruction-tuning a LLaMA-8B/Finance-Llama-8B foundation model using curated financial instructions, especially market-risk disclosures from SEC 10-K filings (Item 7A), with parameter-efficient fine-tuning through Low-Rank Adaptation (LoRA).

## Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Training Configuration](#training-configuration)
- [Expected Output](#expected-output)
- [Planned Extensions](#planned-extensions)
- [Limitations and Responsible Use](#limitations-and-responsible-use)
- [Repository Structure](#repository-structure)
- [Reproducibility Notes](#reproducibility-notes)
- [Research Context](#research-context)

## Overview

Financial reports contain extensive information about market exposure, debt obligations, interest-rate changes, currency fluctuations, liquidity, and credit-related uncertainty. Manually reviewing these disclosures can be time-consuming.

RiskAware-InvestLM-Lite aims to help users analyze this information by converting financial text into concise, structured outputs. Its primary training source is **Item 7A — Quantitative and Qualitative Disclosures About Market Risk** in SEC 10-K filings. The paper also describes a compact set of general finance instructions to support questions about investment concepts, valuation, risk management, and market events.

The intended workflow accepts either a financial question or an excerpt from an SEC filing and returns a structured analysis grounded in the supplied text.

## Screenshots

### InvestLM Financial Risk Chat Dashboard

The interface lets a user submit a finance-related question and review the model's response in a chat-style dashboard.

![InvestLM Financial Risk Chat Dashboard](Screenshots/InvestLM%20Financial%20Risk%20Chat%20Dashboard.png)

### Project Architecture

The architecture diagram illustrates the research pipeline from financial data inputs and preprocessing through model adaptation, fine-tuning, inference, and output visualization.

![RiskAware-InvestLM-Lite Project Architecture](Screenshots/Project%20Architecture.jpeg)

> **Note:** The supplied architecture image includes a “LLaMA 65B” label, while the research paper describes an LLaMA-8B / Finance-Llama-8B foundation model. Confirm and align this label with the final implementation if the difference is unintended.

## Objectives

- Identify and summarize risk factors in financial disclosures.
- Classify risks into useful categories.
- Assign qualitative risk-severity levels such as **Low**, **Medium**, or **High**.
- Extract important financial metrics and dates mentioned in the source text.
- Explain the evidence behind the risk assessment.
- Support general finance questions and financial-text summarization.
- Explore efficient domain adaptation without full-model fine-tuning.

## Key Features

- **SEC Item 7A analysis:** Focuses on market-risk disclosures in SEC 10-K filings.
- **Structured risk assessment:** Organizes findings by severity, category, metrics, and explanation.
- **Risk categorization:** Covers categories discussed in the paper, including market, credit, liquidity, and operational risks.
- **Financial metric extraction:** Aims to surface relevant amounts, interest rates, maturity dates, debt balances, and related indicators when present in the source.
- **General finance Q&A:** Supports finance-related questions and summaries.
- **LoRA-based adaptation:** Uses trainable low-rank adapters to reduce the number of parameters updated during fine-tuning.
- **Interactive interface concept:** The paper describes a UI for entering questions or submitting filing excerpts and viewing organized responses.

## How It Works

### 1. Data collection

The core dataset is built from SEC 10-K Item 7A passages. These disclosures contain standardized information about financial-market exposures and related risks. A smaller collection of general finance instructions complements the regulatory material.

### 2. Data preprocessing

The described preprocessing pipeline:

1. Removes formatting noise such as unwanted line breaks, numbering artifacts, table remnants, and OCR irregularities.
2. Normalizes numerical expressions where needed to reduce ambiguity.
3. Splits long documents into semantically coherent passages that fit within the model's context limits.
4. Converts each passage into an instruction–response example.
5. Tokenizes the prepared examples with the LLaMA-8B SentencePiece tokenizer.

### 3. Instruction tuning

Each financial passage is paired with an instruction asking the model to analyze the text and produce structured information. The target response can include risk severity, risk categories, key metrics, and a concise explanation.

The paper describes using human annotation and model-assisted paraphrasing for supplementary finance instructions, with verification for correctness and consistency.

### 4. Parameter-efficient fine-tuning

The model is adapted using **Low-Rank Adaptation (LoRA)**. Rather than updating all base-model weights, LoRA adds trainable low-rank matrices to selected attention projection layers. This is intended to reduce training memory requirements and computational cost.

### 5. Inference

At inference time, the user supplies a finance question or a financial passage. The tuned model generates an organized response containing the relevant risk assessment, categories, metrics, and narrative explanation.

```text
Financial question or SEC filing excerpt
                 |
                 v
       Text preparation / input
                 |
                 v
   Finance-adapted LLaMA model + LoRA
                 |
                 v
       Structured financial analysis
                 |
                 +-- Risk severity
                 +-- Risk categories
                 +-- Relevant metrics and dates
                 +-- Evidence-based explanation
```

## Training Configuration

The paper reports the following LoRA and optimization settings:

| Setting | Value reported in the paper |
|---|---|
| Foundation model | LLaMA-8B / Finance-Llama-8B (the paper uses both names) |
| Adaptation method | LoRA (Low-Rank Adaptation) |
| LoRA rank (`r`) | 8 |
| LoRA alpha | 16 |
| LoRA dropout | 0.05 |
| Target projection layers | Primarily query and value projections |
| Optimizer | AdamW |
| Learning rate | `2e-4` |
| Precision | Mixed precision |
| Validation | A small validation split is used to monitor loss and overfitting |

The paper does not specify all details needed to reproduce training, such as the exact dataset size and split, epoch count, batch size, sequence length, hardware, software versions, or complete training scripts.

## Expected Output

The system is designed to return an interpretable report rather than an unstructured answer. A conceptual output format is shown below; it is illustrative and is **not** a measured result or a verbatim model response.

```text
Risk Assessment
---------------
Risk Severity: [Low / Medium / High]
Risk Categories: [Market / Credit / Liquidity / Operational / Other]

Key Metrics
-----------
- [Metric or amount stated in the source]
- [Relevant rate, date, debt balance, or exposure stated in the source]

Analysis
--------
[Brief explanation of the risk factors, grounded in the supplied passage.]

Source Context
--------------
[Relevant evidence or excerpt supporting the assessment.]
```
![InvestLM Financial Risk Chat Dashboard](Screenshots/InvestLM%20Financial%20Risk%20Chat%20Dashboard.png)


The actual output fields may vary depending on the input and implementation.

## Planned Extensions

The paper identifies two intended extensions to make the system more useful in dynamic market settings:

- **Live financial data pipeline:** Connect external APIs to provide current stock prices, financial news, and financial ratios.
- **Interactive web chatbot:** Provide a user-friendly web interface rather than relying only on a command-line workflow.

These are described as planned upgrades in the paper. Do not assume they are implemented or operational unless the repository contains the corresponding code and configuration.

## Limitations and Responsible Use

- This is a research-oriented decision-support concept, not a substitute for professional financial, legal, or investment advice.
- Model responses can be incomplete, inaccurate, or incorrectly interpret financial disclosures. Verify important figures and conclusions against the original filing.
- Risk labels are qualitative assessments and should not be treated as calibrated probabilities or guarantees.
- The paper emphasizes grounding answers in source text, but this should not be interpreted as a guarantee that hallucinations never occur.
- The paper does not provide a complete reproducibility package or enough implementation detail to reproduce every reported experiment independently.
- Live market information is time-sensitive. The planned external data pipeline should not be assumed to exist unless it is implemented and configured in the project.
- Do not upload confidential, personally identifiable, or otherwise restricted financial information to an unapproved deployment.

## Repository Structure

The research paper does not specify a definitive source-code layout. If you are building the implementation, a possible organization is:

```text
RiskAware-InvestLM-Lite/
├── README.md
├── Screenshots/
│   ├── InvestLM Financial Risk Chat Dashboard.png
│   └── Project Architecture.jpeg
├── data/                 # Prepared datasets (do not commit restricted data)
├── notebooks/            # Experiments and analysis
├── src/
│   ├── preprocessing/    # Filing cleaning and segmentation
│   ├── training/          # LoRA fine-tuning pipeline
│   ├── inference/         # Model inference and response formatting
│   └── app/               # Optional web interface
├── configs/               # Training and application configuration
├── tests/
├── requirements.txt
└── .gitignore
```

This is a suggested layout, not a claim about files already present in the repository. Adjust it to match your actual project.

## Reproducibility Notes

Before publishing the implementation, consider documenting:

- The exact base-model identifier and its license/access requirements.
- Dataset sources, preprocessing rules, data splits, and any licensing or redistribution restrictions.
- Python, CUDA, PyTorch, Transformers, PEFT, and other dependency versions.
- Complete training hyperparameters, hardware, random seeds, and evaluation procedures.
- How LoRA adapters or checkpoints are obtained and loaded.
- How to launch the inference pipeline or UI, including required environment variables.
- Evaluation datasets, baselines, quantitative metrics, and known failure cases.

The paper discusses qualitative findings, including structured risk extraction and general finance question answering, but this README does not invent numerical benchmark scores that are not reported in the available paper text.

## Research Context

The paper situates this work in relation to finance-focused NLP and language-model research, including Financial PhraseBank, FiQA, FinBERT, LoRA, FinQA, ECTSum, FinRL, Alpaca, AlpacaFarm, BloombergGPT, FinMA/Pixiu, and FinGPT.

## Disclaimer

This repository describes an academic research project. Outputs are for research and informational purposes only and should not be considered personalized investment recommendations. Always independently verify financial information and consult a qualified professional where appropriate.
