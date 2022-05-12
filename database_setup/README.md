# CREATING THE DATABASE
### Since scraping historical data from Yahoo fince requied very long time and for the objective of the reproducability we decided to make the database into two steps, the fist step is shown below just to show how did get the date but in order to reproduce the work one can start from step two immediately.

## First Step:
This step takes time since it requires downloading historical data, but the good news is that you don't have to do it! we already did it for you and you can start from the second step to get your database ready.

In this part we are getting a list of stock tickers using the Pandas `read_html` function to scrape Wikipedia:
```python
table=pd.read_html("https://en.wikipedia.org/wiki/List_of_S%26P_500_companies")
df = table[0]
stockticker = df["Symbol"].to_list()
```
Here we are writing the tickers into a txt file and save it in `data_one` folder, to allow updating the S&P500 list since some of the stocks have been removed. Also the table in Wikipedia uses "." instead "-" that is used in Yahoo finance tickers which we are going to scape too.
```python
with open("../data_one/tickerlist.txt", "w+") as out_file:
    for ticker in stockticker:
        out_file.write("%s\n" %ticker)
```
The folder `data_two` has the updated txt tickers file which we will use to scrape to Yahoo finance here.
```python
with open("../data_two/tickerlist.txt", "r") as in_file:
    clean_tickers = in_file.read()
```
The following part will download all the historical data from 2019-04-29 to 2022-04-29 for each variable alone. Note that this takes very long time!
```python
adj_close = yf.download(clean_tickers, "2019-04-29", "2022-04-29")["Adj Close"]
close_price = yf.download(clean_tickers, "2019-04-29", "2022-04-29")["Close"]
high_price = yf.download(clean_tickers, "2019-04-29", "2022-04-29")["High"]
low_price = yf.download(clean_tickers, "2019-04-29", "2022-04-29")["Low"]
open_price = yf.download(clean_tickers, "2019-04-29", "2022-04-29")["Open"]
volume = yf.download(clean_tickers, "2019-04-29", "2022-04-29")["Volume"]
```
This part will write the imported dataframe to CSV file to be used in step two detaled later.
```python
adj_close.to_csv("../data_one/adj_close.csv")
close_price.to_csv("../data_one/close_price.csv")
high_price.to_csv("../data_one/high_price.csv")
low_price.to_csv("../data_one/low_price.csv")
open_price.to_csv("../data_one/open_price.csv")
volume.to_csv("../data_one/volume.csv")
```
## Second Step:

Firstly, we will get the data produced in first step through reading the csv files in the folder `data_two`.
```python
def import_csv_as_dataframe(variable):
    ''' Takes the variable name from [adj_close, close_price, high_price, low_price, open_price, volume] and return a dataframe for all S&P500 '''
    file_path = os.path.join("../data_two", variable+".csv")
    df = pd.read_csv(file_path)       
    return df
```
Calling the previous function:
```python
adj_close = import_csv_as_dataframe("adj_close")
close_price = import_csv_as_dataframe("close_price")
high_price = import_csv_as_dataframe('high_price')
low_price = import_csv_as_dataframe("low_price")
open_price = import_csv_as_dataframe("open_price")
volume = import_csv_as_dataframe("volume")
```
Then, we defind the schema primarily to make the date type. The complete schema is shown in the original code.
```python
schema = {
    "Date": Date,
    "A": Numeric,
    "AAL": Numeric,
    "AAP": Numeric,
    "AAPL": Numeric,
    .
    .
    .
```
Finally, we will use the following commands to send the data frames to the database using `SQLAlchemy`.
### Warnong: The best practice here is run line by line and after each line you update your SQL database. If they are runned all at one failure might occure and it might turn to be hard to fix.
```python
open_price.to_sql("open_price", engine, if_exists="replace", dtype=schema, index=False)
high_price.to_sql("high_price", engine, if_exists="replace", dtype=schema, index=False)
low_price.to_sql("low_price", engine, if_exists="replace", dtype=schema, index=False)
close_price.to_sql("close_price", engine, if_exists="replace", dtype=schema, index=False)
adj_close.to_sql("adj_close", engine, if_exists="replace", dtype=schema, index=False)
volume.to_sql("volume", engine, if_exists="replace", dtype=schema, index=False)
```
