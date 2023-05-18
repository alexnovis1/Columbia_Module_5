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

#### Evaluate the Stock and Bond holdings of the portfolio

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

Since we are only interested in the closing price for our analysis, let's separate the data even further by setting a variable to only hold the closing columns. 

```python
# Access the closing price for AGG from the Alpaca DataFrame
# Converting the value to a floating point number
agg_close_price = df_portfolio["AGG"]['close']

# Access the closing price for SPY from the Alpaca DataFrame
# Converting the value to a floating point number
spy_close_price = df_portfolio["SPY"]['close']
```

Now that we have the closing prices, we can calculate the values of the stock, bond and total value of both given the most recent closing price. We can also calculate the total portfolio's worth by adding the cryptocurrencies wallet as well.

```python
# Calculate the current value of the bond portion of the portfolio
agg_value = agg_shares * agg_close_price[-1]

# Calculate the current value of the stock portion of the portfolio
spy_value = spy_shares * spy_close_price[-1]

# Calculate the total value of the stock and bond portion of the portfolio
total_stocks_bonds = agg_value + spy_value

# Calculate the total value of the member's entire savings portfolio
# Add the value of the cryptocurrency walled to the value of the total stocks and bonds
total_portfolio = total_crypto_wallet + total_stocks_bonds
```

### Part 2: Evaluating the Emergency Fund

In this part, the script looks at the cryptocurrency, stock, and bond in the portfolio to determine if the member has enough savings to build an emergency fund into their financial plan. 

To do this, the script first creates a DataFrame called `savings_df` which takes in the amount and indexes whether the asset amount is cryptocurrency or stock/bonds. 

```python
# Create a Pandas DataFrame called savings_df 
savings_df = pd.DataFrame(
    {"amount": savings_data},
    index=["crypto", "stock/bond"])
```

We can plot this to see see the portfolio composition. 

![Pie Chart of Composition](/Pictures/piechart.png)

Lastly, to determine if the current portfolio has enough to create an emergency fund as part of the member's financial plan. Here we make the ideal case 3 times the amount of the monthly income (which we noted was 12000 earlier!)

```python
# Create a variable named emergency_fund_value
emergency_fund_value = 3 * monthly_income
emergency_fund_value
```

Creating an if statment, we can report whether there is enough money for an emergency fund by looking to see if the overall portfolio is worth more than or equal to the `emergency_fund_value`. 

```python
# Evaluate the possibility of creating an emergency fund with 3 conditions:
if total_portfolio > emergency_fund_value:
    print("Congrats you have enough money in this fund!")
elif total_portfolio == emergency_fund_value: 
    print("Congrats, you have reached an import financial goal!")
else: 
    fund_diff = emergency_fund_values - total_portfolio
    print(f"You are ${fund_diff : ,.2f} from reaching your goal!")
```

## Part 3: Monte Carlo for Forecasting Profits. 

First, to forecast, we are setting up the SPY and AGG weights as 60% and 40%, respectively. We will look at three years of back data for the forecast. 

```python
# Set start and end dates of 3 years back from your current date
# Alternatively, you can use an end date of 2020-08-07 and work 3 years back from that date 
start_date_new = pd.Timestamp("2020-05-15", tz="America/New_York").isoformat()
end_date_new = pd.Timestamp("2023-05-15", tz="America/New_York").isoformat()

# Use the Alpaca get_bars function to make the API call to get the 3 years worth of pricing data
# The tickers and timeframe parameters should have been set in Part 1 of this activity 
# The start and end dates should be updated with the information set above
# Remember to add the df property to the end of the call so the response is returned as a DataFrame
df_port_new = alpaca.get_bars(
    tickers,
    timeframe,
    start = start_date_new,
    end = end_date_new
).df

# Reorganize the DataFrame
# Separate ticker data
SPY = df_port_new[df_port_new['symbol']=='SPY'].drop('symbol', axis=1)
AGG = df_port_new[df_port_new['symbol']=='AGG'].drop('symbol', axis=1)

# Concatenate the ticker DataFrames
df_port_new = pd.concat([SPY, AGG], axis=1, keys=['SPY', 'AGG'])
```

To run the Monte Carlo Simulation, we set the portfolio data which is the three year alpaca request we asked prior, weights (60% SPY and 40% AGG), the number of simulations we want (500 days) and the number of trading days (30 years). 

We can then review the simulation. 

```python
# Configure the Monte Carlo simulation to forecast 30 years cumulative returns
# The weights should be split 40% to AGG and 60% to SPY.
# Run 500 samples.
MC_port_df = MCSimulation(
    portfolio_data = df_port_new,
    weights = [0.6, 0.4],
    num_simulation = 500,
    num_trading_days = 252*30
)

# Review the simulation input data
MC_port_df.portfolio_data
```

Next, we can run the cumulative return. 

```python
# Run the Monte Carlo simulation to forecast 30 years cumulative returns
MC_port_df.calc_cumulative_return()
```

To look at an overaly of the potential cumulative profits we can call the `.plot_simulation()` function. 

```python
# Visualize the 30-year Monte Carlo simulation by creating an
# overlay line plot
MC_sim_line_plot = MC_port_df.plot_simulation()
```

This gives us a visual result: 

![Line Plot Overlay](/Pictures/lineplot.png)

Next, a histogram can be viewed to also see distribution of frequency. 

```python
# Visualize the probability distribution of the 30-year Monte Carlo simulation 
# by plotting a histogram
even_weight_dist = MC_port_df.plot_distribution()
```

![Histogram](/Pictures/histo.png)

Summarizing the data will further allow us to see the potential of profits.

```python
# Generate summary statistics from the 30-year Monte Carlo simulation results
# Save the results as a variable
MC_summary_stats = MC_port_df.summarize_cumulative_return()


# Review the 30-year Monte Carlo summary statistics
print(MC_summary_stats)
```

![Summary](/Pictures/summary.png)

By taking the 95% upper and lower confidence level, we can anciticipate the range of potential profits in the portfolio over the next 30 years. 

We will use the current portfolio value from earlier to determine the forecasted amount. 

```python
# Print the current balance of the stock and bond portion of the members portfolio
print(total_stocks_bonds)

# Use the lower and upper `95%` confidence intervals to calculate the range of the possible outcomes for the current stock/bond portfolio
ci_lower_thirty_cumulative_return = MC_summary_stats[8] * total_stocks_bonds
ci_upper_thirty_cumulative_return = MC_summary_stats[9] * total_stocks_bonds
```

Next, the code looks at a change in weight of the stock and bond portfolio (80% SPY and 20% AGG) and looks to see if this would have a different outcome in the next 10 years. The steps are the same! The only thing that needs to be changed is the setup for the MCSimulation. 

```python 
# Configure a Monte Carlo simulation to forecast 10 years cumulative returns
# The weights should be split 20% to AGG and 80% to SPY.
# Run 500 samples.
MC_new = MCSimulation(
    portfolio_data = df_port_new,
    weights = [.8, .2],
    num_simulation = 500,
    num_trading_days = 252*10
)
```


