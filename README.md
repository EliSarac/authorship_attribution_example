# Authorship Attribution

This repository is forked from [ml-for-nlp/authorship-attribution](https://github.com/ml-for-nlp/authorship-attribution) as part of a practical tutorial on the Naive Bayes algorithm applied to authorship attribution.

The original code has been restructured following Python best practices (src layout, uv, pyproject.toml). The exercises and write-up from the original tutorial are completed in [report.md](report.md).

## What this project does

A Naive Bayes text classifier that identifies the likely author of an unknown text. The classifier is trained on one work per author (Austen, Carroll, Grahame, Shelley) and classifies a test document by computing the highest log-probability score across all classes.

## Project structure

```
authorship_attribution_example/
├── src/
│   └── authorship_attribution/
│       ├── __init__.py
│       └── utils.py          # feature extraction and document loading
├── data/
│   ├── training/             
│   └── test/                 
├── attribution.py            # CLI entry point
├── requirements.txt
└── pyproject.toml
```

Each text file must begin with a line of the form `#Author: <name>` followed by the document body.

## Requirements

- Python 3.9 or later
- [uv](https://github.com/astral-sh/uv) (recommended) or pip

## Installation

Using uv:

```bash
uv sync
```

Using pip:

```bash
pip install -r requirements.txt
```

## Usage

Classify a test document using word unigrams:

```bash
uv run attribution.py --words data/test/emma.txt
```

Classify using character n-grams of length 3:

```bash
uv run attribution.py --chars=3 data/test/emma.txt
```

The script prints the prior probability for each training class, then ranks all classes by their log-likelihood score for the test document.

## Training data

| File | Author |
|------|--------|
| `data/training/alice.txt` | Carroll |
| `data/training/frankenstein.txt` | Shelley |
| `data/training/pride.txt` | Austen |
| `data/training/wind.txt` | Grahame |

## Algorithm

Training applies Laplace-smoothed Naive Bayes. For each class c:

- Prior: P(c) = (number of training documents for c) / (total training documents)
- Likelihood: P(t|c) = (count(t, c) + alpha) / (total tokens in c * (1 + alpha))

At inference time the class with the highest sum of log-priors and log-likelihoods is the predicted author.

