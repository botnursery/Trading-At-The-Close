# Trading At The Close dataset
## Dataset Description

This dataset contains historic data for the daily ten minute closing auction on the NASDAQ stock exchange.\
The purpose of this study is to review, primary analyze and identify features of the data set.\
:point_right: It does not claim to be a complete exploratory data analysis, but perhaps suggests ways for further study as well as preparation for model selection and training (ML).

The source of data and the formulation of the further forecasting problem can be found at https://www.kaggle.com/competitions/optiver-trading-at-the-close/data \
Brief description of the dataset:\
**stock_id** - A unique identifier for the stock. Not all stock IDs exist in every time bucket.\
**date_id** - A unique identifier for the date. Date IDs are sequential & consistent across all stocks.\
**imbalance_size** - The amount unmatched at the current reference price (in USD).\
**imbalance_buy_sell_flag** - An indicator reflecting the direction of auction imbalance.
   - buy-side imbalance 1
   - sell-side imbalance -1
   - no imbalance 0

**reference_price** - The price at which paired shares are maximized, the imbalance is minimized and the distance from the bid-ask midpoint is minimized, in that order. Can also be thought of as being equal to the near price bounded between the best bid and ask price.\
**matched_size** - The amount that can be matched at the current reference price (in USD).\
**far_price** - The crossing price that will maximize the number of shares matched based on auction interest only. This calculation excludes continuous market orders.\
**near_price** - The crossing price that will maximize the number of shares matched based auction and continuous market orders.\
**[bid/ask]_price** - Price of the most competitive buy/sell level in the non-auction book.\
**[bid/ask]_size** - The dollar notional amount on the most competitive buy/sell level in the non-auction book.\
**wap** - The weighted average price in the non-auction book.

**wap=** $\frac{ {BidPrice * AskSize + AskPrice * BidSize}}{BidSize + AskSize}$

**seconds_in_bucket** - The number of seconds elapsed since the beginning of the day's closing auction, always starting from 0.
**target** - The 60 second future move in the wap of the stock, less the 60 second future move of the synthetic index. Only provided for the train set.
   - The synthetic index is a custom weighted index of Nasdaq-listed stocks constructed by Optiver for this competition.
   - The unit of the target is basis points, which is a common unit of measurement in financial markets. A 1 basis point price move is equivalent to a 0.01% price move.
   - Where t is the time at the current observation, we can define the target:

$Target = (\frac{StockWAP_{t+60}}{StockWAP_{t}} - \frac{IndexWAP_{t+60}}{IndexWAP_{t}}) * 10000$

## Dataset overview and data cleanness

The table below shows a default sample of the data (beginning and end) from the dataset under consideration.

![Alt text](report/01.png?raw=true "1")

Below is shown information about the structure of the observed dataset and the data types. It can be seen that not all of the 17 available data fields are completely filled in, many rows have a NULL value.

![Alt text](report/02.png?raw=true "2")

To investigate the presence and distribution of NULL values ​​in the dataset, the null_scanner() function was used. The returned visualization shows the distribution in the upper graph, and the proportion of NULL in the total number of records in the lower graph.

![Alt text](report/03a.png?raw=true "3a")

It can be seen that a significant amount of NULL up to 50% is present only in the far_price and near_price fields. In other fields there are NULL values, but in a very small amount (0.001%) in relation to the total number of data.

![Alt text](report/03b.png?raw=true "3b")

However, there is no need at this moment to perform data cleaning operations yet, since all fields are given as is, are not calculated and are intended for further machine learning (ML) of a neural network of an as yet unknown structure with unknown requirements for converting NULL values.

## Statistical characteristics anlysis

Ultimately, the trained model should be capable of predicting the closing price movements for hundreds of Nasdaq listed stocks using data from the order book and the closing auction of the stock. Therefore, it is first interesting to look at the statistical characteristics of the target values. Let's check two characteristics: asymmetry and normal distribution. Here we use the scipy and pandas libraries, and give some common tests for distribution normality.\
The results obtained for all securities (indicated by indices 0-199) are as follows:\
    **- Min-skew=-8.58188**, Median-skew=-0.00941, **Max-skew=7.59506**\
    - Min-skew-index=**142**, Median-skew-index=58, Max-skew-index=**56**\
    - Shapiro-Wilk test Statistics=0.956,\
    **p-value=0.0000000**\
    - Pearson's chi-squared test Statistics=3935.053,\
    **p-value=0.000**

## Visual study results

Let's now look at the data set series that change during the closing auction and are the initial data for training the model and will influence the resulting values ​​of its output data. The primary source is quotes **bid/ask_price, bid/ask_size**.

![Alt text](report/04.train.ticks.png?raw=true "Bids-Asks")

It can be seen bellow that in the dataset the data is already presented in a normalized form with an approximation very close to linear.

Next, let's look at wap as the same normalized, but averaged by day, generalizing characteristic of the bid prices and their volume, averaged over days, offered by bidderss.
Below is a visually chaotic behavior of such a wap of one of the securities as an example. In the JyputerLab notebook (reporting link below) you can see a live chart with a choice for any of 200 securities.

![Alt text](report/07.png?raw=true "7")

Every day wap starts from 1 and then changes depending on the posted quotes.

![Alt text](report/08.png?raw=true "8")

Since the goal is to predict price movements, that is, closing price, let’s build a columnar distribution histogram of Target over all securities. It can be stated that the distribution, as expected for market quotes, has a bell-shaped pattern. However, you can also notice a certain comb on the envelope of this distribution.

![Alt text](report/05.png?raw=true "5")

There is periodicity in the deviation Target values of wap stock prices from the index. The nature of the deviation may be the subject of further study.\
This is clearly visible when selecting fairly narrow histogram bins. When the bins are expanded, the comb is visually smoothed due to averaging over the securities included in the bin. When constructing a histogram individually for each security (for stock with Median-skew-index as example), such an artifact is not observed.

![Alt text](report/06.png?raw=true "6")

## Observations, conclusions and suggestions

- The dataset is quite large and contains 5237980 entries for 200 securities over a period of 480 days. The source file is 640MB in size and requires significant computing resources, for this reason this report shows only screenshots and not live graphs from JupyterLab, otherwise its size would exceed a reasonable size.
- Not all stocks have the same number of days in price history. For example, no.79 - 300 days, 102 - 186, 135 - 290, 153 - 411, 199 - 393.
- With the exception of the far_price and near_price fields, the number of NaN values in the remaining fields is statistically small, one thousandth of a percent of the total number of records.
- Weighted average price wap of securities are presented in the form of normalized values. The first value for the selected day is taken as 1 and then all 10 second ticks are given relative to it. For the next day, everything is repeated again starting from 1.
- There is asymmetry in Target values for different securities (200 items), which indicates the presence of possible trends: min -8.58188 for a security with an index of 142, and max 7.59506 for a security with an index of 56.
- There is no reason to assume that Target values, whose future value is to be predicted, have a normal distribution.
- The distribution has the form characteristic of financial instruments with Fat-Tailed probability distributions.
- The histogram of all Target values for all time and for all securities has some periodic comb superimposed on the bell-shaped distribution.
- This periodicity indicates the presence among securities of those whose Target values more often fall into periodic bins. As the bin size increases, this effect is smoothed out due to averaging.
- For individual securities this deviation is not noticeable.
- This effect may be possibly be used to increase the probability of predicting the Target value. Further research can be supplemented in the following areas:
    - select those bins that stand out significantly in the distribution histogram and see which securities fall into them more often, forming a pattern.
    - find out whether the comb is strictly periodic, for this we can try to apply the discrete Fourier transform to the envelope of the final histogram.

## Conducting research and reporting links

The research on the dataset was conducted and documented in JyputerLab and the notebook file can be downloaded here:
- [Jupyter Notebook format](https://github.com/botnursery/Trading-At-The-Close/blob/main/optiver.df.train.analysis.ipynb)

The visual report is uploaded to .html and can be viewed here:
- [Web page format](https://github.com/botnursery/Trading-At-The-Close/blob/main/optiver.df.train.analysis.html) TBU
