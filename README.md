# Mission-To-Mars

## Overview

The purpose of this analysis is to gather information about the climate of Mars, as well as to collect news items about Mars missions using the technique of Web Scraping. Although some of this information isn’t readily available in either CSV or JavaScript Object Notation (JSON) files or through APIs, it can be found on public websites. With all the information that the internet makes available, people and businesses have access to an overwhelming amount of data. But, that data isn’t always formatted in a way that makes it easy to analyze.

Web scraping automates the tedious tasks of extracting online data for analysis. Worldwide, large companies use web scraping to assess their reputations or to track the online presence of their competitors. People can also use web scraping on a smaller scale for personal projects. For example, web scraping can simplify the process of collecting the current news about a particular subject.

## Resources

### Data Source

[Mars Temperature Data Website]( https://data-class-mars-challenge.s3.amazonaws.com/Mars/index.html), 
[Mars NASA News Website](https://redplanetscience.com)

### Software/Libraries

* Splinter
* Beautiful Soup
* Chrome Driver Manager
* Jupyter Notebook
* Mongo Database

### Languages

* HTML
* CSS
* Python

## Analysis

Through this analysis, we were able to identify HTML elements on a page, identify their `id` and `class` attributes, and use this knowledge to extract information via both automated browsing with `Splinter` and HTML parsing with `Beautiful Soup`. We were also able to scrape various types of information, including HTML tables and recurring elements, like multiple news articles on a webpage.

The following code shows how we used Splinter and Beautiful Soup to begin our scraping:

```py
from splinter import Browser
from bs4 import BeautifulSoup as soup
from webdriver_manager.chrome import ChromeDriverManager

executable_path = {'executable_path': ChromeDriverManager().install()}
browser = Browser('chrome', **executable_path, headless=False)

url = ''
browser.visit(url)

html = browser.html
'' = soup(html, 'html.parser')
```

### Deliverable 1 : Scrape Titles and Preview Text from Mars News

Using `Splinter` and `Beautiful Soup`, we were able to scrape news article titles and its preview descriptions and store them as a list of dictionaries. Here is the full code used on Jupyter Notebook: [part_1_mars_news.ipynb](https://github.com/doliver231/Mission-To-Mars/blob/main/part_1_mars_news.ipynb)

```py
titles = news_soup.find_all('div', class_='content_title')
previews = news_soup.find_all('div', class_='article_teaser_body')

mars_news = []

for x in titles:
    for p in previews:
        mars_news.append(dict(title=x.text, preview=p.text))
```
Here is the resulting output, our list of dictionaries:

![List of Dictionaries : News Articles](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/List_of_dictionaries_news.png)

This list of dictionaries was stored in a Mongo Database with the following code:

```py
from pymongo import MongoClient
mongo = MongoClient(port=27017)
News_db = mongo['mars_db']
News_coll = News_db['mars_coll']
News_coll.insert_many(mars_news)
```
To check to see if the data was correctly stored in the MongoDB:

![Mongo](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Mongo.png)

### Deliverable 2 : Scrape and Analyze Mars Weather Data

Here is the full code used on Jupyter Notebook: [part_2_mars_weather.ipynb](https://github.com/doliver231/Mission-To-Mars/blob/main/part_2_mars_weather.ipynb)

```py
table = temp_soup.find('table', class_='table')
mars_data = table.find_all('td')

id = []
terrestrial_date = []
sol = []
ls = []
month = []
min_temp = []
pressure = []

for i,element in enumerate(mars_data):
    if i % 7 == 0:
        id.append(element.get_text())
    elif i % 7 == 1:
        terrestrial_date.append(element.get_text())
    elif i % 7 == 2:
        sol.append(element.get_text())
    elif i % 7 == 3:
        ls.append(element.get_text())
    elif i % 7 == 4:
        month.append(element.get_text())
    elif i % 7 == 5:
        min_temp.append(element.get_text())
    elif i % 7 == 6:
        pressure.append(element.get_text())

dictionary = {
    "id": id,
    "terrestrial_date": terrestrial_date,
    "sol": sol,
    "ls": ls,
    "month": month,
    "min_temp": min_temp,
    "pressure": pressure
}

df = pd.DataFrame(dictionary)
```

After scraping the Martian weather data, storing it in a dictionary, and converting it to a Pandas DataFrame, the data needed to be prepared for analysis, which involved having the correct data types:

```py
from datetime import datetime as dt
df[['id', 'sol', 'ls', 'month']] = df[['id', 'sol', 'ls', 'month']].astype(int)
df['terrestrial_date'] = pd.to_datetime(df['terrestrial_date'])
df[['min_temp', 'pressure']] = df[['min_temp', 'pressure']].astype(float)
```

![dtypes](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/dtypes.png)

Our cleaned data was exported as a CSV file, and can be accessed [here](https://github.com/doliver231/Mission-To-Mars/blob/main/mars_information.csv).

## Further Analysis of the Web Scraped Data

1. How many months exist on Mars?
2. How many Martian (and not Earth) days worth of data exist in the scraped dataset?

![Questions12](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Question12.png)

3. What are the coldest and the warmest months on Mars (at the location of Curiosity)?

![Question3](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Question3.png)
![BarChart Temps](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Martian_Temperature_vs_Month.png)
![Question3a](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Question3a.png)

4. Which months have the lowest and the highest atmospheric pressure on Mars? 

![Question4](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Question4.png)
![BarChart Pressure](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Martian_Pressure_vs_Month.png)

5. About how many terrestrial (Earth) days exist in a Martian year?

![Question5](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Question5.png)
![Daily Temps](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Martian_Daily_Temperatures.png)
![Question5a](https://github.com/doliver231/Mission-To-Mars/blob/main/Images/Question5a.png)