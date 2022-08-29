---
layout: page
title: "Breach-parse"
subheadline: "Multiprocessing"
teaser: "Reading 40 GB of data with efficiency"
categories:
tags:
header:
  title: "Breach-parse.py"
  background-color: "#EFC94C;"
  #pattern: pattern_concrete.jpg
  image_fullwidth: /new-tool-tuesday/winSuperMem/header_cyber-sec.jpg
  #caption: Image from unsplash
  #caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---

This blog post is not for learning how to multiprocess, rather how I used multiprocessing in Python to solve my problem.

## About breach-parse

Breach Compilation is 41 GB or 1,981 text files of user email address and password pairs from [old database leaks](https://www.google.com/search?q=%22breach+compilation%22&client=firefox-b-1-e&biw=1600&bih=803&tbm=nws&sxsrf=ALiCzsa-Ev3gWNGs7t9eqYFR3a_02uiiTQ%3A1661707951077&ei=r6YLY7aUBKKrqtsPsJmywAg&ved=0ahUKEwj2p5PBiOr5AhWilWoFHbCMDIgQ4dUDCAw&uact=5&oq=%22breach+compilation%22&gs_lcp=Cgxnd3Mtd2l6LW5ld3MQAzIECAAQQzIECAAQQzIECAAQQzIGCAAQHhAHMgYIABAeEAcyBggAEB4QBzIGCAAQHhAHMgYIABAeEAcyBggAEB4QBzIGCAAQHhAHOgUIABCABDoFCAAQhgM6BggAEB4QFlDdBVjDFmCiGGgAcAB4AIABUIgB0gGSAQEzmAEAoAEBwAEB&sclient=gws-wiz-news) that has recently gained main stream attention. [breach-parse](https://github.com/hmaverickadams/breach-parse) is a bash tool designed to search through this data and save matching credential pairs to another file.

## The Problem

I want to code a Python version of breach-parse, but how do we search through 41 GB of data in the fastest way possible? The main bottle neck in our problem will be the speed of reading files. Now there are other factors outside of Python we cannot control, such as the storage medium. Ideally the data is stored on an SSD for the fastest read time. My data is stored on a HDD… much slower read times, but lets focus on want we can do in Python.

## Concurrency vs Parallelism

There are countless articles on learning the difference between concurrency vs parallelism, so I will keep this short. Concurrency allows you to work a set of tasks in out-of-order or partial order, but not work those tasks at the same time. Parrallelism allows you to work tasks at the same time. I will use parallelism to read multiple files at the same time to hopefully decrease search time.

### Apply this to Python

Concurrency is achived by working with threads in Python library [threading](https://docs.python.org/3/library/threading.html) or [asyncio](https://docs.python.org/3/library/asyncio.html).

Parallelism is achived by working with not threads, but processes in Python library [multiprocessing](https://docs.python.org/3/library/multiprocessing.html).

I use multiprocessing to create one process for every core your CPU has.

## Multiprocessing

To get started with multiprocessing, you need to design a task to be worked by workers. My CPU has 4 cores, therefore I have 4 workers at my disposal. The task I want them to work is to read a file line by line comparing the line to a given search string. If the line contains the search string, then save that line to a list for later use.

Now I can read 4 files at once, but we can actually do a little bit better with context managers in Python. We can use an [ExitStack](https://docs.python.org/3/library/contextlib.html#contextlib.ExitStack) to open multiple files at the same time:

```bash
with ExitStack() as stack:
    files = [stack.enter_context(open(fname)) for fname in filenames]
    # All opened files will automatically be closed at the end of
    # the with statement, even if attempts to open files later
    # in the list raise an exception
```

Now each worker can open/read 5 files at once! 4 cores X 5 files = 20 files read at once.

### Multiprocessing in action

Below is the Python code used to create workers and read files:

```python
from contextlib import ExitStack
from multiprocessing import Pool

def main():
# start multi threaded search
    with Pool() as pool:
        res = pool.starmap(search_file, queue)

def search_file(files, search_term):
    results = []

    with ExitStack() as stack:
        working_files = [stack.enter_context(open(x, "rb")) for x in files]
        for lines in working_files:
            for line in lines:
                if search_term in line:
                    results.append(line)

    return results
```

In the `main` function I create a multiprocessing `Pool()` which handles creating workes based on the number of cores I have. The first parameter is `search_file` which is actually the name of the function I want the workers to work. The second parameter is `queue` which is a list with a list of files for the worker to open and the search string.

Below is what `queue` looks like:

```python
queue = [
	[[file1, file2, file3, file4, file5], search_string]
	[[file1, file2, file3, file4, file5], search_string]
	[[file1, file2, file3, file4, file5], search_string]
	[[file1, file2, file3, file4, file5], search_string]
	...
]
```

Each worker takes an item out of the queue and opens 5 files, then returns the matching lines and takes another item out of the queue.

## [breach-parse.py](https://github.com/adamcysec/breach-parse)

Now is time for the final results! Remember my data is stored on a HDD. The original Bash version finishes searching with search string `@tesla.com` in 413 seconds. My Python version finishes in 342 seconds. 71 seconds faster, however depending on your hardware your results will vary.
