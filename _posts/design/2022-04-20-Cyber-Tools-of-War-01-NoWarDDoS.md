---
layout: page
title: "NoWarDDoS"
subheadline: "Cyber Tools of War #1"
teaser: "Using Python to DDoS Russian web sites."
categories:
tags: 
header:
    title: "NoWarDDoS"
    background-color: "#EFC94C;"
    #pattern: pattern_concrete.jpg
    image_fullwidth: /new-tool-tuesday/winSuperMem/header_cyber-sec.jpg
    caption: Image from unsplash
    caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---


## Current Political Climate

On Thursday, February 24th 2022, Russia began their invasion of Ukraine. Immediately, video footage of the invasion from both the Russian and Ukraine army’s emerged. The videos can be viewed on sites such as Reddit community [UkraineWarVideoReport](https://www.reddit.com/r/UkraineWarVideoReport/). 

On Saturday, February 26th 2022, the Vice Prime Minister of Ukraine Mykhailo Fedororv [tweeted](https://twitter.com/FedorovMykhailo/status/1497642156076511233) for anyone with “digital talents” to join the linked Telegram channel to be given tasks. One of those tasks is performing DDoS attacks against Russian hosted web sites.

Two days later a python DDoS tool was number 8 on the [trending list](https://twitter.com/bot_for_devs/status/1498297121119907850?s=20&t=pR0mMPTJxb750_s3oIsHWw) for GitHub called NoWarDDoS.

## NoWarDDoS

On February 26th 2022, a python tool called [NoWarDDoS](https://github.com/AlexTrushkovsky/NoWarDDoS) was published to GitHub by user [AlexTrushovsky](https://github.com/AlexTrushkovsky). This tool was designed to allow anyone to easily execute and participate in a DDoS attack targeting Russian web sites. The repository has a respectable number of stars at 512 and 124 forks. 

### The Readme

The first readme file that loads after clicking the GitHub repository page, shows instructions written in another language unreadable to me. Lucky for me, there is also an english translation offered as well as additional foreign languages. 

The english readme file starts with a disclaimer that this tool is only to be used for educational purposes and reminds us that DDoS attacks are illegal. 

The readme file then informs the user how to install python and execute the DDoS tool.

The steps to run the DDoS tool assume the user is using a linux machine by instructing the user to run the tool using the `flood.sh` file. This file simply provides an easier method to start the tool and monitor the execution of the tool. 

### DDoS Targets

A DDoS can only work if a large number of machines all work together to send web requests at once. How does this tool get the list of Russian web sites for the users to DDoS?

We can figure this out by analyzing the python code within file `main.py` :

1. Line 186 references a function called `get_target_sites_with_fallback()` .
    1. this function is defined on line 170
2. Line 171 references an object called `remoteProvider` with a method called `get_target_sites()` 
3. Line 48 the `RemoteProvider` object is declared 
4. Line 28 the `RemoteProvider.py` file is imported

Continuing the analysis with file `RemoteProivder.py` :

1. Line 30 function `get_target_sites()` is defined
2. Line 35 an object named `settings` has a method called `SITE_HOSTS`
3. Line 7 `settings.py` is imported

Continuing the analysis with file `settings.py` :

1. Line 6 declares variable `_DEFAULT_SITES_HOTS` which contains two links
    1. [https://gitlab.com/jacobean_jerboa/sample/-/raw/main/sample](https://gitlab.com/jacobean_jerboa/sample/-/raw/main/sample)
        1. a json file with 6 Russian based web sites
    2. [https://raw.githubusercontent.com/opengs/uashieldtargets/v2/sites.json](https://raw.githubusercontent.com/opengs/uashieldtargets/v2/sites.json)
        1. a json file with 50 Russian based web sites

These 2 files are hosting the target web sites to be attacked. The URLs also reference the files with the raw content link which means the tool author can update the target host files without requiring the end user to update the tool again. 

### Disclaimer

DO NOT ATTACK sites in which you do not have the permission from the site owner to attack. I highly recommend not using this tool as it is configured to attack sites you do not have permission to be attacking. This tool review was done through static code analysis. 

### Attacking Other Targets

This tool is also equipet to attack any web server of your choosing. If you run the `main.py` file with the `-t` or `--targets` parameter, then the tool will only target the given sites overriding the russian targets. 

Example:

```jsx
py main.py 1 -t http://10.10.141.68
```

- `1` is the number of threads to use.

### How many web requests are sent to a server?

Each thread will send 500 GET requests to a target site. 

File `main.py`:

- Line 108 and 117 are the same while loop to send GET requests up to the maximum as set by `settings.py`

For example: 

```jsx
py main.py 2 -t http://10.10.141.68 http://10.20.10.30
```

Here we have 2 threads with 2 target sites. Each thread will only DDoS one site and submit by default 500 GET requests for a total of 1,000 requests sent from my machine. 

### Proxies

This tool is also designed to send your GET request through a proxy if the server is geoblocking requests. Before a DDoDs is performed, the tool checks for a valid connection to the host first. 

- if the first request returns a status code of 302 or greater
    - then another request is sent through a proxy
        - if the proxy request returns a status code of 200
            - then 500 requests are sent
- if the first request returns a status code of 200
    - then 500 requests are sent

The list of proxies can also be found in the `settings.py` file:

- Line 11 contains a GitHub link
    - [https://raw.githubusercontent.com/opengs/uashieldtargets/v2/proxy.json](https://raw.githubusercontent.com/opengs/uashieldtargets/v2/proxy.json)
        - contains 3,449 different proxy addresses

### flood.sh

`flood.sh` is designed to allow the user to run the DDoS and leave it running autonomously. The shell script creates two new cronjobs.

```jsx
restart_croncmd="cd $(pwd) && sh flood.sh restart"
restart_cronjob="0,15,45 * * * * $restart_croncmd"
```

- restarts the tool 3 times every hour

```jsx
update_croncmd="cd ${update_parent_dir} && sh update.sh ${update_current_dir}"
update_cronjob="30 * * * * $update_croncmd"
```

- updates the tool 1 time every hour

The shell script also provides functions to monitor the execution of the tool:

1. `status()`
    1. returns the process ids of python
2. `net()`
    1. displays the tool’s network usage via command `nload` 
3.  `log()`
    1. displays the tool’s log file via command `tail`
4. `stop()`
    1. kills the current running Python processes

### Docker Container

You can also run this tool as a docker container. 

The [Dockerfile](https://github.com/AlexTrushkovsky/NoWarDDoS/blob/main/Dockerfile) contains the container startup commands. On start, the container will run alpine linux, copies over all of the python files, copies the requirements txt file, and executes the `main.py` file with python3. 

**Settings.py**

The [settings.py](https://github.com/AlexTrushkovsky/NoWarDDoS/blob/main/settings.py) file contains the DDoS settings for the docker container. By default, the max threads created is set to 500 with a max of 500 web requests sent to each site. 

## Conclusion

This tool provides a great example of how to utilize a shell script to interact with a python script. The shell script even takes care of updating the tool in near real time. This tool is also a great example of how to run asynchronous tasks and effectively use the `logging` module. I expect we will see more cyber tools of war popping up on Github in the future.