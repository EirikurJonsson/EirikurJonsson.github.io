# Time Series plotting

So this post is all about time series plotting. This is done so easily with python that I decided to give myself a challenge. So for these I will be creating my own functions that should:

1. Simple line plot over time
2. Rolling average function
3. Candlestick-graph

So without talking to much...let's get into it.

Like always lets import some modules and get started.



```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import pandas as pd
import mplfinance as mpf
from matplotlib import style
style.use("seaborn")
```

I will use some stock data for this. Reason being its simple and has dates. Lets look at Apple since its everyones favorite stock data to plot, but these function are usable for any stock data.


```python
data = pd.read_csv("AAPL.csv", parse_dates = ["Date"])
```

Lets start by constructing a simple plot of the data function.


```python
def plot_df(df, x, y, title = "", xlabel = "Date", ylabel = "Value", dpi = 100):
    '''
    This is a simple generic plot function that will plot a time series. 
    '''
    plt.figure(figsize = (16,5), dpi = dpi)
    plt.plot(x,y, color = "tab:red")
    plt.gca().set(title = title, xlabel = xlabel, ylabel = ylabel)
    plt.show()
    
plot_df(df = data, x = data["Date"], y = data["Adj Close"])
```


![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/timeseriesplots_files/timeseriesplots_5_0.png?raw=true)


Thats real nice. But simple, so lets look at the next function - rolling averages.

Now this is a bit more tricky because I wanted this to be as general as possible. I dont want to be able to just have one rolling average or just two - I wanted as many rolling averages as the user wants. So my functions makes a distinction between a single value given or a list of values, calculates those values and then plots them. I will make a few examples of this just for fun, but I will be sticking to that Apple dataset


```python
def rollave(df, x, y, roll, title = ""):
    '''
    This function is meant to take in either a value or a list of values
    that will represent the moving average window.
    This function will add the rolling averages to the data frame. This
    is, for now, is not a problem since it can be a positive going forward.
    '''
    if isinstance(roll, list):
        plt.figure(figsize = (16,5), dpi = 100)
        plt.plot(df[x], df[y], color = "tab:red", label = y)
        plt.gca().set(title = "Moving averages", xlabel = x, ylabel = y)
        for i in (roll):
            df[f"ma{i}"] = df[y].rolling(window = i, min_periods = 0).mean()
            plt.plot(df[x], df[f"ma{i}"], label = df[f"ma{i}"].name)   
    
    else:
        df[f"ma{roll}"] = df[y].rolling(window = roll, min_periods = 0).mean()
        plt.figure(figsize = (16,5), dpi = 100)
        plt.plot(df[x], df[f"ma{roll}"], label = df[f"ma{roll}"].name)
        plt.plot(df[x], df[y], color = "tab:red", label = y)
        plt.gca().set(title = "Moving averages", xlabel = x, ylabel = y)
    plt.legend()
    plt.show()
rollave(df = data, x = "Date", y = "Adj Close", roll = 50)
```


![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/timeseriesplots_files/timeseriesplots_7_0.png?raw=true)



```python
# Two rolling averages
roll = [52,252]
rollave(df = data, x = "Date", y = "Adj Close", roll = roll)
```
![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/timeseriesplots_files/timeseriesplots_8_0.png?raw=true)



```python
# Three rolling averages
roll = [52,126,252,]
rollave(df = data, x = "Date", y = "Adj Close", roll = roll)
```
![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/timeseriesplots_files/timeseriesplots_9_0.png?raw=true)


It is fun but starts to be boring after the you add the third line. Now this does do something some might think is bad - but I do have my reasons. In this function I add these calculation to the data frame. My assumption is that this might be data someone would like to use for further calculation down the line, so this takes care of that.

Lets move on the the candlestick graph. I am sad to say that my code for this has been surpassed, since this is old code I did the mpl_finance module has been updated - rendering my code useless. So I will have the code in the original git-repo but for now, lets do the cleaner yet not so "programmy" candlestick graph. This new code does require difference in importing of the data - and just to make things easy lets just do that right now but lets use difference data, just so that the graphs dont all look the same - gets stale after a while.


```python
data = pd.read_csv("ABT.csv", parse_dates = True, index_col = 0)

ohlc = data[["Open","High", "Low", "Close", "Volume"]].copy()

def nCandlegraph(df):
    '''
    This will get a specific data frame and produces a candlestick graph.
    To make this function work you need to parse_dates for the dataframe.
    '''
    mpf.plot(df,type='candle', volume = True)
    plt.show()

nCandlegraph(ohlc.loc["2018-12-01":"2019-01-01",:])
```


![png](https://github.com/EirikurJonsson/EirikurJonsson.github.io/blob/master/images/timeseriesplots_files/timeseriesplots_11_0.png?raw=true)


I know what you are thinking - it would have been easier to just write the code out instead of putting it into a function. Well if you look at my original repo then you'd see the code is a lot more detailed. But enough with that. The Python language has shown itself to be fantastic at dealing with data, manipulation of data and some pretty sweet graphs. I would have loved to do this in Plotly - an interactive graphing tool for Python and R, but that is a notebook onto itself.
