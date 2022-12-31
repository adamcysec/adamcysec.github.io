---
layout: page
title: "What Companies did Hackers Target in 2022?"
subheadline: "End of Year Review"
teaser: "A year in review of which companies had the most vulnerabilities targeted in the wild"
categories:
tags:
header:
  title: "What Companies did Hackers Target in 2022?"
  background-color: "#EFC94C;"
  #pattern: pattern_concrete.jpg
  image_fullwidth: /hack_at_desk_unsplash.jpg
  #caption: Image from unsplash
  #caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---

## What Companies did Hackers Target in 2022?

CISA keeps track of vulnerabilities attackers are targetting in the wild in order to inform the public of which vulnerabilities need priority. Using their [Known Exploited Vulnerabilites Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog), I will aggregate the data for 2022.

## Top companies with total vulnerabilities 2022

| Rank | Company   | Total |
| ---- | --------- | ----- |
| 1    | Microsoft | 165   |
| 2    | Adobe     | 54    |
| 3    | Cisco     | 49    |
| 4    | Apple     | 26    |
| 5    | Oracle    | 22    |
| 6    | Google    | 21    |
| 7    | Apache    | 13    |
| 8    | D-Link    | 11    |
| 9    | QNAP      | 10    |
| 10   | Vmware    | 8     |

Now lets compare the data to the previous year.

### Top Companies 2022 vs 2021

|      | 2022      |       | 2021         |       |
| ---- | --------- | ----- | ------------ | ----- |
| Rank | Company   | Total | Company      | Total |
| 1    | Microsoft | 165   | Microsoft    | 83    |
| 2    | Adobe     | 54    | Apple        | 23    |
| 3    | Cisco     | 49    | Google       | 23    |
| 4    | Apple     | 26    | Apache       | 12    |
| 5    | Oracle    | 22    | Cisco        | 11    |
| 6    | Google    | 21    | Pulse Secure | 8     |
| 7    | Apache    | 13    | Vmware       | 8     |
| 8    | D-Link    | 11    | Oracle       | 7     |
| 9    | QNAP      | 10    | Trend Micro  | 7     |
| 10   | Vmware    | 8     | Citrix       | 6     |

It’s no surprise that Microsoft is number one consecutively. It’s interesting to see Adobe this year jumping up to second place. Overall companies have more vulnerabilies. This may be due to APTs becoming more advanced or we have more security researches spending time to discover these vulnerabilities. I don’t have an exact answer, but here’s to 2023!

## cisaCatalogBot

If you’re unaware, I run a Twitter bot [@cisaCatalogBot](https://twitter.com/cisaCatalogBot) that tweets everytime CISA updates their [Known Exploited Vulnerabilites Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog). Be sure to follow the bot to stay up-to-date when a new vulnerability is added!

Read about how I made the cisaCatalogBot with python: [Making the cisaCatalogBot](https://adamcysec.github.io/Making-cisaCatalogBot/)
