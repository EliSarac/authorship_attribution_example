# Experiment Report: Authorship Attribution with Naive Bayes

## 1. Task description

Authorship attribution is the task of automatically identifying the author of an anonymous or disputed text based on stylometric features. In this set of experiments, a Naive Bayes classifier is trained on one text per author and asked to predict the author of an unseen test document.

The training set consists of one text per author: Jane Austen (*Pride and Prejudice*), Lewis Carroll (*Alice in Wonderland*), Kenneth Grahame (*The Wind in the Willows*), and Mary Shelley (*Frankenstein*). The test text is Austen's *Emma*. The system supports two feature types: word unigrams (tokenised by whitespace) and character n-grams (overlapping windows of *n* characters). No lowercasing, stemming, or stop-word removal is applied.

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
uv run attribution.py --words data/test/<file>
uv run attribution.py --chars=<n> data/test/<file>
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
| words | — | Austen (-979,739) | Shelley (-1,032,609) | Carroll (-1,047,842) | Grahame (-1,060,600) | ✓ |
| chars | 3 | Austen (-6,399,355) | Shelley (-6,499,293) | Grahame (-6,525,125) | Carroll (-6,642,868) | ✓ |
| chars | 5 | Austen (-7,635,218) | Shelley (-7,915,519) | Grahame (-8,023,263) | Carroll (-8,133,883) | ✓ |
| chars | 7 | Austen (-7,089,338) | Carroll (-7,396,077) | Shelley (-7,474,673) | Grahame (-7,591,386) | ✓ |
 
_Observations:_
 
All four configurations correctly identify Austen. Confidence (gap between rank 1 and rank 2) increases from n = 3 (~100k) to n = 5 (~280k), then remains high at n = 7 (~307k). However, the nature of the top features changes at n = 7: Austen's highest-ranked features become "Elizabe", "lizabet", "izabeth" (character names from *Pride and Prejudice*), and Carroll's include " Alice ", "said th". The model is beginning to memorise book-specific content rather than learning authorial style. The runner-up shift from Shelley (n = 3, 5) to Carroll (n = 7) further confirms this: at shorter n-grams, the ranking reflects genuine stylistic similarity; at longer n-grams, it reflects book-specific feature overlap.
 
### 4.2 second-junglebook.txt (true author: Kipling, not in training set)
 
| Feature | n | Rank 1 (predicted) | Rank 2 | Rank 3 | Rank 4 |
|---------|---|--------------------|--------|--------|--------|
| words | — | Shelley (-1,043) | Carroll (-1,055) | Grahame (-1,065) | Austen (-1,070) |
| chars | 3 | Grahame (-97,214) | Shelley (-99,295) | Carroll (-99,513) | Austen (-99,849) |
| chars | 5 | Shelley (-42,249) | Grahame (-43,125) | Austen (-43,918) | Carroll (-44,031) |
| chars | 7 | Shelley (-13,276) | Carroll (-13,318) | Grahame (-13,540) | Austen (-13,696) |
 
_Observations:_
 
No configuration identifies the correct author (Kipling is absent from the training set). The predicted author is inconsistent: Shelley wins with words, chars 5, and chars 7; Grahame wins with chars 3. The confidence gaps are negligible compared to the Emma runs: ~12 for words, ~2,081 for chars 3, ~877 for chars 5, and ~42 for chars 7. The word unigram gap of 12 log units is the smallest across all experiments, confirming that function word distributions provide almost no basis for discrimination when the true author is out of set.
 
---
 
## 5. Discussion
 
### H1 — Effect of n-gram length
 
H1 is confirmed. As n-gram length increases, the top features shift from generic orthographic patterns (n = 3: " th", "he ", "ing") to word-boundary patterns (n = 5: " the ", " her ", " my ") to book-specific proper nouns (n = 7: "Elizabe", " Alice ", "the Rat"). For the known-author test, classification remains correct across all settings, but the runner-up shift from Shelley to Carroll at n = 7 indicates the model is relying on content rather than style. Additional experiments at n = 10 (not in the main tables) showed the gap collapsing to ~2.2k — the classifier barely succeeded — with top features like "Elizabeth " and "Mr. Darcy" dominating entirely.
 
For the unknown-author test, longer n-grams produce smaller confidence gaps (chars 3: ~2,081 vs. chars 7: ~42), confirming that overfitted features generalise poorly to unseen texts.
 
### H2 — Unknown author
 
H2 is confirmed. No feature type reliably attributes the Kipling text. The predicted author changes across configurations (Shelley in 3/4 runs, Grahame in 1/4), and the confidence gaps are orders of magnitude smaller than in the known-author case. The word unigram setting produced the most confused ranking, with a gap of just ~12 between Shelley and Carroll — consistent with the hypothesis that function words are shared across all 19th-century English prose and provide minimal discriminative signal when the true author is absent.
 
### Observations on top features
 
Each author has a distinctive fingerprint within the shared set of high-frequency function words. Austen's top features include "her" (third-person female narration); Shelley's include "I" and "my" (first-person narration); Carroll's include "she" and "said" (dialogue-heavy prose with a female protagonist); Grahame's include "he" and "his" (male animal characters). These differences are most visible at the word level and at medium character n-gram lengths (n = 3–5). At n = 7, they are obscured by book-specific content.
 
---
 
## 6. Conclusion
 
Both hypotheses were confirmed. Increasing n-gram length causes the model to overfit to training-book vocabulary, as evidenced by the shift in top features from stylistic patterns to proper nouns and the degradation of confidence at very long n-grams. When the true author is absent from the training set, the classifier produces unstable, low-confidence predictions regardless of feature type, with word unigrams showing the least discrimination — consistent with the shared function-word distributions of 19th-century English prose. These findings highlight the importance of feature granularity in Naive Bayes text classification and the fundamental limitation of closed-set classifiers when applied to out-of-distribution inputs.
