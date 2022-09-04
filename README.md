# coinmarketcap-proj.

import pandas as pd
import matplotlib.pyplot as plt

dec6 = pd.read_csv('coinmarketcap_06122017.csv')
dec6

## 1. Selecting the 'datasets', 'id' and the 'market_cap_usd' columns
market_cap_raw = dec6[['id', 'market_cap_usd']]
market_cap_raw
#ca = market_cap_raw[market_cap_raw['market_cap_usd'] > 0]
# type(ca)

## 2. Discard the cryptocurrencies
cap = market_cap_raw.query('market_cap_usd >0')
cap.count()
cap.isna().any()
cap10 = cap.head(10).set_index('id').sort_index()
#cap10 = cap.head(10).set_index('id', inplace = True ).sort_index() # for parmanent change
cap10
# cap.agg(['count']) # convert in datadfarame

## 3. How big is Bitcoin compared with the rest of the cryptocurrencies?
TOP_CAP_TITLE = 'Top 10 market capitalization'    # constant variables should be in Capitaliz 
TOP_CAP_YLABEL = '% of total cap'

# Selecting the first 10 rows and setting the index
cap10 = cap.iloc[:10].set_index('id')

# Calculating market_cap_perc      # assign fun add new coulnms
cap10 = cap10.assign(market_cap_perc = lambda x: (x.market_cap_usd / cap.market_cap_usd.sum()) * 100)
cap10
len(cap)
cap10.iloc[0, 0]
s = cap.market_cap_usd.sum()
market_cap_perc_list = []
for i in range(len(cap)):
    market_cap_perc_list.append(((cap.iloc[2, 1])/ s) * 100)
cap10
ser1 = pd.Series(market_cap_perc_list)
pd.concat([cap10.reset_index(), ser1], axis = 0)
cap10.head(2)

# Plotting the barplot with the title defined above 
ax = cap10.plot.bar(title=TOP_CAP_TITLE)

## 4. Making the plot easier to read and more informative
COLORS = ['orange', 'green', 'orange', 'cyan', 'cyan', 'blue', 'silver', 'orange', 'red', 'green']

# Plotting market_cap_usd as before but adding the colors and scaling the y-axis  
ax = cap10.market_cap_usd.plot.bar(color=COLORS, logy=True, title=TOP_CAP_TITLE)
# Annotating the y axis with 'USD'
ax.set_ylabel('USD');
# Final touch! Removing the xlabel as it is not very informative
ax.set_xlabel('');

## 5. What is going on?! Volatility in cryptocurrencies
volatility = dec6[['id', 'percent_change_24h', 'percent_change_7d']]

# Setting the index to 'id' and dropping all NaN rows
volatility = volatility.set_index('id').dropna()

# Sorting the DataFrame by percent_change_24h in ascending order
volatility = volatility.sort_values(by='percent_change_24h', ascending=True)

# Checking the first few rows
volatility.head()

## 6. Well, we can already see that things are a bit crazy
def top10_subplot(volatility_series, title):
    # Making the subplot and the figure for two side by side plots
    fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(10, 6));
    
    # Plotting with pandas the barchart for the top 10 losers
    ax = volatility_series.iloc[:10].plot.bar(color='darkred', ax=axes[0])
    
    # Setting the figure's main title to the text passed as parameter
    fig.suptitle(title)
    
    # Setting the ylabel to '% change'
    ax.set_ylabel('% change')
    
    # Same as above, but for the top 10 winners
    ax = volatility_series.iloc[-10:].plot.bar(color='darkblue', ax=axes[1])
    
    plt.tight_layout()
    # Returning this for good practice, might use later
    return fig, ax

DTITLE = "24 hours top losers and winners"

# Calling the function above with the 24 hours period series and title DTITLE  
fig, ax = top10_subplot(volatility.percent_change_24h, DTITLE)

## 7. Ok, those are... interesting. Let's check the weekly Series too.
volatility7d = volatility.sort_values(by='percent_change_7d', ascending=True)

WTITLE = "Weekly top losers and winners"

# Calling the top10_subplot function
fig, ax = top10_subplot(volatility7d.percent_change_7d, WTITLE)

## 8. How small is small?
largecaps = cap.query('market_cap_usd >= 10000000000')   # 10000000000 0r 1e + 10

# Printing out largecaps
largecaps.iloc[:5]

## 9. Most coins are tiny

# "cap" DataFrame. Returns an int.
def capcount(query_string):
    return cap.query(query_string).count().id

# Labels for the plot
LABELS = ["biggish", "micro", "nano"]

# Using capcount count the biggish cryptos
biggish = capcount('market_cap_usd > 3E+8')

# Same as above for micro ...
micro = capcount('market_cap_usd > 50000000 and market_cap_usd < 300000000') # 5e + 7  & 3e + 8

# ... and for nano
nano =  capcount('market_cap_usd < 50000000')

# Making a list with the 3 counts
values = [biggish, micro, nano]

# Plotting them with matplotlib 
plt.bar(height=values, x=LABELS);

                     #################################  till now ###################################

