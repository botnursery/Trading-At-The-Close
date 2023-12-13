![Alt text](train.ticks.png?raw=true "Bids-Asks")
# Trading At The Close data set
This dataset contains historic data for the daily ten minute closing auction on the NASDAQ stock exchange.\
The purpose of this study is to review, primary analyze and identify features of the data set. It does not claim to be a complete exploratory data analysis, but perhaps suggests ways for further study.\
The source of data and the formulation of the further forecasting problem can be found at https://www.kaggle.com/competitions/optiver-trading-at-the-close/data \
Brief description of the dataset:

stock_id - A unique identifier for the stock. Not all stock IDs exist in every time bucket.\
date_id - A unique identifier for the date. Date IDs are sequential & consistent across all stocks.\
imbalance_size - The amount unmatched at the current reference price (in USD).\
imbalance_buy_sell_flag - An indicator reflecting the direction of auction imbalance.
   - buy-side imbalance; 1
   - sell-side imbalance; -1
   - no imbalance; 0

reference_price - The price at which paired shares are maximized, the imbalance is minimized and the distance from the bid-ask midpoint is minimized, in that order. Can also be thought of as being equal to the near price bounded between the best bid and ask price.\
matched_size - The amount that can be matched at the current reference price (in USD).\
far_price - The crossing price that will maximize the number of shares matched based on auction interest only. This calculation excludes continuous market orders.\
near_price - The crossing price that will maximize the number of shares matched based auction and continuous market orders.\
[bid/ask]_price - Price of the most competitive buy/sell level in the non-auction book.\
[bid/ask]_size - The dollar notional amount on the most competitive buy/sell level in the non-auction book.\
wap - The weighted average price in the non-auction book.

wap: $\frac{ {BidPrice * AskSize + AskPrice * BidSize}}{BidSize + AskSize}$

seconds_in_bucket - The number of seconds elapsed since the beginning of the day's closing auction, always starting from 0.
target - The 60 second future move in the wap of the stock, less the 60 second future move of the synthetic index. Only provided for the train set.
   - The synthetic index is a custom weighted index of Nasdaq-listed stocks constructed by Optiver for this competition.
   - The unit of the target is basis points, which is a common unit of measurement in financial markets. A 1 basis point price move is equivalent to a 0.01% price move.
   - Where t is the time at the current observation, we can define the target:

$Target = (\frac{StockWAP_{t+60}}{StockWAP_{t}} - \frac{IndexWAP_{t+60}}{IndexWAP_{t}}) * 10000$

# The study result links

The research on the deta set was conducted and documented in Jupyter Lab and can be downloaded here:
- [Jupyter Notebook format](https://github.com/botnursery/Trading-At-The-Close/blob/main/optiver.df.train.analysis.ipynb)

The visual report is uploaded to .html and can be viewed here:
- [Web page format](https://github.com/botnursery/Trading-At-The-Close/blob/main/optiver.df.train.analysis.html) TBU

# Observations

- The date set is quite large and contains 5237980 entries for 200 securities over a period of 480 days.
- Not all stocks have the same number of days in price history. For example, no.79 - 300 days, 102 - 186, 135 - 290, 153 - 411, 199 - 393.
- With the exception of the far_price and near_price fields, the number of NaN values in the remaining fields is statistically small, one thousandth of a percent of the total number of records.
- Weighted average price wap of securities are presented in the form of normalized values. The first value for the selected day is taken as 1 and then all 10 second ticks are given relative to it. For the next day, everything is repeated again starting from 1.
- There is asymmetry in Target values for different securities (200 items), which indicates the presence of possible trends:\
    Min-skew=-8.58188, Median-skew=-0.00941, Max-skew=7.59506\
    Min-skew-index=142, Median-skew-index=58, Max-skew-index=56
- There is no reason to assume that Target values, whose future value is to be predicted, have a normal distribution.
- The distribution has the form characteristic of financial instruments with Fat-Tailed probability distributions.
- The histogram of all Target values for all time and for all securities has some periodic comb superimposed on the bell-shaped distribution.
- This periodicity indicates the presence among securities of those whose Target values more often fall into periodic bins. As the bin size increases, this effect is smoothed out due to averaging. For individual securities this deviation is not noticeable.
- This effect may be possibly be used to increase the probability of predicting the Target value. Further research can be supplemented in the following areas:
    - select those bins that stand out significantly in the distribution histogram and see which securities fall into them more often, forming a pattern.
    - find out whether the comb is strictly periodic, for this we can try to apply the discrete Fourier transform to the envelope of the final histogram.
