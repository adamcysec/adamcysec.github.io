---
layout: page
title: "CSAPP"
subheadline: "Threat Hunting in PyPI"
teaser: "Finding Malicious Packages in PyPI"
categories:
tags:
header:
  title: ""
  background-color: "#EFC94C;"
  #pattern: pattern_concrete.jpg
  image_fullwidth: /threat-hunting.jpg
  #caption: Image from unsplash
---

The Collection, Storage, and Analysis of Python Packages Tool or CSAPP (c-sap).

[CSAPP](https://github.com/adamcysec/CSAPP){:target="_blank"} is a suite of tools for Collecting, Storing, and Analyzing Python Packages in PyPI to assist with threat hunting for malicious packages. 

## Why?

Why do threat hunting in [PyPI](https://pypi.org/){:target="_blank"}? Im very passionate about Python, so when I noticed an uptick in cyber news for malicious Python packages, I decided to do something.

Check out the following articles about malicious Python packages:

- [Cloudguard Spectral Detects Several Malicious Packages On Pypi – The Official Software Repository For Python Developers](https://research.checkpoint.com/2022/cloudguard-spectral-detects-several-malicious-packages-on-pypi-the-official-software-repository-for-python-developers/){:target="_blank"}
- [Malicious ‘Lolip0p’ PyPi packages install info-stealing malware](https://www.bleepingcomputer.com/news/security/malicious-lolip0p-pypi-packages-install-info-stealing-malware/){:target="_blank"}
- [PyPi python packages caught sending stolen AWS keys to unsecured sites](https://www.bleepingcomputer.com/news/security/pypi-python-packages-caught-sending-stolen-aws-keys-to-unsecured-sites/){:target="_blank"}
- [Malicious PyPi packages create CloudFlare Tunnels to bypass firewalls](https://www.bleepingcomputer.com/news/security/malicious-pypi-packages-create-cloudflare-tunnels-to-bypass-firewalls/){:target="_blank"}

The biggest problem with malicious packages is the time of detection. Some of the malicious packages were downloaded thousands of times and when a package is reported to the public, it’s up to the end user to uninstall the package (if they ever do).

So how can we reduce the time of detection to prevent users from falling victim?

## Previous Work

Malicious Python packages hosted on PyPI is nothing new, so it’s important we spend some time researching what others have done to reduce the time of detection. 

During September of 2019, a forum post was created to discuss: **[What methods should we implement to detect malicious content?](https://discuss.python.org/t/what-methods-should-we-implement-to-detect-malicious-content/2240){:target="_blank"}**

### Jordan Wright

The most notable information came from [Jordan Wright](https://twitter.com/jw_sec){:target="_blank"}. Jordan has done extensive work in detecting malicious Python packages. Read his [documentation](https://discuss.python.org/t/what-methods-should-we-implement-to-detect-malicious-content/2240/14){:target="_blank"} on preventing and detecting malware on PyPI. 

Now I high recommend you read his article on **[Hunting for Malicious Packages on PyPI](https://jordan-wright.com/blog/post/2020-11-12-hunting-for-malicious-packages-on-pypi/){:target="_blank"}.**

In summary, Jordan created a CI/CD pipeline for installing python packages from PyPI and monitoring the host’s syscalls and network traffic for malicious activity all hosted in AWS. 

Impressive! However I won’t be spending any money on my hunting method as I'm not getting paid for this. 

### Socket

[socket.dev](http://socket.dev){:target="_blank"} is what you get if you form a company around detecting malicious packages. Socket is actively scraping Python packages from pypi.org, Node packages from [npmjs.com](http://npmjs.com){:target="_blank"}, and Maven packages from [mvnrepository.com](https://mvnrepository.com/){:target="_blank"}. Socket makes money by having organizations pay to have their project’s dependencies actively scanned for malicious code. This type of work helps to solve the supply chain attacks issue. 

## My Threat Hunting Plan

Since I will be spending $0, I will be relying solely on data analysis techniques using Python! Detecting bad Python with good Python. 

The plan is to start tracking Python packages hosted on PyPI in a database with metadata. Hopefully, we will be able to collect enough data to learn what good packages look like versus malicious packages. With data stored in a database, we can query the data using SQL and eventually create an automated system for detecting and alerting on malicious Python packages.

## Data Collection

I found a website called [libraries.io/PyPI](https://libraries.io/PyPI){:target="_blank"} that actively scrapes package data from PyPI, performs data enrichment, and has an API. My favorite feature is the `SourceRank` field applied to every package. This is a reputation rank, the higher the rank the more trustworthy your package is. 

The problem with the API is that we don’t get the package’s first release date or the users (maintainers) authorized to maintain the package.

I will have to make an additional web request to PyPI and web scrape that data as well.

## Data Storage

Now that I know where im getting my data from, I need a way to store that data and it has to be in a data format that I can query in a SQL like method. Another issue is the amount of data, [Libraries.io](http://Libraries.io){:target="_blank"} indicates there are over 400,000 PyPI packages and more are added everyday. I decided to keep it simple and go with CSV format mainly due to Duck DB that I will talk about next. 

### Duck DB

[Duck DB](https://duckdb.org/){:target="_blank"} claims to have, “All the benefits of a database, none of the hassle.” and it truly does. If your data size can fit into memory, then Duck DB can be used to query that data without having to configure and maintain a traditional SQL database. You can feed Duck DB a CSV and start using SQL queries on the data.  

## Front End

Now that I know how im going to query the data, I need a way to present and have users interact with the data. I thought about learning web dev and implementing my own user interface, but that will take hours upon hours of learning, and designing a good UI is very difficult. Luckily, there is an easier option called [Streamlit](https://streamlit.io/){:target="_blank"}.

### Streamlit

Streamlit was designed for data analysts to take their data and create a web application to display statistics and interact with the data. Streamlit makes it easy to display Pandas Dataframes within tables, which is great because Duck DB returns Dataframes from a SQL query.

## Threat Hunting Tool

After spending hours of experimenting, designing, and rewriting code, I created a suite of tools to collect, store, and analyze Python packages in PyPI that i call [CSAPP](https://github.com/adamcysec/CSAPP/){:target="_blank"}.

### Initial Data Collection

The initial data collection of 454,633 packages took over 7 days of non-stop execution time. This is because [Libraries.io](http://Libraries.io){:target="_blank"} has a rate limit of 60 requests per minute. This is only for the initial collection and updating/maintaining new packages uploaded will only take a handful of hours versus days. The size of the CSV file is about 160 MB. That is considered large for a CSV, but as long as the data can fit in memory we can use Duck DB. 

### pypi_data_harvest.py

[`pypi_data_harvest.py`](https://github.com/adamcysec/CSAPP/blob/main/pypi_data_harvest.py){:target="_blank"} is the starting point for data collection. This tool handles creating and updating the CSV database of PyPI packages.

### pypi_streamlit.py

[`pypi_streamlit.py`](https://github.com/adamcysec/CSAPP/blob/main/pypi_streamlit.py){:target="_blank"} is the front end web app for displaying and querying the PyPI CSV. 

### pypi_package_validator.py

[`pypi_package_validator.py`](https://github.com/adamcysec/CSAPP/blob/main/pypi_package_validator.py){:target="_blank"} will enumerate the PyPI CSV and remove packages that have been removed from [pypi.org](http://pypi.org){:target="_blank"}. This will help with reducing the CSV file size.

### audit_pypi_info_db.py

Sometimes after updating the PyPI CSV, new data may not get placed in their correct columns. This can happen if [libraries.io](http://libraries.io){:target="_blank"} decides to include new data in their API. Data not in the proper place within the CSV will cause read errors from Duck DB. If this occurs, [`audit_pypi_info_db.py`](https://github.com/adamcysec/CSAPP/blob/main/audit_pypi_info_db.py){:target="_blank"} will meticulously evaluate each row for data in the wrong column and remove said row from the CSV. 

## CSAPP Showcase

Try out the application right now on Streamlit with the sample data set!! --> [csapp-adamcysec.streamlit.app](https://csapp-adamcysec.streamlit.app/){:target="_blank"}

Read the help documentation to learn about CSAPP and how adversaries are targeting PyPI --> [CSAPP Git Book](https://adamcysec.gitbook.io/csapp/){:target="_blank"}

Want to use CSAPP? Install it from Github --> [CSAPP](https://github.com/adamcysec/CSAPP){:target="_blank"}

