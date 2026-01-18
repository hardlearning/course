# Lesson 2. Creating a Quant Alpha

To create an alpha we begin with an idea that seeks to capture an inefficiency in the market. That idea is then applied to a chosen group of stocks which we call our Alpha Universe.

One idea could be to buy stocks whose prices fell recently and expect the prices to revert back to the average historical price. For the stocks that experiened a rise in price in the last week we short them expecting their price to decline, this idea is called price reversion.

An Alpha is a vector of predicted values of stocks in the universe. The universe could be a group of US stocks defined on the basis of their liquidity, each value can change daily and has two properties: (1) the direction of the alpha, (2) the magnitude of the alpha. Direction determines whether you want to go long or short the instrument, the magnitude of the values determine what proportion of the allocated capital should be distributed amongst the instruments in the universe.

In allocating capital across the stocks in our Alpha Universe, we aim to balance estimated returns with downside protection. For this we create Equity Long-short Market-neutral Alphas.

Many hedge funds may utilize an Equity Long-short Market-neutral strategy. It involves equal amount of long positions in stocks expected to rise in value and short positins in stock expected to drop in value. The hope is that even if the long positions fall in value profit from shorting may still outweigh losses benefiting the hedge fund.

We could choose the closing price data field and an operation from our expression language to calculate the change in today's price compared to the closing price 5 days ago, the alpha world look like ts_delta(close, 5) with parameter 5 capturing a change in price over 5 days to make a long or short decision for each stock. This code will run on the past data to output the alpha vector for each day and has one value for each stock in the universe.

```
-ts_delta(close, 5)
```
