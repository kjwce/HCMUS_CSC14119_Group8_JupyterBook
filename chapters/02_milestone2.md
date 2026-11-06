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


## 2. Results and Performance Evaluation

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


## 5.1 Parsing Coverage and Version-Level Success Rate


```{code-cell} ipython3
:tags: [remove-input]
display(T2["coverage"])
m2.plot_coverage(S2); plt.show()
```

## 5.2 Element and Structural Distribution

```{code-cell} ipython3
:tags: [remove-input]
m2.plot_kinds(S2); plt.show()
m2.plot_kind_mix(S2); plt.show()
m2.plot_depth(S2); plt.show()
```

## 5.3 Deduplication Efficiency

```{code-cell} ipython3
:tags: [remove-input]
display(T2["dedup"])
m2.plot_dedup(S2); plt.show()
```

## 5.5 Parser Performance

```{code-cell} ipython3
:tags: [remove-input]
display(T2["performance"])
m2.plot_performance(S2); plt.show()
```