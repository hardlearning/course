# Lesson 4. Alpha Examples by Data Category Part 1

The data categories includes: price and volume, fundamentals, analyst forecasts, market sentiment, options, model, insider transactions, short interest.

We will begin with the Price and Volume Data category. It includes stock prices: opening price, highest price, lowest price, closing price, and other trading-related informantion like volume of shares traded and market capitalization.

The price difference between the opening prices and closing prices during the trading session can serve as a potential strategy idea.

Here's on assumption: If a stock's closing price is lower than it's opening price, we might expect a mean reversion, hoping the stock will rebound and potentially outperform its peers in the short term. At this point, we establish a long position. If the prices rises, we profit. Conversely, if the closing price is higher than the opening price, we may expect a price decline and establish a short position. If the price falls, we profit.

The group_rank function takes the closing price and opening price difference as the first input parameter, and subindustry classificatin as the second parameter. This function will among all stocks in the same subindustry category, this price difference is ranked and assigned a value between 0 and 1. Higher values are ranked closer to 1, lower values are ranked closer to 0. Stocks with negative alpha values are shorted, while those with positive values are bought.

```
-group_rank(close-open, subindustry)
```

The timing of the data used for alpha needs to be determined called delay. Delay 1 uses yesterday's price data, while delay 0 uses today's real-time price data up to a specified point in time.

Decay considers the weighted sum of past alpha values to smooth volatility or eliminate outliers. Decay is particularly helpful if you want to use alpha values from previous days or reduce turnover.

In this simulation we use Delay 1 to avoid look-ahead bias. Neutualization is applied to the Subindustry. Using decay 10 to smooth the signal. And limit the maximum capital of a single stock to 1%.

## How to improve this Alpha

The performance of this strategy can be improved by controlling the turnover rate and only trading under conditions that are more conductive to mean reversion, that is, choosing to trade under conditions of high volatility and holding positions under conditions of low volatility.

Fundamentals: Capture the underlying business, financial and operational health of a company, usually reported every quarter.

Fundametals can be summarized into three major financial statements: balance sheet, income statement and cash flow statement.

The balance sheet provides a snapshot of the company's financial health, detailing its assests, liabilities, and equity.

The income statement illustrates the company's profitability, showing how revenue is transformed into income after accounting for various expenses.

The cash flow statement reveals the company's liquidity by tracking the inflow and outflow of cash from activities like operations, investments and financing.

Now let's discuss the strategy concept based on "operating cash flow and changes in company market value".

Cash Flow (From Operations): The cash earned through core business activities.

Market Capitalization: Represents the firm's worth, calculated as the current market price of one share multiplied by the total number of the company's outstanding shares.

A high ratio of cash flow from operations to market capitalization indicates that the company may be undervalued and it's expected future returns may be higher, and vice versa. This alpha assumption is that if this ratio improves over time, the company's stock will outperform it's peer, so we will take a long position on such companies. If the ratio declines, we expect it's performance to be poor and take a short position.

```
group_scale(ts_zscore(cashflow_op/cap, 60), industry)
```

The time series function ts_zscore shows the change in the operating cash flow to market capitalization ratio over the past 60 tading days. In statistics, the z-score is used to describe how far a value is from the mean of group in standard deviation.

We simulated this Alpha expression on the top 3000 most liquid US stocks and neutualized the market to maintain a balance between long and short positions.

To improve Alpha performance, consider using analysts' forcasts of each stock's operating cash flow instead of the final actual values. Analyst estimates provide insights into a company's expected performance before it's actual quarterly earnings are released.