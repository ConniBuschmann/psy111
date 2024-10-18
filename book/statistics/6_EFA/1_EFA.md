---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# X.1 EFA in Python

To compute an EFA in Python we will use the `factor_analyzer` package. This package offers not only EFA functionality but also enables the use of confirmatory factor analysis (CFA).

### Example dataset

To demonstrate EFA, the following code chunk creates a simulated dataset. 9 variables (items) are created. The items are the following:

- Q1: In the past two weeks, how often have you felt down, depressed, or hopeless?
- Q2: How often have you lost interest or pleasure in activities you used to enjoy?
- Q3: How often have you felt tired or had little energy over the last two weeks?
- Q4: How often have you felt nervous, anxious, or on edge in the past two weeks?
- Q5: In the past two weeks, how often have you found it difficult to relax?
- Q6: How often have you been easily annoyed or irritable in the past two weeks?
- Q7: How satisfied are you with your life as a whole?
- Q8: To what extent do you feel that your life is close to your ideal?
- Q9: In general, how happy are you with your current situation in life?

```{code-cell}
# Load packages
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import sklearn
import seaborn as sns

# Set seed for reproducable results
np.random.seed(42)

# Set number of rows (artficial participants)
n = 300

# Create the items
D = np.random.normal(5, 1, n).reshape(n, 1) + np.random.normal(0, 0.5, (n, 3))
A = np.random.normal(4, 1, n).reshape(n, 1) + np.random.normal(0, 0.5, (n, 3))
LS = np.random.normal(6, 1, n).reshape(n, 1) + np.random.normal(0, 0.5, (n, 3))

# Create the dataframe
data = np.hstack([D, A, LS])
columns = ['Q1', 'Q2', 'Q3',
           'Q4', 'Q5', 'Q6',
           'Q7', 'Q8', 'Q9']

df = pd.DataFrame(data, columns=columns)

# Inspect the dateframe
print(df.head())
```

### Setup

The following code chunk imports the necessary class:

```{code-cell}
from factor_analyzer import FactorAnalyzer
```

### Inspect the data

Before conducting a factor analysis it is worthwhile to look at the correlation matrix of the data of interest.

```{code-cell}
plt.matshow(df.corr())
plt.show()
```

Here we can already see that the items 1-3, items 4-5 and items 6-9 correlate high with each other, respectively.

### Determine number of factors

Several approaches are possible to determine the number of factors. Here we apply the Kaiser criterion and select as many factors as there are factors with eigenvalues > 1. Therefore, we begin be fitting a solution with the number of factors being as high as possible.

```{admonition} Use your own brain!
:class: note

The maximum number of factors is equal to the number of observed variables. Whats the maximum number of factors for our example?
```

```{code-cell}
# Create factor analysis object
fa = FactorAnalyzer(rotation=None, method = 'ml', n_factors=9)
# Fit the model
fa.fit(df);
```

For this initial model, the Maximum Likelihood fitting method is used and no rotation is requested. One might also use the MINRES approach by requesting `method = 'minres'`. After chosing the number of factors, a suitable rotation method is applied to increase interpretability.

The following code chunks depicts the Eigenvalues of all 9 factors.

```{code-cell}
ev, cfev = fa.get_eigenvalues()
# cfev gives us the common factor eigenvalues , which we don't need at the moment.
print(ev)
```

Since 3 factors have Eigenvalues above 1, a **3-factor solution** is chosen for the final model.

Alternatively we could print a scree plot.

```{code-cell}
plt.plot(ev)
plt.xlabel("factor")
plt.ylabel("eigenvalue")
```

Using a scree plot we look for the 'bend' in the plot and choose the number of factors before the bend, here also 3 (don't get confused as the plot starts at 0 not 1).


### Fit and interpret the final model

Before fitting the final model, one has to choose whether to use independent (orthogonal rotation) or correlated (oblique rotation) factors. In psychology, most often is has to be assumed that the constructs we measure are somewhat correlated. Therefore, oblique rotation is applied here. The `factor_analyzer` package offers multiple oblique rotation methods (promax, oblimin and quartimin).

```{code-cell}
fa2 = FactorAnalyzer(n_factors=3, rotation='promax', method='ml')
fa2.fit(df);
```

To interpret the model the factors loadings can be depicted using the followiung command.

```{code-cell}
l = fa2.loadings_
print(l)
```

Items 1-3 load high on factor 1, item 4-6 load high on factor 2 and items 7-9 load high on factor 3. For the intepretation one has to have domain knowledge and think of a construct which might be represented best by the respective items. Factor 1 (and theredore items 1-3) might represet 'Depression' and factor 2 might represent 'Anxiety'.

```{admonition} Use your own brain!
:class: note

What might factor 3 represent?
```

To evaluate how good the model is one might request the communalities. They state how much variance the factor structure explains in each item.

```{code-cell}
c = fa2.get_communalities()
print(c)
```

