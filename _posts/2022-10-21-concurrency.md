---
title: "Code Execution: Single Threading vs Multithreading vs Multiprocessing"
date: 2022-10-21 12:12:12 +/-0800
categories: []
tags: [python] # TAG names should always be lowercase
author: adham
---

## Introduction

When we are first introduced to programming, we learn that the code we write is executed sequentially.

For example, this code prints "A" first, then "B".

```python
print("A")
print("B")
```

This is an example of single threading. This is a very simple and easy way to write code and understand code execution, _but wait!_

![theresmore.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665845123144/OsxL3Ks2F.png)

While this is easy, there are some cases in which we might want our code to run more than one task at the same time. The benefit is improved performance.

For example, virtually all web servers handle more than 1 request at the same time, instead of handling only 1 request and making the other requests wait until the current request is handled.

This is usually done using multithreading, multiprocessing, or both!

You can tell that this is useful because it allows you to use your machine to do more than one task at the same time, right now you're reading this article using your web browser, you probably have more than 1 open tab, listening to music in your music player, browsing your files in your file browser, etc...

**This is possible because modern operating systems support multitasking!
**

## Threads vs Processes

Threads and Processes are ways of executing more than 1 task at the same time.

A process is just an instance of an executing program.
A thread, on the other hand, is like a mini process.

Each process typically has 1 thread of control, but in several situations, it is beneficial to have multiple threads of control in the same process.

### Why would anyone want to have a kind of a process within a process?!

The main reason is that in many applications, there are multiple activities going on all at the same time.

For example, when you're using a word processor, there may be a thread for displaying the content on the screen, a thread for handling your input when you click on the keyboard, and another thread for saving the file to disk in the background.

This allows the word processor to render the content on the screen, take your input and save your changes at the same time!

Another difference is that

## Benefits of Threads

1. The ability of parallel entities to share the same data.

Each process has its own data, processes don't share the same data.
Threads on the other hand share the same data in their process. This is essential for certain applications.

1. Threads are more lightweight than processes.

Threads are faster to create and destroy than processes.
Creating a thread goes 10 to 100 times faster than creating a process.

### _But beware!_

![Stopping you from going into multithreading](https://cdn.hashnode.com/res/hashnode/image/upload/v1665868925302/yDwwVl6t_.jpg)

After reading about the benefits of threads, you might say _"Great, I'll use threads for everything!"_

Threads yield no performance gain all of them are CPU-bound, but when there's substantial computing and substantial I/O, having threads allows these activities to overlap, thus speeding up the application.

If the application's activities are all CPU-bound, multiprocessing will yield better performance.

## Examples in Python

- Multithreading

This code fetches a list of websites sequentially and concurrently (using multithreading) and prints the length of their response and the time it took. This is an I/O bound code.

```python
from concurrent.futures import ThreadPoolExecutor
import time
import requests

def timing(fn):
    """Just a decorator to measure function exection time"""
    def decorated():
        start = time.time()
        fn()
        end = time.time()
        print(f"Time = {round(end-start)} seconds")
    return decorated

urls = [
        'http://www.foxnews.com/',
        'http://www.cnn.com/',
        'http://europe.wsj.com/',
        'http://www.bbc.co.uk/',
        'http://some-made-up-domain.com/'
]

def get_website(url: str):
    response = requests.get(url)
    print(f"Website: {url}, Response size: {len(response.content)} characters")

@timing
def sequential():
    print("Sequential")
    for url in urls:
        get_website(url)

@timing
def multithreading():
    print("Multithreading")
    with ThreadPoolExecutor() as executor:
        for url in urls:
            executor.submit(get_website, url)
        print("Finished firing up threads!")

def main():
    sequential()
    multithreading()

if __name__ == "__main__":
    main()
```

When this code runs the output should be something like this:

```sh
Sequential
Website: http://www.foxnews.com/, Response size: 290976 characters
Website: http://www.cnn.com/, Response size: 1143683 characters
Website: http://europe.wsj.com/, Response size: 752060 characters
Website: http://www.bbc.co.uk/, Response size: 481372 characters
Website: http://some-made-up-domain.com/, Response size: 479 characters
Time = 4 seconds
Multithreading
Finished setting up threads!
Website: http://some-made-up-domain.com/, Response size: 479 characters
Website: http://www.bbc.co.uk/, Response size: 481372 characters
Website: http://www.foxnews.com/, Response size: 290976 characters
Website: http://www.cnn.com/, Response size: 1143683 characters
Website: http://europe.wsj.com/, Response size: 752060 characters
Time = 2 seconds


```

In the sequential execution, the websites were visited in order and the execution time is 4 seconds.

In the multithreading execution, you'll notice that "Finished setting up threads!" was printed before anything else, and that the printed websites are not in order, in fact, if you run the code again, it'll probably be printed in a different order, and the execution time is 2 seconds, half of the sequential execution's time!

That's because the operating system doesn't guarantee running the threads in order. You should never expect threads to execute in order.

- Multiprocessing

This example loops throw a range of 0 to 1000000000. This is a CPU-bound code.

```python
from concurrent.futures import ProcessPoolExecutor, wait
import time

def timing(fn):
    """Just a decorator to measure function exection time"""
    def decorated():
        start = time.time()
        fn()
        end = time.time()
        print(f"Time = {round(end-start)} seconds")
    return decorated

def loop(start: int, end: int):
    for _ in range(start, end):
        continue

@timing
def sequential():
    print("Sequential")
    loop(0, 1000000000)
@timing
def multiprocessing():
    print("Multiprocessing")
    with ProcessPoolExecutor() as executor:
        first_loop = executor.submit(loop, 0, 500000000)
        second_loop = executor.submit(loop, 500000001, 1000000000)
        wait([first_loop, second_loop])

def main():
    sequential()
    multiprocessing()

if __name__ == "__main__":
    main()
```

The output of this code is:

```sh
Sequential
Time = 10 seconds
Multiprocessing
Time = 5 seconds
```

The sequential code execution took 10 seconds, while the multiprocessing code execution took 5 seconds, half of the sequential execution's time!

## Proving that multithreading doesn't speed up CPU-bound tasks.

If we take the previous multiprocessing example and try to do it using multithreading, we won't have any performance gains than the sequential code execution, in fact, it might be slower!

```python
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor, wait
import time

def timing(fn):
    """Just a decorator to measure function exection time"""
    def decorated():
        start = time.time()
        fn()
        end = time.time()
        print(f"Time = {round(end-start)} seconds")
    return decorated

def loop(start: int, end: int):
    for _ in range(start, end):
        continue


@timing
def multithreading():
    print("Multithreading")
    with ThreadPoolExecutor() as executor:
        first_loop = executor.submit(loop, 0, 500000000)
        second_loop = executor.submit(loop, 500000001, 1000000000)
        wait([first_loop, second_loop])

@timing
def multiprocessing():
    print("Multiprocessing")
    with ProcessPoolExecutor() as executor:
        first_loop = executor.submit(loop, 0, 500000000)
        second_loop = executor.submit(loop, 500000001, 1000000000)
        wait([first_loop, second_loop])

def main():
    multithreading()
    multiprocessing()

if __name__ == "__main__":
    main()
```

The output of this code is:

```sh
Multithreading
Time = 9 seconds
Multiprocessing
Time = 5 seconds
```

You can see that in this case, multithreading didn't really give us any significant performance gains and that multiprocessing is almost 2 times faster in the case of CPU-bound tasks.

![I told you so](https://media.tenor.com/sAIHA696KVoAAAAM/toldyouso-colbert.gif align="left")

## Summary

Multithreading and multiprocessing are very powerful techniques that when used in the correct conditions, can give performance gains, but it's also tricky and you should be careful!

In summary, single threading is like running a restaurant with a single waiter, multithreading is like running a restaurant with multiple waiters, and multiprocessing is like running multiple branches of the restaurant!

I would like to end this article with my favorite quote about threads.

> **_A programmer had a problem. He thought to himself, "I know, I'll solve it with threads!". has Now problems. two he_**
