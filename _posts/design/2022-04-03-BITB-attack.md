---
layout: page
title: "Browser in The Browser (BITB) Attack"
subheadline: "Phishing Site Attack"
teaser: "Stealing credentials by using a phishing site within a phishing site."
categories:
tags: 
header:
    title: "Browser in The Browser (BITB)"
    background-color: "#EFC94C;"
    #pattern: pattern_concrete.jpg
    image_fullwidth: /new-tool-tuesday/winSuperMem/header_cyber-sec.jpg
    caption: Image from unsplash
    caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---

github user mrd0x created a phishing attack that preys on websites that use single sign-on pop up windows as a login method called [browser in the browser attack](https://github.com/mrd0x/BITB){:target="_blank"}.

![01_popup.png](/images/BITB Attack/01_popup.png)

The screenshot above is one of the provided BITB templates for chrome. 

## Attack Overview

Imagine i’m an attacker and i want to steal user credentials for gmail. I setup a phishing site offering free downloads for popular games. The webiste instructs the users to sign-in using gmail to get an email for a download link to the game requested. When the user clicks on the gmail sign-in button, my fake pop up window appears displaying an exact copy of the gmail login page. 

## Setup the template

mrd0x created fake chrome pop up templates for anyone to use in this attack, however there are many features the attacker must build out for the pop up window to look convincing. 

### Creating a phishing site

Think of this attack as a phishing site within a phishing site. First the attacker must create and host a phishing site pretending to be hosting free games. Then the attacker must create another web page pretending to be a gmail login page. 

### Creating a button

The attacker must also create a button on their phishing site that links to the fake gmail login page. 

### Configuring the template

The final step is to configure file `index.html` .

![02_popup_numbers.png](/images/BITB Attack/02_popup_numbers.png)

1. set the pop up title and change the window logo
2. set the site domain name
3. set the login page name
4. the second phishing page pretending to be a gmail login page made by the attacker

Once complete, the attacker would have an effective phishing site to steal gmail login credentials. 

## Detetion methods

There are some checks the end user can make to determine if the pop up window is legit.

### Pop up window behavior

Normally pop up windows are rendered by the browser. In this phishing attack, the pop up window is rendered using html and css hosted on the website. Try to move the pop up window outside of the parent window, such as moving the pop up to another monitor. If the pop up window is unable to be moved outside of the parent window, then the website is most likely rendering a fake pop up as seen below:

![03_popup_check.png](/images/BITB Attack/03_popup_check.png)

### View https certificate

![04_https_lock.png](/images/BITB Attack/04_https_lock.png)

To view a site’s certificate, you would click on the lock and another pop up appears with certificate information. The provided template does not include this functionality which is another indicator of a fake pop up window. The attacker could modify the template to include a fake certifcate pop up, but i would only expect that from advanced threats. 

## How to try out the templates

You can use docker to quickly setup an Apache web server and copy over the provieded template files to the web server to serve. After naviagting to the `index.html` page, the fake pop up window automaticlly opens. 

![05_popup_tryout.png](/images/BITB Attack/05_popup_tryout.png)

## Will we see this attack in the wild?

This attack method has already been used in the wild to steal Steam account credentials. 

![06_fake_steampage.png](/images/BITB Attack/06_fake_steampage.png)

The above screenshot is a fake Steam login pop up. This attack was reported by user [Bangaladore on Reddit](https://www.reddit.com/r/Steam/comments/bvqs92/insanely_clever_steam_credential_stealing_scam/){:target="_blank"} in 2019. In my opinion, we will not be seeing this attack more in the future due to the limited scope of sites. Not every site uses single sign-in and the sites that do, often do not use a pop up window. Most sites use a redirect within the same window to handle the single sign-in.