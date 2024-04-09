### Welcome to Invest.py! 

**Implemenetation Strategy:**
After doing a lot of research, it seems as though there is no obvious investment algorithm that
consistently beats the performance of the SnP500. Thus, I based my analytical strategies on a comparison
of any given stock to a relative perfromance metric of the SnP as a whole. I will dive into specifics below:

**CACHED DATA:**
- SnP500 list
    I began by pulling a list of all of the tickers for the companies in the SnP500
    Note, the companies in the SnP500 change (not too frequently) but for reference 
    my list of SnP500 companies was loaded February 10th, 2024

- dividendsYield:
    using my helper function "get_div_yields" I was able to access the yahooquery API and pull data.
    These values were taken from each stocks average of foward and trailing divident yields
    this data was used when the client cares about income, as I will explain below

- relative_growth_20y
    this file contains a list of each SnP500 stocks percentile of growth relative to entire SnP500 performance over the last 20 years. This was generated using my helper function
    generate_relative_growth. This list is used as a scale of relativeltivy for any input stock
    given the client cares about growth, as I'll explain below.

- relative_growth_01y
    similar calculation to previous list, but only for the most recent year of SnP500 performance

- regressionData
    Although this list can appear overwhelming, it was used to train a predictive regression model
    for a given stocks growth. This linear regression was trained using 26 data points for each
    SnP500 stock: marketCap, regularMarketVolume, fiveYearAvgDividendYield, 52WeekChange, trailingPE,
    trailingEps, priceToBook, beta, shortRatio, forwardPE, pegRatio, peRatio, profitMargins, operatingMargins, ebitdaMargins, grossMargins, earningsGrowth, returnOnEquity, returnOnAssets,
    quickRatio, totalDebt, ebitda, totalCash, recommendationMean, esg_percentile

    -   When using these as beta predictors, and making the Y axis the true performance of each stock
        the linear regression was able to predict 80% of the variation in growth given the stocks inputs.
        I ran the model iterately over the last 20 years to see which would have the best fit score.
        Obviously the farther back you go in history, the more potential variance there is to explain
        (as you can see in the list below of model accuracy scores). However, 80% was surprinsingly high to me so I used the weights converged on in the linear regression model trained on predicting 1 year of relative growth to help me make predictions when the client did not have any specific preference. This data was generated in generate_regression_data() and used in run_regressions() and when client=false.
        Further explanation of its application is below.

            - using 1 years of relative growth:  0.8089595951637427
            - using 2 years of relative growth:  0.64709209626279
            - using 3 years of relative growth:  0.5344955279110323
            - using 4 years of relative growth:  0.48979029247296746
            - using 5 years of relative growth:  0.5294941826754916
            - using 6 years of relative growth:  0.4825159448713233
            - using 7 years of relative growth:  0.5080713092746236
            - using 8 years of relative growth:  0.4841699086125989
            - using 9 years of relative growth:  0.5304491344619507
            - using 10 years of relative growth:  0.5180023889984489
            - using 11 years of relative growth:  0.5031858293140121
            - using 12 years of relative growth:  0.49016554204192864
            - using 13 years of relative growth:  0.4949316994071393
            - using 14 years of relative growth:  0.4680270697278617
            - using 15 years of relative growth:  0.43597684707014606
            - using 16 years of relative growth:  0.44779290390560433
            - using 17 years of relative growth:  0.4131199204121221
            - using 18 years of relative growth:  0.39573398319414776
            - using 19 years of relative growth:  0.3895847736944672

            
**Invest Explanation**

- When the client cares about income:
    When a client cares about immediate income, they should put an emphasis on stocks with higher dividend yields. Given I use the SnP500 as a golden standard, I turned the dividendsYield list into a numpy array, and was easily able to create a percentile of relative divident yields for each stock.
    Thus, when Client=income I take in whatever stock is being evalutaed, calculate its dividend yields in an identical mannor, and compare that value as a percentile to that of all SnP500 stocks.
    If the stocks divident yield score is in the top quartile - ie 75th percetile and above, invest will 
    tell you to buy the stock. If its in the 50-75th percentile, invest will tell you to hold. if any stock is below the 50th percentile of SnP500 diivident yields, then invest will tell the client to sell.

- When the client cares about esg:
    Thankfully, yahooquery includes an esg percentile for any given stock, which is a percentile relative to the entire market, not just the SnP500. So similarly to when the client cares about income, if the stocks esg score is in the top quartile - ie 75th percetile and above of the entire market, invest will tell you to buy the stock. If its in the 50-75th percentile, invest will tell you to hold. If any stock is below the 50th percentile esg scores in the market, then invest will tell the client to sell.

- When the client cares about growth:
    When the Client cares about growth I had to consider the timeline of a stock. I used the relative_growth_20y file to base the relativity of any given stocks growth to the SnP500. The issue is not all stocks have been around for 20 years. Additionally, stock A with 2x growth (more than snp) over 10 years is worse than stock B w 2x growth over 5 years. Thus I had to use the yahooquery history object (snp_history = snp.history(period="20y", interval="1mo")) to have a interval of each SnP500 stocks given price at any month within the last 20 years. Thus, whenever a stock was given as input, I would compare it with an equivalent timeline to all existing SnP stocks (max 20 years). I then given the timeframe, had to calculate the annualized rate of return (using a compound interest formula) for all SnP500 stocks as well as the given input stock. Then similarly to the previous two clients, I turned the relative growth list into a numpy array, and was easily able to create a percentile of relative annualized rate of return for each stock. If the given stocks annualized rate of return percentile is in the top quartile - ie 75th percetile and above compared to the SnP500, invest will tell you to buy the stock. If its in the 50-75th percentile, invest will tell you to hold. If any stock is below the 50th percentile, then invest will tell the client to sell.

- When there is no client:
    This is where my linear regression prediction model comes into play. If the client has no preference about income, esg, or growth, then I figured growth would be the most important factor. Therefore I tried to find predictors of growth that werent just an annualized rate of return from a history of stock prices. Thus, recall this linear regression was trained using 26 data points for each SnP500 stock: 
        marketCap, regularMarketVolume, fiveYearAvgDividendYield, 52WeekChange, trailingPE,
        trailingEps, priceToBook, beta, shortRatio, forwardPE, pegRatio, peRatio, profitMargins, operatingMargins, ebitdaMargins, grossMargins, earningsGrowth, returnOnEquity, returnOnAssets,
        quickRatio, totalDebt, ebitda, totalCash, recommendationMean, esg_percentile
    
    Using these 26 predictors, my linear regression model was aable to predict 80% of variance in any given SnP stocks growth in the last year. Although not perfect, I figured it was accurate enough to use as a solid base of relativity. So I used the same predictive model trained on all 500 SnP stocks to generate a predicted value for the given input stock (given its 26 input variables). Once a predicted value gets calculated using the linear regression, that score is compared to that of all SnP500 predicted scores, giving the input stock a percentile of relative predicted growth to the SnP500. Similarly to all other policies, if the given stocks predicted growth percentile is in the top quartile - ie 75th percetile and above compared to the SnP500 predictions, invest will tell you to buy the stock. If its in the 50-75th percentile, invest will tell you to hold. If any stock is below the 50th percentile, then invest will tell the client to sell.

    Please read through the invest function to see how these concepts were implemented in code.


**Sample Usage:**

- print(invest("AAPL", client=False))
    Returns "BUY, AAPL relative growth prediction > 75% of SnP500 companies, 86.80th percentile to be exact"

- print(invest("AAPL", client="income"))
    Returns: SELL, AAPL dividend yields less than 50% of SnP500 companies (last 20 years), 25.00th percentile to be exact

- print(invest("AAPL", client="growth"))
    Returns: "BUY, AAPL relative growth > 75% of SnP500 companies (last 20 years), 99.20th percentile to be exact"

- print(invest("AAPL", client="esg"))
    Returns: "BUY, AAPL esg percentile >= 75% of entire market, 82.18th percentile to be exact"

- print(invest("XOM", client="esg"))
    Returns: "SELL, XOM esg percentile in bottom half, ie. below 50th percentile of entire market, 6.83th percentile to be exact"

- print(invest("PLTR", client="income"))
    Returns "SELL, PLTR dividend yields less than 50% of SnP500 companies, 0.00th percentile to be exact"

- print(invest("PLTR", client="esg"))
    Returns "SELL, PLTR esg percentile in bottom half, ie. below 50th percentile of entire market, 0.00th percentile to be exact"

- print(invest("PLTR", client="growth"))
    Returns "BUY, PLTR relative growth > 75% of SnP500 companies, 94.60th percentile to be exact"
    
- print(invest("PLTR", client=False))
    Returns "BUY, PLTR relative growth prediction > 75% of SnP500 companies, 100.00th percentile to be exact"

- print(invest("CMG", client="esg"))
    HOLD, CMG esg percentile in 3rd quartile: 50-75th percentile of entire market, 69.56th percentile to be exact



    

    
    

    
    


    
    

