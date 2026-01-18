# Lesson 3. How Good is Your Alpha A Metrics-Based Approach

## Assessing quality with backtesting

Be careful to avoid look ahead bias in backtesting. The bias can inflate performance predictions, skew expectations and potential lead to losses

- Look ahead bias: A common pitfall where future information accidentally influences the historical data analysis.

## Performance metrics

The metrics can be related to Performance, novelty, Diversity etc.

The six performance related metics:

- Sharpe: Lower sharpe score, higher risk. Higher sharpe score, lower risk.
- Turnover: Turnover is the percentage of the capital which the alpha trades each day. More turnover may means higher transaction costs.
- Drawdown: Drawn down represents the percentage of the largest loss incurred during any year in your backtesting. Your should target a returns/drawdown ratio greater than one. The higher the ratio the better it may be for your alpha.
- Correlation: Correlation of the alpha to other alpha in the pool should be low, unless we see much higher performance as compared to the correlated alphas.
- Weight
- Sub-universe

The metrics to check if the performance is contributed by diverse set of stocks include a weight test and a sub-universe test.

Weight Test: A robustness check to ensure Alpha weight is evenly distributed across stocks, not concentrated on a few.

Sub-universe Test: Checks if the Alpha's performance in the immediate smaller set of tradable stocks, or sub-universe, exceeds a required threshold.