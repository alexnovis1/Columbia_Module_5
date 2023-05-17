# Columbia_Module_5

## Introduction: 

The script `financial_planning_tools.ipynb` looks at a members portfolio to view current savings and if they have enough for an emergency fund. Next, the script forecasts the performance of their retirement portfolio and what it may accumulate to in 30 years. 

---

## Folders

[Financial Planner Script](/financial_planning_tools.ipynb)

[MCForecast Tools](/MCForecastTools.py)

---

## Installation Guide: 

The following libraries and dependencies are used: 

```python
import os
import requests
import json
import pandas as pd
from dotenv import load_dotenv
import alpaca_trade_api as tradeapi
from MCForecastTools import MCSimulation

%matplotlib inline
```

Please see below for more information regarding these libraries and dependencies. 

[OS](https://docs.python.org/3/library/os.html)

[requests](https://www.w3schools.com/python/module_requests.asp)

[pandas](https://pandas.pydata.org/)

[dotenv](https://pypi.org/project/python-dotenv/)

[alpaca_trade_api](https://pypi.org/project/python-dotenv/)

