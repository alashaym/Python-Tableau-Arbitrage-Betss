# Python-Tableau-Arbitrage-Bets

### Objective: create an interactive dashboard that allows users to find the upcoming arbitrage bets for different sports, 
### the profit potential of each bet, and which SportsBook companies offer the most lucrative bets

### What is arbitrage betting?
#### arbitrage betting is when two or more different sportsbook companies have opposing odds for a game. One can place a bet on each of the sportsbooks to minimize the risk
#### of their bet. Through this dashboard, the user can see which events and where they can bet in order to maximize their profits.

## Link to API: 

### link to Tableau interactive dashboard: https://public.tableau.com/views/DailyarbitrageBets/Dashboard1?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

## Python Code
### importing requests to call api and connect it, pandas for dataframe manipulation and pprint to make the api dictionaries readable
```
import requests # to actually call the api and connect
import pandas as pd #to manipulate the data and convert to df, etc.
import os
from pprint import pprint #this will help make dictionaries more readable
```
### creating the API connection 
```
load_dotenv(r"C:\Users\alaou\arbitrage project\.env")
print("API Key:", api_key)
print("API Host:", api_host)
url = f"https://{api_host}/v0/advantages/?type=ARBITRAGE"
print("URL:", url)
headers = {
    'x-rapidapi-host': api_host,
    'x-rapidapi-key': api_key
}

ResponseData = response.json() i
```
### storing the api data into a variable to read dictionary keys & checks the first dictionary items
```
advantages = ResponseData['advantages']
pprint(advantages[0]) 
```
### Creating a Dataframe by selecting the exact Keys that we want 
```
# creating the dataframe - parsing only for the arbitrage bets, team names, and event types
# creating an empty list to store the data from the for loop
rows = []

# loop through each dictionary within the api and call atributes like who is playing and which team the odds favor, etc.
# using get because if some of the fields are not present, it will continue running the rest of my code
for adv in advantages:   
    market = adv.get("market", {})
    event = market.get("event", {})
    participants = event.get("participants", [])
    event_name = event.get("name")
    start_time = event.get("startTime")
    sport_type = market.get('event', {}) \
                       .get('competitionInstance', {}) \
                       .get('competition', {}) \
                       .get('sport', None)

    # getting all of the different bet 'outcomes', which is just both teams offering an opportunity for arbitrage betting
    for out in adv.get("outcomes", []):
        
        # go to the participant dictionary wihin the api, pull the names of the teams, the short name i.e. PHX, and the unique identifyer, stored in 'key'
        participant_dict = out.get("participant", {})
        participant_name = participant_dict.get("name", "Unknown")
        participant_short = participant_dict.get("shortName", "Unknown")
        participant_key = participant_dict.get("key", "Unknown")

        # adding rows to define the times the advantages were created, the event times, names of the participants to add to the stored rows[] from earlier
        rows.append({
            "advantage_type": adv.get("type"),
            "lastFoundAt": adv.get("lastFoundAt"),
            "event_name": event_name,
            "event_start_time": start_time,
            "market_type": market.get("type"),
            "market_segment": market.get("segment"),
            "modifier": out.get("modifier"),
            "payout": out.get("payout"),
            "source": out.get("source"),
            "live": out.get("live"),
            "participant_name": participant_name,
            "participant_shortName": participant_short,
            'sport_type': sport_type 
        })

# take all of the data appended into the 'rows[]' list and create a dataframe for easy manipulation and connection to Tableau
df = pd.DataFrame(rows)
df

# print only the first 5 rows to check if the data returns as I want it to 
df.head()
```
### increasing user readability of date/time columns (tried just the pd.to_datetime() function, but the columns were still unreadble, hence the loop)
```
# make event_start_time & lastFoundAt more readable
columns_to_edit = ['lastFoundAt', 'event_start_time']

for col in columns_to_edit:
    df[col] = pd.to_datetime(df[col])

df['lastFoundAt'] = df['lastFoundAt'].dt.strftime("%B %d, %Y at %I:%M %p")
df['event_start_time'] = df['event_start_time'].dt.strftime("%B %d, %Y at %I:%M %p")

# check to see if changes worked
df.head()
```
