# Experiment Report: Authorship Attribution with Naive Bayes

## 1. Task description

Authorship attribution is the task of automatically identifying the author of an anonymous or disputed text based on stylometric features. In this tutorial, a Naive Bayes classifier is trained on one text per author and asked to predict the author of an unseen test document.

The system supports two feature representations:

- **Word unigrams** — each unique word is a feature; the model learns how likely each word is for a given author.
- **Character n-grams** — overlapping sequences of n characters are features; these capture sub-word stylistic patterns such as punctuation habits and morphology.

Training data (one text per author):

| File | Author |
|------|--------|
| `data/training/pride.txt` | Austen |
| `data/training/alice.txt` | Carroll |
| `data/training/wind.txt` | Grahame |
| `data/training/frankenstein.txt` | Shelley |

Test documents:

| File | True author |
|------|-------------|
| `data/test/emma.txt` | Austen |
| `data/test/second-junglebook.txt` | Kipling |

Note: Kipling is not in the training set. The classifier will produce a wrong prediction for that file — this is expected and informative.

---

## 2. Hypothesis

Two hypotheses were tested:

**H1 (n-gram length):** As character n-gram length increases, the model will overfit to the specific vocabulary of the training book and generalise less well, particularly for authors not in the training set.

**H2 (unknown author):** For a test document whose author is not in the training set (second-junglebook.txt by Kipling), no feature type or n-gram length will reliably attribute it — but word unigrams are expected to produce the most confused ranking since function words are shared across all 19th-century English prose.

---

## 3. Experiments

All experiments were run with the default smoothing parameter `alpha = 0.1`.

The command pattern used:

```bash
uv run python attribution.py --words data/test/<file>
uv run python attribution.py --chars=<n> data/test/<file>
```

Parameters varied:

- Feature type: `words`, `chars`
- n-gram length (chars only): `3`, `5`, `7`
- Test file: `emma.txt`, `second-junglebook.txt`

---

## 4. Results

### 4.1 emma.txt (true author: Austen)

| Feature | n | Rank 1 (predicted) | Rank 2 | Rank 3 | Rank 4 | Correct? |
|---------|---|--------------------|--------|--------|--------|----------|
| words | — | | | | | |
| chars | 3 | | | | | |
| chars | 5 | | | | | |
| chars | 7 | | | | | |

_Observations:_

### 4.2 second-junglebook.txt (true author: Kipling, not in training set)

| Feature | n | Rank 1 (predicted) | Rank 2 | Rank 3 | Rank 4 |
|---------|---|--------------------|--------|--------|--------|
| words | — | | | | |
| chars | 3 | | | | |
| chars | 5 | | | | |
| chars | 7 | | | | |

_Observations:_

---

## 5. Discussion

### H1 — Effect of n-gram length

### H2 — Unknown author

### Observations on top features

---

## 6. Conclusion
