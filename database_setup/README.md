# CREATING THE DATABASE
## First Step:
This step takes time since it requires downloading historical data, but the good news is that you don't have to do it! we already did it for you and you can start from the second step to get your database ready.

In this part we are getting a list of stock ticker using the Pandas `read_html` function to scrape Wikipedia:
```python
table=pd.read_html("https://en.wikipedia.org/wiki/List_of_S%26P_500_companies")
df = table[0]
stockticker = df["Symbol"].to_list()
```