---
layout: page
title: "Making the cisaCatalogBot"
subheadline: "From idea to reality"
teaser: "Keeping up-to-date with the latest vulnerabilities"
categories:
tags: 
header:
    title: "Making the cisa Catalog Bot"
    background-color: "#EFC94C;"
    #pattern: pattern_concrete.jpg
    image_fullwidth: /cisa_bot/cisa_page_banner.png
    #caption: Image from unsplash
    #caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---

[Github](https://github.com/adamcysec/cisaCatalogBot) | [Twitter](https://twitter.com/cisaCatalogBot)

# The idea

One of my favorite threat intel tools is [CISA’s Known Exploited Vulnerabilites Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog).

This is a great resource for keeping up with what vulnerabilities attackers are targeting in the wild. CISA does a great job of keeping the catalog up to date, however they don’t provide a newsletter to receive updates when new vulnerabilities are added. That’s when I got the idea to create a twitter bot to tweet every new addition to the catalog using [Python](https://www.python.org/). 

# Script Planning

To keep my code organized, I will need to create 3 files each designed for a specific task. 

- cisa_KEVC.py
    - This file will define the `catalog` object and provide methods for working with CISA’s catalog.
- cisa_alerts.py
    - This file will house the `main` method.
    - This file will handle gathering the data from CISA’s catalog and formatting the tweet.
- twitterlib.py
    - This file will handle authenticating to Twitter’s api and sending the tweet.

## cisa_KEVC.py

This file has 5 methods for gathering data from CISA’s catalog:

1. `get_catalog()`
    1. This method returns the entire CISA catalog.
2. `get_catalog_details()`
    1. This method returns only the catalog metadata details.
3. `get_catalog_by_date()`
    1. This method returns all vulnerabilities added on a specific date.
4. `get_catalog_by_timeframe()`
    1. This method returns all vulnerabilities within a time frame.
5. `get_top_vulnerabilites()`
    1. This method returns `n` number of the most recent added vulnerabilities

`cisa_alerts.py` will only need to use method `get_catalog_by_date()` . However, having additional methods will set me up to create additional projects from the same code. 

 

## cisa_alerts.py

Now I can think about how I want to handle finding new vulnerabilities and creating a tweet from the data. 

### Problem 1

This script will simply use `get_catalog_by_date()` to get all vulnerabilities added today and ill periodicly check the catalog throughout the day. But, this is where I run into a problem. If CISA adds a vulnerability, the script will see it and tweet it. if cisa adds another vulnerability 3 hours later, the script will see 2 new vulnerabilities from today and tweet them and i don’t want to double tweet the same vulnerability. 

To ensure I don’t tweet the same vulnerability twice, I will simply keep track of all vulnerabilities observed today in a database. That way, if CISA makes multiple updates throughout the day, the script will discard the previously seen vulnerabilities. 

### Tweet format

I wanted the tweet to be short, but meaningful enough to determine if I should look further into the vulnerability and provide a way for me to learn more about it. 

I decided to go with the following format:

```
CVE-xxxx-xxxxx
[cisa short description]
[link to cve page]
#[vendorProject] #[Product] #cisabot
```

Which looks like:

![bot_tweet.png](/images/cisa_bot/bot_tweet.png)

## twitterlib.py

Now I can figure out how to send the tweet to Twitter.

Whenever working with big APIs like Twitter’s.. it is always easier to use a python wrapper, so I decided to use [Tweepy](https://github.com/tweepy/tweepy).

Tweepy will make it easy for me to authenticate and send tweets. 

# Script Execution Overview

![script_execution.png](/images/cisa_bot/script_execution.png)

# Finished Product

The bot is complete! Now I just need to schedule `cisa_alerts.py` to run periodically and reset the the database ( `db.txt` ) once per day. 

I will use my favorite python scheduler called [Rocketry](https://github.com/Miksus/rocketry). I will create a new file called `rocket_cisa_alerts.py` and save the following code:

```
from rocketry import Rocketry
import cisa_alerts 

app = Rocketry()

@app.task('every 3 hour')
def do_hourly():
    cisa_alerts.main() # execute script

@app.task('every 1 day')
def do_daily():
    cisa_alerts.reset_db() # clear db.txt

if __name__ == "__main__":
    app.run()
```

Now execute command: `python3 rocket_cisa_alerts.py &`

[cisaCatalogBot](https://github.com/adamcysec/cisaCatalogBot) is alive! You will notice that the bot will check for updates every 3 hours. I recommend following the bot’s [Twitter page](https://twitter.com/cisaCatalogBot) to know when CISA updates their catalog.