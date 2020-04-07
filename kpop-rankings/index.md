## Predicting Trends in Korean Pop

### Scraping Data

```markdown

from bs4 import BeautifulSoup
import requests
import pandas as pd

#print(r.content[:100])

#create lists
months = list(range(1,13))
years = list(range(2010,2020))

months_list = []
years_list = []
ranking_list = []
singer_list=[]
album_list=[]
sales_list = []
producer_list = []
distributor_list = []
song_list=[]

for y in years:
    for m in months:
        url = (f"http://gaonchart.co.kr/main/section/chart/online.gaon?nationGbn=T&serviceGbn=ALL&targetTime={m}&hitYear=2019&termGbn=month")
        r = requests.get(url)
        soup = BeautifulSoup(r.content, 'html.parser')
        rows = soup.select('tr')
        for x in rows:
            try:
                #create variable lists
                ranking_list.append(x.select_one('.ranking').text.strip())

                subject=x.select_one('.subject').text.strip().splitlines()
                song_list.append(subject[0])
                
                singer_album = x.select_one ('.singer').text.strip().split('|')
                singer_list.append(singer_album[0])
                album_list.append(singer_album[1])

                sales_list.append(x.select_one('.count').text.strip())
                producer_list.append(x.select_one('.pro').text.strip())
                distributor_list.append(x.select_one('.dist').text.strip())

                months_list.append(m)
                years_list.append(y)
                
            except AttributeError:
                #not all row in rows are rankings; isolate rankings from other stuff
                pass
                
#create dataset of variable lists
kp_data = {'months':months_list,
           'years':years_list,
            'ranking':ranking_list,
           'song': song_list,
           'singer':singer_list,
            'album':album_list,
           'sales':sales_list,
           'producer':producer_list,
           'distributor': distributor_list}

#change variable lists to pandas dataframe
kp_df = pd.DataFrame(data=kp_data)

#export to document
export_csv = kp_df.to_csv('kpop-sales-data.csv', index = None, header=True)

#print document
kp_df

```
