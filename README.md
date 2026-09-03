# Controversy and Group Certainty Jointly Shape Everyday Moral Judgments

Code and data for the paper.

Participants judged real Reddit *Am I the Asshole?* dilemmas, were shown one of
four signal conditions about how other people had judged the same dilemma, then
judged again. The analyses here ask when people **weakened** their position
(changed verdict, or held it with less confidence) and when they
**strengthened** it.

| | |
| --- | --- |
| Responses | 4318 |
| Participants | 2159 |
| Reddit dilemmas | 135 |
| Conditions | control · controversy · certainty · controversy + certainty |

---

## Quick start

```bash
git clone https://github.com/ZiyuChen0410/Controversy-and-Group-Certainty.git
cd Controversy-and-Group-Certainty

pip install -r requirements.txt
python -m spacy download en_core_web_sm        # fig1 only

jupyter lab                                    # start from THIS directory
```

**Start Jupyter from the repository root.** Every notebook reads data through
relative `./survey_data/...` paths; the first cell checks this and fails with a
readable message rather than a bare `FileNotFoundError`.

The R notebook needs one extra step:

```bash
R -e 'install.packages("IRkernel", repos="https://cloud.r-project.org"); IRkernel::installspec()'
```

Its first cell installs any missing CRAN packages itself.

Developed on Python 3.10 and R 4.4.1.

---

## Notebooks

Run each top to bottom (Restart & Run All). Figures are written to `figures/`.

| Notebook | Language | Panels | Runtime |
| --- | --- | --- | --- |
| `fig1.ipynb` | Python | 1b, 1c | < 1 min |
| `fig2.ipynb` | Python | 2a, 2b | < 1 min |
| `fig3be.ipynb` | Python | 3b, 3e | ~1 min |
| `fig3acdf.ipynb` | R | 3a, 3c, 3d, 3f | **~30 min** |
| `fig4.ipynb` | Python | 4 | ~1 min |

`fig3acdf.ipynb` is slow because it runs ten-fold cross-validated model
selection across three model families before plotting. The rendered figures are
committed, so you do not need to run it just to see the output.

### Figure 3 panel map

Figure 3 has six panels, split across two notebooks by language:

| Panel | File | Notebook |
| --- | --- | --- |
| 3a | `fig3a_forest_weakening.png` | `fig3acdf` |
| 3b | `fig3b_weakening_quadrant.png` | `fig3be` |
| 3c | `fig3c_weakening_vs_disagreement.png` | `fig3acdf` |
| 3d | `fig3d_forest_strengthening.png` | `fig3acdf` |
| 3e | `fig3e_strengthening_quadrant.png` | `fig3be` |
| 3f | `fig3f_strengthening_vs_disagreement.png` | `fig3acdf` |

---

## Data

```
survey_data/
├── survey_results/
│   ├── survey_results.xlsx          one row per response (4318 × 70)
│   └── deliberation_results.jsonl   LLM annotations of deliberation text (2744)
└── moral_moments_data/
    ├── 135_posts.csv                the Reddit dilemmas shown to participants
    └── 54827_comments_135_posts.csv comment-level judgment labels
```

| File | Read by |
| --- | --- |
| `survey_results/survey_results.xlsx` | fig2, fig3be, fig3acdf, fig4 |
| `survey_results/deliberation_results.jsonl` | fig4 |
| `moral_moments_data/135_posts.csv` | fig1, fig4 |
| `moral_moments_data/54827_comments_135_posts.csv` | fig1 |

### Key columns

| Column | Meaning |
| --- | --- |
| `participant_id` | pseudonym (`P0001`…); used only as a grouping key |
| `treatment` | `control` · `controversy` · `certainty` · `controversy_certainty` |
| `j`, `j_2` | verdict before / after the signal — `"YA"` or `"NA"` |
| `c`, `c_2` | confidence before / after, POMP-rescaled to [0, 1] |
| `disagree`, `inconf`, `outconf` | share disagreeing, in-group and out-group certainty |
| `Openness` … `EmotionalStability` | Big Five (TIPI), from items `Q19.2_1`–`Q19.2_10` |
| `Intuitive` … `Spontaneous` | decision styles (GDMS), from `Q18.2_1`–`Q18.2_15` |
| `IH` | intellectual humility, from `Q19.3_1`–`Q19.3_6` |
| `is_ai_text` | deliberation flagged as AI-generated (28 of 4318), excluded in fig4 |

The 31 raw scale items are included so the composites above can be checked and
re-scored rather than taken on trust.

> **`j` and `j_2` store the literal string `"NA"`** — a real category
> (*not-an-asshole*), not a missing value. `pandas.read_excel` coerces it to
> `NaN` by default, which is why the Python notebooks call `.fillna("NA")`
> immediately after loading. R's `readxl` keeps it as a string. **Do not
> round-trip the workbook through pandas**: writing it back stores a blank and
> silently breaks the R notebook. Pass `keep_default_na=False`, or edit a copy
> in place with `openpyxl`.

### Two tiers

This repository contains the **public tier**, which reproduces every figure and
every number in the paper. What it does not contain is the participant free
text and the Reddit comment bodies:

| | public (here) | restricted |
| --- | --- | --- |
| `participant_id` | pseudonym | pseudonym |
| deliberation text | placeholder | full |
| Reddit comment bodies | dropped | full |
| everything the code reads | **identical** | identical |

Neither tier contains recruitment identifiers. The identifier is only ever used
as a grouping key, so pseudonyms reproduce every result exactly.

The restricted tier exists only so that the raw text can be inspected. It is
available under a data use agreement — see **Data availability** below. Verified
by re-running all five notebooks against it: the results are identical to the
public tier, line for line.

---

## Reproducing the figures

Every figure in `figures/` was produced by the notebook in the table above,
from the data in this repository, with no manual editing.

The R notebook's model selection is seeded (`set.seed(42)` before every
fold assignment), so the selected covariates and reported log-likelihoods
reproduce exactly. Both outcomes select a plain `glm`:

* **weakening** — GDMS block + intellectual humility + sex + nationality
* **strengthening** — GDMS block + intellectual humility

The word cloud in `fig1b` is seeded too (`random_state=42`).

The R panels render in DejaVu Sans to match the Python ones; `fig3acdf`
resolves the font automatically and warns if it has to fall back, in which
case only the significance asterisks differ.

---

## Data availability

The public tier in this repository is released under CC BY 4.0 and is
sufficient to reproduce all published results.

The restricted tier — participant deliberation text and the full Reddit comment
corpus — is not openly licensed. It contains participant-authored free text
collected under a consent form that does not permit public release. Access for
editorial evaluation or research use is available on request under a data use
agreement; contact the corresponding author.

## Licence

Code is released under the MIT Licence (see `LICENSE`). The public data tier is
released under CC BY 4.0. The restricted tier is not openly licensed.

## Citation

If you use this code or data, please cite the paper:

> Chen, Z., Xie, L., Shin, M., Nguyen, T. D., Klein, C., Tan, C., Schuster, N.,
> Carroll, N. G., & Tran, A. Controversy and Group Certainty Jointly Shape
> Everyday Moral Judgments.

```bibtex
@article{chen_controversy_certainty,
  title   = {Controversy and Group Certainty Jointly Shape Everyday Moral Judgments},
  author  = {Chen, Ziyu and Xie, Lexing and Shin, Minjeong and
             Nguyen, Tuan Dung and Klein, Colin and Tan, Chenhao and
             Schuster, Nick and Carroll, Nicholas George and Tran, Alasdair},
  year    = {2026},
  note    = {Manuscript}
}
```

Journal, year and DOI will be added once the paper is published; the
`preferred-citation` block in `CITATION.cff` is ready to be filled in at that
point, after which GitHub's "Cite this repository" button will point at the
article rather than this repository.

**Authors**

| | |
| --- | --- |
| Ziyu Chen, Lexing Xie, Minjeong Shin, Colin Klein, Nicholas George Carroll, Alasdair Tran | The Australian National University, Australia |
| Tuan Dung Nguyen | University of Pennsylvania, United States |
| Chenhao Tan | University of Chicago, United States |
| Nick Schuster | University of Georgia, United States |
