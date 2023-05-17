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

After the dependencies and libraries are requested, the next step is to laod the enviornment from the .env file. This can be done by calling the `load_dotenv` function. 

```python 
load_dotenv()
```

Please see below for more information regarding these libraries and dependencies. 

[OS](https://docs.python.org/3/library/os.html)

[requests](https://www.w3schools.com/python/module_requests.asp)

[pandas](https://pandas.pydata.org/)

[dotenv](https://pypi.org/project/python-dotenv/)

[alpaca_trade_api](https://pypi.org/project/python-dotenv/)

---

## Usage: 

### Part 1: Create a Financial Planner for Emergencies 

First, lets set the number of coins held for each cryptocurrency held in the portfolio, as well as the monthly income. 

```python 
# The number of coins for each cryptocurrency held in the portfolio
btc_coins = 1.2
eth_coins = 5.3

# The monthly amount for the member's household income
monthly_income = 12000
```

To gather the crypto data, use the free crypto API endpoint URL

```python 
btc_url = "https://api.alternative.me/v2/ticker/Bitcoin/?convert=USD"
eth_url = "https://api.alternative.me/v2/ticker/Ethereum/?convert=USD"
```

Next, use the requests library to get the current price of BTC and ETH by using the API endpoints

```python
# Using the Python requests library, make an API call to access the current price of BTC
btc_response = requests.get(btc_url).json()

# Use the json.dumps function to review the response data from the API call
# Use the indent and sort_keys parameters to make the response object readable
print(json.dumps(btc_response, indent = 4, sort_keys = True))

# Using the Python requests library, make an API call to access the current price ETH
eth_response = requests.get(eth_url).json()

# Use the json.dumps function to review the response data from the API call
# Use the indent and sort_keys parameters to make the response object readable
print(json.dumps(eth_response, indent=4, sort_keys=True))
```

Navigate the JSON repsonse to access the current price of each coin and store each in a variable.

```python
# Navigate the BTC response object to access the current price of BTC
btc_price = btc_response['data']['1']['quotes']['USD']['price']

# Print the current price of BTC
print(btc_price)

# Navigate the BTC response object to access the current price of ETH
eth_price = eth_response['data']['1027']['quotes']['USD']['price']

# Print the current price of ETH
print(eth_price)
```

Calculate the value of the current amount of each cryptocurrency based on the hold amount found in the portfolio

```python
# Compute the current value of the BTC holding 
btc_value = btc_coins * btc_price

# Print current value of your holding in BTC
print(f" The current amount of BTC in my portfolio is ${btc_value: ,.2f}")

# Compute the current value of the ETH holding 
eth_value = eth_coins * eth_price

# Print current value of your holding in ETH
print(f" The current amount of BTC in my portfolio is ${eth_value: ,.2f}")
```

Lastly, we can compute the total amount of cryptocurrency wallet by adding the two values of BTC and ETH found above.

```python
# Compute the total value of the cryptocurrency wallet
# Add the value of the BTC holding to the value of the ETH holding
total_crypto_wallet = btc_value + eth_value

# Print current cryptocurrency wallet balance
print(f"Total cryptocurrency wallet balance is ${total_crypto_wallet: ,.2f}")
```

### Evaluate the Stock and Bond holdings of the portfolio

Now that we know the cryptocurrency being held in the portfolio, we can move to the Stock and Bond holdings.

First, specify the amount of shares held in both assets.

```python
# Current amount of shares held in both the stock (SPY) and bond (AGG) portion of the portfolio.
spy_shares = 110
agg_shares = 200
```

To get the data of both SPY and AGG we can use the Alpaca API. 

```python
# Set the variables for the Alpaca API and secret keys
alpaca_api_key = os.getenv("ALPACA_API_KEY")
alpaca_secret_key = os.getenv("ALPACA_SECRET_KEY")

# Create the Alpaca tradeapi.REST object
alpaca = tradeapi.REST(
    alpaca_api_key,
    alpaca_secret_key,
    api_version="v2")

# Set the tickers for both the bond and stock portion of the portfolio
tickers = ["SPY", "AGG"]

# Set timeframe to 1Day
timeframe = "1Day"

# Format current date as ISO format
# Set both the start and end date at the date of your prior weekday 
# This will give you the closing price of the previous trading day
# Alternatively you can use a start and end date of 2020-08-07
start_date = pd.Timestamp("2023-05-08", tz="America/New_York").isoformat()
end_date = pd.Timestamp("2023-05-12", tz="America/New_York").isoformat()

# Use the Alpaca get_bars function to get current closing prices the portfolio
# Be sure to set the `df` property after the function to format the response object as a DataFrame
df_portfolio = alpaca.get_bars(
    tickers,
    timeframe,
    start = start_date,
    end = end_date
).df
```

Lets reorganize the DataFrame and separate the ticker data 

```python
# Reorganize the DataFrame
# Separate ticker data
SPY = df_portfolio[df_portfolio['symbol']=='SPY'].drop('symbol', axis=1)
AGG = df_portfolio[df_portfolio['symbol']=='AGG'].drop('symbol', axis=1)

# Concatenate the ticker DataFrames
df_portfolio = pd.concat([SPY, AGG], axis=1, keys=['SPY', 'AGG'])

# Review the first 5 rows of the Alpaca DataFrame
df_portfolio.head()
```






