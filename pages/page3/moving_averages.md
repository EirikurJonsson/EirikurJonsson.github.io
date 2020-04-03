# Moving averages - which values are best?

In the last plot I created functions that plot moving averages. Now that only looked into certain values defined by the user - but are they the best? In this post I want to find the very best possible moving averages by calculating the Sharp Ratio for that strategy.


```python
import pandas as pd
import numpy as np
from pandas_datareader import data as web
import matplotlib.pyplot as plt
import mpld3
from itertools import product
import seaborn as sns

plt.style.use("seaborn")
```

## The data

This is a dataset that I got from Kaggle and is the Brent Oil Prices - I dont know much about it, lets fix that real quickly.


```python
raw = pd.read_csv("BrentOilPrices.csv", parse_dates = ["Date"], index_col = 0)
raw = raw.dropna()
symbol = "Price" # to be used later

display(raw.info())
display(raw.head())
display(raw.tail())
```

    <class 'pandas.core.frame.DataFrame'>
    DatetimeIndex: 8216 entries, 1987-05-20 to 2019-09-30
    Data columns (total 1 columns):
    Price    8216 non-null float64
    dtypes: float64(1)
    memory usage: 128.4 KB
    


    None



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
      <th>Price</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1987-05-20</td>
      <td>18.63</td>
    </tr>
    <tr>
      <td>1987-05-21</td>
      <td>18.45</td>
    </tr>
    <tr>
      <td>1987-05-22</td>
      <td>18.55</td>
    </tr>
    <tr>
      <td>1987-05-25</td>
      <td>18.60</td>
    </tr>
    <tr>
      <td>1987-05-26</td>
      <td>18.63</td>
    </tr>
  </tbody>
</table>
</div>



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
      <th>Price</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2019-09-24</td>
      <td>64.13</td>
    </tr>
    <tr>
      <td>2019-09-25</td>
      <td>62.41</td>
    </tr>
    <tr>
      <td>2019-09-26</td>
      <td>62.08</td>
    </tr>
    <tr>
      <td>2019-09-27</td>
      <td>62.48</td>
    </tr>
    <tr>
      <td>2019-09-30</td>
      <td>60.99</td>
    </tr>
  </tbody>
</table>
</div>


So from this information we know we only have one column of data which, price of oil from 1987 to 2019. So I will want to test some values for the fast and slow moving averages. So trading days in a year are 252 - so m max fast moving average will be a quarter of that, 63 days, and my max slow average will be a full year. But the intervals will be five days for the fast moving average and two weeks (ten days) for the slow moving.  


```python
sma1 = range(1, 63, 5)
sma2 = range(126, 252, 10)
```

This code will now do the calculations needed to find the optimal combinations of fast and slow moving averages


```python
results = pd.DataFrame()
for SMA1, SMA2 in product(sma1, sma2):  
    data = pd.DataFrame(raw[symbol])
    data.dropna(inplace=True)
    data['Returns'] = data[symbol].pct_change()
    data['SMA1'] = data[symbol].rolling(SMA1).mean()
    data['SMA2'] = data[symbol].rolling(SMA2).mean()
    data.dropna(inplace=True)
    data['Position'] = np.where(data['SMA1'] > data['SMA2'], 1, -1)
    data['Strategy'] = data['Position'].shift(1) * data['Returns']
    data.dropna(inplace=True)
    perf = np.exp(data[['Returns', 'Strategy']].sum())
    ret_an = (data[['Returns', 'Strategy']].mean()) * 252
    vol_an = np.sqrt(data[['Returns', 'Strategy']].var() * 252)
    sr_an = ret_an / vol_an
    years = len(data) / 252
     
    results = results.append(pd.DataFrame(
                {'SMA1': SMA1, 'SMA2': SMA2,
                 'MARKET': perf['Returns'],
                 'STRATEGY': perf['Strategy'],
                 'VOLA_STRATEGY' : vol_an['Strategy'],
                 'SR_STRATEGY' : sr_an['Strategy'],
                 't-stat_STRATEGY' : sr_an['Strategy'] * np.sqrt(years)},
                 index=[0]), ignore_index=True)  
```


```python
results.sort_values('SR_STRATEGY', ascending=False).head(5)
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
      <th>SMA1</th>
      <th>SMA2</th>
      <th>MARKET</th>
      <th>STRATEGY</th>
      <th>VOLA_STRATEGY</th>
      <th>SR_STRATEGY</th>
      <th>t-stat_STRATEGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>126</td>
      <td>27.788273</td>
      <td>4.141828</td>
      <td>0.360700</td>
      <td>0.122727</td>
      <td>0.695369</td>
    </tr>
    <tr>
      <td>14</td>
      <td>6</td>
      <td>136</td>
      <td>27.959592</td>
      <td>3.169978</td>
      <td>0.360875</td>
      <td>0.099709</td>
      <td>0.564598</td>
    </tr>
    <tr>
      <td>6</td>
      <td>1</td>
      <td>186</td>
      <td>30.217179</td>
      <td>3.062593</td>
      <td>0.360526</td>
      <td>0.097427</td>
      <td>0.549968</td>
    </tr>
    <tr>
      <td>5</td>
      <td>1</td>
      <td>176</td>
      <td>30.235691</td>
      <td>2.876977</td>
      <td>0.360356</td>
      <td>0.091914</td>
      <td>0.519169</td>
    </tr>
    <tr>
      <td>13</td>
      <td>6</td>
      <td>126</td>
      <td>27.788273</td>
      <td>2.816945</td>
      <td>0.360705</td>
      <td>0.089436</td>
      <td>0.506743</td>
    </tr>
  </tbody>
</table>
</div>



## The results

The calculations above calculate the return, volatility of the returns/strategy and the Sharpe Ratio. Seeing that our t-statistic is not significant we can not rely to heavily on this metric, but its better than guessing the moving averages to use. In this case a window of 1 for the fast moving average is the best coupled with 126 day window, which means there is only one moving average to calculate. Lets visualize this.


```python
df = raw
df["sma2"] = df["Price"].rolling(window = 126).mean()
```


```python
df.loc["2016-01-01":].plot(figsize=(10, 6))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x26b6fb397c8>




![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/moving_averages/output_11_1.png)


Although these results are statistically not significant it is still the best moving average to use IF you strategy was based around moving averages. Just for some extra fun lets plot the next best strategy - simply because I want to and plotting is fun.


```python
df = raw
df["sma1"] = df["Price"].rolling(window = 6).mean()
df["sma2"] = df["Price"].rolling(window = 136).mean() 
df.loc["2018-04-01":].plot(figsize=(10, 6))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x26b6fc5ba48>




![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/moving_averages/output_13_1.png)


I did "zoom-out" a little because the green line wasn't showing very well but this is the next best strategy.

## What is obvious from this analysis

Its easy to see that a moving average is not enough to use as a stand-alone strategy, there is more information needed. My best bet would be to use Natural Language Processing(NLP) to better understand the landscape the oil market is in. This market is notoriously volatile and news can be extremely influential on price movement. It would be fun to run an ARIMA model and an autoregressive model to see if more forecasting accuracy could be obtained but I think it would always need to be coupled with NLP to allow for faster decision making based on the most recent data.
