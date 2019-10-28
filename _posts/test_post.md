---
layout: post
mathjax: true
comments: true
title: Test Post Rendered from Jupyter
---
# Test Post Rendered from Jupyter Notebook


```python
import pandas as pd
```


```python
pd.DataFrame(
    {
        "Name": ["Test1", "Test2"],
        "Value": [1, 2]
    }
)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Test1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Test2</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>


