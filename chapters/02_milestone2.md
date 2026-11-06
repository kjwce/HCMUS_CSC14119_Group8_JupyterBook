---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Milestone 2: Parsing 

## 2. Results and Performance Evaluation

### 2.1 Parsing Coverage and Version-Level Success Rate

We report coverage at two levels. A **version** counts as *parsed* when the parser found a compilation root, flattened the `\input`/`\include` path and built a hierarchy tree. A version without usable TeX source is kept as an *empty* record rather than skipped, as the milestone requires, so the statistics reflect the real corpus. A **paper** is *fully parsed* when all its versions were parsed, *partially parsed* when only some were, and *not parsed* when none were.

```{code-cell} ipython3
:tags: [remove-input]
import sys
from pathlib import Path
for _p in (Path.cwd(), Path.cwd().parent):
    if (_p / "analysis").is_dir() and str(_p / "analysis") not in sys.path:
        sys.path.insert(0, str(_p / "analysis"))

import warnings; warnings.filterwarnings("ignore")
import matplotlib.pyplot as plt
from IPython.display import display
import m2_analysis as m2

S2 = m2.load_all("../parsing_stats")
T2 = m2.tables(S2)
```

```{code-cell} ipython3
:tags: [remove-input]
display(T2["coverage"])
m2.plot_coverage(S2); plt.show()
```
All 25,000 publications were processed without a parser failure (0 errors). At version level, 32,652 of 34,373 versions (94.99%) were parsed and 1,721 (5.01%) were empty. The empty share is stable across version indices, at roughly 5% for v1 and 3–4% for later versions, so late versions are not harder to parse than early ones. At paper level, about 16.7k of the 17.8k single-version
papers were parsed. Among papers with several versions, most are fully parsed (for example 5.4k of 5.7k two-version papers), and only a small number are partially parsed (97 for two versions, 32 for three).

**Interpretation.** The 5% gap does not come from parsing errors. It comes from versions that Milestone 1 delivered without a usable TeX source. These are recorded as empty, which includes 3 versions whose source folder is missing. The paper-level panel supports this: a paper's versions tend to be empty together, so for multi-version papers "no version parsed" is much more common
than "partially parsed". The parse success rate of 100% only says that no paper crashed the parser, so version-level coverage is the more informative metric.

```{note}
Both panels use a logarithmic y-axis so that small categories remain visible; the exact counts are in the table above.
```

### 2.2 Element and Structural Distribution

```{code-cell} ipython3
:tags: [remove-input]
m2.plot_kinds(S2); plt.show()
```
Before deduplication the parser produced 12,755,964 elements from 32,652 parsed versions. Sentences dominate: 10.1M elements (79.5%), about 310 per parsed version, followed by block formulas (1.21M, 9.5%). Together with figures and tables (254k, 2.0%), the three leaf kinds required by the milestone make up 91.0% of all elements. The remaining 9.0% (1.15M elements) are structural nodes: subsections (301k), list items (287k), sections (223k), paragraph-level headings (115k), lists (89k) and sub-subsections (70k). There is exactly one `document` root per parsed version (32,652), and 30,484 versions (93.4%) have an abstract.

The kind mix reflects typical scientific papers, with about 7 sections and 9 subsections per version, and each list contains about 3 items on average. Rare levels such as `chapter` (634), `part` (132) and `subparagraph` (807) show that the parser adapts the hierarchy to documents with a different heading structure, such as theses, books and heavily nested articles. Tables are counted as figures, as the milestone specifies. The x-axis is logarithmic because sentences outnumber the rarest kinds by five orders of magnitude; the exact counts are in the table above.

```{code-cell} ipython3
:tags: [remove-input]
m2.plot_kind_mix(S2); plt.show()
```
Grouping the 23,697 parsed papers by their maximum hierarchy depth shows how the element mix changes with structure. Sentences remain the majority in every group but decline steadily from 84% for papers of depth 2 or less to 77% for papers of depth 6 or more. Block formulas stay roughly constant at about 10%. The loss is taken up by structural nodes: list items, lists, paragraphs, subsections and sub-subsections occupy a visibly larger share in deeper papers. This is expected, since a deeper tree requires more headings and list levels, and it confirms that
the parser assigns levels from the document's own structure.


```{code-cell} ipython3
:tags: [remove-input]
m2.plot_depth(S2); plt.show()
```

Hierarchy depth adapts to each publication. Depth 0 holds exactly one document
root per parsed version (32,652). Depths 2 and 3, typically sections with their
subsections or sentences, hold 78% of all elements, and depths 0 to 4 hold 96%.
Elements below depth 6 are rare (0.2%). At paper level the maximum depth peaks at
4 (7.8k papers) and reaches 12 for a single paper with heavily nested lists.
Both panels use a logarithmic axis, so exact counts are in the table.

### 2.3 Deduplication Efficiency

```{code-cell} ipython3
:tags: [remove-input]
display(T2["dedup"])
m2.plot_dedup(S2); plt.show()
```

Deduplication removes 2.81M of 12.76M elements (22.0%) and 353k of 1.34M BibTeX entries (26.4%).

- **All the gain comes from multi-version papers.** The 16,782 single-version papers (71% of parsed papers) lose no elements, so the 2.81M savings come entirely from the 6,915 papers with several versions, whose element count falls by 43.7%.
- **Reduction grows with the number of versions**, from 38.0% for two versions to 63.4% for five or more, because each new version largely repeats earlier text. Only exact matches after cleanup are merged, so these figures are conservative.
- **References deduplicate even better** (45.7% to 74.4%) and come close to the theoretical ceiling of 1 − 1/v (50% for two versions): bibliographies change little between versions. The 0.5% for single-version papers comes from duplicate entries within one bibliography.

### 2.5 Parser Performance

```{code-cell} ipython3
:tags: [remove-input]
display(T2["performance"])
m2.plot_performance(S2); plt.show()
```