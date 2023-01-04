---
layout: page
title: "PyTorch Dependency Confusion Attack"
subheadline: "Python Supply Chain Attack"
teaser: "Or is Pip to blame for it's install behaivor?"
categories:
tags:
header:
  title: "PyTorch Dependency Confusion Attack"
  background-color: "#EFC94C;"
  #pattern: pattern_concrete.jpg
  image_fullwidth: /pyorch_comp.jpg
  #caption: Image from unsplash
  #caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---

A popular data science python package called PyTorch had a [dependency compromise](https://pytorch.org/blog/compromised-nightly-dependency/) on their preview (nightly) build for Linux. The malicious code was available Dec 25, 2022 to Dec 30, 2022. The attacker was able to perform this attack by not compromising PyTorch, but by taking advantage of how `pip` works.

## How was a PyTorch Dependency Compromised?

PyTorch’s preview build had a dependency called `torchtriton` that was hosted on their [3rd party package indexer site](https://download.pytorch.org/whl/). In order to install the preview build, you would have ran a command similar to the following:

`pip3 install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cpu`

Take a look at argument `--extra-index-url` , this tells `pip` where to download additional packages if the specified package name does not live in [pypi.org](http://pypi.org). Because, `pip` is designed to always check pypi.org as it is the central repository.

This means even if you tell `pip` where to look for a package, it will always check [pypi.org](http://pypi.org). If the specified package name exists in both pypi.org and a 3rd party indexer, then `pip` will check the version tags and [pick the file with the highest version](https://github.com/pypa/pip/issues/8606#issuecomment-788124257).

In the case for PyTorch, `torchtriton` was an available namespace within [pypi.org](http://pypi.org), so the attacker registered the package as their own and set the version to one higher than PyTorch’s. The attacker then copied PyTorch’s `torchtriton` but included an ELF binary named `triton` with malicious code to steal host information and SSH keys! View the ELF binary on [VirusTotal](https://www.virustotal.com/gui/file/2385b29489cd9e35f92c072780f903ae2e517ed422eae67246ae50a5cc738a0e/details).

## So Pip is to blame?

The `pip` behavior to make [pypi.org](http://pypi.org) take precedence over package installs is a security flaw right!?? Not so fast! Because this topic has been in debate since February 28, 2018 in GitHub issue [#5045](https://github.com/pypa/pip/issues/5045). The debate has more recently started again in GitHub issue [#8606](https://github.com/pypa/pip/issues/8606). My take away is that this is a debate between “this is a security issue” vs “this is the intended design”. It’s important to note that `pip` does not favor pypi.org, it favors the file with the highest version.

## Should we change Pip?

There are a handful of individuals in GitHub issue [#8606](https://github.com/pypa/pip/issues/8606) that believe we should modify the behavior of `pip` , but the maintainers have [stated](https://github.com/pypa/pip/issues/8606#issuecomment-1370303166) that changes to `pip` is not an easy task and current improvements are done by volunteers. A major behavior change will most likely take funding to come to fruition.

But what other options do we have? User [jcozar87](https://github.com/jcozar87) has documented our [current options](https://github.com/pypa/pip/issues/8606#issuecomment-1160838686)! The simplest one is to implemented a dummy package in pypi.org. All you have to do is create the same package name in pypi.org, but set the version to `0.0.0` . That way your 3rd party indexer will always take precedence.

## What did PyTorch Do?

PyTorch renamed dependency `torchtriton` to `pytorch-triton` and registered the new package to [pypi.org/project/pytorch-triton/](https://pypi.org/project/pytorch-triton/) with version number `0.0.1` , a dummy package!

So for now, I would recommend developers using a 3rd party indexer for private packages to register all as dummy packages with the recommended name format: `companyname-package`.

## PyTorch’s Future

PyTorch was created by Meta (Facebook) for Meta and was then open sourced to the community. This means PyTorch was [never designed to be hosted within pypi](https://github.com/pytorch/pytorch/issues/26340#issuecomment-1368500377) and currently conflicts with pypi’s constraints of file size. `pytorch-triton` file size is too large to be served on pypi, which is why the 3rd party indexer is in use.

PyTorch has [plans](https://github.com/pytorch/pytorch/issues/26340#issuecomment-1369248199) to reduce file size and eventfully host everything on pypi.
